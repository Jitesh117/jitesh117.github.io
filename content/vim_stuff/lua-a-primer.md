+++
title = "Lua: a Primer for Neovim"
date = 2024-10-30T17:19:38+05:30
draft = false
showToc = true
cover.image = "/images/lua_primer.jpg"
tags = ["vim","linux","neovim"]
+++

One of the most annoying things new neovim users do is skip learning lua and directly jump to "writing a config", eventually just yanking and pasting a youtube tutorial's github repository. I know this because I've been there too.

In this article I've emphasised more on lua code and have added useful comments in the code blocks itself for easy understanding instead of describing them in paragraphs.

## Why Learn Lua?

Before diving into configurations and plugin development, understanding Lua gives you several advantages:

### 1. **Control over your environment**:

- Understand what each line in your config actually does
- Debug issues effectively
- Make informed decisions about performance tradeoffs
- Customize behavior exactly to your needs

### 2. **Better debugging**:

- Read and understand error messages
- Use built-in debugging tools effectively
- Track down issues in your config systematically
- Fix problems without relying on community support

### 3. **Custom solutions**:

- Write your own plugins
- Modify existing plugins to suit your needs
- Create specific functionality for your workflow
- Contribute back to the Neovim community

### 4. **Performance optimization**:

- Write efficient configurations
- Understand and optimize startup time
- Implement lazy loading correctly
- Profile and improve slow operations

## Understanding the Neovim Lua Environment

Before diving into Lua syntax, it's important to understand how Lua integrates with Neovim:

### The Runtime Path

```lua
-- Understanding where Neovim looks for Lua files
print(vim.inspect(vim.api.nvim_list_runtime_paths()))

-- Common locations:
-- ~/.config/nvim/lua/       -- Your Lua modules
-- ~/.config/nvim/plugin/    -- Auto-loaded Lua/VimL files
-- ~/.config/nvim/after/     -- Files loaded after plugins
```

### Lua Module Loading

```lua
-- Different ways to require modules
local mymodule = require('mymodule')          -- Looks in lua/mymodule.lua
local config = require('config.options')      -- Looks in lua/config/options.lua

-- Protected requires (recommended)
local status_ok, module = pcall(require, 'module_name')
if not status_ok then
    vim.notify('Failed to load module: ' .. module, vim.log.levels.ERROR)
    return
end

-- Understanding package.loaded
local function reload_module(name)
    package.loaded[name] = nil
    return require(name)
end
```

## Basic Syntax Deep Dive

### Variables and Scope

```lua
-- Local variables - ALWAYS prefer these
-- They're faster and prevent naming conflicts
local my_var = "hello"

-- Global variables - avoid unless absolutely necessary
-- These pollute the global namespace
my_global = "world"

-- Explicit global variables - use when you really need them
-- At least this makes it clear you meant to make it global
_G.my_explicit_global = "explicit global"

-- Scope demonstration
do
    -- x only exists within this block
    local x = 10
    print(x)           -- This works
end
-- This will error because x doesn't exist here
print(x)

-- Closure example - demonstrates variable lifetime
local function create_counter()
    -- count persists between function calls
    local count = 0
    -- Returns a function that "closes over" count
    return function()
        count = count + 1
        return count
    end
end

-- Create a new counter
local counter = create_counter()
print(counter()) -- Prints: 1 (count is remembered)
print(counter()) -- Prints: 2 (count is still remembered)
```

### Tables in Depth

Tables are Lua's most powerful data structure. After reading more lua code you'll notice that almost everything is a table in lua. Just like most things in Go are structs. Understanding them is crucial for Neovim configuration:

```lua
-- Tables can be used as dictionaries (key-value pairs)
local simple_table = {
    key = "value",              -- Standard key = value syntax
    [1] = "numeric key",        -- Explicit numeric index
    ["complex key"] = "value"   -- String key with spaces needs brackets
}

-- Tables as arrays (sequential numeric indices)
local array = {1, 2, 3, 4, 5}
print(#array)  -- Length operator (#) works on sequential tables

-- Mixed-use tables (both array and dictionary features)
local mixed = {
    "first",                -- Gets index 1 automatically
    "second",               -- Gets index 2 automatically
    key = "value",          -- Named key
    nested = {              -- Tables can contain other tables
        deep = "value"
    }
}

-- Common table operations
table.insert(array, "new value")  -- Add to end of table
table.remove(array, 1)            -- Remove first element
table.concat(array, ", ")         -- Join elements with delimiter

-- Neovim-specific table operations
-- Deep merge two tables (useful for config options)
local opts = vim.tbl_deep_extend("force",
    { default = "value" },    -- Base table
    { override = "new" }      -- Overrides take precedence
)

-- Two main ways to iterate over tables:
-- 1. pairs() - for any key-value pairs (unordered)
for key, value in pairs(mixed) do
    print(key, value)
end

-- 2. ipairs() - for numeric indices only (ordered)
for index, value in ipairs(array) do
    print(index, value)
end
```

### Functions and Closures

