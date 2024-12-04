+++
title = "Master Normal Mode in Vim"
date = 2024-12-04T07:40:04+05:30
draft = false
showToc = true
cover.image = 'images/normal_mode_vim.jpg'
tags = ['vim']
+++

Vim's Normal mode is the foundation of efficient text editing. Unlike traditional editors where typing immediately inserts text, Vim's modal approach separates navigation and manipulation from insertion. This separation enables incredibly powerful and precise editing capabilities.

### Why Normal Mode?

- Keeps your hands on the home row
- Provides atomic operations that can be combined
- Reduces repetitive strain by minimizing hand movement
- Enables powerful combinations of commands

## Basic Navigation

### Character Navigation

The fundamental movement keys (hjkl) are your bread and butter:

```
h - left
j - down
k - up
l - right
```

Example:

```
This is a sam|ple text
```

Press `hhhh` →

```
This is |a sample text
```

### Word Navigation

Word-based movements are more efficient than character-by-character:

- `w` - move to start of next word
- `e` - move to end of current/next word
- `b` - move backward to start of current/previous word
- `W`, `E`, `B` - same as above but for WORDS (space-separated)

Example with `w`:

```
The |quick brown fox jumps
```

Press `w` twice →

```
The quick |brown fox jumps
```

### Line Navigation

Master these for quick horizontal movement:

```
0  - start of line
^  - first non-blank character
$  - end of line
g_ - last non-blank character
```

Example sequence:

```
    Hello |world!
```

Press `0` →

```
|    Hello world!
```

Press `^` →

```
    |Hello world!
```

### Sentence and Paragraph Movement

- `)` - next sentence
- `(` - previous sentence
- `}` - next paragraph
- `{` - previous paragraph

Example:

```
First sentence. Second |sentence. Third sentence.

New paragraph starts here.
```

Press `)` →

```
First sentence. Second sentence. |Third sentence.

New paragraph starts here.
```

## Advanced Navigation

### Screen Position Jumps

```
H - High (top of screen)
M - Middle of screen
L - Low (bottom of screen)
zt - move current line to top
zz - move current line to middle
zb - move current line to bottom
```

### File Position Jumps

```
gg - start of file
G  - end of file
42G - jump to line 42
```

### Scrolling

```
Ctrl-e - scroll down one line
Ctrl-y - scroll up one line
Ctrl-f - scroll forward one screen
Ctrl-b - scroll backward one screen
Ctrl-d - scroll down half screen
Ctrl-u - scroll up half screen
```

## Text Manipulation

### Basic Deletion Commands

```
x  - delete character under cursor
X  - delete character before cursor
dd - delete current line
D  - delete from cursor to end of line
```

Example sequence:

```
The quiick |brown fox
```

Press `x` →

```
The quiick|brown fox
```

Press `X` →

```
The quic|brown fox
```

### Deletion with Movement

Combine `d` with any movement command:

```
dw  - delete to next word
db  - delete to previous word
d$  - delete to end of line
d0  - delete to start of line
d)  - delete to next sentence
d}  - delete to next paragraph
```

Example:

```
The quick |brown fox jumps
```

Press `dw` →

```
The quick |fox jumps
```

### Change Commands

Similar to delete but enters Insert mode:

```
cw  - change word
cc  - change entire line
C   - change to end of line
ct{char} - change until character
```

Example:

```
The |quick brown fox
```

Press `ct ` →

```
The |brown fox
```

(now in Insert mode)

## Advanced Editing Commands

### Text Objects

Text objects combine with operations:

```
iw - inner word
aw - a word (including space)
is - inner sentence
as - a sentence
ip - inner paragraph
ap - a paragraph
i" - inner quotes
a" - a quotes (including quotes)
i( - inner parentheses
a( - a parentheses (including parentheses)
```

Examples:

```
The "quick |brown" fox
```

Press `ci"` →

```
The "|" fox
```

(in Insert mode)

### Block Operations

Visual block mode (`Ctrl-v`):

```
Ctrl-v - start visual block
I      - insert at start of block
A      - append at end of block
c      - change block
d      - delete block
```

Example:

```
first  |item
second item
third  item
```

Press `Ctrl-v jj I prefix ` →

```
prefix first  item
prefix second item
prefix third  item
```

## Pattern Matching and Search

### Basic Search

```
/pattern - search forward
?pattern - search backward
n       - next match
N       - previous match
*       - search word under cursor forward
#       - search word under cursor backward
```

### Search and Replace

```
:s/old/new/    - replace first occurrence in line
:s/old/new/g   - replace all occurrences in line
:%s/old/new/g  - replace all occurrences in file
:%s/old/new/gc - replace with confirmation
```

Example of search and replace with confirmation:

```vim
:%s/colour/color/gc
replace with color (y/n/a/q/l/^E/^Y)?
```

## Marks and Jumps

