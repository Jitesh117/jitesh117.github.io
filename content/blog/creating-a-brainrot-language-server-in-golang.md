+++
title = "Creating a Brainrot Language Server in Golang"
date = 2024-11-08T09:56:17+05:30
draft = false
cover.image = '/images/brainrot.jpg'
showToc = true
tags = ["Golang", "Tech"]
+++

I was reading the language server protocol docs and about json-rpc for collaborating on a new project which will need an LSP server. Since it was getting a little boring to just read so much of documentations and RFCs, I thought of making a small and fun project around LSPs.

{{< callout  emoji="🌐" >}}
You can find the source code of this project in this [**github repository**](https://github.com/Jitesh117/brainrot-lsp)
{{< /callout >}}

## What's an LSP?

The Language Server Protocol (LSP) is a game-changer in how we build development tools. Before Microsoft created LSP for VS Code, each editor (like Vim, Emacs, Sublime) needed to implement language support separately for each programming language. Microsoft's decision to make LSP open source and editor-agnostic meant that one language server could now work with any LSP-compatible editor, making it more efficient for the community and providing consistent experiences across different editors.

LSP defines a protocol for common features like:

- Code completion
- Hover information
- Go to definition
- Find all references
- Symbol search
- Formatting
- Refactoring

On a high level this is how an LSP works:

{{< mermaid >}}

sequenceDiagram
participant User
participant Editor
participant LSP
User ->> Editor: Open file / Type code
Editor ->> LSP: Send request (e.g., syntax check, completion)
LSP -->> Editor: Respond with diagnostics or completions
User ->> Editor: Hover over code
Editor ->> LSP: Request symbol info
LSP -->> Editor: Respond with symbol information (tooltip)
User ->> Editor: Request go to definition
Editor ->> LSP: Request definition location
LSP -->> Editor: Respond with definition location
Editor ->> User: Navigate to definition
User ->> Editor: Format code
Editor ->> LSP: Request code formatting
LSP -->> Editor: Return formatted code
Editor ->> User: Display formatted code

{{< /mermaid >}}

Now, you can write one language server that works with any LSP-compatible editor. The language server runs as a separate process and communicates with the editor through JSON-RPC messages.

## Why did I use Golang?

I'm going to be honest here – I didn't choose Go because of some grand technical reasons about performance or concurrency. The truth is much simpler: Go just clicked with me.

When I started learning Go in August this year, I wasn't expecting much. Microsoft created LSP, so JavaScript would have been the obvious choice, especially with the extensive TypeScript LSP examples out there. But Go has this way of growing on you.

What made me stick with Go for this project?

1. **It's Refreshingly Simple**: No class inheritance hierarchies to wrestle with, no complex type systems to navigate. Just structs, interfaces, and straightforward functions. When you're trying to understand something as complex as LSP, this simplicity is a blessing.

2. **The Standard Library is Your Friend**: Need to handle JSON? `encoding/json` is right there. Want to work with streams? `io` has your back. No need to dive into npm looking for the right package.

3. **Clear Error Handling**: Yes, some people hate the `if err != nil` pattern, but when you're debugging why your language server isn't responding, explicit error handling is actually pretty nice.

4. **Fast Iteration**: Write code, run `go build`, test. No build configs, no transpilation, no bundling. For a project where you're constantly testing with different editors, this quick feedback loop is invaluable.

Could I have built this in JavaScript/TypeScript? Absolutely. Would it have worked just as well? Probably. But sometimes you pick a tool not because it's the "best" choice, but because it makes the development process enjoyable. For me, that tool happened to be Go.

## The Brainrot Language Server

For this project, I decided to create something fun: a language server that helps you write in peak internet slang. It provides snippets and completions for creating social media style posts, rants, and reactions.

This is how it looks like:
![demo](/images/demo.gif)

### Setting up the project

First, let's create our project structure:

```bash
mkdir brainrot-lsp
cd brainrot-lsp
go mod init github.com/yourusername/brainrot-lsp
```

We'll use the [`glsp`](https://github.com/tliron/glsp) library for LSP implementation:

```bash
go get github.com/tliron/glsp
```

### Project Structure

Our project follows this structure:

```
brainrot-lsp
├── go.mod
├── go.sum
├── handlers
│   └── completionHandler.go
├── main.go
└── mappers/
    ├── brainrotMapper.go
    └── snippets.go
```

### Implementing the Core

Let's look at the main server implementation:
Intialize and shutdown are two of the mandatory LSP lifecycle methods. Fortunately, [glsp](https://github.com/tliron/glsp) has took care of the heavy lifting for us by providing an elegant `protocol.Handler` struct.

```go
package main

import (
	"github.com/Jitesh117/brainrot-lsp/handlers"
	"github.com/tliron/commonlog"
	"github.com/tliron/glsp"
	protocol "github.com/tliron/glsp/protocol_3_16"
	"github.com/tliron/glsp/server"

	_ "github.com/tliron/commonlog/simple"
)

const lsName = "Brainrot Autocomplete Language Server"

var (
	version string = "0.0.1"
	handler protocol.Handler
)

func main() {
	commonlog.Configure(2, nil)
	handler = protocol.Handler{
		Initialize:             initialize,
		Shutdown:               shutdown,
		TextDocumentCompletion: handlers.TextDocumentCompletion,
	}
	server := server.NewServer(&handler, lsName, true)
	server.RunStdio()
}

func initialize(context *glsp.Context, params *protocol.InitializeParams) (any, error) {
	commonlog.NewInfoMessage(0, "Initializing Brainrot server...")
	capabilities := handler.CreateServerCapabilities()

	trueVar := true
	capabilities.CompletionProvider = &protocol.CompletionOptions{
		ResolveProvider: &trueVar,
	}

	return protocol.InitializeResult{
		Capabilities: capabilities,
		ServerInfo: &protocol.InitializeResultServerInfo{
			Name:    lsName,
			Version: &version,
		},
	}, nil
}

func shutdown(context *glsp.Context) error {
	return nil
}
```

### The Fun Part: Autocomplete and Snippets

The heart of our LSP is the snippet system. We define templates for various social media scenarios:

```go
package mappers

type BrainrotEntry struct {
	Term        string
	Description string
}

var BrainrotMapper = map[string]BrainrotEntry{
	"yes":           {"ongod frfr 💯🔥", "An enthusiastic affirmation, absolutely true."},
	"no":            {"nah fam 🚫🙅", "Casual denial or rejection."},
    // SIMILAR BRAINROT SLANGS
}

var CodeSnippets = map[string]CodeSnippet{
    "#story": {
        Body: "no cap frfr ${1:name} was ${2:activity} and suddenly 💀 ${3:unexpected_event} happened ong\neveryone was like sheeeesh 😱",
        Description: "Generate a dramatic story template",
    },
    "#rant": {
        Body: "NAH FR THO 😤 ${1:topic} is actually WILD 💯 like how are people even ${2:action} fr⁉️",
        Description: "Create a passionate rant template",
    },
    // ... MORE SNIPPETS
}
```

### Completion Handler

The completion handler brings our snippets to life:

```go
package handlers

import (
	"fmt"

	"github.com/Jitesh117/brainrot-lsp/mappers"
	_ "github.com/tliron/commonlog/simple"
	"github.com/tliron/glsp"
	protocol "github.com/tliron/glsp/protocol_3_16"
)

func TextDocumentCompletion(
	context *glsp.Context,
	params *protocol.CompletionParams,
) (interface{}, error) {
	var completionItems []protocol.CompletionItem

	// FOR AUTOCOMPLETE BRAINROT
	for word, entry := range mappers.BrainrotMapper {
		term := entry.Term
		description := entry.Description
		detail := fmt.Sprintf("%s\n%s", term, description)
		completionItems = append(completionItems, protocol.CompletionItem{
			Label:      word,
			Detail:     &detail,
			InsertText: &term,
		})
	}
     // FOR SNIPPETS
	for label, snippet := range mappers.CodeSnippets {
		insertText := snippet.Body
		detail := snippet.Description
		kind := protocol.CompletionItemKindSnippet
		textFormat := protocol.InsertTextFormatSnippet

		completionItems = append(completionItems, protocol.CompletionItem{
			Label:            label,
			Detail:           &detail,
			InsertText:       &insertText,
			Kind:             &kind,
			InsertTextFormat: &textFormat,
		})
	}

	return completionItems, nil
}
```

Now run `go mod tidy` to install any missing packages used.

### Using the Language Server

To use the server in VSCode, create a client extension or use an existing LSP client. But if you use neovim(btw), you can just add this to your current neovim session:

```lua
vim.lsp.start({
	name = "brainrot-lsp",
	cmd = { "./bin/brainrot-lsp" },
	root_dir = vim.fn.getcwd(),
})
```

Then run the command `:source <filename>.lua`. After this the language server should be working now.

And if you want to add it to your neovim config for global usage. You can add this to your config's `init.lua` file:

```lua
local function start_brainrot_lsp()
  local filetype = vim.bo.filetype
  if filetype == "text" or filetype == "markdown" then
    vim.lsp.start {
      name = "brainrot-lsp",
      cmd = { "path-to/brainrot-lsp" },
      root_dir = vim.fn.getcwd(),
    }
  end
end

vim.api.nvim_create_autocmd("BufEnter", {
  pattern = "*",
  callback = start_brainrot_lsp,
})

```

Now you can type `story` and get a template like:

```
no cap frfr John was studying and suddenly 💀 a ghost appeared happened ong
everyone was like sheeeesh 😱
```

## Future Improvements

This is only a basic language server. We'd need to add more stuff to it to make it "production grade". Some features we could add:

1. Emoji suggestions based on context
2. Auto-capitalization for emphasis
3. Code actions

## Conclusion

Building an LSP might seem daunting, but Go and modern libraries make it approachable. This fun project shows how LSP can enhance any text editing experience, even if it's just helping you write more brainrot slangs lol.

The full code is available on [GitHub](https://github.com/Jitesh117/brainrot-lsp). Feel free to contribute and make it even more 🔥!
