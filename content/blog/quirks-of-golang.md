+++
title = 'Golang: The quirks'
date = 2024-08-15T18:19:09+05:30
draft = false
cover.Image = "/images/golang_quirks.jpg"
showToc = true
+++

Go, also known as Golang, is a popular programming language known for its simplicity and efficiency. However, like any language, it has its share of quirks that can surprise developers. This article explores some of the interesting features and potential gotchas in Go.

## 1. Unused Variables Are Errors

Let's chat about Go's compiler - it's like that friend who always points out the tiniest details in your stories. Sometimes, it feels like it's trying to win a "Most Meticulous Code Reviewer" award! Imagine you're working on a function to process user data:

```go
func processUserData(userID int) error {
    user, err := fetchUserFromDB(userID)
    if err != nil {
        return fmt.Errorf("failed to fetch user: %w", err)
    }
    data, err := parseUserData(user)
    if err != nil {
        return fmt.Errorf("failed to parse user data: %w", err)
    }
    result := analyzeData(data)
    saveAnalysisResult(result)

    return nil
}
```

Now, let's say you're debugging and want to skip the analysis part. You comment out those lines.

```go
func processUserData(userID int) error {
    user, err := fetchUserFromDB(userID)
    if err != nil {
        return fmt.Errorf("failed to fetch user: %w", err)
    }

    data, err := parseUserData(user)
    if err != nil {
        return fmt.Errorf("failed to parse user data: %w", err)
    }

    // result := analyzeData(data)
    // saveAnalysisResult(result)

    return nil
}
```

Boom! The compiler throws a fit: "data declared and not used". Alright, let's use an underscore:

```go
func processUserData(userID int) error {
    user, err := fetchUserFromDB(userID)
    if err != nil {
        return fmt.Errorf("failed to fetch user: %w", err)
    }
    _, err := parseUserData(user)
    if err != nil {
        return fmt.Errorf("failed to parse user data: %w", err)
    }
    // result := analyzeData(data)
    // saveAnalysisResult(result)
    return nil
}
```

But wait! The compiler's not done. "No new variables on left side of :=". Right, `err` is already declared. One more tweak:

```go
func processUserData(userID int) error {
    user, err := fetchUserFromDB(userID)
    if err != nil {
        return fmt.Errorf("failed to fetch user: %w", err)
    }

    _, err = parseUserData(user) // used assignment operator instead of declaration operator
    if err != nil {
        return fmt.Errorf("failed to parse user data: %w", err)
    }

    // result := analyzeData(data)
    // saveAnalysisResult(result)

    return nil
}
```

Finally, it compiles! But seriously, we had to change our code twice just to comment out two lines. It's like playing a game of "Whack-a-Mole" with the compiler.

Sure, we appreciate Go's commitment to clean code. But sometimes, when you're deep in debugging mode, you just want to make a quick change without feeling like you're negotiating with a particularly picky code robot.

Wouldn't it be nice if Go had a "debug mode" where unused variables were just friendly warnings? Imagine how much smoother our edit-compile-debug tango would be!

## 2. No Ternary Operator

Go doesn't have a ternary operator (`?:`), which can be surprising for developers coming from other languages:

```go
// Instead of: result := (condition) ? a : b
result := a
if !condition {
    result = b
}
```

It's like Go is saying, "Oh, you want a shortcut? How about a nice walk through if-else land instead?" Sure, it's more explicit, but sometimes you just want to get to the point without feeling like you're writing a small novel.

## 3. Naked Returns

Go allows "naked" returns in functions, which can improve readability in short functions but may reduce clarity in longer ones:

```go
func split(sum int) (x, y int) {
    x = sum * 10
    y = sum - x
    return // Returns x and y
}
```

At first glance, it's neat and tidy. But let's dive deeper into why naked returns might not be the best idea after all:

1. `The Return Without Arguments Dilemma`: The `return` statement lacks its expected arguments. To figure out what's being returned, you need to play detective and deduce it from the function signature and variable assignments in the code. It's like a mystery novel where you have to piece together the plot from scattered clues.
2. **`The Readability Trap`**: A Tour of Go notes that naked returns "can harm readability in longer functions." But let's be real - they can harm readability, period. Here's why:
   - **`The Signature Scavenger Hunt`**: When you see a naked `return`, you're forced to scroll up to the function signature to determine what values are supposed to be returned. It's like trying to remember the beginning of a long joke to understand the punchline.
   - **`The Backward Scan Challenge`**: You need to scan backwards through the code to find the last assignments of the variables. This can be particularly tricky in longer functions. It's like trying to remember where you parked your car in a multi-story parking lot.
   - **`The 'err' Trap`**: Developers often use `err` as a return variable name and habitually shadow this variable. Combine these practices, and finding the last actual assignment becomes a game of "Where's Waldo?" but for errors.
