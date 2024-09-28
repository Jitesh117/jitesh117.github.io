+++
title = "How I Use fzf to Supercharge My Workflow"
date = 2024-09-28T18:19:24+05:30
draft = false
cover.image = '/images/fzf_cover.jpg'
tags = ["linux", "Tech productivity"]
showToc = true
+++

I spend most of my time (almost 99%) on my terminal. So obviously whatever I'm doing on it should be as fast as possible, requiring me to type as few keystrokes as possible. Although [tmux](https://jitesh117.github.io/blog/why-to-use-tmux/) solves a good chunk of this problem, sometimes you've to navigate between different types of lists a lot within the same terminal window. And that's where `fzf` shines.

`fzf` is a command line utility which makes searching files a breeze. Although it has many great flags in by itself, it really shines when combined with other unix commands, utilising its power to the fullest.

In this article, I'll be going through some of my most used `fzf` bash functions which take my developer experience to the next level. You can use these bash functions in your `.zshrc` or `.bashrc` for global usage.

**NOTE**: I've used the nvim command in many of these bash functions, you can replace it with your favorite code editor of choice. For VSCode users, you can just replace the `nvim` command with `code` command.

## Quick Navigation with `fcd`

One of the most common tasks in the terminal is navigating between directories. With `fcd`, you can quickly find and change to any directory:

```bash
fcd() {
  local dir
  dir=$(find ${1:-.} -type d 2> /dev/null | fzf --preview 'tree -C {} | head -n 100' +m) && cd "$dir"
}
```

Just type `fcd` and you'll be presented with a list of directories. Start typing to filter, and press Enter to change to the selected directory.

### Explanation:

- `find ${1:-.} -type d`: Searches for directories, starting from the current directory (or a specified directory)
- `fzf --preview 'tree -C {} | head -n 100'`: Uses fzf to select a directory, with a preview showing the directory structure
- `cd "$dir"`: Changes to the selected directory

![fcd](/images/fcd.png)

## Fuzzy File Editing with `fzp`

Open files for editing becomes a breeze with this funciton. It basically fuzzy finds the file you're looking for and shows a nice looking preview pane for your convenience.

```sh
fzp() {
  local file
  file=$(fzf --preview 'batcat --style=numbers --color=always --theme=TwoDark {} || cat {}' \
             --height=100% --layout=reverse --border \
             --preview-window=right:60%:wrap)
  if [[ -n "$file" ]]; then
    nvim "$file"
  fi
}
```

### Explanation:

- `fzf --preview '...'`: Uses fzf to select a file, with a preview using `batcat` (or `cat` as fallback)
- `nvim "$file"`: Opens the selected file in Neovim

![fzp](/images/fzp.png)

## Command History Search with `fzh`

Remembering complex commands can be challenging. With `fzh`, you can easily search and re-run your command history:

```bash
fzh() {
  local selected_command
  selected_command=$(history | fzf --tac --height 40% --reverse --query="$LBUFFER" | sed 's/^[ ]*[0-9]\+[ ]*//')
  if [ -n "$selected_command" ]; then
    eval "$selected_command"
  fi
}
```

### Explanation:

- `history`: Lists command history
- `fzf --tac --height 40% --reverse --query="$LBUFFER"`: Uses fzf to select a command, with reverse order and initial query
- `sed 's/^[ ]*[0-9]\+[ ]*//'`: Removes the command number from the history output
- `eval "$selected_command"`: Executes the selected command

![fzh](/images/fzh.png)

Just type `fzh` and start searching your command history.

## Fuzzy Copy with `fzcp`

The `fzcp` function allows you to interactively select a source file and a destination directory for copying:

```bash
fzcp() {
  local src_file dest_dir
  src_file=$(find . -type f | fzf --preview 'batcat --style=numbers --color=always --theme=TwoDark --line-range :500 {}' --height=40%)
  [[ -n "$src_file" ]] && dest_dir=$(find . -type d | fzf --preview 'tree -C {} | head -100' --height=40%) && \
  cp "$src_file" "$dest_dir"
}
```

### Explanation:

- `find . -type f`: Lists all files in the current directory and subdirectories
- First `fzf` command: Allows selection of the source file, with a preview using `batcat`
- Second `fzf` command: Allows selection of the destination directory, with a preview using `tree`
- `cp "$src_file" "$dest_dir"`: Copies the selected file to the selected directory

Usage: Simply type `fzcp` in your terminal. You'll first be prompted to select a file to copy, then a directory to copy it to.

I didn't include the screenshot as it's very similar to the `fcd` function.

## Fuzzy Move with `fzmv`

