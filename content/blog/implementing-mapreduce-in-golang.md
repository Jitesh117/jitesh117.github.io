+++
title = "Implementing MapReduce in Golang"
date = 2025-11-21T08:14:37+05:30
draft = false
showToc = true
cover.image = '/images/mapreduce.png'
tags = ['golang', 'paper implementation']
+++

Ever wondered how Google processes massive amounts of data? Or how systems like Hadoop work under the hood? The secret is MapReduce - a simple but powerful way to process huge datasets by breaking them into smaller pieces.

In this post, we'll build our own MapReduce system in Go. I'll be basically be implementing the famous [Google paper](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf) in golang!

{{< callout type="info" >}}
You can find my implementation of the paper in this [github repository](https://github.com/Jitesh117/mapReduceGo/tree/master/mapreduce).
{{< /callout >}}

## Why I Built This

While learning about distributed systems, I came across Google's 2004 MapReduce paper.
Instead of just reading it, I decided to implement it myself in Go to truly understand
how it works. This post walks through my implementation and explains the key concepts
I learned along the way.

If you've ever been curious about how systems like Hadoop process terabytes of data,
or wondered what "MapReduce" actually means, this post is for you.

## What is MapReduce?

Think of MapReduce like organizing a massive library:

### The Problem:

You have 1 million books and need to count how many times each word appears across all of them.

### The Naive Way:

Read every book one by one. This would take forever.

### The MapReduce Way:

1. Hire 100 people (workers)
2. Give each person 10,000 books (map phase)
3. Each person counts words in their books
4. Combine everyone's counts together (reduce phase)

This is exactly what MapReduce does with data!

![MapReduce Design Overview](/images/mapreduce_design.png)

## The Two Phases

### Phase 1: Map (Break it Down)

The Map phase takes big chunks of data and breaks them into key-value pairs.

**Example:** Processing the text "foo bar baz foo"

```
Input: "foo bar baz foo"

Map Function Output:
foo → 1
bar → 1
baz → 1
foo → 1
```

Each word becomes a key, and we give it a value of "1" (meaning we saw it once).

### Phase 2: Reduce (Combine Results)

The Reduce phase takes all values for the same key and combines them.

**Example:** Combining the counts for "foo"

```
Input: foo → [1, 1, 1, 1]
Reduce Function Output: foo → 4
```

We saw "foo" four times, so the final count is 4.

## The Architecture

Our MapReduce system has three parts:

### 1. The Master (The Boss)

The Master is like a project manager:

- Knows what work needs to be done
- Assigns tasks to workers
- Keeps track of who's doing what
- Stores the results

```go
type Master struct {
    inputs       []string        // The data to process
    mapStatus    []TaskState     // Which map tasks are done?
    reduceStatus []TaskState     // Which reduce tasks are done?
    intermediate map[int]map[string][]string  // Temporary storage
    outputs      map[int]string  // Final results
}
```

### 2. The Workers (The Doers)

Workers are like employees who:

- Ask the Master: "What should I do?"
- Do the work (map or reduce)
- Report back: "I'm done!"
- Repeat until all work is finished

```go
type Worker struct {
    ID       int
    Master   *Master
    mapFn    MapFunc      // How to do map work
    reduceFn ReduceFunc   // How to do reduce work
}
```

### 3. The Tasks (The Work Items)

Each task contains:

- What type of work (map or reduce)
- Which piece of data to work on
- An ID number

```go
type Task struct {
    Type       TaskType  // MapTask, ReduceTask, or NoTask
    TaskID     int
    Data       string    // The actual data to process
}
```

## How It All Works Together

Let's trace through a complete example with the input:

```
["foo bar baz foo", "bar baz qux", "foo qux qux qux"]
```

### Step 1: Starting Up

```go
func main() {
    // Create the master with 3 pieces of data and 3 reducers
    master := NewMaster(inputs, 3)

    // Hire 4 workers
    for i := 0; i < 4; i++ {
        w := NewWorker(i, master, mapF, reduceF)
        go w.Run(&wg)  // Each worker runs independently
    }
}
```

### Step 2: Map Phase (Breaking Down)

**Worker 0 asks for work:**

```
Worker 0: "Hey Master, got any work?"
Master: "Yes! Process this: 'foo bar baz foo'"
```

**Worker 0 does the mapping:**

```go
func doMap(task Task) {
    // Call the map function
    kvs := mapF("doc-0", "foo bar baz foo")
    // Result: [
    //   {Key: "foo", Value: "1"},
    //   {Key: "bar", Value: "1"},
    //   {Key: "baz", Value: "1"},
    //   {Key: "foo", Value: "1"}
    // ]
}
```

**Worker 0 sorts these by reducer:**

This is important! We use a hash function to decide which reducer handles which words:

```go
partitions := make(map[int][]KeyValue)
for _, kv := range kvs {
    reducerNum := ihash(kv.Key) % 3  // Pick a reducer (0, 1, or 2)
    partitions[reducerNum] = append(partitions[reducerNum], kv)
}

// Might look like:
// Reducer 0: ["foo", "foo"]
// Reducer 1: ["bar"]
// Reducer 2: ["baz"]
```

**Worker 0 reports back:**

```
Worker 0: "Done! Here's my data organized by reducer."
Master: "Thanks! I'll store this."
```

**Meanwhile, other workers do the same:**

- Worker 1 processes "bar baz qux"
- Worker 2 processes "foo qux qux qux"
- Worker 3 waits (no work left)

All of this happens **at the same time**!

### Step 3: Waiting for Everyone

The Master keeps track of progress:

```go
func (m *Master) RequestTask() Task {
    // First, try to give out map tasks
    for i := 0; i < m.numMap; i++ {
        if m.mapStatus[i].State == "idle" {
            return Task{Type: MapTask, TaskID: i, Data: m.inputs[i]}
        }
    }

    // Are all map tasks done?
    if !allMapTasksDone() {
        return Task{Type: NoTask}  // "Sorry, wait a bit"
    }

    // All maps done! Now give out reduce tasks
    // ...
}
```

### Step 4: Reduce Phase (Combining)

Once all maps are done, the Master has intermediate data organized like this:

```
Reducer 0: {
    "foo": ["1", "1", "1", "1"],
    "qux": ["1", "1", "1", "1"]
}

Reducer 1: {
    "bar": ["1", "1"]
}

Reducer 2: {
    "baz": ["1", "1"]
}
```

**Worker 0 asks for work again:**

```
Worker 0: "Map tasks done. What now?"
Master: "Great! Do reduce task 0."
```

**Worker 0 does the reducing:**

```go
func doReduce(task Task) {
    // Get my partition
    partition := Master.GetReducePartition(0)
    // {"foo": ["1","1","1","1"], "qux": ["1","1","1","1"]}

    // Sort keys alphabetically
    keys := ["foo", "qux"]

    // Reduce each key
    for _, key := range keys {
        result := reduceF(key, partition[key])
        // reduceF("foo", ["1","1","1","1"]) returns "4"
        output += "foo 4\n"
    }

    // Report: "foo 4\nqux 4"
}
```

### Step 5: Final Output

After all reducers finish:

```
--- reducer 0 output ---
foo 4
qux 4

--- reducer 1 output ---
bar 2

--- reducer 2 output ---
baz 2
```

Let's trace the word "foo" from start to finish:

**Input Documents:**

1. "foo bar baz foo" (doc 0)
2. "bar baz qux" (doc 1)
3. "foo qux qux qux" (doc 2)

**Map Phase:**

- Worker 0 processes doc 0: emits `foo→1, bar→1, baz→1, foo→1`
- Worker 1 processes doc 1: emits `bar→1, baz→1, qux→1`
- Worker 2 processes doc 2: emits `foo→1, qux→1, qux→1, qux→1`

**Partitioning (using hash):**

- `ihash("foo") % 3 = 0` → All "foo" goes to Reducer 0
- Reducer 0 receives: `foo: [1, 1, 1]` from the three mappers

**Reduce Phase:**

- Reducer 0 gets: `{"foo": ["1","1","1"], "qux": ["1","1","1","1"]}`
- Sorts keys: `["foo", "qux"]`
- Processes "foo": counts 3 values → outputs "foo 3"

**Final Output:**

```
foo 3
```

The word "foo" appeared in docs 0 and 2, and our system correctly counted all occurrences!

## The Secret Sauce: Concurrency

The magic of this implementation is that **everything happens in parallel**.

### Without Concurrency (Slow)

```
Process doc 1 → Process doc 2 → Process doc 3
Total time: 3 seconds
```

### With Concurrency (Fast)

```
Process doc 1 ┐
Process doc 2 ├─ All at once!
Process doc 3 ┘
Total time: 1 second
```

### How Go Makes This Easy

Go has goroutines - lightweight threads that make concurrency simple:

```go
// Start 4 workers running at the same time
for i := 0; i < 4; i++ {
    go w.Run(&wg)  // "go" means "run this concurrently"
}

wg.Wait()  // Wait for everyone to finish
```

### Avoiding Chaos with Mutexes

When multiple workers access shared data, we need locks to prevent chaos:

```go
func (m *Master) RequestTask() Task {
    m.mu.Lock()           //  Lock the door
    defer m.mu.Unlock()   //  Unlock when done

    // Now only ONE worker can be in here at a time
    if m.mapStatus[0].State == "idle" {
        m.mapStatus[0].State = "in-progress"
        return Task{Type: MapTask, TaskID: 0}
    }
}
```

Without locks:

```
Worker 1: "Is task 0 idle? Yes!"
Worker 2: "Is task 0 idle? Yes!"
Both workers start task 0!  (Bad!)
```

With locks:

```
Worker 1:  "Is task 0 idle? Yes! I'll take it."
Worker 2:  (waiting for lock...)
Worker 1:  (done)
Worker 2:  "Is task 0 idle? No, Worker 1 took it. I'll try task 1."
```

## Fault Tolerance: When Workers Fail

What if a worker crashes? The Master has timeouts:

```go
// If a task has been "in-progress" for too long...
if time.Since(m.mapStatus[i].AssignedAt) > 5*time.Second {
    // Assume the worker died. Give it to someone else!
    return Task{Type: MapTask, TaskID: i}
}
```

This makes the system resilient - work never gets lost!

## The User's Job

Users of this MapReduce framework only need to write two functions:

### 1. Map Function (How to process one chunk)

```go
func mapF(docID string, contents string) []KeyValue {
    words := strings.Fields(contents)
    var kvs []KeyValue
    for _, word := range words {
        kvs = append(kvs, KeyValue{
            Key:   strings.ToLower(word),
            Value: "1",
        })
    }
    return kvs
}
```

"Split text into words, emit each word with count 1"

### 2. Reduce Function (How to combine values)

```go
func reduceF(key string, values []string) string {
    return fmt.Sprintf("%d", len(values))
}
```

"Count how many values we got"

That's it! The framework handles all the hard stuff:

- Distributing work to workers
- Running tasks in parallel
- Handling failures
- Organizing intermediate data
- Collecting final results

## Real-World Applications

This pattern is used everywhere:

**Web Search:**

- Map: Find keywords in each webpage
- Reduce: Rank pages by relevance

**Log Analysis:**

- Map: Parse each log entry
- Reduce: Count errors by type

**Machine Learning:**

- Map: Process training examples
- Reduce: Average the results

**E-commerce:**

- Map: Process each user's purchases
- Reduce: Find most popular products

## Why This Design Works

**1. Simplicity:** Users only write two functions - map and reduce

**2. Scalability:** Need to go faster? Add more workers!

**3. Fault Tolerance:** Workers can crash and restart

**4. Parallelism:** Everything that can run simultaneously does

**5. Clean Separation:** Framework code vs. user code are separate

## What We Learned

We built a complete MapReduce system that:

- Processes data in parallel using multiple workers
- Coordinates work with a Master
- Handles worker failures gracefully
- Provides a simple interface for users

This is the same pattern Google used to index the entire web, and it's the foundation of systems like Hadoop and Spark!

The beauty of MapReduce isn't in complex algorithms - it's in the simple idea that big problems can be broken into small pieces, processed independently, and combined back together.

The possibilities are endless once you understand the fundamentals.