3. **`The Consistency Conundrum`**: If you use naked returns for short functions but explicit returns for longer ones, you're creating a Jekyll and Hyde situation in your codebase. As functions evolve and change length, you'll need to constantly reevaluate your return style choices. It's like having different rules for casual Fridays depending on the office temperature.

In the end, naked returns are like using acronyms in a presentation - they might save you a few keystrokes, but they leave everyone else scratching their heads. For the sake of your future self and your fellow developers, it's often better to be explicit about what you're returning. Your code reviewers will thank you!

## 4. Slice Behavior

Slices in Go can behave unexpectedly due to their underlying array structure:

```go
a := make([]int, 5)
b := a[1:3]
b[0] = 10 // This also modifies a[1]
```

Surprise! You thought you were just changing `b`, but a got a makeover too. It's the programming equivalent of finding out that the wall you're painting in your apartment is actually shared with your neighbor. "Oops, hope you like blue!"

## 5. Defer and Loop Variables

Using `defer` inside a loop can lead to unexpected results due to how closure captures loop variables:

```go
for i := 0; i < 5; i++ {
    defer fmt.Println(i) // Will print 4, 4, 4, 4, 4
}
```

You might expect it to print 0, 1, 2, 3, 4. But no, it's more like a time-travel movie where everyone ends up in the same timeline. It's Go's way of saying, "You thought you understood closure? That's cute."

## 6. Nil Slices vs. Empty Slices

In Go, there's a distinction between nil slices and empty slices, which can affect behavior in certain situations:

```go
var s []int // nil slice
s = []int{} // empty slice
```

It's like the difference between an empty room and a room with a sign that says "This room is empty." Functionally similar, philosophically distinct, and occasionally the source of bugs that make you question your life choices.

## 7. Date Formatting: The "What Year Is It?" Conundrum

Just when you thought you had seen all of Go's quirks, along comes date formatting to make you question your sanity. In most programming languages, you'd use something intuitive like `YYYY-MM-DD` for year-month-day. But Go? Go decided to be "special."

```go
// Instead of something logical like:
// fmt.Println(time.Now().Format("YYYY-MM-DD HH:mm:ss"))

// In Go, you write:
fmt.Println(time.Now().Format("2006-01-02 15:04:05"))
```

Yes, you read that right. The reference date in Go is `January 2, 2006 at 3:04:05 PM`. Or in Go's mind: `01/02 03:04:05PM '06 -0700`. Let's break down this madness:

- Month: 01
- Day: 02
- Hour: 03 (15 for 24-hour time)
- Minute: 04
- Second: 05
- Year: 06
- Time zone: -0700

It's like Go's creators were visited by a time-traveling numerologist with a peculiar sense of humor. This format is so ingrained in Go that it's often referred to as the "magic date" or the "reference time."

Why did they choose this? Well, it's supposed to be more intuitive because the numbers line up (01, 02, 03, 04, 05, 06). But let's be honest, the first time you encounter this, it feels about as intuitive as using a banana for a phone.

The fun doesn't stop there. Want to format just the year and month?

```go
fmt.Println(time.Now().Format("2006-01"))
```

Need day of the week?

```go
fmt.Println(time.Now().Format("Monday"))
```

It's like Go is constantly asking "What would the world be like if it were January 2, 2006?" And we're all just along for the ride.

This unique approach to date formatting is a perfect example of Go's philosophy: do things differently, make it consistent (once you know the trick), and who cares if the rest of the programming world thinks you've lost your mind?

So the next time you're formatting dates in Go and find yourself writing "2006," just remember: in Go's world, it's always 2006 somewhere. It's not a bug, it's a feature! A feature that might make you want to time-travel back to when this decision was made and have a long, serious talk with the Go team.

It's so confusing. It's as if they were trying to port [.NET's formatting syntax](https://docs.microsoft.com/en-us/dotnet/standard/base-types/custom-date-and-time-format-strings) to a slightly more verbose version of [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck).

## Conclusion

Go is pretty strict with the clean code philosphy which although seems useful in the first look but can be a little hard to work with if you're not used to its quirks, occasionally making you feel like you're solving a puzzle just to do simple tasks. Now excuse me while I go comment out a line of code and accidentally rewrite my entire function.
