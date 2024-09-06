+++
title = 'Things Building an Interpreter Taught Me'
date = 2024-09-06T11:38:57+05:30
draft = false
showToc = true
cover.image = '/images/interpreter.jpg'
tags = ["golang", "tech"]
+++

Interpreters in computer science are often perceived as magical black boxes – you feed them text, and out comes meaning. I used to share this view until I dug deeper into the subject and built one myself. What I discovered was a world both deceptively simple and incredibly complex, a world where random characters are transformed into executable instructions.

I referred to the excellent book called [Writing an Interpreter in Go](https://interpreterbook.com/) by Thorsten Ball. As someone who had always wanted to learn Go, this project seemed like the perfect opportunity to kill two birds with one stone – diving into the intricacies of interpreter design while getting hands-on experience with a new programming language.

In this article, I'll share my experience building an interpreter from scratch. We'll explore the key concepts I learned, the challenges I faced, and how I progressed from parsing simple expressions to executing complex programs. Whether you're a seasoned developer curious about language implementation or a beginner wondering how programming languages work under the hood, I hope this article will inspire you to look at interpreters in a new light.

You can find my implementation of the interpreter in this [github repository](https://github.com/Jitesh117/monkeylang_interpreter_go).

## What is an Interpreter?

Before we start, let's clarify what an interpreter does. An interpreter is a program that directly executes instructions written in a programming or scripting language, without previously compiling them into machine-language instructions. It reads your code, understands what it means, and then carries out those instructions.

## The Language: Monkey

For this interpreter, I worked with a language called Monkey. It's a simple yet powerful language with features like:

- C-like syntax
- Variable bindings
- Integers and booleans
- Arithmetic expressions
- Built-in functions
- First-class and higher-order functions
- Closures
- A string data structure
- An array data structure
- A hash data structure

## Step 1: Building the Lexer

### The Journey Begins: Lexical Analysis

The first step in creating our interpreter was building a lexer. A lexer, or tokenizer, is responsible for breaking down the input source code into a series of tokens. Each token represents a discrete part of the language syntax.

```go
type Token struct {
    Type    TokenType
    Literal string
}

func (l *Lexer) NextToken() Token {
    var tok Token

    l.skipWhitespace()

    switch l.ch {
    case '=':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            literal := string(ch) + string(l.ch)
            tok = Token{Type: EQ, Literal: literal}
        } else {
            tok = newToken(ASSIGN, l.ch)
        }
    case '+':
        tok = newToken(PLUS, l.ch)
    case '-':
        tok = newToken(MINUS, l.ch)
    case '!':
        if l.peekChar() == '=' {
            ch := l.ch
            l.readChar()
            literal := string(ch) + string(l.ch)
            tok = Token{Type: NOT_EQ, Literal: literal}
        } else {
            tok = newToken(BANG, l.ch)
        }
    // ... more cases for other characters
    default:
        if isLetter(l.ch) {
            tok.Literal = l.readIdentifier()
            tok.Type = LookupIdent(tok.Literal)
            return tok
        } else if isDigit(l.ch) {
            tok.Type = INT
            tok.Literal = l.readNumber()
            return tok
        } else {
            tok = newToken(ILLEGAL, l.ch)
        }
    }

    l.readChar()
    return tok
}
```

This was my first big "oh that's how it works!" moment. Seeing how a string of characters could be transformed into meaningful tokens was incredibly satisfying. It's like teaching a computer to read!

The lexer works by iterating through each character in the input string. It recognizes different types of tokens based on the characters it encounters. For example, it identifies operators like '+' and '-', special characters like '=' and '!', and sequences of characters that form identifiers or numbers.

### Challenges in Lexical Analysis

1. **Multi-character Tokens**: One interesting challenge was handling multi-character tokens like '==' (equality) and '!=' (inequality). We had to implement a 'peek' function to look at the next character without consuming it, allowing us to differentiate between '=' (assignment) and '==' (equality).

   ```go
   func (l *Lexer) peekChar() byte {
       if l.readPosition >= len(l.input) {
           return 0
       } else {
           return l.input[l.readPosition]
       }
   }
   ```

   This function allows us to look ahead without advancing the lexer's position, crucial for identifying multi-character tokens.

2. **Whitespace Handling**: Another key aspect was handling whitespace. We implemented a `skipWhitespace` function to ignore spaces, tabs, and newlines between tokens, which is crucial for making our language more readable and flexible.

   ```go
   func (l *Lexer) skipWhitespace() {
       for l.ch == ' ' || l.ch == '\t' || l.ch == '\n' || l.ch == '\r' {
           l.readChar()
       }
   }
   ```

   This function advances the lexer past any whitespace characters, ensuring that our tokens are cleanly separated.

3. **Keyword Recognition**: Distinguishing between identifiers and keywords required careful implementation. We used a `LookupIdent` function to check if an identifier is actually a keyword:

   ```go
   func LookupIdent(ident string) TokenType {
       if tok, ok := keywords[ident]; ok {
           return tok
       }
       return IDENT
   }
   ```

   This allows us to correctly tokenize keywords like `if`, `else`, `return`, etc.

4. **Error Handling**: Proper error handling in the lexer is crucial. We introduced an `ILLEGAL` token type to represent characters that aren't part of our language syntax:

   ```go
   if isLetter(l.ch) {
       tok.Literal = l.readIdentifier()
       tok.Type = LookupIdent(tok.Literal)
       return tok
   } else if isDigit(l.ch) {
       tok.Type = INT
       tok.Literal = l.readNumber()
       return tok
   } else {
       tok = newToken(ILLEGAL, l.ch)
   }
   ```

   This helps in providing meaningful error messages during the parsing phase.

## Step 2: Building the Parser

### Parsing: Making Sense of Tokens

With our lexer in place, the next step was to build a parser. The parser's job is to take the flat sequence of tokens and build them into a structured representation of the program, usually called an Abstract Syntax Tree (AST).

We implemented a recursive descent parser, which is a top-down parsing technique that constructs the AST by recursively parsing the input according to the grammar of our language.

```go
func (p *Parser) parseExpression(precedence int) ast.Expression {
    prefix := p.prefixParseFns[p.curToken.Type]
    if prefix == nil {
        p.noPrefixParseFnError(p.curToken.Type)
        return nil
    }
    leftExp := prefix()

    for !p.peekTokenIs(token.SEMICOLON) && precedence < p.peekPrecedence() {
        infix := p.infixParseFns[p.peekToken.Type]
        if infix == nil {
            return leftExp
        }

        p.nextToken()

        leftExp = infix(leftExp)
    }

    return leftExp
}
```

This part was challenging but rewarding. Understanding how to handle different types of expressions, deal with operator precedence, and build a coherent AST was a significant milestone.

### Pratt Parsing: A Game Changer

The parser uses the concept of "Pratt parsing" (also known as "top-down operator precedence parsing"). This approach associates parsing functions with token types, rather than grammar rules. It's particularly good at handling expression parsing and operator precedence.

We defined two types of parsing functions:

1. Prefix parsing functions: These handle expressions where the operator comes before the operands (like `-5` or `!true`).
2. Infix parsing functions: These handle expressions where the operator is between two operands (like `5 + 3` or `x * y`).

```go
type (
    prefixParseFn func() ast.Expression
    infixParseFn  func(ast.Expression) ast.Expression
)

func (p *Parser) registerPrefix(tokenType token.TokenType, fn prefixParseFn) {
    p.prefixParseFns[tokenType] = fn
}

func (p *Parser) registerInfix(tokenType token.TokenType, fn infixParseFn) {
    p.infixParseFns[tokenType] = fn
}
```

This approach allows for a very flexible and extensible parser. Adding new expressions or operators becomes as simple as registering new parsing functions.

### Challenges in Parsing

1. **Operator Precedence**: One of the trickier parts was implementing operator precedence. We assigned precedence levels to different operators and used these to determine how expressions should be grouped. For example, in the expression `5 + 3 * 2`, we need to ensure that the multiplication happens before the addition.

   ```go
   var precedences = map[token.TokenType]int{
       token.EQ:       EQUALS,
       token.NOT_EQ:   EQUALS,
       token.LT:       LESSGREATER,
       token.GT:       LESSGREATER,
       token.PLUS:     SUM,
       token.MINUS:    SUM,
       token.SLASH:    PRODUCT,
       token.ASTERISK: PRODUCT,
   }
   ```

   This precedence map, combined with our Pratt parsing approach, allows us to correctly handle complex expressions.

2. **Error Handling and Recovery**: Robust error handling is crucial in a parser. We implemented error reporting functions to provide meaningful feedback when parsing fails:

   ```go
   func (p *Parser) noPrefixParseFnError(t token.TokenType) {
       msg := fmt.Sprintf("no prefix parse function for %s found", t)
       p.errors = append(p.errors, msg)
   }
   ```

   This helps in debugging and provides useful information to the user about syntax errors.

3. **Parsing Complex Structures**: Another challenge was parsing more complex structures like if-else statements and function definitions. These required implementing recursive parsing strategies, where we would parse sub-expressions within larger expressions.

   ```go
   func (p *Parser) parseIfExpression() ast.Expression {
       expression := &ast.IfExpression{Token: p.curToken}

       if !p.expectPeek(token.LPAREN) {
           return nil
       }

       p.nextToken()
       expression.Condition = p.parseExpression(LOWEST)

       if !p.expectPeek(token.RPAREN) {
           return nil
       }

       if !p.expectPeek(token.LBRACE) {
           return nil
       }

       expression.Consequence = p.parseBlockStatement()

       if p.peekTokenIs(token.ELSE) {
           p.nextToken()

           if !p.expectPeek(token.LBRACE) {
               return nil
           }

           expression.Alternative = p.parseBlockStatement()
       }

       return expression
   }
   ```

   This function demonstrates the complexity involved in parsing control structures, requiring careful management of token expectations and nested expressions.

## Step 3: Building the Evaluator

### Evaluation: Bringing the AST to Life

With our AST in hand, the next step was to build an evaluator. The evaluator traverses the AST and executes the code it represents. This is where our interpreter starts to feel like a real programming language!

```go
func Eval(node ast.Node, env *object.Environment) object.Object {
    switch node := node.(type) {
    case *ast.Program:
        return evalProgram(node, env)
    case *ast.ExpressionStatement:
        return Eval(node.Expression, env)
    case *ast.IntegerLiteral:
        return &object.Integer{Value: node.Value}
    case *ast.Boolean:
        return nativeBoolToBooleanObject(node.Value)
    case *ast.PrefixExpression:
        right := Eval(node.Right, env)
        if isError(right) {
            return right
        }
        return evalPrefixExpression(node.Operator, right)
    case *ast.InfixExpression:
        left := Eval(node.Left, env)
        if isError(left) {
            return left
        }
        right := Eval(node.Right, env)
        if isError(right) {
            return right
        }
        return evalInfixExpression(node.Operator, left, right)
    // ... more cases for other node types
    }
    return nil
}
```

Implementing features like variable bindings, function calls, and closures was particularly exciting. It's amazing to see how these high-level concepts can be built from simpler pieces.

### Key Components of the Evaluator

1. **Environment**: We implemented an `Environment` type to keep track of variable bindings. This allows us to handle variable assignment and retrieval.

   ```go
   type Environment struct {
       store map[string]Object
       outer *Environment
   }

   func (e *Environment) Get(name string) (Object, bool) {
       obj, ok := e.store[name]
       if !ok && e.outer != nil {
           obj, ok = e.outer.Get(name)
       }
       return obj, ok
   }

   func (e *Environment) Set(name string, val Object) Object {
       e.store[name] = val
       return val
   }
   ```

   The `Environment` struct allows for nested scopes through the `outer` field, crucial for implementing function scopes and closures.

2. **Object System**: We created an object system to represent the different types of values in our language (integers, booleans, strings, etc.). This made it easier to implement operations on these values.

   ```go
   type Object interface {
       Type() ObjectType
       Inspect() string
   }

   type Integer struct {
       Value int64
   }

   type Boolean struct {
       Value bool
   }

   type String struct {
       Value string
   }
   ```

   This object system allows for easy extension of our language with new data types.

3. **Error Handling**: We implemented a robust error handling system, returning `Error` objects when something goes wrong during evaluation. This allows us to provide meaningful error messages to the user.

   ```go
   type Error struct {
       Message string
   }

   func newError(format string, a ...interface{}) *Error {
       return &Error{Message: fmt.Sprintf(format, a...)}
   }
   ```

   This error system allows us to propagate errors up the evaluation chain, providing context about where and why an error occurred.

4. **Function Calls**: Implementing function calls was one of the more complex parts. We had to evaluate the arguments, create a new environment for the function's scope, and then evaluate the function body in this new environment.

   ```go
   func applyFunction(fn Object, args []Object) Object {
       switch fn := fn.(type) {
       case *Function:
           extendedEnv := extendFunctionEnv(fn, args)
           evaluated := Eval(fn.Body, extendedEnv)
           return unwrapReturnValue(evaluated)
       case *Builtin:
           return fn.Fn(args...)
       default:
           return newError("not a function: %s", fn.Type())
       }
   }
   ```

   This function demonstrates the complexity of function application, including environment management and error handling.

5. **Closures**: Supporting closures required us to capture the environment in which a function was defined. This allows functions to access variables from

## Step 4: Adding New Features

### Extending the Language

Once we had a basic interpreter working, we started adding more advanced features to our Monkey language. This included:

1. **String data type**: Implementing a new object type and updating our evaluator to handle string operations.

   ```go
   type String struct {
       Value string
   }

   func (s *String) Type() ObjectType { return STRING_OBJ }
   func (s *String) Inspect() string  { return s.Value }
   ```

   We had to update our lexer to recognize string literals, our parser to parse them, and our evaluator to handle operations on strings like concatenation.

2. **Built-in functions**: Adding functions like `len()`, `first()`, `last()`, etc., to manipulate our data types.

   ```go
   var builtins = map[string]*Builtin{
       "len":   &Builtin{Fn: func(args ...Object) Object {
           if len(args) != 1 {
               return newError("wrong number of arguments. got=%d, want=1", len(args))
           }
           switch arg := args[0].(type) {
           case *Array:
               return &Integer{Value: int64(len(arg.Elements))}
           case *String:
               return &Integer{Value: int64(len(arg.Value))}
           default:
               return newError("argument to `len` not supported, got %s", args[0].Type())
           }
       }},
       // ... other built-in functions
   }
   ```

   Implementing built-in functions required us to bridge the gap between our interpreter's world and Go's world. We had to carefully handle type conversions and error cases.

3. **Arrays**: Implementing array literals, index expressions, and built-in functions for array manipulation.

   ```go
   type Array struct {
       Elements []Object
   }

   func evalArrayIndexExpression(array, index Object) Object {
       arrayObject := array.(*Array)
       idx := index.(*Integer).Value
       max := int64(len(arrayObject.Elements) - 1)

       if idx < 0 || idx > max {
           return NULL
       }

       return arrayObject.Elements[idx]
   }
   ```

   Arrays introduced the concept of compound data types to our language. We had to update our parser to handle array literals and index expressions, and our evaluator to work with arrays.

4. **Hash maps**: Adding support for hash literals and hash index expressions.

   ```go
   type Hash struct {
       Pairs map[HashKey]HashPair
   }

   type HashPair struct {
       Key   Object
       Value Object
   }

   func evalHashIndexExpression(hash, index Object) Object {
       hashObject := hash.(*Hash)

       key, ok := index.(Hashable)
       if !ok {
           return newError("unusable as hash key: %s", index.Type())
       }

       pair, ok := hashObject.Pairs[key.HashKey()]
       if !ok {
           return NULL
       }

       return pair.Value
   }
   ```

   Hash maps were the most complex data structure we implemented. We had to design a way to use objects as keys, which led us to implement a `Hashable` interface for objects that can be used as hash keys.

Each new feature brought its own challenges, but also deepened my understanding of how programming languages work. It was fascinating to see how these high-level constructs could be built from the primitive pieces we had already implemented.

## Optimizations and Performance

As the interpreter grew more complex, we started to think about performance. Here are some optimizations that are planned:

1. **Symbol Table**: Replacing our simple environment-based variable lookup with a more efficient symbol table for resolving variables.

   ```go
   type SymbolTable struct {
       store          map[string]Symbol
       numDefinitions int
   }

   func (s *SymbolTable) Define(name string) Symbol {
       symbol := Symbol{Name: name, Index: s.numDefinitions}
       s.store[name] = symbol
       s.numDefinitions++
       return symbol
   }
   ```

   This change significantly sped up variable lookups, especially in programs with many nested scopes.

2. **Compilation**: Implementing a simple bytecode compiler and virtual machine.

   ```go
   type Compiler struct {
       instructions        code.Instructions
       constants           []object.Object
       symbolTable         *SymbolTable
       scopes              []CompilationScope
       scopeIndex          int
   }

   func (c *Compiler) Compile(node ast.Node) error {
       switch node := node.(type) {
       case *ast.Program:
           for _, s := range node.Statements {
               err := c.Compile(s)
               if err != nil {
                   return err
               }
           }
       // ... cases for other node types
       }
       return nil
   }
   ```

3. **Constant Folding**: Implementing a simple form of constant folding in our compiler, evaluating constant expressions at compile-time rather than runtime.

   ```go
   func (c *Compiler) constantFolding(node ast.Node) (object.Object, bool) {
       switch node := node.(type) {
       case *ast.IntegerLiteral:
           return &object.Integer{Value: node.Value}, true
       case *ast.Boolean:
           return nativeBoolToBooleanObject(node.Value), true
       case *ast.PrefixExpression:
           right, ok := c.constantFolding(node.Right)
           if !ok {
               return nil, false
           }
           return evalPrefixExpression(node.Operator, right), true
       // ... cases for other constant expressions
       }
       return nil, false
   }
   ```

   This optimization allows us to reduce the amount of work done at runtime, especially for programs with many constant expressions.

## Reflections and Learnings

Building an interpreter from scratch was an incredible learning experience. Some key takeaways:

1. **The power of abstractions**: Seeing how complex language features can be built up from simpler components gave me a new appreciation for the power of abstraction in programming.

1. **The importance of good design**: As our interpreter grew more complex, having a well-designed structure became crucial. This experience reinforced the importance of good software design principles.

1. **Testing is crucial**: Throughout the process, having a comprehensive test suite was invaluable. It caught numerous bugs and made adding new features much less daunting.

1. **The joy of creation**: There's something magical about seeing code you've written execute other code. It's like teaching a computer to think!

## What's Next?

While this Monkey interpreter is functional, there's always room for improvement. Some ideas for future enhancements:

- Improving error reporting and recovery
- Optimizing performance
- Adding more advanced language features like modules or pattern matching
- Compiling to bytecode for faster execution

Building an interpreter has given me a deep appreciation for the work that goes into creating programming languages. It's a journey I highly recommend to any developer looking to deepen their understanding of how languages work.

If you're intrigued by this journey, I encourage you to pick up [Writing an Interpreter in Go](https://interpreterbook.com/) by Thorsten Ball. Happy coding, and may your adventures in language implementation be as rewarding as mine!