```lua
-- Multiple return values
-- Multiple return values - common in Neovim API
local function get_buffer_info()
    -- Get information about current buffer
    local bufnr = vim.api.nvim_get_current_buf()
    local filetype = vim.bo.filetype
    local name = vim.api.nvim_buf_get_name(bufnr)
    -- Lua can return multiple values
    return bufnr, filetype, name
end

-- Capturing multiple return values
local bufnr, ft, name = get_buffer_info()

-- Variadic functions (variable number of arguments)
local function sum(...)
    -- Pack arguments into a table using ...
    local args = {...}
    local total = 0
    -- Loop through all arguments
    for _, value in ipairs(args) do
        total = total + value
    end
    return total
end

-- Function that returns multiple functions
-- Useful for creating related functionality
local function create_counter_suite()
    local count = 0
    -- Return table of related functions
    return {
        increment = function()
            count = count + 1
            return count
        end,
        decrement = function()
            count = count - 1
            return count
        end,
        reset = function()
            count = 0
            return count
        end
    }
end
```

## Advanced Configuration Patterns

### Modular Configuration Structure

```lua
-- Different ways to set Vim options
local opt = vim.opt  -- For convenience

-- Boolean options
opt.number = true         -- Shows line numbers
opt.relativenumber = true -- Shows relative line numbers

-- String options
opt.clipboard = "unnamed,unnamedplus"  -- Use system clipboard

-- Number options
opt.tabstop = 4          -- Tab width
opt.shiftwidth = 4       -- Indentation width

-- List options (options that can have multiple values)
opt.fillchars = {
    vert = "│",          -- Vertical split character
    fold = "·",          -- Folding character
    diff = "─",          -- Diff mode character
}

-- Appending to list-style options
opt.shortmess:append("c")  -- Add 'c' to shortmess option

-- Buffer-local options
vim.bo.expandtab = true    -- Use spaces instead of tabs

-- Window-local options
vim.wo.wrap = false        -- Don't wrap lines
```

### Keymaps

```lua
-- Basic keymap structure
-- mode, keys, action, options
vim.keymap.set('n', '<leader>w',
    '<cmd>write<CR>',
    {desc = 'Save buffer'}
)

-- Mapping with a Lua function
-- Mapping with a Lua function
vim.keymap.set('n', '<leader>f', function()
    -- Check if any LSP clients are attached to current buffer
    local clients = vim.lsp.get_active_clients({ bufnr = 0 })
    if #clients > 0 then
        vim.lsp.buf.format()
    else
        -- Fallback to basic formatting
        vim.cmd('normal! gg=G')
        vim.notify('No LSP clients attached, using basic formatting', vim.log.levels.WARN)
    end
end, {desc = 'Format buffer'})

-- Buffer-local mapping
vim.keymap.set('n', 'K', vim.lsp.buf.hover, {
    buffer = true,  -- Only in current buffer
    desc = 'Show hover documentation'
})

-- Complex mapping example
vim.keymap.set('n', '<leader>s', function()
    -- Get current word under cursor
    local word = vim.fn.expand('<cword>')
    -- Replace in whole file with confirmation
    vim.cmd('%s/'..word..'//gc')
end, {desc = 'Replace word under cursor'})
```

## Error handling and debugging

```lua
-- Different notification levels
local notify = vim.notify  -- For convenience

-- Basic usage
notify("Basic message")  -- Uses INFO level by default
notify("Warning message", vim.log.levels.WARN)
notify("Error message", vim.log.levels.ERROR)

-- Notification levels explained:
-- vim.log.levels.DEBUG  - For debug information
-- vim.log.levels.INFO   - For general information
-- vim.log.levels.WARN   - For warning messages
-- vim.log.levels.ERROR  - For error messages

-- Enhanced error handling pattern
local function safe_require(module_name)
    local status_ok, module = pcall(require, module_name)
    if not status_ok then
        notify(
            string.format("Error loading module '%s': %s", module_name, module),
            vim.log.levels.ERROR
        )
        return nil
    end
    return module
end

-- Debug utilities
local function debug_variable(name, value)
    -- Pretty print tables and complex values
    notify(
        string.format("%s = %s", name, vim.inspect(value)),
        vim.log.levels.DEBUG
    )
end

-- Error handling in functions
local function example_function(arg)
    -- Input validation
    if type(arg) ~= "string" then
        error("argument must be a string")
    end

    -- Try-catch pattern
    local success, result = pcall(function()
        -- Potentially dangerous operation
        return vim.fn.system("some_command " .. arg)
    end)

    if not success then
        notify("Operation failed: " .. result, vim.log.levels.ERROR)
        return nil
    end

    return result
end

-- Debugging helper for keymaps
local function debug_mapping(mode, lhs, rhs)
    local existing = vim.fn.maparg(lhs, mode, false, true)
    debug_variable("Existing mapping", existing)
end

```

## Conclusion

Learning Lua for Neovim is a journey that pays dividends in the long run. Rather than seeing it as an obstacle, think of it as an investment in your development environment. Happy Vimming!
