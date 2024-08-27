+++
title = 'Search and Replace'
date = 2024-08-27T20:32:40+05:30
draft = false
cover.Image = '/images/search_replace.jpg'
showToc = true
+++

Search and replace is a powerful feature in vim that can save you a lot of time when editing text. Whether you're working on a single file or across multiple files, vim provides robust tools to find and replace text quickly. In this article, we'll explore the basics of search and replace, as well as some advanced tips to help you make the most of this feature.

## But why would one use this when LSPs can do the same thing but better?

When LSP (Language Server Protocol) is configured in vim, many tasks related to code refactoring, navigation, and error checking become more streamlined. LSP often provides more precise and context-aware tools for renaming variables, refactoring code, or finding references across a project.

However, vim's search and replace still holds significant value even with LSP:

1. **Simple Text Manipulation**: For non-code files or simple text edits (like updating documentation or configuration files), search and replace is quick and effective.

2. **Broad Replacements**: Sometimes, you may need to replace text that isn't necessarily a code symbol, such as changing a term in comments or updating a common phrase in multiple places where LSP might not be applicable.

3. **Custom Patterns**: vim's powerful regular expressions allow you to perform complex replacements that might be beyond the scope of LSP’s capabilities.

4. **Efficiency**: For quick, small-scale changes, vim’s search and replace can be faster than invoking an LSP command, especially if you're already familiar with the exact replacement needed.

5. **Outside LSP Scope**: If your project contains files or text that the LSP doesn't cover (like certain configuration files, markdown files, etc.), vim's search and replace remains invaluable.

6. **SSH and Remote Editing**: When SSHing into someone else's computer or a server, the environment may not have LSP configured or available. In such situations, vim's search and replace functionality becomes crucial for making quick edits without relying on LSP features.

## Basic Search

Before diving into search and replace, it's essential to understand how to search for text in vim. To search for a word or phrase, follow these steps:

### 1. **Search Forward**:

Press `/`, type the word you want to find, and press `Enter`. vim will highlight the first occurrence of the word in the text. Use `n` to move to the next occurrence and `N` to move to the previous one.

### 2. **Search backward**:

Press `?`, type the word, and press `Enter`. This command works like the forward search but moves in the opposite direction.

### 3. **Case sensitivity**:

By default, searches are case-sensitive. To make them case-insensitive, you can add `\c` to the search query, like `/word\c`, or you can set `ignorecase` in your `.vimrc`/`init.vim`/`init.lua` file.

```vim
:set ignorecase
```

### 4. **Exact matches**:

To search for an exact word, use `\b` before and after the word, like this: `/\bword\b`

## Basic search and Replace

vim's basic search and replacecommand is simple and effective. Here's how you can use it:

### 1. **Replace in the current line:**

```vim
:s/old/new
```

### 2. **Replace All in the current line:**

```vim
:s/old/new/g
```

Adding `g` at the end replaces all occurrences of `old` with `new` in the current line.

### 3. **Replace across the entire file:**

```vim
:%s/old/new/g
```

The `%` symbol tells vim to apply the command to al llines in the file. This command replaces all occurrences of `old` with `new` throughout the file.

### 4. **Confirm each replacement**

```vim
:%s/old/new/gc
```

Adding `c` makes vim prompt you for confirmation before each replacement. Press `y` to confirm, `n` to skip, `a` to replace all without further prompts, or `q` to quit.

## Advanced search and replace

Once you're comfortable with the basics, you can explore more advanced search and replace options:

### 1. **Using regular expressions:**

vim supports regular expressions, which allow you to search for patterns instead of just fixed text. For example:

```vim
:%s/\<old\>/new/g
```

The `\<` and `\>` symbols ensure that only whole words "old" are replaced, avoiding partial matches.

### 2. **Replacing with the content of the registers**:

You can use the contents of a register in your replacement. For example, if you'vecopied something into register `a`, you can replace "old" with the content of `a`:

```vim
:%s/old/\=@a/g
```

### 3. **Search and replace in multiple files:**

If you're working with multiple files, you can perform search and replace across all open files using the args command or by using the vim command with wildcards:

```vim
:argdo %s/old/new/gc | update

```

## Conclusion

The power of search and replace in vim is in its ability to streamline your workflow and tackle complex edits with ease. As you continue to explore and experiment with these features, you'll discover even more ways to enhance your efficiency and precision. Keep pushing the boundaries of what you can achieve, and let vim be the tool that helps you master your craft.

Happy vimming!
