+++
title = 'Open Sourcing Vim Royale'
date = 2026-06-05T11:18:14+05:30
draft = false
showToC = true
cover.image = "/images/open_sourcing.png"
+++

I launched [Vim Royale](https://vimroyale.com/) 3 weeks back. Got a LOT of feedback/feature requests, and I was shipping out code whenever I found fit. It was an interesting experience, building Vim Royale. Learnt a lot of stuff, and wanted to share what was the process like. So here goes.

Also, yes. Vim Royale is now open source.

{{< callout type="info" >}}
You can find the code here: [github.com/Jitesh117/vim_royale](https://github.com/Jitesh117/vim_royale)
{{< /callout >}}

## The Inception

I'd be honest, the name vim royale is not my own creation. It was directly taken from one of Primeagen's ongoing project. But I was disheartened to know that he didn't finish it. So last year, I thought of giving life to this project, since I too, wanted to play a multiplayer vim game, as I'm a die-hard vim user too.

The initial idea was quite simple, there would be two players matched with each other, with a given `TargetCode` and an initial buffer state. The goal is to edit your buffer to match the target code exactly. And I thought how hard could it be to do this? And I was right. It wasn't that hard. I mean the implementation sure wasn't. But a lot of thought went into designing the system how it works now.

The smallest version of the game loop was basically this:

```go
type Match struct {
	ID           string
	PlayerA      *Client
	PlayerB      *Client
	Status       MatchStatus
	TargetCode   string
	PollutedCode string
	StartedAt    time.Time
	FinishedAt   *time.Time
	WinnerID     string
	LastSeqByID  map[string]int64
}
```

A match had two players, a target code, a polluted code, and a small sequence map so stale websocket messages could be ignored.

That was the first shape of the system.

And then, as usual, the simple thing became not-so-simple once actual users started touching it.

## The "connection" between users

The first important design decision was that players never directly talk to each other. Both players are connected to the backend over WebSockets, and the backend acts like the referee.

So if player A edits their buffer, the flow is:

1. Player A sends a `BUFFER_UPDATE`.
2. Backend validates that player A belongs to that match.
3. Backend checks if the message sequence is fresh.
4. Backend forwards it to player B.
5. Spectators, if any, also get the update.

The backend has a small envelope format around every message:

```go
type Envelope struct {
	Type      MessageType     `json:"type"`
	MatchID   string          `json:"matchId,omitempty"`
	PlayerID  string          `json:"playerId,omitempty"`
	Seq       int64           `json:"seq,omitempty"`
	Timestamp int64           `json:"timestamp,omitempty"`
	Payload   json.RawMessage `json:"payload,omitempty"`
}
```

And the message types started out very small:

```go
const (
	MsgHello          MessageType = "HELLO"
	MsgHelloAck       MessageType = "HELLO_ACK"
	MsgQueueJoin      MessageType = "QUEUE_JOIN"
	MsgGameStart      MessageType = "GAME_START"
	MsgBufferUpdate   MessageType = "BUFFER_UPDATE"
	MsgPlayerFinished MessageType = "PLAYER_FINISHED"
	MsgGameOver       MessageType = "GAME_OVER"
	MsgError          MessageType = "ERROR"
)
```

I liked this model because the frontend did not need to know much about "networking". It just sent events and reacted to events.

The server was the source of truth.

This was very important, because if I let the frontend decide too much, then every small race condition would become a game logic issue. And dear god, race conditions did show up anyway.

## The "Flow"

The flow currently looks roughly like this:

```txt
Client connects
  |
  v
Client sends HELLO
  |
  v
Server attaches identity
  |
  v
Server sends HELLO_ACK
  |
  v
Client sends QUEUE_JOIN
  |
  v
Server puts client in matchmaking queue
  |
  v
Server finds opponent
  |
  v
Server sends GAME_START to both players
  |
  v
Players exchange BUFFER_UPDATE through server
  |
  v
First player to match target sends PLAYER_FINISHED
  |
  v
Server persists match, updates rating, sends GAME_OVER
```

The `HELLO` step might look like extra ceremony, but it made a lot of later features possible.

Initially it was just token or player id:

```go
type HelloPayload struct {
	Token    string `json:"token,omitempty"`
	PlayerID string `json:"playerId,omitempty"`
}
```

Later, when guest mode and tournaments came in, this expanded:

```go
type HelloPayload struct {
	Token                  string `json:"token,omitempty"`
	PlayerID               string `json:"playerId,omitempty"`
	GuestName              string `json:"guestName,omitempty"`
	GuestSessionToken      string `json:"guestSessionToken,omitempty"`
	TournamentID           uint   `json:"tournamentId,omitempty"`
	TournamentSessionToken string `json:"tournamentSessionToken,omitempty"`
}
```

This is one of those changes where I was glad the protocol had a proper handshake. If the connection starts with an identity negotiation step, adding more ways to identify a player becomes much less painful.

The server attaches the identity like this:

```go
func (h *Hub) AttachIdentity(client *Client, playerID string) error {
	h.mu.Lock()
	defer h.mu.Unlock()

	if existing, exists := h.clientsByID[playerID]; exists && existing != client {
		return fmt.Errorf("player_id_already_connected")
	}

	client.ID = playerID
	h.clientsByID[playerID] = client
	return nil
}
```

That `clientsByID` map became more important than I initially expected.

You don't just want to know which websocket connections are alive. You also want to know if the same player has opened two tabs, refreshed during matchmaking, disconnected mid-game, or tried to reconnect while the old socket is still around.

Multiplayer games are basically distributed systems wearing a fun hat.

## How matchmaking was initially

After I was certain that the match is finally working, the next thing that remained was to make a matchmaking algorithm, so players can join a queue and match with each other.

Initially I had a very simple queue data structure in place, which would match players based on first come first serve.

The first version was literally a slice.

```go
type Hub struct {
	mu sync.Mutex

	clients     map[*Client]struct{}
	clientsByID map[string]*Client
	matches     map[string]*Match
	waitingQ    []*Client

	Register   chan *Client
	Unregister chan *Client
	incoming   chan InboundMessage
}
```

And matchmaking was basically:

```go
func (h *Hub) enqueueForMatch(client *Client) {
	h.mu.Lock()
	defer h.mu.Unlock()

	if client.ID == "" {
		h.sendErrorLocked(client, "unauthenticated", "send HELLO before queueing")
		return
	}

	if client.MatchID != "" {
		h.sendErrorLocked(client, "already_in_match", "player already in an active match")
		return
	}

	for _, queued := range h.waitingQ {
		if queued.ID == client.ID {
			h.sendErrorLocked(client, "already_queued", "player already in queue")
			return
		}
	}

	h.waitingQ = append(h.waitingQ, client)

	for len(h.waitingQ) >= 2 {
		pA := h.waitingQ[0]
		pB := h.waitingQ[1]
		h.waitingQ = h.waitingQ[2:]

		if pA == pB || pA.ID == "" || pB.ID == "" {
			continue
		}

		// start match
	}
}
```

And honestly, for the first version, this was fine.

Two people join, they get matched. Done.

But quickly I realised this won't be a good idea, since after a point, most people would deviate from their initial ratings based on the skill level. And it won't be a good idea to match an 800 rated player with a 1500 rated player.

So I googled a bit and after going through some blog posts and StackOverflow pages, I stumbled on the idea of a rating bucket approach for matchmaking. And it made total sense.

## Rating buckets

The idea is pretty simple.

Instead of one queue, you maintain multiple queues. Each queue represents a rating bucket.

```go
type ratingBucket struct {
	mu      sync.Mutex
	players []*Client
}

const numBuckets = 41

type Hub struct {
	mu sync.Mutex

	clients        map[*Client]struct{}
	clientsByID    map[string]*Client
	matches        map[string]*Match
	waitingBuckets [numBuckets]*ratingBucket
}
```

Then when a player joins matchmaking, we put them in the bucket corresponding to their rating.

```go
func (h *Hub) enqueueForRankedMatch(client *Client) {
	h.mu.Lock()
	defer h.mu.Unlock()

	if !h.isClientActiveLocked(client) {
		return
	}
	if client.ID == "" {
		h.sendErrorLocked(client, "unauthenticated", "send HELLO before queueing")
		return
	}
	if client.MatchID != "" {
		h.sendErrorLocked(client, "already_in_match", "player already in an active match")
		return
	}

	bucketIdx := client.Rating
	if bucketIdx < 0 || bucketIdx >= numBuckets {
		bucketIdx = 0
	}

	bucket := h.waitingBuckets[bucketIdx]

	bucket.mu.Lock()
	defer bucket.mu.Unlock()

	for _, queued := range bucket.players {
		if queued.ID == client.ID {
			h.sendErrorLocked(client, "already_queued", "player already in queue")
			return
		}
	}

	bucket.players = append(bucket.players, client)
	client.QueueType = "ranked"
	client.EnqueuedAt = time.Now()
	client.markAwaitingQueuePong()

	h.tryMatchBucket(bucket)
}
```

At this point I thought, "huh, that was easy".

Boy was I wrong.

The first mental model I had was: players should be strictly confined to their rating buckets.

If you are in bucket 10, you should only match with another player in bucket 10. If you are in bucket 20, same thing. This sounds fair, right?

It is fair on paper.

But in practice, a game where you wait forever is not a game.

If two players are in nearby buckets and both have waited enough, it is better to start a slightly imperfect match than to let both of them stare at a matchmaking screen. This was one of those product decisions hidden inside a backend algorithm.

So the later version added match expansion.

```go
func (h *Hub) runMatchExpansion() {
	ticker := time.NewTicker(1 * time.Second)
	defer ticker.Stop()

	for range ticker.C {
		h.expandStaleMatches()
	}
}
```

Every second, the backend looks at old queue entries and slowly expands the acceptable bucket range.

```go
func (h *Hub) expandStaleMatches() {
	h.mu.Lock()
	defer h.mu.Unlock()

	now := time.Now()

	for i := 0; i < numBuckets; i++ {
		src := h.waitingBuckets[i]
		if src == nil {
			continue
		}

		src.mu.Lock()
		if len(src.players) == 0 {
			src.mu.Unlock()
			continue
		}

		h.tryMatchBucket(src)

		if len(src.players) == 0 {
			src.mu.Unlock()
			continue
		}

		oldest := src.players[0]
		wait := now.Sub(oldest.EnqueuedAt)

		if wait < 5*time.Second {
			src.mu.Unlock()
			continue
		}

		spread := int(wait / (5 * time.Second))
		if spread > 5 {
			spread = 5
		}
		if spread < 1 {
			spread = 1
		}

		src.mu.Unlock()

		for delta := 1; delta <= spread; delta++ {
			for _, j := range []int{i - delta, i + delta} {
				if j < 0 || j >= numBuckets {
					continue
				}

				dst := h.waitingBuckets[j]
				if dst == nil {
					continue
				}

				dst.mu.Lock()
				if len(dst.players) == 0 {
					dst.mu.Unlock()
					continue
				}

				candidate := dst.players[0]
				dst.players = dst.players[1:]
				dst.mu.Unlock()

				src.mu.Lock()
				src.players = append(src.players, candidate)
				h.tryMatchBucket(src)
				src.mu.Unlock()
				break
			}
		}
	}
}
```

This looks a little weird at first, but the core idea is simple:

```txt
0s wait   -> only same bucket
5s wait   -> nearby buckets
10s wait  -> slightly wider range
15s wait  -> wider range
...
```

This made the matchmaking feel much better.

Strict fairness is nice, but getting to actually play the game is nicer.

## Starting a match

Once two players are selected, the backend creates the match and sends both players a `GAME_START`.

```go
func (h *Hub) startMatchLocked(pA, pB *Client, tournamentID *uint) {
	matchID := newMatchID()
	now := time.Now().UTC()

	targetCode := utils.PickTargetCode()
	pollutedCode := utils.PolluteCode(targetCode, pA.Rating)

	match := &Match{
		ID:           matchID,
		PlayerA:      pA,
		PlayerB:      pB,
		TournamentID: tournamentID,
		Status:       MatchPlaying,
		TargetCode:   targetCode,
		PollutedCode: pollutedCode,
		StartedAt:    now,
		LastSeqByID: map[string]int64{
			pA.ID: 0,
			pB.ID: 0,
		},
		PlayerABuffer: pollutedCode,
		PlayerBBuffer: pollutedCode,
	}

	h.matches[matchID] = match
	pA.MatchID = matchID
	pB.MatchID = matchID

	h.sendLocked(pA, MsgGameStart, matchID, pA.ID, 0, GameStartPayload{
		MatchID:      matchID,
		OpponentID:   pB.ID,
		OpponentName: pB.DisplayName,
		Role:         "A",
		StartedAt:    now.Unix(),
		TargetCode:   targetCode,
		PollutedCode: pollutedCode,
	})

	h.sendLocked(pB, MsgGameStart, matchID, pB.ID, 0, GameStartPayload{
		MatchID:      matchID,
		OpponentID:   pA.ID,
		OpponentName: pA.DisplayName,
		Role:         "B",
		StartedAt:    now.Unix(),
		TargetCode:   targetCode,
		PollutedCode: pollutedCode,
	})

	go h.runRoundTimeout(matchID)
}
```

There are a few small details here that came later:

- `PlayerABuffer` and `PlayerBBuffer` are kept on the server for spectators and replay.
- `RoundDurationS` exists because matches should not run forever.
- `TournamentID` is optional because regular matchmaking and tournament matchmaking share the same match engine.
- The polluted code can depend on rating, so stronger players can get harder edits.

I like this kind of code because it is boring in the right way. It creates the thing, stores the thing, notifies both clients, and starts a timeout.

Boring backend code is underrated.

## Buffer updates

Once the game starts, players keep sending buffer changes.

The earliest version just forwarded the whole content:

```go
type BufferUpdatePayload struct {
	Content string `json:"content"`
	Cursor  int    `json:"cursor,omitempty"`
}
```

This is the simplest thing you can do. And for small snippets of code, it works.

But later, for replay and spectators, I needed more structure. So the payload evolved into:

```go
type BufferUpdatePayload struct {
	Content *string      `json:"content,omitempty"`
	Delta   *BufferDelta `json:"delta,omitempty"`
	Cursor  int          `json:"cursor,omitempty"`
	Replay  *ReplayMeta  `json:"replay,omitempty"`
}

type BufferDelta struct {
	Ops []BufferDeltaOp `json:"ops"`
}

type BufferDeltaOp struct {
	Type string `json:"type"`
	Pos  int    `json:"pos,omitempty"`
	Text string `json:"text,omitempty"`
	From int    `json:"from,omitempty"`
	To   int    `json:"to,omitempty"`
}
```

Now instead of always sending the full buffer, the frontend can send operations like:

```json
{
  "ops": [
    { "type": "delete", "from": 10, "to": 14 },
    { "type": "insert", "pos": 10, "text": "name" }
  ]
}
```

The backend applies these deltas to its own copy of the player buffer.

```go
func applyBufferDelta(base string, delta *BufferDelta) string {
	if delta == nil || len(delta.Ops) == 0 {
		return base
	}

	insertions := make(map[int][]string)

	type deleteRange struct {
		from int
		to   int
	}

	deletes := make([]deleteRange, 0)

	for _, op := range delta.Ops {
		switch op.Type {
		case "insert":
			pos := clamp(op.Pos, 0, len(base))
			insertions[pos] = append(insertions[pos], op.Text)

		case "delete":
			from := clamp(op.From, 0, len(base))
			to := clamp(op.To, 0, len(base))
			if to <= from {
				continue
			}
			deletes = append(deletes, deleteRange{from: from, to: to})
		}
	}

	// merge deletes, apply inserts, return new buffer
	return string(out)
}
```

This function is not very glamorous. But it is one of those pieces that made the rest of the system possible.

Because once the backend has the latest buffers, it can support:

- spectators joining mid-match
- replay reconstruction
- bot matches
- better game over payloads
- cleaner debugging

At first I thought the backend only had to relay messages.

Later I realised the backend had to understand the match.

Not the Vim part, but the state part.

## Reducing WebSocket payloads by roughly 90%

One optimization that made a surprisingly big difference was reducing the WebSocket payload size.

Initially, every keystroke sent the entire buffer snapshot.

```tsx
const sendBufferUpdate = useCallback(
  (content: string) => {
    const seq = seqRef.current++;

    sendEnvelope({
      type: "BUFFER_UPDATE",
      matchId: matchStateRef.current.matchId,
      seq,
      payload: { content },
    });
  },
  [sendEnvelope],
);
```

This was easy to reason about.

If the player typed one character, send the whole buffer. If the opponent receives it, replace the whole opponent editor content.

```tsx
onBufferUpdate: (content: string) => {
  replaceOpponentContent(content);
};
```

Simple. Beautiful. Wasteful.

For small challenges, it was okay. But Vim Royale sends updates on every edit. If the buffer is 1000 characters and the user presses `x`, sending 1000 characters again is silly. The actual change was just "delete one character".

Instead of sending full buffer snapshots on every keystroke, I started sending buffer deltas.

The payload changed from this:

```ts
export type BufferUpdatePayload = {
  content: string;
  cursor?: number;
};
```

to this:

```ts
export type BufferDeltaOp =
  | { type: "insert"; pos: number; text: string }
  | { type: "delete"; from: number; to: number };

export type BufferDelta = {
  ops: BufferDeltaOp[];
};

export type BufferUpdatePayload = {
  content?: string;
  delta?: BufferDelta;
  cursor?: number;
};
```

Now a tiny edit produces a tiny message.

For example, instead of sending this on every keystroke:

```json
{
  "type": "BUFFER_UPDATE",
  "matchId": "match_123",
  "seq": 42,
  "payload": {
    "content": "function greet(name) {\n  console.log('hello ' + name)\n}\n"
  }
}
```

the frontend can send this:

```json
{
  "type": "BUFFER_UPDATE",
  "matchId": "match_123",
  "seq": 42,
  "payload": {
    "delta": {
      "ops": [{ "type": "insert", "pos": 15, "text": "user" }]
    }
  }
}
```

For most edits, this reduced the payload by roughly 90%.

The implementation used CodeMirror's `ChangeSet`, which already knows what changed in the document. So I did not have to invent a diffing algorithm from scratch.

```tsx
const changesToDelta = useCallback((changes: ChangeSet): BufferDelta => {
  const ops: BufferDelta["ops"] = [];

  changes.iterChanges((from, to, _insertedFrom, _insertedTo, inserted) => {
    if (from !== to) {
      ops.push({ type: "delete", from, to });
    }

    if (inserted.toString()) {
      ops.push({
        type: "insert",
        pos: from,
        text: inserted.toString(),
      });
    }
  });

  return { ops };
}, []);
```

Then instead of sending the full content on every editor update:

```tsx
const handleContentChange = useCallback(
  (_content: string, changes: ChangeSet) => {
    if (viewStateRef.current !== "playing") return;

    const delta = changesToDelta(changes);
    sendBufferUpdate(undefined, delta);

    const content = getPlayerContent();

    if (!finishSentRef.current && content === targetCodeRef.current) {
      finishSentRef.current = true;
      sendPlayerFinished();
    }
  },
  [changesToDelta, sendBufferUpdate, getPlayerContent, sendPlayerFinished],
);
```

The opponent side also became delta-aware.

```tsx
const applyDelta = useCallback((delta: BufferDelta) => {
  if (!rightViewRef.current) return;

  const view = rightViewRef.current;
  const changes: { from: number; to?: number; insert?: string }[] = [];

  for (const op of delta.ops) {
    if (op.type === "delete") {
      changes.push({ from: op.from, to: op.to });
    } else if (op.type === "insert") {
      changes.push({ from: op.pos, insert: op.text });
    }
  }

  view.dispatch({
    changes: changes as any,
  });
}, []);
```

And for backwards compatibility, the message parser still accepted either a full snapshot or a delta.

```tsx
export function parseBufferUpdatePayload(
  payload: unknown,
): BufferUpdatePayload | null {
  if (!isRecord(payload)) return null;

  const content =
    typeof payload.content === "string" ? payload.content : undefined;

  const cursor =
    typeof payload.cursor === "number" ? payload.cursor : undefined;

  let delta: BufferDelta | undefined = undefined;
  const deltaObj = payload.delta as Record<string, unknown> | undefined;

  if (deltaObj && Array.isArray(deltaObj.ops)) {
    delta = { ops: deltaObj.ops as BufferDelta["ops"] };
  }

  if (!content && !delta) return null;

  return { content, delta, cursor };
}
```

This change not only reduced the egress, but also opened doors for some new features!

Because once every edit is represented as an operation, replay becomes much easier.

A replay is not just "store final buffer". A replay needs to know how the buffer changed over time. Sending deltas meant I already had a stream of operations that looked like:

```json
[
  {
    "timestamp": 1716100000000,
    "ops": [{ "type": "delete", "from": 32, "to": 33 }]
  },
  {
    "timestamp": 1716100000300,
    "ops": [{ "type": "insert", "pos": 32, "text": "." }]
  }
]
```

Spectator mode also benefited from the same idea.

A spectator does not need the entire buffer every time a player presses a key. They need the initial snapshot when they join, and then a stream of deltas after that.

So the backend eventually started keeping player buffers in memory:

```go
type Match struct {
	ID            string
	PlayerA       *Client
	PlayerB       *Client
	Status        MatchStatus
	TargetCode    string
	PollutedCode  string
	PlayerABuffer string
	PlayerBBuffer string
	LastSeqByID   map[string]int64
}
```

When a delta comes in, the backend applies it to its own copy too.

```go
if payload.Delta != nil {
	if sender == match.PlayerA {
		match.PlayerABuffer = applyBufferDelta(match.PlayerABuffer, payload.Delta)
	} else if sender == match.PlayerB {
		match.PlayerBBuffer = applyBufferDelta(match.PlayerBBuffer, payload.Delta)
	}
} else if payload.Content != nil {
	if sender == match.PlayerA {
		match.PlayerABuffer = *payload.Content
	} else if sender == match.PlayerB {
		match.PlayerBBuffer = *payload.Content
	}
}
```

This made spectator mode much cleaner:

```txt
spectator joins
  |
  v
server sends current PlayerA and PlayerB buffer snapshots
  |
  v
server streams future deltas
```

Same for replay:

```txt
match starts with pollutedCode
  |
  v
apply delta 1
  |
  v
apply delta 2
  |
  v
apply delta 3
  |
  v
reconstruct how the game actually happened
```

So that one commit did three things at once:

- reduced WebSocket payload size by roughly 90%
- made spectator mode cheaper and simpler
- made replay data more natural to store and reconstruct

I wish I could say I planned all of this from the beginning, but no. The first goal was just "stop sending the whole buffer every time someone types `j`".

The nice architecture came as a side effect.

## Relaying updates safely

The relay path also evolved.

The server does not blindly forward messages. It checks if the sender is in a match, if the match exists, if the match is playing, and if the sequence number is newer than the last one.

```go
func (h *Hub) relayBufferUpdate(sender *Client, message Envelope) {
	h.mu.Lock()
	defer h.mu.Unlock()

	if sender.MatchID == "" || message.MatchID == "" || sender.MatchID != message.MatchID {
		h.sendErrorLocked(sender, "invalid_match", "buffer update does not belong to active match")
		return
	}

	match, ok := h.matches[sender.MatchID]
	if !ok || match.Status != MatchPlaying {
		h.sendErrorLocked(sender, "match_not_active", "match is not active")
		return
	}

	lastSeq := match.LastSeqByID[sender.ID]
	if message.Seq <= lastSeq {
		return
	}

	match.LastSeqByID[sender.ID] = message.Seq

	var payload BufferUpdatePayload
	if err := json.Unmarshal(message.Payload, &payload); err == nil {
		if payload.Delta != nil {
			if sender == match.PlayerA {
				match.PlayerABuffer = applyBufferDelta(match.PlayerABuffer, payload.Delta)
			} else if sender == match.PlayerB {
				match.PlayerBBuffer = applyBufferDelta(match.PlayerBBuffer, payload.Delta)
			}
		} else if payload.Content != nil {
			if sender == match.PlayerA {
				match.PlayerABuffer = *payload.Content
			} else if sender == match.PlayerB {
				match.PlayerBBuffer = *payload.Content
			}
		}
	}

	opponent := match.opponentOf(sender)
	if opponent == nil {
		h.sendErrorLocked(sender, "opponent_missing", "opponent is unavailable")
		return
	}

	message.PlayerID = sender.ID
	message.Timestamp = time.Now().UTC().Unix()

	h.sendEnvelopeLocked(opponent, message)
}
```

The sequence check is small but important.

WebSockets preserve order on a connection, but once reconnections, retries, stale UI state, and multiple tabs enter the picture, having a sequence number gives the backend a cheap sanity check.

I don't want an old update coming in and overwriting newer state. That would be very funny for exactly 2 seconds, and then it would become a support issue.

## "That" React bug

One of the most annoying bugs I hit was around matchmaking.

The symptom looked backend-ish.

Sometimes users would connect, matchmaking would behave weirdly, sockets would close, and I kept thinking:

"Yeah, this has to be a WebSocket bug."

It was not a WebSocket bug.

It was React.

In dev mode, React Strict Mode can run effects twice. In production, the effect runs once. This created a very annoying difference between local testing and production behavior.

At one point the frontend had two effects involved in starting matchmaking. One effect assigned a ref, another effect called the ref.

Simplified, it looked like this:

```tsx
const beginMatchmakingRef = useRef<() => void>(() => {});

useEffect(() => {
  beginMatchmakingRef.current();
}, []);

const beginMatchmaking = useCallback(() => {
  disconnect();
  cleanupEditors();
  connect(callbacks);
}, [disconnect, cleanupEditors, connect]);

useEffect(() => {
  beginMatchmakingRef.current = () => {
    beginMatchmaking();
  };
}, [beginMatchmaking]);
```

In dev, because effects were firing twice, it accidentally behaved like the ref eventually got assigned and called properly.

In production, it fired once, and the mount effect could call the empty placeholder.

So I was staring at the backend, checking queue state, looking at websocket close handlers, wondering why the hub was betraying me.

But the actual bug was effect sequencing.

The fix was to make the mount effect call `beginMatchmaking` directly, instead of routing through a ref that may or may not have been assigned yet.

```tsx
const beginMatchmaking = useCallback(() => {
  disconnect();
  cleanupEditors();

  finishSentRef.current = false;
  targetCodeRef.current = "";
  pollutedCodeRef.current = "";

  setStatusText("Connecting to matchmaking server...");
  setViewState("matchmaking");

  connect(callbacks);
}, [disconnect, cleanupEditors, connect]);

useEffect(() => {
  beginMatchmaking();
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);
```

Later, when guest sessions and async setup came in, I added a run id to prevent stale async matchmaking runs from continuing after a newer one had started.

```tsx
const matchmakingRunRef = useRef(0);

const beginMatchmaking = useCallback(
  async (providedRunId?: number) => {
    const runId = providedRunId ?? matchmakingRunRef.current + 1;

    if (providedRunId === undefined) {
      matchmakingRunRef.current = runId;
    }

    const guestProfile = await ensureGuest();

    if (matchmakingRunRef.current !== runId) {
      return;
    }

    disconnect();
    cleanupEditors();
    connect(callbacks);
  },
  [ensureGuest, disconnect, cleanupEditors, connect],
);
```

Frontend is not the focus of this article, but I had to mention this one because it was such a classic bug.

The backend was not innocent either, though. This bug also forced me to make the backend stricter about dead clients.

## Dead clients and queue ghosts

When users refresh a page, close a tab, lose connection, or get hit by some frontend weirdness, the backend has to clean up properly.

Initially, unregistering a client was simple:

```go
func (h *Hub) handleUnregister(client *Client) {
	h.mu.Lock()
	defer h.mu.Unlock()

	delete(h.clients, client)
	h.removeFromQueueLocked(client)

	if client.ID != "" {
		if current, ok := h.clientsByID[client.ID]; ok && current == client {
			delete(h.clientsByID, client.ID)
		}
	}

	close(client.send)
}
```

This is fine when the world is kind.

The world is not kind.

Later, I added explicit closed state and active checks:

```go
type Client struct {
	ID         string
	Rating     int
	EnqueuedAt time.Time
	Conn       *websocket.Conn
	Hub        *Hub
	send       chan []byte
	closed     bool
	MatchID    string
}
```

And then before matching anyone:

```go
func (h *Hub) isClientActiveLocked(client *Client) bool {
	if client == nil || client.closed {
		return false
	}

	_, ok := h.clients[client]
	if !ok {
		return false
	}

	return client.isHeartbeatFresh()
}
```

The queue matcher now checks that both players are still active before starting a match.

```go
func (h *Hub) tryMatchBucket(bucket *ratingBucket) {
	for len(bucket.players) >= 2 {
		pA := bucket.players[0]
		pB := bucket.players[1]
		bucket.players = bucket.players[2:]

		validA := h.isClientActiveLocked(pA) && pA.ID != ""
		validB := h.isClientActiveLocked(pB) && pB.ID != ""

		if pA == pB || !validA || !validB {
			if validB && pA != pB {
				bucket.players = append([]*Client{pB}, bucket.players...)
			}
			if validA {
				bucket.players = append([]*Client{pA}, bucket.players...)
			}
			continue
		}

		h.startMatchLocked(pA, pB, nil)
	}
}
```

This fixed a class of bugs where a stale client could sit inside a queue and get matched later.

There are few things more cursed than matching a real user with a ghost websocket.

## Finishing a match

The win condition is simple from the player's perspective.

If your buffer equals the target buffer, you win.

The frontend detects that and sends `PLAYER_FINISHED`. But the backend still owns the final transition.

```go
func (h *Hub) finishMatch(client *Client, message Envelope) {
	h.mu.Lock()
	defer h.mu.Unlock()

	if client.MatchID == "" {
		h.sendErrorLocked(client, "invalid_match", "no active match to finish")
		return
	}

	match, ok := h.matches[client.MatchID]
	if !ok || match.Status != MatchPlaying {
		h.sendErrorLocked(client, "match_not_active", "match is not active")
		return
	}

	opponent := match.opponentOf(client)
	now := time.Now().UTC()

	match.Status = MatchFinished
	match.FinishedAt = &now
	match.WinnerID = client.ID

	// persist match, update rating, send GAME_OVER
}
```

A match can end in a few ways now:

- one player finishes first
- one player disconnects
- round timer expires
- tournament result is reported
- bot match completes

This is where the backend started to feel less like a relay and more like a small match engine.

## Ratings

The rating system is Elo-based.

The core function is tiny:

```go
func CalculateElo(
	playerRating,
	opponentRating float64,
	isPlayerWinner bool,
) (float64, float64) {
	const k = 32.0

	playerExpected := 1.0 / (1.0 + math.Pow(10, (opponentRating-playerRating)/400))
	opponentExpected := 1.0 / (1.0 + math.Pow(10, (playerRating-opponentRating)/400))

	playerScore := 0.0
	opponentScore := 0.0

	if isPlayerWinner {
		playerScore = 1.0
	} else {
		opponentScore = 1.0
	}

	newPlayerRating := playerRating + k*(playerScore-playerExpected)
	newOpponentRating := opponentRating + k*(opponentScore-opponentExpected)

	return newPlayerRating, newOpponentRating
}
```

But the important part is not this function. The important part is doing the match creation and rating update together.

```go
func CreateMatchAndSendRatingDelta(
	db *gorm.DB,
	winnerID,
	loserID,
	targetCode,
	pollutedCode string,
) (
	matchID string,
	winnerDelta,
	loserDelta,
	winnerNewRating,
	loserNewRating float64,
	err error,
) {
	err = db.Transaction(func(tx *gorm.DB) error {
		usersByPlayerID, lookupErr := lockUsersByPlayerID(tx, winnerID, loserID)
		if lookupErr != nil {
			return lookupErr
		}

		winner := usersByPlayerID[winnerID]
		loser := usersByPlayerID[loserID]

		oldWinnerRating := winner.Rating
		oldLoserRating := loser.Rating

		newWinnerRating, newLoserRating := utils.CalculateElo(
			winner.Rating,
			loser.Rating,
			true,
		)

		match := &models.Match{
			MatchID:      uuid.New(),
			PlayerAID:    &winner.ID,
			PlayerBID:    &loser.ID,
			TargetCode:   targetCode,
			PollutedCode: pollutedCode,
			WinnerID:     winner.ID,
			Outcome:      "decisive",
			FinishedAt:   time.Now().UTC(),
		}

		if err := tx.Create(match).Error; err != nil {
			return err
		}

		winner.Rating = newWinnerRating
		loser.Rating = newLoserRating

		if err := UpdateUserStats(tx, winner, true); err != nil {
			return err
		}
		if err := UpdateUserStats(tx, loser, false); err != nil {
			return err
		}

		matchID = match.MatchID.String()
		winnerDelta = winner.Rating - oldWinnerRating
		loserDelta = loser.Rating - oldLoserRating
		winnerNewRating = winner.Rating
		loserNewRating = loser.Rating

		return nil
	})

	return
}
```

The users are locked before rating updates:

```go
func lockUsersByPlayerID(tx *gorm.DB, playerIDs ...string) (map[string]*models.User, error) {
	var users []models.User

	if err := tx.
		Clauses(clause.Locking{Strength: "UPDATE"}).
		Where("CONCAT(provider, ':', provider_id) IN ?", playerIDs).
		Order("id ASC").
		Find(&users).Error; err != nil {
		return nil, err
	}

	usersByPlayerID := make(map[string]*models.User, len(users))

	for i := range users {
		user := &users[i]
		usersByPlayerID[user.Provider+":"+user.ProviderID] = user
	}

	return usersByPlayerID, nil
}
```

This avoids two simultaneous match results updating the same player rating in weird ways.

Again, not glamorous. But this is the stuff that matters.

## What changed after launch

After launch, the backend started getting features pretty quickly.

Some of them were expected:

- leaderboard
- rating changes
- match history
- replay
- bot mode
- guest mode
- spectator mode
- private lobbies
- tournament mode

Some of them came directly from user feedback.

For example, guest mode changed a lot of assumptions. Initially I could assume a player had a provider id like:

```txt
google:123456789
github:123456789
```

But guest players needed their own identity flow. Tournament guests needed another identity flow. Authenticated players playing against guests needed mixed rating logic.

So the match persistence path eventually had to handle cases like:

```go
winnerIsAuthenticated := providerIDLike.MatchString(winner.ID)
loserIsAuthenticated := providerIDLike.MatchString(loser.ID)

switch {
case winnerIsAuthenticated && !loserIsAuthenticated:
	// authenticated player beat a guest
	// update authenticated player's rating

case !winnerIsAuthenticated && loserIsAuthenticated:
	// guest beat authenticated player
	// update authenticated player's rating as a loss
}
```

This is why I always laugh a little when I say "just add guest mode".

There is no "just". The word "just" has destroyed more engineering estimates than any programming language ever could.

## Why I am open sourcing Vim Royale

A few users asked if they could contribute to Vim Royale.

That felt very cool.

Also a little scary.

Until now, Vim Royale was mostly me shipping things whenever I found fit. If I saw a bug, I fixed it. If I wanted bot mode, I built it. If I wanted spectator mode, I hacked on it until it worked.

But open source is a different kind of responsibility.

I want to experience what it feels like to maintain a product that other people use, read, improve, complain about, and hopefully enjoy contributing to.

There is a different skill in making code contributor-friendly. It is not just about writing code that works. It is about writing code that someone else can understand without needing a brain dump from you at 2 AM.

I am not claiming Vim Royale is perfect. It is definitely not. Some parts are clean. Some parts are battle scars wearing a trench coat.

But I think that is also the fun of open sourcing it now.

It is real software. It has real users. It has real bugs. It has real design tradeoffs.

And now it can have real contributors too.

## Things contributors can help with

If you want to contribute, here are some areas that I think would be really useful:

- Better matchmaking tuning
- More Vim challenges
- Better replay viewer
- More bot personalities
- Tournament improvements
- Better tests around the match engine
- UI/UX improvements
- Accessibility fixes
- Better docs for local setup
- More observability around WebSocket failures

The backend especially has a lot of fun problems.

Matchmaking sounds simple until you deal with waiting time, ratings, reconnects, duplicate tabs, stale sockets, guests, tournaments, and people rage-quitting mid-match.

And that is exactly why it is fun.

## Closing thoughts

Building Vim Royale taught me that even a "small multiplayer game" has a surprising amount of backend design hiding inside it.

You need a protocol.
You need identity.
You need queues.
You need fair matchmaking.
You need to handle disconnects.
You need persistence.
You need rating updates.
You need to not match people with ghosts.
You need to debug React bugs that look like WebSocket bugs.

Classic software stuff, basically.

I built Vim Royale because I wanted to play a multiplayer Vim game. The fact that other people also wanted to play it, and now some people want to contribute to it, makes me very happy.

If you like Vim, Go, React, WebSockets, or simply enjoy breaking things in useful ways, come contribute.

{{< callout type="info" >}}
You can find the code here: [github.com/Jitesh117/vim_royale](https://github.com/Jitesh117/vim_royale)
{{< /callout >}}

See you in the queue!
