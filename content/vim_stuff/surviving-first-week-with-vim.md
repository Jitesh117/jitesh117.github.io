+++
title = "Surviving First Week With Vim"
date = 2025-01-29T19:16:34+05:30
draft = false
showToc = true
cover.image = '/images/vim_first_week.jpg'
tags = ["vim"]
+++

So, you’ve finally done it. You’ve taken the leap. You’ve closed VS Code (or maybe it crashed on you one too many times), and you’ve opened Vim. And now… you’re stuck.

Don’t worry—I’ve been there. Everyone has. The first week with Vim feels like you’ve been thrown into the deep end of a pool without knowing how to swim. But don’t panic! If you stick with it, you’ll start to see why so many developers swear by it. Let’s get you through this week, step by step.

## Day 1: Escaping the Void

The first thing you probably noticed is that typing doesn’t work as expected. That’s because Vim starts in _Normal mode_, where keys are commands, not text input.

- **Try pressing `i`** – Now you can type! You’re in _Insert mode_.
- **Want to get back to Normal mode?** Press `Esc`.
- **Want to save and quit?** Type `:wq` and press `Enter`.
- **Messed up and want to quit without saving?** Type `:q!` and press `Enter`.

🎯 **Goal for today**: Get comfortable switching between `i` (insert mode) and `Esc` (normal mode).

## Day 2: Moving Like a Vim User

You’ve probably been reaching for the arrow keys. I get it. But let’s try something different.

Use these instead:

- `h` → move left
- `l` → move right
- `j` → move down
- `k` → move up

At first, it feels like learning to walk again. But once you get used to it, it’s way faster.

### Bonus: Faster Movement

- `w` → jump to the start of the next word
- `b` → jump to the start of the previous word
- `0` → go to the beginning of the line
- `^` → go to the first non-whitespace character of the line
- `$` → go to the end of the line

🎯 **Goal for today**: Move around a file using `hjkl`, `w`, `b`, and `$`. Avoid the arrow keys!

## Day 3: Editing Without a Mouse

Now that you can move, let’s learn how to delete, copy, and paste.

- `x` → delete a character
- `dd` → delete a whole line
- `yy` → copy (yank) a line
- `p` → paste after the cursor
- `P` → paste before the cursor

🎯 **Goal for today**: Practice deleting, copying, and pasting lines without reaching for the mouse.

## Day 4: Undo, Redo, and Searching

We all make mistakes. Here’s how to fix them:

- `u` → undo
- `Ctrl-r` → redo
- `/word` → search for "word" in the file
- `n` → go to the next search result
- `N` → go to the previous search result

🎯 **Goal for today**: Make a mess, then use `u` and `Ctrl-r` to fix it. Search for words in your text.

## Day 5: Opening, Saving, and Switching Files

Vim isn’t just a text editor—it’s a whole environment. Let’s get better at handling files.

- `:e filename` → open a file
- `:w` → save the current file
- `:q` → quit Vim
- `:wq` → save and quit
- `:q!` → quit without saving

Working with multiple files? Try:

- `:split filename` → open a file in a horizontal split
- `:vsplit filename` → open a file in a vertical split
- `Ctrl-w w` → switch between splits

🎯 **Goal for today**: Open, save, and quit files without panicking.

## Day 6: Making Vim Feel Like Home

Vim is powerful, but out of the box, it’s pretty minimal. Let’s add some quality-of-life improvements.

Edit your Vim config:

```sh
vim ~/.vimrc  # For Vim
vim ~/.config/nvim/init.vim  # For Neovim
```

Here are some simple settings to make life easier:

```vim
set number          " Show line numbers
set relativenumber  " Show relative line numbers
set mouse=a         " Enable mouse support
set clipboard=unnamedplus  " Use system clipboard
```

🎯 **Goal for today**: Add some settings to your `.vimrc` or `init.vim`, or if you're feeling really excited, `init.lua`.

## Day 7: Surviving and Thriving

If you’ve made it this far, congrats! You’ve survived the toughest part. Now, here’s what’s next:

1. **Learn more motions:** `gg` (go to top), `G` (go to bottom), `{` and `}` (jump paragraphs).
2. **Get used to `.`** – This repeats the last command. A real time-saver!
3. **Try out plugins** – Look into a plugin manager like `vim-plug` to enhance Vim.
4. **Practice, practice, practice** – The more you use Vim, the more natural it becomes.

🎯 **Goal for today**: Review what you’ve learned and start making Vim your own.

## Final Thoughts

Vim has a steep learning curve, but it rewards you with speed and efficiency. The key is to take it slow and resist the urge to go back to your old editor. Once you get past the first week, you’ll start to see why people never go back.

So keep going, and happy Vimming! 🚀