### Setting Marks

```
m{a-zA-Z} - set mark
'{a-z}    - jump to line of mark
`{a-z}    - jump to position of mark
```

Example:

```
First line
Second |line
Third line
```

Press `ma` to set mark 'a', move around, then `` `a `` to return

### Jump List

```
Ctrl-o - jump back
Ctrl-i - jump forward
:jumps - show jump list
```

## Text Objects

### Inner Text Objects

```
i{object} - inner object
iw - inner word
is - inner sentence
ip - inner paragraph
i" - inner quotes
i' - inner single quotes
i) - inner parentheses
i] - inner square brackets
i} - inner curly brackets
it - inner tag
```

### A Text Objects (includes delimiter)

```
a{object} - a object
aw - a word
as - a sentence
ap - a paragraph
a" - a quotes
a' - a single quotes
a) - a parentheses
a] - a square brackets
a} - a curly brackets
at - a tag
```

## Macros and Advanced Automation

### Recording Macros

```
q{register} - start recording
q          - stop recording
@{register} - play macro
@@         - repeat last macro
```

Example macro to wrap word in quotes:

```
The |quick brown fox
```

Press `qa i" <Esc> ea " <Esc> q` →

```
The "|quick" brown fox
```

### Complex Macro Example

Create numbered list:

```
|apple
banana
cherry
```

Record: `qa I1. <Esc> j q`
Play twice: `@a @a`
Result:

```
1. apple
2. banana
3. cherry
```

## Register Operations

### Named Registers

```
"ay  - yank into register a
"ap  - paste from register a
"by  - yank into register b
"bp  - paste from register b
```

### Special Registers

```
""   - unnamed register (last delete/yank)
"0   - last yank
"1-9 - last deletes
"-   - small delete register
"/   - last search pattern
":   - last command
".   - last inserted text
"%   - current file name
"#   - alternate file name
"=   - expression register
"*   - system clipboard (X11 primary)
"+   - system clipboard (X11 clipboard)
"_   - black hole register
```

## Window Management

### Split Windows

```
:sp[lit]     - horizontal split
:vsp[lit]    - vertical split
Ctrl-w s     - horizontal split
Ctrl-w v     - vertical split
```

### Window Navigation

```
Ctrl-w h     - move to left window
Ctrl-w j     - move to window below
Ctrl-w k     - move to window above
Ctrl-w l     - move to right window
Ctrl-w w     - cycle through windows
```

### Window Resizing

```
Ctrl-w =     - make windows equal size
Ctrl-w _     - maximize height
Ctrl-w |     - maximize width
Ctrl-w >     - increase width
Ctrl-w <     - decrease width
Ctrl-w +     - increase height
Ctrl-w -     - decrease height
```

## Best Practices and Tips

### Efficiency Guidelines

1. Avoid repeating keys more than twice
2. Use text objects over character movements
3. Think in operations + motions
4. Learn one new command per week

### Common Patterns

1. Change inside quotes: `ci"`
2. Delete until character: `dt{char}`
3. Change until character: `ct{char}`
4. Delete around parentheses: `da)`
5. Yank inside paragraph: `yip`

### Movement Strategy

1. For small movements (1-3 characters): use `h/j/k/l`
2. For word movements: use `w/b/e`
3. For line movements: use `0/$`
4. For screen movements: use `H/M/L`
5. For file movements: use `gg/G`

## Common Patterns and Workflows

### Code Editing Patterns

```
ci(  - change inside parentheses
ca{  - change around curly braces
yi"  - yank inside quotes
da]  - delete around square brackets
```

Example:

```
function hello(|old_name) {
```

Press `ci(` →

```
function hello(|) {
```

(in Insert mode)

### Text Processing Workflows

1. Sorting lines:

   ```
   |apple
   cherry
   banana
   ```

   Press `vip:sort` →

   ```
   apple
   banana
   cherry
   ```

2. Indenting blocks:
   ```
   if (condition) {
   |first line
   second line
   third line
   }
   ```
   Press `>i{` →
   ```
   if (condition) {
       first line
       second line
       third line
   }
   ```

## Troubleshooting

### Common Issues

1. **Stuck in Insert Mode**: Press `<Esc>`
2. **Stuck in Visual Mode**: Press `<Esc>`
3. **Stuck in Command Mode**: Press `<Esc>` twice
4. **Lost in File**: Use `gg` or `G` to orient yourself

### Recovery Commands

```
u        - undo
Ctrl-r   - redo
:earlier 1m - go back 1 minute
:later 1m  - go forward 1 minute
```

## Conclusion

Mastering Normal mode in Vim is a journey that requires patience and practice. Start with basic movements, gradually incorporate text objects, and eventually master advanced techniques like macros and registers. Remember that the goal is not to memorize every command but to build muscle memory for the commands you use most frequently.

Happy Vimming!
