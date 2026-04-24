# Lexical Structure

## Comments

- Single-line comments: `// ...`
- Multi-line comments: `/* ... */`

## Statement Terminators

Semicolons are required for simple statements.

Block/control statements such as `if`, `match`, `while`, and `for` do not require a trailing semicolon.

## Whitespace and Identifiers

- Spaces, tabs, and newlines may appear freely between tokens unless they would split a token.
- Newlines are not semantically significant by themselves.
- Identifiers start with an ASCII letter or `_`, followed by zero or more ASCII letters, digits, or `_`.
- Keywords may not be used as identifiers except where explicitly allowed by a later grammar form.

## Literals

### Integers

Supported forms:

- decimal: `123`
- hex: `0xDEADBEEF` | `0xDEAD_BEEF`
- binary: `0b10101100` | `0b1010_1100`
- underscore separators: `1_000_000`

Underscore separators are optional and exist only for readability.

Integer semantics:

- Compile-time-proven overflow is a compile-time error.
- Compile-time-proven invalid shifts are compile-time errors.
- Compile-time-proven division or modulo by zero is a compile-time error.
- Runtime integer overflow wraps; it does not trap.
- Runtime wrapping applies to integer `+`, `-`, `*`, and unary `-`.
- Wrapping is modulo `2^N`, where `N` is the bit width of the operand type.
- For signed integers, the wrapped bit pattern is interpreted as a two's-complement value of the same width.
- Shift amounts greater than or equal to the bit width count as invalid shifts.
- Runtime shifts with a negative count or a count greater than or equal to the bit width panic.
- Left shift `<<` is equivalent to multiplication by `2^count` modulo `2^N` when the shift count is valid.
- Right shift `>>` is logical for unsigned integers and arithmetic (sign-extending) for signed integers when the shift count is valid.
- Negating the minimum signed value wraps to itself.
- Integer division truncates toward zero.
- Integer division by zero panics.
- Integer modulo by zero panics.
- Signed `MIN / -1` wraps to `MIN`.
- Signed `MIN % -1` evaluates to `0`.
- Integer literals that do not fit the target type are compile-time errors.
- Bitwise operations are defined over the fixed-width representation.

### Floats

Supported forms:

- `3.14`
- `2.0e5`
- `6.9`
- `NaN`
- `Infinity`

Float semantics follow IEEE-754 for the target width.

- Floating-point overflow produces `Infinity`.
- `NaN` compares unequal to everything, including itself.

### Strings

- Only double-quoted strings are allowed.
- Escape sequences include `\\`, `\"`, `\n`, `\r`, `\t`, `\0`, and `\u{...}`.
- Unknown escapes are errors.
- Multiline strings are allowed without a separate literal form.

String storage is UTF-8. String length and indexing are defined in Unicode scalar values rather than UTF-8 bytes. Indexing by scalar value is `O(n)`.

## Active Keywords

The active keyword set is currently:

- Flow: `if`, `else`, `match`, `for`, `while`, `break`, `continue`, `return`
- Declarations: `let`, `const`, `fn`, `inline`, `extern`, `struct`, `type`, `trait`, `enum`, `pub`, `import`
- Other language keywords: `mut`, `in`, `from`, `as`, `where`, `satisfies`, `Self`
- Special types: `void`, `null`
- Literal keywords: `true`, `false`, `NaN`, `Infinity`

## Operators

### Postfix and Primary Forms

Primary expression forms include:

- literals
- identifiers
- parenthesized expressions: `(expr)`
- tuple literals: `(a, b, c)`
- array literals: `[a, b, c]`
- struct literals: `User { id: 1, name: "a" }`
- anonymous functions: `fn(x: i32) -> i32 { ... }`

Postfix expression forms include:

- call: `f(x, y)`
- member access: `value.name`
- method call: `value.method(x)`
- indexing: `value[index]`
- explicit type arguments on call-like forms: `f<i32>(1)`

Calls, indexing, and member access bind tighter than every unary or binary operator and group left to right.

### Unary and Logical

- unary `-` numeric negation
- `!` logical NOT
- `&&`, `||` logical AND/OR

### Arithmetic and Bitwise

- `*`, `/`, `%`
- `+`, `-`
- `<<`, `>>`
- `&`
- `^`
- `|`

Integer arithmetic uses the overflow rules defined above. Floating-point arithmetic follows IEEE-754 for the target width.

Concretely:

- `+`, `-`, `*`, and unary `-` wrap at runtime and do not panic on overflow.
- `/` and `%` panic only on zero divisors.
- `<<` and `>>` panic only on invalid runtime shift counts.
- Overflow is therefore distinct from invalid operations: overflow wraps, while invalid division/modulo/shift operations panic.

`+` also concatenates strings. When one side is `string`, the other side must be explicitly convertible or already typed as `string`; there is no implicit numeric-to-string conversion.

### Comparison and Equality

- `<`, `<=`, `>`, `>=`
- `==`, `!=`

Vek provides:

- `==` and `!=` for value equality

`!=` is defined as the logical negation of `==`.

### Casts

Vek supports explicit casts only, using `as`.

```vek
let x: i32 = 42;
let y: f32 = x as f32;
```

There are no implicit casts. Mixed-type operations require explicit conversion.

Supported built-in cast domains include the core primitive types. User-defined type-to-type/custom casts are not supported.

### Assignment

`=` assigns to an assignable place expression.

Compound assignment operators are statement-form shorthand:

- `+=`
- `-=`
- `*=`
- `/=`
- `%=`
- `<<=`
- `>>=`
- `&=`
- `^=`
- `|=`

`place op= value` has the same type rules and result as `place = place op value`,
except the place expression is evaluated only once. For example, `xs[i] += 1`
evaluates `xs` and `i` once, reads the current element, computes the addition,
and writes the result back.

Assignable place expressions are:

- a mutable local binding declared with `let`
- a `mut` parameter
- a field reached through a mutable receiver
- an index reached through a mutable receiver

Assignment is a statement form, not an expression form.

### Precedence and Associativity

From tightest to loosest:

1. postfix forms: call, indexing, member access
2. unary prefix: `!`, unary `-`
3. multiplicative: `*`, `/`, `%`
4. additive: `+`, `-`
5. shifts: `<<`, `>>`
6. bitwise AND: `&`
7. bitwise XOR: `^`
8. bitwise OR: `|`
9. comparisons: `<`, `<=`, `>`, `>=`
10. equality: `==`, `!=`
11. logical AND: `&&`
12. logical OR: `||`
13. cast: `as`

Binary operators of the same precedence group associate left to right. `as` associates left to right and has the loosest precedence among expression operators, so `a + b as f32` parses as `(a + b) as f32`.

### Evaluation Order

- Expression operands are evaluated left to right.
- Function call arguments are evaluated left to right after the callee expression.
- Member access evaluates the receiver before selecting the member.
- Indexing evaluates the receiver, then the index.
- `&&` and `||` short-circuit; the right-hand side is evaluated only when needed.
- Only the selected branch of `if`, `match`, and conditional loop bodies is evaluated.

### Conditions

Conditions in `if` and `while` must be typed as `bool`.

`&&` and `||` require `bool` operands and produce `bool`.

### Operator Overloading

Only equality is customizable for user-defined types in the current design.

- `==` may have custom/user-defined behavior through `Equal<T>`
- all other operators are built-in only and may not be overloaded
