+++
title = "Master Command Line Mode in Vim"
date = 2024-10-02T17:23:20+05:30
draft = true
cover.image = '/images/vim_command.jpg'
tags = ["vim", "linux"]
showToc = true
+++

Vim has always been my go-to text editor because of how flexible and powerful it is, especially when it comes to speeding up my workflow. Out of all its modes, I’ve found command-line mode to be a real game-changer. It’s where you can execute more complex tasks, access a whole range of hidden features, and even tweak your editing setup to make it just right for you. In this article, I’ll walk you through what makes command-line mode so special and how you can make the most of it.

## What is Command-Line Mode?

Command-line mode in Vim is a special mode activated by typing `:` (colon) when in Normal mode. It allows you to execute a wide range of commands, from basic file operations to complex text manipulations and Vim configurations.

## Accessing Command-Line Mode

To enter command-line mode:

1. Ensure you're in Normal mode (press `Esc` if unsure).
2. Type `:` (colon).
3. The cursor will move to the bottom of the screen, where you can type your command.

## Basic Commands

Here are some fundamental commands you can execute in command-line mode:

- `:w` - Write (save) the current file
- `:q` - Quit the current window (fails if there are unsaved changes)
- `:wq` or `:x` - Write and quit
- `:q!` - Quit without saving (force quit)
- `:e filename` - Open a file for editing
- `:help keyword` - Open help for a specific keyword

## Text Manipulation Commands

Command-line mode offers powerful text manipulation capabilities:

- `:s/old/new` - Replace first occurrence of 'old' with 'new' on the current line
- `:%s/old/new/g` - Replace all occurrences of 'old' with 'new' throughout the file
- `:5,10s/old/new/g` - Replace all occurrences of 'old' with 'new' between lines 5 and 10

## File and Buffer Management

Manage multiple files and buffers efficiently:

- `:bd` - Close the current buffer
- `:bn` - Go to the next buffer
- `:bp` - Go to the previous buffer
- `:ls` - List all open buffers

## Window Management

Control Vim's windowing system:

- `:sp filename` - Open a file in a new horizontal split window
- `:vsp filename` - Open a file in a new vertical split window
- `:resize 10` - Resize the current window to 10 lines height
- `:vertical resize 80` - Resize the current window to 80 characters width

## Vim Settings

Vim's command-line mode allows you to customize your editing environment on-the-fly. Here's an expanded list of useful settings and commands:

### Display Settings

- `:set number` / `:set nu` - Show line numbers
- `:set nonumber` / `:set nonu` - Hide line numbers
- `:set relativenumber` / `:set rnu` - Show relative line numbers
- `:set cursorline` / `:set cul` - Highlight the current line
- `:set cursorcolumn` / `:set cuc` - Highlight the current column
- `:set list` - Show hidden characters (tabs, trailing spaces, etc.)
- `:set nolist` - Hide hidden characters
- `:set wrap` - Enable line wrapping
- `:set nowrap` - Disable line wrapping

### Indentation and Formatting

- `:set tabstop=4` / `:set ts=4` - Set tab width to 4 spaces
- `:set shiftwidth=4` / `:set sw=4` - Set indent width to 4 spaces
- `:set expandtab` / `:set et` - Use spaces instead of tabs
- `:set noexpandtab` / `:set noet` - Use tabs instead of spaces
- `:set autoindent` / `:set ai` - Enable auto-indentation
- `:set smartindent` / `:set si` - Enable smart indentation

### Search Settings

- `:set ignorecase` / `:set ic` - Make searches case-insensitive
- `:set smartcase` / `:set scs` - Case-sensitive if search contains uppercase
- `:set hlsearch` / `:set hls` - Highlight all search matches
- `:set incsearch` / `:set is` - Show matches as you type

### Editor Behavior

- `:set paste` - Enable paste mode (for pasting text from outside Vim)
- `:set nopaste` - Disable paste mode
- `:set mouse=a` - Enable mouse support in all modes
- `:set scrolloff=5` / `:set so=5` - Keep 5 lines above/below cursor
- `:set backspace=indent,eol,start` - Make backspace behave as expected

### File Type Specific Settings

- `:set filetype=python` - Set file type to Python
- `:set syntax=javascript` - Set syntax highlighting for JavaScript

### Viewing and Saving Settings

- `:set` - View changed settings
- `:set all` - View all settings
- `:set {option}?` - View the current value of a specific option
- `:setlocal` - Set an option for the current buffer only

### Persistent Settings

To make settings persistent across Vim sessions, add them to your `~/.vimrc` file:

```vim
:edit ~/.vimrc  " Open vimrc file
:write          " Save changes
:source %       " Reload vimrc without restarting Vim
```

Remember, you can combine multiple settings in a single command:

```vim
:set number relativenumber cursorline expandtab tabstop=2 shiftwidth=2
```

These commands offer a glimpse into Vim's extensive customization options. I'll write a detailed article on vim settings in a later post. So stay tuned!

## Advanced Features

Explore some of Vim's more advanced capabilities:

- `:make` - Run the make command from within Vim
- `:!command` - Execute an external shell command
- `:r !command` - Insert the output of a shell command into the current buffer
- `:earlier 15m` - Revert the document to how it was 15 minutes ago

## Command-Line Mode History

Vim keeps a history of commands you've executed:

- Press `:` and then use the up/down arrow keys to cycle through previous commands
- Type part of a command and use the up/down arrows to search through commands that start with what you've typed

## Command-Line Window

For more complex command editing:

1. Press `q:` in Normal mode to open the command-line window
2. Edit your command as you would normal text
3. Press `Enter` to execute the command on the current line

## Tips for Mastering Command-Line Mode

1. Use tab completion to auto-complete commands, filenames, and options
2. Leverage the power of wildcards in file operations (e.g., `:w *.txt`)
3. Create custom commands with `:command`
4. Use `:set` to view current settings and `:set all` to see all options
5. Combine commands with `|` (pipe) for more complex operations

But the beauty of vim lies in the fact that you can do everything the command mode offers by creating custom keymappings/keybindings. I'll cover this in a later blog post.

## Conclusion

Vim's command-line mode is a versatile and powerful tool that significantly enhances editing efficiency. By mastering this mode, you can perform complex operations quickly, customize your editing environment on-the-fly, and tap into Vim's extensive feature set. Regular practice and exploration of Vim's vast command set will help you become a more proficient and productive Vim user.

Remember, the key to mastering Vim's command-line mode is consistent practice and gradual exploration of new commands and features. Happy Vimming!
