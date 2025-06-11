+++
title = 'Must Know Vim Commands'
date = 2025-06-11T22:50:15+05:30
draft = false
cover.Image = "/images/must_know_vim_commands.jpg"
+++

Vim can feel like a boss fight when you're starting out — blinking cursor, no mouse, and zero clues. But once you get past the "what even is this" phase, it becomes a power tool that makes other editors feel… clunky.

So in this article, I would list down the vim commands which I use THE MOST. Many people on twitter were asking me to make a list of commands I use daily. So here it is!

This one would be mostly a cheatsheet sort of article and not a "How To" one.

## Navigation Basics

### `h`, `j`, `k`, `l` – The Arrow Keys

You’ll see these everywhere.

```vim
h → left
j → down
k → up
l → right
```

### `w`, `b`, `e` – Word Navigation

- `w`: jump to the start of the next word
- `b`: back to the beginning of the previous word
- `e`: end of the current/next word

```vim
:w  → Save file
dd  → Delete current line
```

### `gg` and `G` – Top and Bottom

```vim
gg → go to the top of the file
G  → go to the bottom
```

### `0`, `^`, `$` – Line Navigation

```vim
0 → Beginning of line
^ → First non-blank character
$ → End of line
```

## Editing Text

### `i`, `I`, `a`, `A` – Insert Modes

```vim
i → insert before cursor
I → insert at start of line
a → append after cursor
A → append at end of line
```

### `o` and `O` – New Lines

```vim
o → open new line below
O → open new line above
```

### `x` and `X` – Delete Characters

```vim
x → delete character under cursor
X → delete character before cursor
```

### `dd`, `yy`, `p`, `P` – Line Operations

```vim
dd → delete (cut) current line
yy → yank (copy) current line
p  → paste below
P  → paste above
```

## Searching and Replacing

### `/pattern` – Search Forward

```vim
/foo → search for "foo" forward
n    → next match
N    → previous match
```

### `:%s/old/new/g` – Replace All

Replace "dog" with "cat" in the whole file:

```vim
:%s/dog/cat/g
```

Replace only the first instance on each line:

```vim
:%s/dog/cat/
```

Ask before replacing:

```vim
:%s/dog/cat/gc
```

## Visual Mode

### `v`, `V`, `Ctrl+v`

```vim
v        → character-wise visual mode
V        → line-wise visual mode
Ctrl+v   → block visual mode (aka column selection)
```

Once selected, you can `d`, `y`, or `p` like usual.

## File Operations

### `:w`, `:q`, `:wq`, `:q!`

```vim
:w   → save file
:q   → quit
:wq  → save and quit
:q!  → quit without saving
```

## Buffers, Windows, and Tabs

### Buffers

```vim
:ls     → list open buffers
:bd     → delete current buffer
:buffer <n> → switch to buffer number n
```

### Windows (Splits)

```vim
:split         → horizontal split
:vsplit        → vertical split
Ctrl+w w       → switch between windows
Ctrl+w q       → close window
```

### Tabs

```vim
:tabnew        → open new tab
gt             → next tab
gT             → previous tab
:tabclose      → close tab
```

## Marks and Jumps

### `m{a-z}` and `'a`, `` `a``

Set a mark:

```vim
ma → set mark 'a' at current position
```

Jump to it:

```vim
'a → jump to line of mark a
`a → jump to exact position of mark a
```

## Other Handy Commands

### `.` – Repeat Last Change

If you just deleted a line with `dd`, pressing `.` will repeat that action.

### `u` and `Ctrl+r`

```vim
u        → undo
Ctrl+r   → redo
```

### `J` – Join Lines

```vim
J → joins the current line with the next line
```

### `>>`, `<<` – Indent and Outdent

```vim
>> → indent current line
<< → unindent current line
```

In visual mode, you can select multiple lines and use `>` or `<` to shift them.

## Config Tweaks Worth Knowing

Put these in your `~/.vimrc` to make Vim more friendly:

```vim
set number          " Show line numbers
set relativenumber  " Relative line numbers
set tabstop=4       " Number of spaces per tab
set shiftwidth=4    " Indentation amount
set expandtab       " Use spaces instead of tabs
set incsearch       " Incremental search
set ignorecase      " Case-insensitive search
```

## Final Tip

You don’t have to memorize all this in one sitting. Start with the basics (`h`, `j`, `k`, `l`, `:wq`) and layer on as you go. Vim rewards muscle memory. The more you use it, the more natural it feels.

Happy Vimming!
