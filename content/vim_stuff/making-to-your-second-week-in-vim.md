+++
title = "Making it to Your Second Week in Vim"
date = 2025-03-05T19:21:30+05:30
draft = false
showToc = true
cover.image = '/images/vim_second_week.jpg'
tags = ['vim']
+++

So, you made it through the first week. You’re moving around with `hjkl`, and you’ve stopped instinctively reaching for your mouse. That’s a win.

But now you’re starting to realize there’s _so much more_ to Vim. And you’re right. The second week is where things get exciting. Let’s take your skills to the next level.

---

## Day 8: Leveling Up Your Movements

By now, you’re comfortable with `hjkl`, `w`, `b`, and `$`, but let’s move even faster.

### Faster Navigation:

- `gg` → Go to the top of the file
- `G` → Go to the bottom of the file
- `{` → Move up by a paragraph
- `}` → Move down by a paragraph
- `Ctrl-d` → Move down half a screen
- `Ctrl-u` → Move up half a screen

### Precision Jumps:

- `fx` → Move to the next occurrence of `x` on the current line
- `tx` → Move **before** the next occurrence of `x`
- `Fx` → Move **backwards** to `x`
- `Tx` → Move **before** `x` (backwards)

🎯 **Goal for today**: Navigate large files with ease. Try moving with `gg`, `G`, `{`, `}` and refine with `f`, `t`, `F`, `T`.

If you want to read more about vim text objects, check [this](https://jitesh117.github.io/vim_stuff/text-objects-in-vim/) out.

---

## Day 9: Mastering Text Objects

One of Vim’s superpowers is **text objects**. They let you operate on words, sentences, and blocks of text efficiently.

### Editing Inside and Around Objects:

- `ciw` → Change _inner_ word (delete & enter insert mode)
- `diw` → Delete _inner_ word
- `yiw` → Yank _inner_ word
- `ci"` → Change everything inside `"`
- `di(` → Delete everything inside `()`
- `da{` → Delete _around_ `{}` (including brackets)

🎯 **Goal for today**: Try `ciw`, `diw`, `ci"`, and `di(` on a text file.

---

## Day 10: Searching and Replacing Like a Pro

You already know how to search with `/word`, but let’s refine it.

### Smarter Searching:

- `/word\c` → Case-insensitive search
- `*` → Search for the word under the cursor
- `#` → Search _backward_ for the word under the cursor

### Global Find and Replace:

- `:%s/old/new/g` → Replace all occurrences of "old" with "new"
- `:%s/old/new/gc` → Replace all with confirmation

🎯 **Goal for today**: Search and replace words in a document efficiently.
Read more about search and replace [here](https://jitesh117.github.io/vim_stuff/search-and-replace/)

---

## Day 11: Buffers, Tabs, and Windows

Vim can manage multiple files. Let’s ditch the old `:e` and get serious. You can check out my blog post on this topic [here](https://jitesh117.github.io/vim_stuff/buffers-windows-and-tabs-in-vim/)

### Buffers:

- `:e file` → Open a new file
- `:ls` → List open buffers
- `:bnext` (`:bn`) → Go to the next buffer
- `:bprev` (`:bp`) → Go to the previous buffer
- `:bd` → Close the current buffer

### Windows (Splits):

- `:split` → Horizontal split
- `:vsplit` → Vertical split
- `Ctrl-w w` → Switch between windows
- `Ctrl-w q` → Close the current window

### Tabs:

- `:tabnew file` → Open a file in a new tab
- `gt` → Move to the next tab
- `gT` → Move to the previous tab
- `:tabclose` → Close the current tab

🎯 **Goal for today**: Manage multiple files using buffers, splits, and tabs.

---

## Day 12: Macros – Automate Everything

I've wrote about macros in great detail [here](https://jitesh117.github.io/vim_stuff/vim_macros/)
Macros let you record a series of commands and repeat them.

### Recording and Playing Macros:

1. Start recording: `q[a-z]` (e.g., `qa` to record in register `a`)
2. Perform some actions
3. Stop recording: `q`
4. Replay macro: `@a`
5. Repeat last macro: `@@`

🎯 **Goal for today**: Record a macro to format or edit multiple lines and replay it.

---

## Day 13: Visual Mode and Block Editing

To read about visual mode in detail, checkout this [blog](https://jitesh117.github.io/vim_stuff/master-visual-mode/) of mine,

### Visual Mode Basics:

- `v` → Start character-wise selection
- `V` → Start line-wise selection
- `Ctrl-v` → Start **block** selection (useful for column editing!)

### Block Editing Magic:

- Select a column with `Ctrl-v`
- Press `I` (insert mode) and type something
- Press `Esc`, and it applies to **all** selected lines!

🎯 **Goal for today**: Use `Ctrl-v` to insert text at the beginning of multiple lines at once.

---

## Day 14: Making Vim _Yours_

Now that you’ve got the basics down, let’s tweak Vim to fit your workflow.

### Essential `.vimrc` Settings:

```vim
set number          " Show line numbers
set relativenumber  " Relative numbers for quick movement
set ignorecase      " Case-insensitive search
set smartcase       " Case-sensitive if uppercase letters are used
set incsearch       " Incremental search
set hlsearch        " Highlight search matches
set wildmenu        " Better command-line completion
set expandtab       " Use spaces instead of tabs
set shiftwidth=4    " Indent by 4 spaces
set tabstop=4       " A tab counts as 4 spaces
```

---

## Final Thoughts

You’ve made it through your second week, and now Vim feels _a little less scary_. You’re moving faster, editing with text objects, jumping around files, and even using macros.

The journey doesn’t end here—Vim mastery takes time. But if you keep practicing, soon you’ll wonder how you ever lived without it.

Happy vimming!