The `fzmv` function is similar to `fzcp`, but it moves the file instead of copying it:

```bash
fzmv() {
  local src_file dest_dir
  src_file=$(find . -type f | fzf --preview 'batcat --style=numbers --color=always --theme=TwoDark --line-range :500 {}' --height=40%)
  [[ -n "$src_file" ]] && dest_dir=$(find . -type d | fzf --preview 'tree -C {} | head -100' --height=40%) && \
  mv "$src_file" "$dest_dir"
}
```

### Explanation:

- The function structure is identical to `fzcp`
- The only difference is the final command: `mv "$src_file" "$dest_dir"`, which moves the file instead of copying it

Usage: Type `fzmv` in your terminal. You'll be prompted to select a file to move, then a directory to move it to.

I didn't include the screenshot as it's very similar to the `fcd` function.

## Process Management with `fzkill`

Need to kill a process but can't remember its PID? `fzkill` to the rescue:

```bash
fzkill() {
  local pid
  pid=$(ps -ef | fzf --preview 'echo {}' | awk '{print $2}') && kill -9 "$pid"
}
```

### Explanation:

- `ps -ef`: Lists all processes
- `fzf --preview 'echo {}'`: Uses fzf to select a process, with a preview of the process details
- `awk '{print $2}'`: Extracts the PID from the selected process
- `kill -9 "$pid"`: Forcefully terminates the selected process

![fzkill](/images/fzkill.png)

Run `fzkill`, select the process, and it's gone!

## Git Branch Management with `fzswitch`

Switching between git branches becomes a visual treat with `fzswitch`:

```bash
fzswitch() {
  local branch
  branch=$(git branch --all | grep -v '\*' | sed 's/remotes\/origin\///' | sort -u |
    fzf --preview 'git log --oneline --color=always $(echo {})')
  if [[ -n "$branch" ]]; then
    git checkout "$branch"
  fi
}
```

### Explanation:

- `git branch --all`: Lists all branches (local and remote)
- `grep -v '\*'`: Excludes the current branch
- `sed 's/remotes\/origin\///'`: Removes the 'remotes/origin/' prefix from remote branches
- `sort -u`: Sorts and removes duplicates
- `fzf --preview 'git log ...'`: Uses fzf to select a branch, with a preview of the branch's commit history
- `git checkout "$branch"`: Checks out the selected branch

![fzswitch](/images/fzswitch.png)

This function shows all branches (local and remote) and even provides a preview of the branch's commit history.

## Smart File Removal with `fzrm` and `fzrmrf`

Deleting files and directories becomes more interactive and safer with these functions:

```bash
fzrm() {
  local file
  file=$(find . -type f | fzf --preview 'batcat --style=numbers --color=always --theme=TwoDark --line-range :500 {}') && rm "$file"
}

fzrmrf() {
  local dir
  dir=$(find ${1:-.} -type d 2> /dev/null | fzf --preview 'tree -C {} | head -100' \
       --height=100% --layout=reverse --border --preview-window=right:60%:wrap)

  if [[ -n "$dir" ]]; then
    echo "Are you sure you want to delete the directory '$dir' and its contents? (y/n)"
    read -r confirm
    if [[ "$confirm" == "y" ]]; then
      rm -rf "$dir"
      echo "'$dir' has been deleted."
    else
      echo "Deletion canceled."
    fi
  fi
}
```

### Explanation

- `find . -type f`: Lists all files in the current directory and subdirectories
- `fzf --preview '...'`: Uses fzf to select a file, with a preview using `batcat`
- `rm "$file"`: Removes the selected file

Explanation for `fzrmrf`:

- `find ${1:-.} -type d`: Lists all directories, starting from the current directory (or a specified directory)
- `fzf --preview '...'`: Uses fzf to select a directory, with a preview of its contents
- Confirmation prompt and `rm -rf "$dir"`: Asks for confirmation before forcefully removing the selected directory and its contents

![fzrmrf](/images/fzrmrf.png)

`fzrm` allows you to interactively select and delete individual files, while `fzrmrf` provides a safe way to delete entire directories with a confirmation step.

## Conclusion

These `fzf`-powered functions have significantly improved my terminal workflow. They allow me to navigate, edit, and manage files and processes with unprecedented speed and ease. By incorporating these into your own setup, you can supercharge your terminal experience and boost your productivity.

Remember, the beauty of these functions is that you can customize them to fit your specific needs. Feel free to modify and extend them to create your perfect terminal environment!

## References

1. [fzf - GitHub Repository](https://github.com/junegunn/fzf)
2. [Why to Use Tmux](https://jitesh117.github.io/blog/why-to-use-tmux/)
