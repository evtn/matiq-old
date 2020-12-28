# matiq
**DISCLAIMER: this project was written in few (~10) hours, with no knowledge on how proper parsers or evaluators work. The code itself is very messy and contains a lot of flaws. This language is *definitely not for production use* (if it is not obvious)**

**Please, don't start raging about "your parser is wrong" etc. But if you have ideas or proposals, I'm ready to listen**    
*This README and project itself are not finished. You can help, but eventually I would polish it by myself anyway*

## Foreword

That's a little programming language, mostly math-oriented, written in Python.    
The goal was (not from the start, though) to build a minimalistic language as an experiment.
It has:
- variables
- numbers and strings
- `while condition` and `for var in range` cycles (with `break`, `continue` and `else` by design)
- control flow (`if`/`elif`/`else`)
- lists
- functions
- ability to run (`use`) other matiq files
- [**unsafe**] ability to use objects from current Python environment (and import modules to it)
- slices

## Installation and quickstart
- Get a Python 3.6+ interpreter (the newer the better, but 3.6 should work)
- Download `matiq` file from repository ( i. e. by `git clone https://github.com/evtn/matiq.git`)
- use `python matiq filename` or `matiq filename` to launch code file (matiq interpreter needs to be in current PATH variable (or in current working directory))

## Hello World
```rust
@"Hello World" // yep, that's it
```
Matiq introduces **@** -- unary print operator. Anything that comes in it, would be printed.    
Unlike Python's print, `@x` returns `x` itself, so you can use it in any expression:
```rust
@@@x // prints x value three times
mult = @x * @x // prints x value twice and writes x * x to mult (afterwards)
```
## Variables
Variables are used pretty straightfoward. You can use any combinations of letters, underscores and numbers (except first symbol can't be a number) for a name:
```rust
п1ривет_䄌 = 10 // valid name
x = 5 // assigns 5 to x
y = x * 2 // assigns x * 2 (10) to y
x += y // equal to x = x + y

list = [1, 2, 3, 4] // list variable
z = list[0] // indexing
```
Moreover, assignments are expressions too. Any assignment returns the value of what was assigned (seems legit):
```rust
x = (y = 3) * 8 // x = 24; y = 3
x = y = 3 * 8 // x = 24; y = 24
// assignment has the lowest priority, so we enclose it in parentheses
```
## Operations
Binary operators are pretty the same as in Python:
```rust
variable = 2 * (5 + 8) - 3
x = 4 ** 2
y = 5x // you can omit * when there is variable on the right and expression on the left
z = (3 + 5)x + (y)x // or even that way
@[x, y, z] // [16, 80, 1408]
```
One important difference: as `//` is used for comments, integer division uses `/_`. So avoid this:
```rust
_var = 5
x = 23/_var // EvaluationError: var is undefined
@x
```

## Unary operators
Besides common `-` (unary minus) and `~` (unary inverse) there are a few more Unary operators in Matiq:
```rust
x = 5
@x // prints x and returns it
>x // asks for input with prompt "5"
?x // depends on type. Here: returns bool(x) (True).

y = [1, 2, 3, 4]
?y // returns y length, here: 4
z = "Hello"
?z // same, 5
```

## Lists
Lists are containers for any values (numbers, strings, lists, functions, etc).    
Lists have many operations, mostly based on indexing:
```rust
x = [10, 67, 89, "Hello", [1, 5, 9, 1]]

@x[0] // prints 10.0 (indexing starts with 0)
x[0] = [1, 2, 3] // replacing by index
@x // prints [[1.0, 2.0, 3.0], 67.0, 89.0, 'Hello', [1.0, 5.0, 9.0, 1.0]]
@x[1..3] // prints [67, 89] (x[1:3])
x[0..1] = 4 // delete 0..1 elements and insert 4 at index 0
x[0..0] = 2 // delete 0..0 elements and insert 2 at index 0
x[1..4] = [8, 9] // delete 1..4 elements and insert 3, 4 at indices 1, 2
x[?x] = 6 // add element to the end (?x - length of x)
y = x[0..?x] // copy x to y

@x // prints [2, 8.0, 9.0, 'Hello', [1.0, 5.0, 9.0, 1.0], 6]
```

## Strings
Strings are similar to Python too.    
You can use either `'` or `"` (no triple quotes, sorry) for string, and any escape sequence supported in Python is supported in Matiq:
```rust
hello = "Hello!"
hi = 'Hi!'
hello_and_hi = hello + hi
multiline = "First line\nSecond Line"
escaped_quote = "What exactly is this \"Matiq\"?"
named = "\N{CEDILLA}" // '¸'
unicode = "\u410c" // '䄌'
```

## Functions
Functions are cool. See more info on how to define functions below (Blocks > fn).
Here's how you call them:
```rust
use cool_functions.mtq

x = one() // function with no arguments
y = func_with_two_arguments(1, 2) // okay, this one is obvoius
z = f2(1, [4, 5]) // two arguments, number and list
```
There are no (still) keyword arguments or default values. Just a fixed number of positional arguments. 

## Blocks
Blocks are an essential part of Matiq. Before you've seen only expressions, but here is an example block:
```rust
if 4 > 5
    @"Now that's impossible"
```
Notice the indent. You can use any number of spaces and/or tabs, as long as it's the same number for all lines in one block, and every block has more indents than its parent.

Although, I don't really recommend mixing tabs and spaces, and even using tabs alone. But, your choice.
Expressions can have contents (inner context/lines), but as block itself is responsible for parsing of block contents, this can be used as comments:
```rust
x = 2 * 3
   There we see a block-like expression.
   This block contents must not be parsed, 
   And code snippet must execute without any problem
@x
```

A distinct feature of blocks is that they have a return value that is used by `else` block

### Block types
There are few block types: [*('iff' is not a typo)*](https://en.wikipedia.org/wiki/If_and_only_if)
- `if condition` -- contents are evaluated iff condition evaluated to True. 
- `elif condition` -- contents are evaluated iff last line evaluated to False and condition evaluated to True. 
- `else` -- contents are evaluated iff last line evaluated to False (can be written as `elif 1`). 
- `while condition` -- contents are evaluated while condition evaluates to True
- `for var in range` -- iterate over a Range with a variable
- `fn name(args)` -- function definition
- `import name [as varname]` -- Python import
- `use name` -- Matiq import
- `ffn name [as varname]` -- foreign object injection
More on those types below
### if
You've already seen `if` block in previous section. If condition is true, runs its content.
Returns condition as its result.
```rust
x = 5
if x == 5
    @"x is definitely 5"
```

### elif / else
You could ask -- why is this a separate section?    
That's because in Matiq, `elif` and `else` are independent from `if`:
```rust
if 4 > 5
    @"Now that's impossible"
else
    @"That would be printed"

for i in 0..10
    if i == 6
        break
else
    @"Not this one"

0
else
    @"But this one"
```
`else` / `elif` blocks run their contents when previous line returned False. This includes:
- any `for`/ `while` cycle which ended without a `break`
- `if`/ `elif` block, when condition is false and there was no `break`
- any falsy expression
- any `import` or `use`
- `ffn` block if injected object is false

The difference between `elif` and `else` is simple: first has condition (like `if`), but second doesn't.    

### for
If you are familiar with programming, you would recognize this.    
Matiq has `for` similiar to Python's `for name in range(start, stop, step)`:
```rust
for i in 0..10 // start..stop
   @i // this would print 0, 1, 2, 3...9

for i in 0..20..2 // start..stop..step
   @i // 0, 2, 4...18
   result = i

@i // EvaluationError (for identifier is unaccessible outside)
@result // 18

for i in 0..20
   if i % 2
       continue // skips every odd iteration
   if i > 15
       break // finishes cycle on i == 16
```
`for` returns true if cycle was finished with break, and false otherwise    

### while
`while` is basically a cycled `if`    
You can use `break` and `continue` just like in `for` cycle:
```rust
i = 10
while i
    @(i -= 1) // 9, 8, 7, 6, 5, 4, 3, 2, 1, 0
```

### fn
Functions are useful whenever you need to use same code:
```rust
fn max(a, b)
    if a > b
        max_val = a
    else
        max_val = b
    max_val
```
Currently, matiq doesn't support `return`, so you need to write return value as the last expression in the function body.
*bonus: alternative implementations*
1. List ternary operator
```rust
fn max(a, b)
    [b, a][a > b]
```
2. Foreign injection - almost cheating
```rust
ffn max
```
### use 
`use` block allows you to inject another matiq files into your code as `use` contents:
```rust
// max.mtq
fn max(a, b)
   [b, a][a > b]

// test.mtq
use max.mtq
@max(3, 4) // 4
```
### import
`import` block allows you to import Python module into current interpreter context.
That can be used with `ffn` blocks (`ffn` described in the next section):
```rust
import random
ffn random.randint

x = randint(0, 100)
@x // prints any random number between 0 and 100
```

## Foreign objects injection
As Matiq executes in a Python interpereter, it can heavily use its functions and objects.
Before going crazy, remember about caveats:
- Matiq type system is much narrower than Python type system. Although Matiq is tolerant for unknown types and has some support for tuples and dicts, using `ffn` can lead to unexpected behaviour sometimes
- There are no keyword args in Matiq. That means you need to implement a Python-side wrapper for keyword arguments support (and dictionary support, too):
```rust
// abstract example, won't work from scratch
ffn dict
ffn some_func_with_keyword_args as f
import kwarg_wrapper
ffn kwarg_wrapper.wrapper as kw

data = dict()
data["a"] = 1
data["b"] = 2
@kw(f, data)
```
### Introduction to `ffn`
This is a basic example using `ffn` (among others below and above):
```rust
// Python str.join method
ffn str.join 
x = ["this", "is", "working!"]
// notice how function name got cut to the last part
@join("\n", x) // "\n".join(x)

// aliasing
ffn str.upper as toUppercase

@toUppercase("so low") // SO LOW
```
You can use any identifier or identifier chain divided by dot as `ffn` source. 
Object would be injected by the name of last part in the chain.
Or an alias (which should be a valid variable identifier) if `as identiier` used

## Debugging
This section is unfinished.
But there is one advice:
```rust
@context // prints current context (variables and tree)
```
