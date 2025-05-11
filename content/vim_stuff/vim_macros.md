+++
title = 'Vim Macros'
date = 2024-08-21T22:06:42+05:30
draft = false
cover.Image = '/images/macro_cover.jpg'
showTOC = true
tags = ["vim", "linux"]
+++

**Vim macros** can seem daunting to many at first glance. However, the truth is, once you unlock their potential, you'll wonder how you ever survived without them. Imagine saving yourself the hassle of repeatedly smashing the dot (`.`) command—Vim macros make that a reality.

Vim macros take a minute to learn but a lifetime to master. Just keep using macros whenever you feel like you're repeating a lot of commands and watch your productivity skyrocket while text editing!

## Why Vim Macros?

Macros are your go-to for automating repetitive tasks. Whether it's tweaking lines, adjusting paragraphs, or even making sweeping changes across files, macros have you covered.

### Example 1: Wrapping Lines in Quotes

Picture this: you need to wrap each line of a multi-line string in quotes. Sound tedious? Not with macros.

```
qa0i"<Esc>A"<Esc>j^q
```

xkHere's what's happening:

1. **`qa`**: Start recording in register `a`.
2. **`0i"`**: Jump to the beginning of the line and insert a quote.
3. **`<Esc>A"`**: Move to the end of the line and append a quote.
4. **`j^`**: Jump to the beginning of the next line.
5. **`q`**: Stop recording.

Here’s how the buffer contents evolve during the macro:

| Keystroke | Buffer Contents           |
| --------- | ------------------------- |
| `qa`      | `one`                     |
| `0i"`     | `"one`                    |
| `<Esc>A"` | `"one"`                   |
| `j^`      | `"one"`                   |
| `q`       | `"one"` (macro ends here) |

Now, with a simple `@a`, you can apply this macro to the current line. Or use `5@a` to repeat it five times in a row. No more manual labor!

### Example 2: Renaming Variables and Navigating the Quickfix List

Macros aren't just for simple edits—they can combine editing with navigation. Suppose you need to rename a variable and then jump to the next item in the quickfix list. Here’s how:

```
qbciwNewVariableName<Esc>:cnext<CR>q
```

This macro:

1. **`qb`**: Start recording in register `b`.
2. **`ciwNewVariableName<Esc>`**: Change the word under the cursor.
3. **`:cnext<CR>`**: Move to the next item in the quickfix list.
4. **`q`**: Stop recording.

Here’s how the buffer contents evolve during the macro:

| Keystroke            | Buffer Contents                     |
| -------------------- | ----------------------------------- |
| `qb`                 | `oldVariable`                       |
| `ciwNewVariableName` | `NewVariableName`                   |
| `<Esc>:cnext<CR>`    | (Moves to next item in quickfix)    |
| `q`                  | `NewVariableName` (macro ends here) |

Now, `@b` can be your quickfire solution to renaming variables across your project.

**NOTE:** if you've LSP configured in your vim setup then this might seem redundant as you can then just use `LSP rename`, but this will come in handy when SSHing into a server and changing a text there repeatedly.

### Example 3: Converting Variable Names to Uppercase and Aligning Assignment

Let’s say you have a list of variables in lowercase that you want to convert to uppercase and align the assignment operators. This is a perfect task for a macro.

Suppose your list looks like this:

```cpp
var_one=1
var_two=2
var_three=3
```

You want to convert this to:

```cpp
VAR_ONE   = 1
VAR_TWO   = 2
VAR_THREE = 3
```

Here’s the macro to accomplish that:

```
qcgUwwf=i <Esc>j^q
```

This macro:

1. **`qc`**: Start recording in register `c`.
2. **`gUw`**: Convert the first word to uppercase.
3. **`wf=`**: Move to the equal sign.
4. **`i <Esc>`**: Insert spaces before the equal sign to align it.
5. **`j^`**: Move to the beginning of the next line.
6. **`q`**: Stop recording.

You can apply this macro to the entire list with `@c` or `5@c` to repeat it five times.

Here’s how the buffer contents evolve during the macro:

| Keystroke | Buffer Contents                 |
| --------- | ------------------------------- |
| `qc`      | `var_one=1`                     |
| `gUw`     | `VAR_ONE=1`                     |
| `wf=`     | `VAR_ONE=1`                     |
| `i <Esc>` | `VAR_ONE = 1`                   |
| `j^`      | `VAR_ONE = 1`                   |
| `q`       | `VAR_ONE = 1` (macro ends here) |

## Conclusion

The beauty of Vim macros lies in their versatility—there’s no end to the creative solutions you can develop as you master them. So keep experimenting, keep refining, and watch your efficiency soar.

Happy Vimming!
