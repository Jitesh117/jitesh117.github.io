+++
title = "How I Built the BrainrotLang Interpreter in Golang"
date = 2025-06-22T19:14:10+05:30
draft = false
showToc = true
cover.image = '/images/brainrotLangInterpreter.jpg'
tags = ['tech', 'golang', 'brainrot']
+++

Last year, when learning about how the LSPs (Language Server Protocol) for different languages work internally, built this [Brainrot LSP](https://github.com/Jitesh117/brainrot-lsp), which was a Language Server for brainrot slangs which can be integrated to your neovim config.

And for some reason, now I want to expand on that idea. So I decided to build a full-fledged Brainrot Programming Language Suite!

In this article, I would walk y'all through the interpreter for The BrainrotLang. This article is the first one in the BrainrotLang Series, and it's going to be crazzyyyy!

{{< callout  emoji="🌐" >}}
You can find the source code of this project in this [**github repository**](https://github.com/Jitesh117/brainrotLang-interpreter)
{{< /callout >}}

So let me make things clear first, I didn't come up with all of the concepts used in this interpreter all by myself. I took help from the very famous book of Thorsten Ball called [Writing an Interpreter in Go](https://interpreterbook.com/).

Okay so the first thing you might want to think about before starting with writing the interpreter is, of course, the syntax, the keywords and stuff.

And after thinking for a lot of time (I'm capping, it was all natural ngl) these are the keywords I could come up with:

```go
var keywords = map[string]TokenType{
	"vibe":  FUNCTION,
	"yeet":  LET,
	"based": TRUE,
	"cap":   FALSE,
	"fr":    IF,
	"sus":   ELSE,
	"slay":  RETURN,
}
```

- `vibe` declares a function.
- `yeet` declares a variable.
- `based` and `cap` are booleans
- `fr` and `sus` are classic booleans
- `slay` is for returning stuff

## The interpreter anatomy

I followed a pretty standard interpreter structure:

1. **Lexer**: turns source code to tokens, which would be consumed by the Parser
2. **Parser**: turns tokens into AST (Abstract Syntax Tree).
3. **Evaluator**: walks the **AST** and executes stuff.

### Lexer/Tokenizer

The lexer reads the raw source code and spits out tokens. Here's how I handled single-character tokens like `=`, `+`, etc.:

```go
switch l.ch {
case '=':
	// Handle '==' for equality comparison
	if l.peekChar() == '=' {
		ch := l.ch
		l.readChar()
		literal := string(ch) + string(l.ch)
		tok = token.Token{Type: token.EQ, Literal: literal}
	} else {
		// Handle '=' for assignment
		tok = newToken(token.ASSIGN, l.ch)
	}
case '+':
	tok = newToken(token.PLUS, l.ch)
case '-':
	tok = newToken(token.MINUS, l.ch)
case '!':
    // .. and so on
    }
```

And identifiers are checked against our keywords map:

```go
// Handle identifiers and numeric literals
if isLetter(l.ch) {
	tok.Literal = l.readIdentifier()
	tok.Type = token.LookupIdent(tok.Literal)
	return tok
} else if isDigit(l.ch) {
	tok.Type = token.INT
	tok.Literal = l.readNumber()
	return tok
} else {
	// Handle unknown or illegal characters
	tok = newToken(token.ILLEGAL, l.ch)
}
```

So, `yeet x = 5;` gets tokenized into something like this:

```go
LET IDENT("x") ASSIGN INT(5) SEMICOLON
```

Chaotic, but so beautiful!

### Parsing the AST

Once we have tokens, we turn them into an Abstract Syntax Tree (AST). This part is where you start seeing the "language" to shape.

Here's a snippet of how I parse `yeet` statements (that is, variable declarations):

```go
func (p *Parser) parseYeetStatement() *ast.YeetStatement {
	stmt := &ast.YeetStatement{Token: p.curToken}
	if !p.expectPeek(token.IDENT) {
		return nil
	}
	stmt.Name = &ast.Identifier{Token: p.curToken, Value: p.curToken.Literal}
	if !p.expectPeek(token.ASSIGN) {
		return nil
	}

	p.nextToken()

	stmt.Value = p.parseExpression(LOWEST)
	for !p.curTokenIs(token.SEMICOLON) {
		p.nextToken()
	}
	return stmt
}
```

So yeah, when the parser sees `yeet x = 5;`, it builds a YeetStatement node with x as the name and 5 as the value.

Everything builds from here.

### Evaluation Time

This is where we bring it all to life. The evaluator walks the AST and executes it. For example, evaluating a `yeet` statement is just storing the variable in the environment:

```go
func Eval(node ast.Node, env *object.Environment) object.Object {
	switch node := node.(type) {
	case *ast.YeetStatement:
		val := Eval(node.Value, env)
		if isError(val) {
			return val
		}
		env.Set(node.Name.Value, val)
    // and other types of node.(type)
	}

	return nil
}
```

Same goes for expressions like `x + 10`, function calls, `fr`/`sus`, `slay`, etc.

Boolean literals like `based` and `cap` are handled like so:

```go
func nativeBoolToBooleanObject(input bool) *object.Boolean {
	if input {
		return BASED
	}
	return CAP
}
```

So an expression like:

```javascript
yeet a = based;
fr (a) {
	slay 1;
} sus {
	slay 0;
}
```

will _actually run_ and slay(return) 1.

## Arrays and Hash Tables in BrainrotLang!

Okay so we've got the basics working - variables, functions, conditionals. Now you would be thinking, okay so what? that's it??? This isn't a serious programming language.

And you would be totally right! That's why I added arrays and hash tables!!!

### Arrays

Arrays are, as the name suggests, literally arrays - ordered, indexable, growing-ish structures. You define them like this:

```javascript
yeet nums = [1, 2, 3, 4, 5];
slay nums[2]; // returns 3
```

The above return `3`, because we're indexing from 0.

Under the hood, the parser treats this like an `ArrayLiteral`, and the evaluator just… evaluates each element, also handling index very elegantly:

```go
func evalArrayIndexExpression(array, index object.Object) object.Object {
	arrayObject := array.(*object.Array)
	idx := index.(*object.Integer).Value
	max := int64(len(arrayObject.Elements) - 1)
	if idx < 0 || idx > max {
		return NULL
	}
	return arrayObject.Elements[idx]
}
```

So yeah, arrays just work!

### Hash Tables (aka Maps/Dicts)

Now _this_ was spicy! BrainrotLang supports hash literal like this:

```cpp
yeet person = {
	"name": "Brainrot",
	"rizzLevel": 100,
};

slay person["rizzLevel"];
```

Returns `100`. Easy.

The interpreter uses a `Hash` object under the hood. Each key must be a hashtable type (like `string`, `int`, `bool`). The literal itself is parsed like this:

```go
func (p *Parser) parseHashLiteral() ast.Expression {
	hash := &ast.HashLiteral{Token: p.curToken}
	hash.Pairs = make(map[ast.Expression]ast.Expression)

	for !p.peekTokenIs(token.RBRACE) {
		p.nextToken()
		key := p.parseExpression(LOWEST)

		if !p.expectPeek(token.COLON) {
			return nil
		}
		p.nextToken()
		value := p.parseExpression(LOWEST)
		hash.Pairs[key] = value
		if !p.peekTokenIs(token.RBRACE) && !p.expectPeek(token.COMMA) {
			return nil
		}
	}
	if !p.expectPeek(token.RBRACE) {
		return nil
	}
	return hash
}
```

And evaluated like this:

```go
func evalHashLiteral(
	node *ast.HashLiteral,
	env *object.Environment,
) object.Object {
	pairs := make(map[object.HashKey]object.HashPair)
	for keyNode, valueNode := range node.Pairs {
		key := Eval(keyNode, env)
		if isError(key) {
			return key
		}
		hashKey, ok := key.(object.Hashable)
		if !ok {
			return newError("unusable as hash key: %s", key.Type())
		}
		value := Eval(valueNode, env)
		if isError(value) {
			return value
		}
		hashed := hashKey.HashKey()
		pairs[hashed] = object.HashPair{Key: key, Value: value}
	}
	return &object.Hash{Pairs: pairs}
}
```

Boom! Basic key-value store ready to rock. Also dynamic keys are supported! How cool is that?!

Next up I might add support for array methods like `.push()` or iteration. But for now, it's all working and stable.

## Demo time!

![demo](/images/interpreterDemo.gif)

{{< callout  emoji="🌐" >}}
You can find the source code of this project in this [**github repository**](https://github.com/Jitesh117/brainrotLang-interpreter)
{{< /callout >}}

## Conclusion

So that's all folks. We now have a working, chaotic brainrot programming language. Next up in this series, I'll write about how the compiler and the LSP of this language works!

Stay unbothered, moisturized, happy, in your lane, focused and keep flourishing!
