# [Understanding Assembly Part I: RISC-V](https://mcyoung.xyz/2021/11/29/assembly-1/)

A [Turing tarpit](https://en.wikipedia.org/wiki/Turing_tarpit) is a programming language that is Turing-complete but very painful to accomplish anything in. One particularly notable tarpit is [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck), which has a reputation among beginner and intermediate programmers as being unapproachable and only accessible to the most elite programmers hence the name, as Wikipedia puts it:

> The language’s name is a reference to the slang term _brainfuck_, which refers to things so complicated or unusual that they exceed the limits of one’s understanding.

Assembly language, the “lowest-level” programming language on any computer, has a similar reputation: difficult, mysterious, and beyond understanding. A Turing tarpit that no programmer would want to have anything to do with.

Although advanced programmers usually stop seeing assembly as mysterious and inaccessible, I feel like it is a valuable topic even for intermediate programmers, and one that can be made approachable and interesting.

This series seeks to be that: assuming you have already been using a compiled language like Rust, C++, or Go, how _is_ assembly relevant to you?

> If you’re here to just learn assembly and don’t really care for motivation, you can just [skip ahead](https://mcyoung.xyz/2021/11/29/assembly-1/#diving-in).
> 
> This series is about learning to _understand_ assembly, not write it. I do occasionally write assembly for a living, but I’m not an _expert_, and I don’t particularly relish it. I do read a ton of assembly, though.

## [What Is It, Anyways?](https://mcyoung.xyz/2021/11/29/assembly-1/#what-is-it-anyways)

As every programmer knows, computers are very stupid. They are very good at following instructions and little else. In fact, the computer is _so_ stupid, it can only process basic instructions serially[^1], one by one. The instructions are very simple: “add these two values”, “copy this value from here to there”, “go run these instructions over here”.

A computer processor implements these instructions as electronic circuits. At its most basic level, every computer looks like the following program:

```c
size_t program_counter = ...;
Instruction *program = ...;

while (true) {
  Instruction next = program[program_counter];
  switch (next.opcode) {
    // Figure out what you're supposed to be doing and do it.
  }
  program_counter++;
}
```

The array `program` is a your program encoded as a sequence of these “machine instructions” in some kind of binary format. For example, in RISC-V programs, each instruction is a 32-bit integer. This binary format is called _machine code_.

For example, when a RISC-V processor encounters the value `5407443` decoding circuitry decides that it means that it should take the value in the “register” `a0`, add `10` to it, and place the result in the register `a1`.

> ### [Decoding Instructions](https://mcyoung.xyz/2021/11/29/assembly-1/#decoding-instructions)
> 
> `5407443` seems opaque, but when viewed as binary, we can see how the processor decodes it:
> 
> ```plaintext
> > 0b 000000000101 00101 000 00110 0010011
> >    \__________/ \___/ \_/ \___/ \_____/
> >     |           |     |   |     |
> >     imm         rs1   fn  rd    opcode
> ```
> 
> `opcode` describes what sort of instruction this is, and what format it’s in; `0b0010011` means it’s an “immediate arithmetic” instruction, which uses the “I-type” format, given above. `fn` further specifies what the operation does; `0b000`, combined with the value of `opcode`, means this is an _addition_ instruction.
> 
> `rs1` is the source: it gives the name of the source register, `a0`, given by it index, `0b00101`, i.e., `10`. Similarly, `rd` specifies the destination `a1` by its index `11`. Finally, `imm` is the value to add, `0b000000000101`, or `10`. The constant value appears immediately in the instruction itself, so it’s called an _immediate_.

However, if you’re a human programming a computer, writing all of this by hand is… very 60s, and you might prefer to have a textual representation, so you can write this more simply as `addi a1, a0, 10`.

`addi a1, a0, 10` is a single line of assembly: it describes a _single_ instruction in text form. Assembly language is “just” a textual representation of the program’s machine code. Your assembler can convert from text into machine instructions, and a _dis_assembler reverses the process.

The simple nature of these instructions is what makes assembly a sort of Turing tarpit: you only get the most basic operations possible: you’re responsible for building _everything_ else.

### [On Architectures](https://mcyoung.xyz/2021/11/29/assembly-1/#on-architectures)

There isn’t “an” assembly language. Every computer has a different instruction set architecture, or “ISA”; I use the terms “instruction set”, “architecture”, and “ISA” interchangeably. Each ISA has a corresponding assembly language that describes that ISA’s specific instructions, but they all generally have similar overall structure.

I’m going to focus on three ISAs for ease of exposition, introduced in this order:

1. RISC-V, a modern and fairly simple instruction set (specifically, the rv32gc variant). That’s Part I.
2. x86\_64, the instruction set of the device you’re reading this on (unless it’s a phone, an Apple M1 laptop, or something like a Nintendo Switch). That’s Part II.
3. MOS 6502, a fairly ancient ISA still popular in very small microcontrollers. That’s Part III.

We’re starting with RISC-V because it’s a particularly elegant ISA (having been developed for academic work originally), while still being representative of the operations most ISAs offer.

In the future, I may dig into some other, more specialized ISAs.

## [But _Why_?](https://mcyoung.xyz/2021/11/29/assembly-1/#but-why)

It’s actually very rare to write actual assembly. Thanks to modern (relatively) languages like Rust, C++, and Go, and even things like Haskell and JavaScript, virtually no programmers need to write assembly anymore.

But that’s only because it’s the leading language written by _computers themselves_. A compiler’s job is, fundamentally, to write the the assembly you would have had to write _for_ you. To better understand what a compiler is doing for you, you need to be able to read its output.

At this point, it may be worth looking at my [article on linkers](https://mcyoung.xyz/2021/06/01/linker-script/#seriously-whats-a-linker) as a refresher on the C compilation model.

For example, let’s suppose we have the very simple C program below.

```square.c
#include <stdio.h>

int square_and_print(int x) {
    x *= x;
    printf("%d\n", x);
    return x;
}
```

Clang, my C compiler of choice, can turn it directly into a library via `clang -c square.c`. `-c` asks the compiler to stop before the link step, outputting the _object file_ `square.o`. We can ask the compiler to stop even sooner than that by writing `clang -S square.c`, which will output `square.s`, the assembly file the compiler produced! For this example, and virtually all others in this post, I’m using a RISC-V target: `-target riscv32-unknown-elf -march=rv32gc`.

If you build with `-Oz` to make the code as small as possible (this makes it easiest to see what’s going on, too), you get something like this:

```square.s
.text
	.file   "square.c"
	.globl  square_and_print
square_and_print:
	addi    sp, sp, -16
	sw      ra, 12(sp)
	sw      s0, 8(sp)
	mul     s0, a0, a0          // !
	lui     a0, %hi(.L.str)
	addi    a0, a0, %lo(.L.str)
	mv      a1, s0
	call    printf              // !
	mv      a0, s0
	lw      s0, 8(sp)
	lw      ra, 12(sp)
	addi    sp, sp, 16
	ret

	.section        .rodata
.L.str:
	.asciz  "%d\n"
```

There’s a lot going on! But pay attention to the two lines with a `// !`: the first is `mul s0, a0, a0`, which is the multiplication `x *= x;`. The second is `call printf`, which is our function call to `printf()`! I’ll explain what everything else means in short order.

Writing assembly isn’t a crucial skill, but being able to read it is. It’s actually so useful, that a website exists for quickly generating the assembly output of a vast library of compilers: the [Compiler Explorer](https://godbolt.org/), frequently just called “godbolt” after its creator, Matt Godbolt. Being able to compare the output of different compilers can help understand what they do! Click on the `godbolt` button in the code fences to a godbolt for it.

“Low-level” languages like C aren’t the only ones where you can inspect assembly output. Godbolt supports Go: for example, click the `godbolt` button below.

```go
package sq

import "fmt"

func SquareAndPrint(x int) int {
    x *= x
    fmt.Printf("%d\n", x)
    return x
}
```

Hopefully this is motivation enough to jump into the language proper. It is very useful to have a godbolt tab open to play around with examples!

## [Diving In](https://mcyoung.xyz/2021/11/29/assembly-1/#diving-in)

So, let’s say you _do_ want to read assembly. How do we do that?

Let’s revisit our `square.c` example above. This time, I’ve added comments explaining what all the salient parts of the code do, including the _assembler directives_, which are all of the form `.blah`. Note that the _actual_ compiler output includes way more directives that would get in the way of exposition.

There’s a lot of terms below that I haven’t defined yet. I’ll break down what this code does gradually, so feel free to refer back to it as necessary, using [this handy-dandy link.](https://mcyoung.xyz/2021/11/29/assembly-1/#big-example)

```square.s
	// This tells the assembler to place all code that
	// follows in the `.text` section, where executable
	// data goes.
	.text

	// This is just metadata that tools can use to figure out
	// how the executable was built.
	.file   "square.c"

	// This asks the assembler to mark `square_and_print`
	// as an externally linkable symbol. Other files that
	// refer to `square_and_print` will be able to find it
	// at link time.
	.globl  square_and_print

square_and_print: // This is a label, which gives this position
		  // in the executable a name that can be
		  // referenced. They're very similar to `goto`
		  // labels from C.
		  //
		  // We'll see more labels later on.


	// This is the function prologue, which "sets up" the
	// function: it allocates stack space and saves the
	// return address, along with other calling-convention
	// fussiness.
	addi    sp, sp, -16
	sw      ra, 12(sp)
	sw      s0, 8(sp)

	// This is our `x *= x;` from before! Notice that the
	// compiler rewrote this to `temp = x * x;` at some
	// point, since the destination register is `s0`.
	mul     s0, a0, a0

	// These two instructions load the address of a string
	// constant; this pattern is specific to RISC-V.
	lui     a0, %hi(.L.str)
	addi    a0, a0, %lo(.L.str)
	
	// This copies the multiplication result into `a1`.
	mv      a1, s0

	// Call to printf!
	call    printf

	// Move `s0` into `a0`, since it's the return value.
	mv      a0, s0

	// This is the function epilogue, which restores state
	// saved in the prologue and de-allocates the stack
	// frame.
	lw      s0, 8(sp)
	lw      ra, 12(sp)
	addi    sp, sp, 16
	
	// We're done; return from the function!
	ret

	// This tells the assembler to place what follows in
	// the `.rodata` section, for read-only constants like
	// strings.
	.section        .rodata

.L.str: // Give our string constant a private name. By convention,
	// .L labels are "private" names emitted by the compiler.

	// Emit an ASCII string into `.rodata` with an extra null
	// terminator at the end: that's what the `z` stands for.
	.asciz  "%d\n"
```

### [The Core Syntax](https://mcyoung.xyz/2021/11/29/assembly-1/#the-core-syntax)

All assemblers are different, but the core syntax tends to be the same. There are three main kinds of syntax productions:

- Instructions, which consist of a _mnemonic_ followed by some number of _operands_, such as `addi sp, sp -16` and `call printf` above. These are the text encoding of machine code.
- Labels, which consist of a symbol followed by a colon, like `square_and_print:` or `.L.str:`. These are used to let instruction operands refer to locations in the program.
- Directives, which vary wildly by assembler. GCC-style assembly like that above uses a `.directive arg, arg` syntax, as seen in `.text`, `.globl`, and `.asciz`. They control the behavior of the assembler in various ways.

An assembler’s purpose is to read the `.s` file and serialize it as a binary `.o` file. It’s kind of like a compiler, but it does virtually no interesting work at all, beyond knowing how to encode instructions.

Directives control how this serialization occurs (such as moving around the output cursor); instructions are emitted as-is, and labels refer to locations in the object file. Simple enough, right?

### [Anatomy of an Instruction](https://mcyoung.xyz/2021/11/29/assembly-1/#anatomy-of-an-instruction)

Let’s look at the very first instruction in `square_and_print`:

```
// RISC-V Assembly
	addi sp, sp, -16
	---- --  --  ---
	 |   |   |    |
	mnemonic |   immediate operand
	     |  input operand
	     |
	    output operand
```

The first token is called the _mnemonic_, which is a painfully terse abbreviation of what the instruction does. In this case, `addi` means “add with immediate”.

`sp` is a _register_. Registers are special variables wired directly into the processor that can be used as operands in instructions. The degree to which only registers are permitted as operands varies by architecture; RISC-V only allows registers, but x86, as we’ll see, does not. Registers come in many flavors, but `sp` is a GPR, or “general purpose register”; it holds a machine word-sized integer, which in the case of 32-bit RISC-V is… 32-bit[^2].

> #### [RISC-V Registers](https://mcyoung.xyz/2021/11/29/assembly-1/#risc-v-registers)
> 
> One of my absolute favorite parts of RISC-V is how it names its registers. It has 32 GPRs named `x0` through `x31`. However, these registers have so-called “ABI names” that specify the _role_ of each register in the ABI.
> 
> The usefulness of these names will be much more apparent when we discuss [the calling convention](https://mcyoung.xyz/2021/11/29/assembly-1/#the-calling-convention), so feel free to come back to this later.
> 
> `x0` is called `zero`, because of its special property: writes to it are ignored, and reads always produce zero. This is handy for encoding certain common operations: for example, it can be used to quickly get a constant value= into a register: `addi rd, zero, 42`.
> 
> `x1`, `x2`, `x3`, and `x4` have special roles and generally aren’t used for general computation. The first two are the link register `ra`, which holds the return address, and `sp`, the stack pointer.
> 
> The latter two are `gp` and `tp`; the global ppointer and the thread ppointer; their roles are somewhat complicated, so we won’t discuss them in this post.
> 
> The remaining registers belong to one of three categories: _argument_ registers, _saved_ registers, and _temporary_ registers, named so for their role in calling a function (as described [below](https://mcyoung.xyz/2021/11/29/assembly-1/#the-calling-convention)).
> 
> The argument registers are `x10` through `x17`, and use the names `a0` through `a7`. The saved registers are `x8`, `x9`, and `x18` through `x27`, called `s0`, through `s11`. The temporary registers are `x5` through `x7` and `x28` through `x31`, called `t0` through `t6`.
> 
> As a matter of personal preference, you may notice me reaching for argument registers for most examples.

`-16` is an _immediate_, which is a literal value that is encoded directly into the instruction. The encoding of `addi sp, sp, -16` will include the binary representation of `-16` (in the case of RISC-V, as a 12-bit integer). \[The decoding example above\]{#decoding-instructions} shows how immediates are literally encoded _immediately_ in the instruction.

Immediates allow for small but fixed integer arguments to be encoded with high locality to the instruction, which is good for code size and performance.

The first operand in RISC-V is (almost) always the output. `addi, rd, rs, imm` should be read as `rd = rs + imm`. Virtually all assembler syntax follows this convention, which is called the [three-address code](https://en.wikipedia.org/wiki/Three-address_code).

Other kinds of operands exist: for example, `call printf` refers to the _symbol_ `printf`. The assembler, which doesn’t actually know where `printf` is, will emit a small note in the object file that tells the linker to find `printf` and splat it into the assembly according to some instructions in the note. These notes are called _relocations_.

The instructions `lui a0, %hi(.L.str)` and `addi a0, a0, %lo(.L.str)` use the `%lo` and `%hi` operand types, which are specific to RISC-V; they load the low 12 bits and high 20 bits of a symbol’s address into the immediate operand. This is a RISC-V-specific pattern for loading an address into a register, which most assemblers provide with the _pseudoinstruction_ `la a0, .L.str` (where `la` stands for “load address”).

Most architectures have their own funny architecture-specific operand types to deal with the architecture’s idiosyncrasy.

### [Types of Instructions](https://mcyoung.xyz/2021/11/29/assembly-1/#types-of-instructions)

Available instructions tend to be motivated by providing one of three classes of functionality:

1. A Turing-complete [register machine](https://en.wikipedia.org/wiki/Register_machine) execution environment. This lends to the Turing tarpit nature of assembly: only the absolute minimum in terms of control flow and memory access is provided.
2. Efficient silicon implementation of common operations on bit strings and integers, ranging from arithmetic to cryptographic algorithms.
3. Building a secure operating system, hosting virtual machines, and actuating hardware external to the processor, like a monitor, a keyboard, or speakers.

Instructions can be broadly classified into four categories: arithmetic, memory, control flow, and “everything else”. In the last thirty years, the bar for general-purpose architectures is usually “this is enough to implement a C runtime.”

#### [Arithmetic Instructions](https://mcyoung.xyz/2021/11/29/assembly-1/#arithmetic-instructions)

Arithmetic makes up the bulk of the instruction set. This always includes addition, subtraction, and bitwise and, or, and xor, as well as unary not and negation.

In RISC-V, these come in two variants: a three-register version and a two-register, one immediate version. For example, `add a0, a1, a2` is the three-register version of addition, while `addi a0, a1, 42` is the immediate version. There isn’t a `subi` though, since you can just use negative immediates with `addi`.

`not` and `neg` are not actual instructions in RISC-V, but pseudoinstructions: `not a0, a1` encodes as `xori a0, a1, -1`, while `neg a0, a1` becomes `sub a0, zero, a1`.

Most instruction sets also have bit shifts, usually in three flavors: left shifts, right shifts, and _arithmetic_ right shifts; arithmetic right shift is defined such that it behaves like division by powers of two on signed integers. RISC-V’s names for these instructions are `sll`, `srl`, and `sra`.

Multiplication and division are somewhat rarer, because they are expensive to implement in silicon; smaller devices don’t have them[^3]. Division in particular is very complex to implement in silicon. Instruction sets usually have different behavior around division by zero: some architectures will fault, similar to a memory error, while some, like RISC-V, produce a well-defined trap value.

There is usually also a “copy” instruction that moves the value of one register to another, which is kind of like a trivial arithmetic instruction. RISC-V calls this `mv a0, a1`, but it’s just a pseudoinstruction that expands to `addi a0, a1, 0`.

Some architectures also offer more exotic arithmetic. This is just a sampler of what’s sometimes available:

- Bit rotation, which is like a shift but bits that get shifted off end up at the other end of the integer. This is useful for a vast array of numeric algorithms, including ARX ciphers like [ChaCha20](https://en.wikipedia.org/wiki/Salsa20#ChaCha_variant).
- Byte reversal, which can be used for changing the endianness of an integer; bit reversal is analogous.
- Bit extraction, which can be used to form new integers out of bitfields of another.
- Carry-less multiplication, which is like long multiplication but you don’t bother to carry anything when you add intermediates. This is used to implement [Galois/Counter mode encryption](https://en.wikipedia.org/wiki/Galois/Counter_Mode).
- Fused instructions, like `xnor` and `nand`.
- Floating point instructions, usually implementing the [IEEE 754](https://en.wikipedia.org/wiki/IEEE_754) standard.

There is also a special kind of arithmetic instruction called a _vector instruction_, but I’ll leave those for another time.

#### [Memory Instructions](https://mcyoung.xyz/2021/11/29/assembly-1/#memory-instructions)

Load instructions fetch memory from RAM into registers, while store instructions write it back. These instructions are what we use to implement pointers.

They come in all sorts of different sizes: RISC-V has `lw`, `lh`, and `lb` for loading 32-, 16-, and 8-bit values from a location; `sw`, `sh`, and `sb` are their store counterparts. 64-bit RISC-V also provides `ld` and `sd` for 64-bit loads and stores.

Load/store instructions frequently take an offset for indexing into memory. `lw a1, 4(a0)`[^4] is effectively `a1 = a0[4]`, treating `a0` like a pointer.

These instructions frequently have an _alignment_ constraint: the pointer value must (or, at least, should) be divisible by the number of bytes being loaded. RISC-V, for example, mandates that `lw` only be used on pointers divisible by 4. This constraint simplifies the microarchitecture; even on architectures that don’t mandate it, aligned loads and stores are typically far faster.

This category also includes instructions necessary for implementing atomics, such as `lock cmpxchg` on x86 and `lr`/`sc` on RISC-V. Atomics are fundamentally about changing the semantics of reading and writing from RAM, and thus require special processor support.

Some architectures, like x86, 65816, and very recently, ARM, provide instructions that implement `memcpy` and its ilk in hardware: in x86, for example, this is called `rep movsb`.

#### [Control Flow Instructions](https://mcyoung.xyz/2021/11/29/assembly-1/#control-flow-instructions)

Control flow is the secret ingredient that turns our glorified calculator into a Turing tarpit: they allow changing the flow of program execution based on its current state.

_Unconditional jumps_ implement `goto`: given some `label`, the `j label` instruction jumps directly to it. `j` can be thought of as writing to a special `pc` register that holds the program counter. RISC-V also provides a _dynamic_ jump, `jr`, which will jump to the address in a register. Function calls and returns are a special kind of unconditional jump.

_Conditional jumps_, often called _branches_, implement `if`. `beq a0, a1, label` will jump to `label` if `a0` and `a1` contain the same value. RISC-V provides branch instructions for all kinds of comparisons, like `bne`, `blt`, and `bge`.

Conditional and unconditional jumps can be used together to build loops, much like we could in C using `if` and `goto`.

For example, to zero a region of memory:

```
// RISC-V Assembly
	// Assume a0 is the start of the region, and a1 the
	// number of bytes to zero.

	// Set a1 to the end of the region.
	addi a1, a0, a1
loop_start:
	// If a0 == a1, we're done!
	beq a0, a1, loop_done

	// Store a zero byte to `a0` and advance the pointer.
	sb zero, 0(a0)
	addi a0, a0, 1

	// Take it from the top!
	j loop_start
loop_done:
```

#### [Miscellaneous Instructions](https://mcyoung.xyz/2021/11/29/assembly-1/#miscellaneous-instructions)

“Everything else” is, well… everything else.

No-op instructions do nothing: `nop`’s only purpose is to take up space in the instruction stream. No-op instructions can be used to pad space in the instruction stream, provide space for the linker to fix things up later, or implement [`nop` sleds](https://en.wikipedia.org/wiki/NOP_slide).

Instructions for poking processor state, like `csrrw` in RISC-V and `wrmsr` in `x86` also belong in this category, as do “hinting” instructions like memory prefetches.

There are also instructions for special control flow: `ecall` is RISC-V’s “syscall” instruction, which “traps” to the kernel for it to do something; other architectures have similar instructions.

Breakpoint instructions and [“fence” instructions](https://www.felixcloutier.com/x86/lfence) belong here, too.

## [The Calling Convention](https://mcyoung.xyz/2021/11/29/assembly-1/#the-calling-convention)

Functions are the core abstraction of all of programming. Assembly is no different: we have functions there, too!

Like in any language, functions are _passed_ a list of arguments, perform some work, and _return_ a value. For example, in C:

```c
int identity(int x) {
  return x;
}

// ...

identity(5)  // Returns 5.
```

Unfortunately, there isn’t anything like function call syntax in assembly. As with everything else, we need do it instruction by instruction. All we _do_ get in most architectures is a `call` instruction, which sets up a return address somewhere, and a `ret` instruction, which uses the return address to jump to where the function was called.

We need some way to pass arguments, return a computed value, and maintain a call stack, so that each function’s return address is kept intact for its `ret` instruction to consume. We also need this to be universal: if I pull in a library, I should be able to call its functions.

This mechanism is called the _calling convention_ of the platform’s ABI. It’s a convention, because all libraries must respect it in their exposed API for code to work correctly at runtime.

### [A Function Call in Slow-Mo](https://mcyoung.xyz/2021/11/29/assembly-1/#a-function-call-in-slow-mo)

At the instruction level, function calls look something like this:

1. Pre-call setup. The caller sets up the function call arguments by placing them in the appointed locations for arguments. These are usually either registers or locations on the stack. a. The caller also saves the _caller-saved registers_ to the stack.
2. Jump to the function. The caller executes a `call` instruction (or whatever the function call instruction might be called – virtually all architectures have one). This sets the program counter to the first instruction of the callee.
3. Function prologue. The callee does some setup before executing its code. a. The callee reserves space on the stack in an architecture-dependent manner. b. The callee saves the _callee-saved registers_ to this stack space.
4. Function body. The actual code of the function runs now! This part of the function needs to make sure the return value winds up wherever the _return slot_ for the function is.
5. Function epilogue. The callee undoes whatever work it did in the prologue, such as restoring saved registers, and executes a `ret` (or equivalent) instruction to return.
6. Post-call cleanup. The caller is now executing again; it can unspill any saved state that it needs immediately after the function call, and can retrieve the return value from the return slot.  
In some ABIs, such as C++’s on Linux, this is where the destructors of the arguments get run. (Rust, and C++ on Windows, have _callee_\-destroyed arguments instead.)
    

When people say that function calls have overhead, this is what they mean. Not only does the `call` instruction cause the processor to slam the breaks on its pipeline, causing all kinds of work to get thrown away, but state needs to be delicately saved and restored across the function boundary to maintain the illusion of a callstack.

Small functions which don’t need to use as many registers can avoid some of the setup and cleanup, and _leaf functions_ which don’t call any other functions can avoid basically all of it!

Almost all registers in RISC-V are caller-saved, except for `ra` and the “saved” registers `s0` and `s11`.

Callee-saved registers are convenient, because they won’t be wiped out by function calls. We can actually see the call to `printf` use this: even though the compiler could have emitted `mul a1, a0, a0` and avoided the `mv`, this is actually less optimal. We need to keep the value around to return, and `a1` is caller-saved, so we would have had to spill `a1` before calling `printf`, regardless of whether `printf` overwrites `a1` or not. We would then have to unspill it into `a0` before `ret`. This costs us a hit to RAM. However, by emitting `mul s0, a0, a0; mv a1, s0`, we speculatively avoid the spill: if `printf` is compiled such that it never touches `s0`, the value never leaves registers at all!

### [Caller-Side](https://mcyoung.xyz/2021/11/29/assembly-1/#caller-side)

We can see steps 1 and 2 in the call to `printf`:

```
// RISC-V Assembly
	lui     a0, %hi(.L.str)
	addi    a0, a0, %lo(.L.str)
	mv      a1, s0
	call    printf
```

Arguments in the usual[^5] RISC-V calling convention, word-sized arguments are passed in the `a0` through `a7` registers, falling back to passing on the stack if they run out of space. If the argument is too big to fit in a register, it gets passed by reference instead. Arguments that fit into two registers can be split across registers.

We can see this in action [above](https://mcyoung.xyz/2021/11/29/assembly-1/#big-example). The first argument, a string, is passed by pointer in `a0`; `lui` and `addi` do the work of actually putting that pointer into `a0`. The second argument `x` is passed in `a1`, copied from `s0` where it landed from the earlier `mul` instruction.

Complex function signatures require much more[^6] work to set up.

Once we’re done getting arguments into place, we `call`, which switches execution over to `printf`’s first instruction. In addition, it stores the return address, specifically, the address of the instruction immediately after the `call`, into an architecture-specific location. On RISC-V, this is the special register `ra`.

### [Callee-Side](https://mcyoung.xyz/2021/11/29/assembly-1/#callee-side)

Meanwhile, steps 3 and 4 occur in `square_and_print`’s prologue/epilogue itself:

```
// RISC-V Assembly
square_and_print: 
	addi    sp, sp, -16
	sw      ra, 12(sp)
	sw      s0, 8(sp)

	// ...

	lw      s0, 8(sp)
	lw      ra, 12(sp)
	addi    sp, sp, 16
	ret
```

`addi sp, sp, -16`, which we stared at so hard above, grows the stack by 16 bytes. `sp` holds the _stack pointer_, which points to the top of the stack at all times. The stack grows downwards (as in most architectures!) and must be aligned to 16-byte boundaries across function calls: even though `square_and_print` only uses eight of those bytes, the full 16 bytes must be allocated.

The two `sw` instructions that follow store (or “spill”) the callee-saved registers `ra` and `s0` to the stack. Note that `s1` through `s11` are _not_ spilled, since `square_and_print` doesn’t use them!

Th this point, the function does its thing, whatever that means. This includes putting the return value in the return slot, which, for a function that returns an `int`, is in `a0`. In general, the return slot is passed back to the caller much like arguments are: if it fits in registers, `a0` and `a1` are used; otherwise, the caller allocates space for it and passes a pointer to the return slot as a hidden argument (in e.g. `a0`)[^7].

The epilogue inverts all operations of the prologue in reverse, unspilling registers and shrinking the stack, followed by `ret`. On RISC-V, all `ret` does is jump to the location referred to by the `ra` register.

Of course, all this work is only necessary to maintain the illusion of a callstack; if `square_and_print` were a leaf function, it would not need to spill anything at all! This results in an almost trivial function:

```c
int square(int x) {
  return x * x;
}
```

```
// RISC-V Assembly
// `x` is already in a0, and the
// return value needs to wind up
// in a0. EZ!
square:
	mul a0, a0, a0
	ret
```

Because leaf functions won’t call other functions, they won’t need to save the caller-saved `tX` registers, so they can freely use them instead of the `sX` registers.

## [The End, for Now](https://mcyoung.xyz/2021/11/29/assembly-1/#the-end-for-now)

Phew! We’re around six thousand words in, so let’s checkpoint what we’ve learned:

1. Computers are stupid, but can at least follow _extremely_ basic instructions, which are encoded as binary.
2. Assembly language is human-readable version of these basic instructions for a particular computer.
3. Assembly language programs consist of _instructions_, _labels_, and _directives_.
4. Each instruction is a _mnemonic_ followed by zero or more _operands_.
5. _Registers_ hold values the machine is currently operating on.
6. Instructions can be broadly categorized as _arithmetic_, _memory_, _control flow_, and “miscellaneous” (plus _vector_ and _float_ instructions, for another time).
7. The _calling convention_ describes the low-level interface of a general function, consisting of some pre-call setup, and a prologue and epilogue in each function.

That’s all for now. RISC-V is a powerful but reasonably simple ISA. Next time, we’ll dive into the much older, much larger, and much more complex Intel x86.

[^1]: This is a hilarious lie that is beyond the scope of this post. See, for example, [https://en.wikipedia.org/wiki/Superscalar\_processor](https://en.wikipedia.org/wiki/Superscalar_processor).
[^2]: What’s a machine word, exactly? It really depends on context. Most popular architectures has a straight-forward definition: the size of a GPR _or_ the size of a pointer, which are the same.  
This is not true of all architectures, so beware.
[^3]: Thankfully, these can be polyfilled using the previous ubiquitous instructions. Hacker’s Delight contains all of the relevant algorithms, so I won’t reproduce them here. The division polyfills are particularly interesting.
[^4]: It’s a bit interesting that we don’t write `lw a1, a0[4]` in imitation of array syntax. This specific corner of the notation is shockingly diverse across assemblers: in ARM, we write `ldr r0, [r1, #offset]`; in x86, `mov rax, [rdx + offset]`, or `movq offset(%rdx), %rax` for AT&T-flavored assemblers (which is surprisingly similar to the RISC-V syntax!); in 6502, `lda ($1234, X)`.
[^5]: The calling convention isn’t actually determined by the architecture in most cases; that’s why it’s called a _convention_. The convention on x86 actually differs on Windows and Linux, and is usually also language-dependent; C’s calling convention is usually documented, but C++, Rust, and Go invent their own to handle language-specific fussiness.  
Of course, if you’re writing assembly, you can do whatever you want (though the silicon may be optimized for a particular recommended calling convention).  
RISC-V defines a recommended calling convention for ELF-based targets: [https://github.com/riscv-non-isa/riscv-elf-psabi-doc](https://github.com/riscv-non-isa/riscv-elf-psabi-doc).
[^6]: The following listing shows how all kinds of different arguments are passed. The output isn’t quite what Clang emits, since I’ve cleaned it up for clarity.  
```c
#include <stdio.h>
#include <stdint.h>
#include <stdnoreturn.h>

struct Pair {
  uint32_t x, y;
};
struct Triple {
  uint32_t x, y, z;
};
struct Packed {
  uint8_t x, y, z;
};

// `noreturn` obviates the
// {pro,epi}logue in `call_it`.
noreturn void all_the_args(
  uint32_t a0,
  uint64_t a1a2,
  struct Pair a3a4,
  struct Triple a5_by_ref,
  uint16_t a6,
  struct Packed a7,
  uint32_t on_the_stack,
  struct Triple stack_by_ref
);

void call_it(void) {
  struct Pair u = {7, 9};
  struct Triple v = {11, 13, 15};
  struct Packed w = {14, 16, 18};
  all_the_args(
    42, -42,  u, v,
     5,   w, 21, v
  );
}
```  
```cpp
// RISC-V Assembly
call_it:
  // Reserve stack space.
  addi    sp, sp, -48

  // Get `&call_it.v` into `a3`.
  lui     a3, %hi(call_it.v)
  addi    a3, a3, %lo(call_it.v)

  // Copy contents of `*a3`
  // into `a0...a2`.
  lw      a0, 0(a3)
  lw      a1, 4(a3)
  lw      a2, 8(a3)

  // Create two copies of `v`
  // on the stack to pass by
  // reference.

  // This is `a5_by_ref`.
  sw      a2, 40(sp)
  sw      a1, 36(sp)
  sw      a0, 32(sp)

  // This is `stack_by_ref`.
  sw      a2, 24(sp)
  sw      a2, 20(sp)
  sw      a0, 16(sp)
  
  // Load the argument regs.
  addi    a0, zero, 42
  addi    a1, zero, -42
  addi    a2, zero, -1
  addi    a3, zero, 7
  addi    a4, zero, 9
  // A pointer to `a5_by_ref`!
  addi    a5, sp, 32
  addi    a6, zero, 5
  // Note that `a7` is three
  // packed bytes!
  lui     a0, 289
  addi    a7, a0, 14

  // Store `21` on the top of
  // the stack (our "a8")
  addi    t0, zero, 21
  sw      t0, 0(sp)

  // Store a pointer to
  // `stack_by_ref` on the 
  // second spot from the
  // stack top (our "a9")
  addi    t0, sp, 16
  sw      t0, 4(sp)

  // Call it!
  call    all_the_args

call_it.v:
  // The constant `{11, 13, 15}`.
  .word   11
  .word   13
  .word   15
```
[^7]: LLVM occasionally does somewhat clueless things around this corner of some ABIs. Given  
```c
typedef struct { char p[100]; } X;

X make_big(int x) {
  return (X) {x};
} 
```  
we get the following from Clang:  
```cpp
// RISC-V Assembly
// NOTE: Return slot passed in `a0`, `x` passed in `a1`.
make_big:
	addi    sp, sp, -16
	sw      ra, 12(sp)
	sw      s0, 8(sp)
	sw      s1, 4(sp)
	mv      s0, a1
	mv      s1, a0
	addi    a0, a0, 1
	addi    a2, zero, 99
	mv      a1, zero
	call    memset
	sb      s0, 0(s1)
	lw      s1, 4(sp)
	lw      s0, 8(sp)
	lw      ra, 12(sp)
	addi    sp, sp, 16
	ret
```  
Note that `sb s0, 0(s1)` stores the input value `x` into the first element of the big array _after_ calling memset. If we move the store to before, we can avoid much silliness, including some unnecessary spills:  
```cpp
// RISC-V Assembly
make_big:
	addi    sp, sp, -16
	sw      ra, 12(sp)
	sb      a1, 0(a0)
	addi    a0, a0, 1
	mv      a1, zero
	addi    a2, zero, 99
	call    memset
	lw      ra, 12(sp)
	addi    sp, sp, 16
	ret
```
