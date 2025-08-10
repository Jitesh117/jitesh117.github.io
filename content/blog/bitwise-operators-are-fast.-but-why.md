+++
title = 'Bitwise Operators Are Fast. But Why'
date = 2025-08-10T11:29:39+05:30
draft = false
showToc = true
cover.Image = '/images/bitwise_operators.png'
+++

Some days back I was solving [this](https://codeforces.com/contest/1891/problem/B) problem on [CodeForces](https://codeforces.com/), and I was getting TLE(Time Limit Exceeded) for my solution, even though I knew it shouldn't take too long to run, since I had already optimized my code, after reading the editorial.

The code segment which was taking the most amount of time was this:

```cpp
for (int i = 0; i < q; i++) {
    if(x[i] > min_processed) continue;
    else{
        min_processed = min(min_processed, x[i]);
        for (int j = 0; j < n; j++) {
            if (a[j] % static_cast < int > (pow(2, x[i])) == 0)
                a[j] += static_cast < int > (pow(2, x[i] - 1));
        }
    }
}
```

After digging more into my solution, I got to know that the `pow` function was taking longer than I had expected. I read the forum for that contest and I found that using bitwise shift operations would be faster for this. This was the code after I replaced the `pow` function with bitwise shifts:

```cpp
for (int i = 0; i < q; i++) {
    if (x[i] >= min_processed) continue;

    min_processed = x[i];
    int two_pow_x = 1 << x[i];
    int two_pow_x_minus_1 = 1 << (x[i] - 1);

    for (int j = 0; j < n; j++) {
        if (a[j] % two_pow_x == 0) {
            a[j] += two_pow_x_minus_1;
        }
    }
}
```

And this solution got accepted. This led to me a rabbithole on learning more about bitwise operators. So I decided to write about the things I learned about bitwise operators, which you are reading now haha!

## What makes bitwise operators so damn fast???

When you write a bitwise operation and run it, you may think it is some kind of "optimization" that the super-genius computer people did. But you'd be wrong. Bitwise-operations are fundamental hardware-level instructions that map directly to single logic gates in the processor.

At the transistor level, the CPU's ALU doesn't think in terms of number or words, it's just a humongous network of 1s and 0s. Here's how the bitwise operators work:

| Operation  | Gate Type | Hardware Implementation              |
| ---------- | --------- | ------------------------------------ |
| `&` AND    | AND gate  | Output = 1 only if both inputs are 1 |
| `` ` `` OR | OR gate   | Output = 1 if either input is 1      |
| `^` XOR    | XOR gate  | Output = 1 if inputs differ          |
| `~` NOT    | Inverter  | Flips the bit (one transistor pair)  |

### CPU Execution: 1 single micro-op or less

Modern CPUs execute instructions in terms of micro-operations (µops). A typical x86-64 AND or OR on registers becomes one µop that the ALU executes in one clock cycle latency with throughput of 1 per cycle per ALU.

Example from Intel Skylake (from the optimization manual):

| Instruction    | Latency | Throughput                          |
| -------------- | ------- | ----------------------------------- |
| `AND r64, r64` | 1 cycle | 0.25 cycles (4 per cycle on 4 ALUs) |
| `OR r64, r64`  | 1 cycle | 0.25 cycles                         |
| `XOR r64, r64` | 1 cycle | 0.25 cycles                         |

### No relying on memory

When both operands are already in registers, the CPU doesn’t have to touch RAM or caches — it just feeds the register bits directly to the ALU.

The real bottleneck in most code isn’t the AND/OR itself — it’s memory access. If you’re pulling operands from memory, you’ll spend tens to hundreds of cycles waiting on cache or RAM. That’s why bitwise ops in tight loops over already-loaded data are stupidly fast.

### That's cool and all, but why is `%2 == 1` faster than `&1` in lower numbers?

When things like this happen, it's not because of modular arithmetic is faster than bitwise ops, it's because of how modern CPUs and compilers behave.

When benchmarking:

#### 1. Branch prediction & data patterns

If your test numbers are small and predictable (e.g., 0, 1, 2, 3...), the % 2 == 1 branch can get perfectly predicted by the CPU’s branch predictor.
That means your `%` version runs in a steady pipeline with no stalls.

#### 2. Micro-op fusion

On x86-64, `% 2 == 1` might compile into:

```asm
test    edi, 1
setne   al
```

which is fused into a single micro-op in modern Intel CPUs.
The `&1` version might compile into:

```asm
and     edi, 1
cmp     edi, 1
sete    al
```

which is slightly less optimal depending on compiler output.

### Why the difference disappears with bigger datasets

If your dataset is less predictable (random numbers), branch prediction becomes imperfect, and both `% 2 == 1`and `& 1` tend to compile to near-identical performance.

### Shifts and Rotates are Just as Native

Bitwise shifts (`<<`, `>>`) and rotates (`ROL`, `ROR`) aren't loops in disguise, they're handled by dedicated hardware blocks called **barrel shifters** inside the CPU.

That means shifting a 64-bit register by 1 bit or by 31 bits takes the same amount of time: one cycle latency, full-width in parallel. No "bit-by-bit" work, no iterative shitting bullshit. Just one instruction, one pass through the barrel shifter.

Example:

```asm
shl rax, 5   ; shift left by 5 bits
shr rbx, cl  ; shift right by the amount in CL
```

Both will finish in 1 cycle latency with throughput of 1 per cycle per execution unit.

## Why bitwise feels instant (and sometimes _Is_)

The magic is that bitwise operations don't really "do" anything in the computational sense. They're pure combinational logic. The inputs come in, the transistors flip instantly according to gate rules, and the output is ready before the next clock tick.

In fact, a lot of "normal" arithmetic is _built_ from these primitves inside the compiler implementation:

- Addition = `XOR`(for sum) + `AND`(for carry) + shifts(to propagage carry)
- Multiplication = repeated shifts + adds
- Division by powers of two = just a shift

That's why you sometimes hear people say "everything is just bitwise operations under the hood." They're not exaggerating at all!

## Don't think of it as magic though

But don't get fooled. Don't always expect that the code will magically run faster just because used Bitwise-operations. Most modern compilers these days already do the optimization on their part even if you didn't explitly used Bitwise-operations.

Let's take one example to demonstrate this(I've took this from [this](https://stackoverflow.com/questions/20393373/performance-wise-how-fast-are-bitwise-operators-vs-normal-modulus) stackoverflow thread):

| Original Calculation | Replacement Calculation |
| -------------------- | ----------------------- |
| y = x / 8            | y = x >> 3              |
| y = x \* 64          | y = x << 6              |
| y = x \* 2           | y = x << 1              |
| y = x \* 15          | y = (x << 4) - x        |

> The last example is perhaps the most interesting one. Whilst multiplying or dividing by powers of 2 is easily converted (manually) into bit-shifts operations, the compiler is generally taught to perform even smarter transformations that you would probably think about on your own and who are not as easily recognized (at the very least, I do not personally immediately recognize that (x << 4) - x means x \* 15).

## TL;DR

Bitwise operations are so damn fast because:

- They’re native hardware gates in the ALU — no math, no iteration.
- They compile to one micro-op with one cycle latency (or better throughput).
- They can be parallelized across multiple ALUs.
- Shifts and rotates use barrel shifters — constant-time for any shift amount.
- They avoid memory latency when operands are in registers.

The CPU literally speaks in bitwise. Everything else is just built on top of it.
