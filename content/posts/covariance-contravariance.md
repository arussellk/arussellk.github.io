---
title: Covariance and Contravariance
---

Covariance and contravariance are properties of code substitution;
they describe what you can and can't do if you want to replace some code
(e.g., a function call)
with some other code
(e.g., a different function call)
in a type-safe way.

I initially found the concept of covariance and contravariance to be difficult
to grasp.
This post starts with types and slowly builds to a practical example of
covariance and contravariance in a way that will hopefully help you understand
it when needed.

## Types

In TypeScript, you could define and use types to describe objects like this:
```ts
// Define the type Foo and use it by name:
type Foo = { name: string, age: number };
const foo: Foo = { name: 'Geralt', age: 98 };

// Use the type directly:
const bar: { a: number, b: number } = { a: 1, b: 2 };
console.log(bar.a); // 1
console.log(bar.b); // 2
```

I think the concept of variance can be more simply and intuitively understood
with just the _existence_ of fields, so let's consider a type system that
contains only object types, and say that the object types can either have a
field or not, and there is no regard for whether the field would hold a string,
boolean, number, etc.
As examples,
this is a type that has the fields name and age: `{ name, age }`,
and this is a type that has the fields a, b, and c: `{ a, b, c }`.

## Subtypes and Supertypes

Subtypes extend other types.
In our simplified type system that only considers the existence of a field,
subtypes have _more_ fields than the simpler types they extend.
Restated, the set of fields of a subtype are a superset of the fields of the
type it extends.

The symbol `<:` means "is a subtype of",
so `S <: T` means that some type `S` is a subtype of `T`.
Using the field syntax from before, this is a chain of subtypes:
```
{ a, b, c } <: { a, b } <: { a } <: { }
```

Going the other way, the symbol `:>` means "is a supertype of",
so `T :> S` means that some type `T` is a supertype of `S`.
This is a chain of supertypes:
```
{ } :> { d } :> { d, e } :> { d, e, f }
```

## Base Code

Let's define a function `foo` that takes an input with `a` and `b` fields
and returns an output with an `x` field.
This code constructs some input for `foo` with the `in` variable,
and collect `foo`s output in the `out` variable.

```
function foo(arg: { a, b }) -> { x }
  return { x: arg.a + arg.b }

var in: { a, b } = { a: 1, b: 2 }
var out: { x } = foo(in)
print(out.x) // 3
```

We are going to modify the above code in several ways to explore when changing
a function's parameter type and return type is valid and invalid.
:white_check_mark: indicates that the modification in the section produces a
valid program.
:x: indicates that the modification produces an invalid program.

### :white_check_mark: The Caller Subtypes the Argument

Let's add another field `c` to the `in` variable before passing it into `foo`,
but keep the implementation of `foo` the same.
This program is still valid because it is valid for any caller of `foo` to pass
in any subtype of `{ a, b }`; after all, `foo` only needs to be able to access
the `a` and `b` fields of the parameter.

```
function foo(arg: { a, b }) -> { x }
  return { x: arg.a + arg.b }

var in: { a, b, c } = { a: 1, b: 2, c: 3 }
var out: { x } = foo(in)
print(out.x) // 3
```

### :white_check_mark: The Function Subtypes the Return Type

This time, let's change `foo`s implementation and add another field `y` to the
return type, but keep the calling code the same.
The code that comes after the call to `foo` only attempts to access the `x`
field, so it ok for `foo` to return any subtype of `{ x }` and this program
remains valid.

```
function foo(arg: { a, b }) -> { x, y }
  return { x: arg.a, y: arg.b }

var in: { a, b } = { a: 1, b: 2 }
var out: { x } = foo(in)
print(out.x) // 3
```

### :x: The Function Subtypes the Argument Type

Let's modify `foo` to require its parameter to have a `c` field,
but keep the calling site the same.
This program is no longer valid because the input to `foo` does not have a `c`.

```
function foo(arg: { a, b, c }) -> { x }
  return { x: arg.a + arg.b + arg.c }

var in: { a, b } = { a: 1, b: 2 }
var out: { x } = foo(in) // Error: 'in' is missing the 'c' field.
print(out.x)
```

### :white_check_mark: The Function Supertypes the Argument Type

This is the same case as when
[the caller subtypes the argument](#-the-caller-subtypes-the-argument),
but from the perspective of replacing the function `foo`.

```
function foo(arg: { a }) -> { x }
  return { x: arg.a }

var in: { a, b } = { a: 1, b: 2 }
var out: { x } = foo(in)
print(out.x) // 1
```

### :x: The Function Supertypes the Return Type

Let's modify `foo` to return the type `{ }` (a supertype of `{ x }`),
but keep the calling site the same.
This program is no longer valid because the output of `foo` does not have an
`x`.

```
function foo(arg: { a, b }) -> { }
  return { }

var in: { a, b } = { a: 1, b: 2 }
var out: { x } = foo(in) // Error: no 'x' on return type of 'foo'.
print(out.x)
```

## TypeScript Example

The following TypeScript is an implementation of the valid and invalid function
substitution that we've looked at so far.

```ts
type A   = { a };
type AB  = { a, b };
type ABC = { a, b, c };
type X   = { x };
type XY  = { x, y };

interface Type {
    foo: (arg: AB) => X
}

interface ValidSubType extends Type {
    // Imposes fewer restrictions on the input it needs and guarantees to
    // return the fields you would expect from calling Type.foo.
    foo: (arg: A) => XY
}

interface InvalidSubTypeBadInput extends Type {
    // Tries to impose a new requirement on calls to foo (that the input will
    // have a "c" field).
    foo: (arg: ABC) => X
}

interface InvalidSubTypeBadOutput extends Type {
    // Does not fulfill the promise made by "foo" (that the return type will
    // have an "x" field).
    foo: (arg: AB) => { }
}
```

If you view the code in the
[TypeScript Playground](https://www.typescriptlang.org/play?noImplicitAny=false#code/C4TwDgpgBAgl8F4oG8oEMoF8DcAoUksAQoiugDRQBGWeB0MRAwlEqmpVZQMa37jQAGvFZkAHn3pRBATVKoxlEH1wBLAHbAIAJwBmabtAAqAlLhFRdAeysAuKAAo02gOb3GASlYA+abky4apo6+oZQAGpoADaqACYAygCuVCaEEGJa6rEAzlCp0MjmIgD0xVAAkgC2YFbZELm6EADuOlDa9cDaqtzAqlbquf1QwAAW0BpgicBQqtPqEBA56FlQLonOaMH1w1ZF8KVtEMCJ2urDY5aqEFFLIFaJUE33N1DpkD2W2laVUNzRMeoXHkBAA6axWEF7Sw2exOVzuLwIXyyfyBDRaPQGaDldQAN2icSSKQERDQsRxk2m6UyS3yZgsByMXW2wCsM2qtWgGHmTUOAEdEqp2pUIJooEM-lEorlWdC2Q5RmhpqNxupKY9VFKoQcRmhcVyoAAibiGy7XWIeSEWcGw5xuYhMRHI1FBDGhbF4gkJZL5UmxADyU3V1NFtNMhQZZQAIlZtuorNNdIkorpNVFztAwF9Kqo6lBKmToFRlIbwaaFbrlRd2sdTsNTE009qyrr9csjWJTanzZaoTbHHb3EQnWQAgEgA),
you will see the errors
`Interface 'InvalidSubTypeBadInput' incorrectly extends interface 'Type'.`
and
`Interface 'InvalidSubTypeBadOutput' incorrectly extends interface 'Type'.`
along with additional information on what is incorrect.

Let's focus on the `ValidSubType` and connect it to covariance and
contravariance:
- `ValidSubType` is a subtype of `Type`,
  and `ValidSubType`'s return type `XY` is a subtype of `Type`'s return type
  `X`.
  Since the type relation of `ValidSubType <: Type` is the same direction
  as `XY <: X`, a function return type is said to be covariant.
- `ValidSubType` is a subtype of `Type`,
  and `ValidSubType`'s argument type `A` is a supertype of `Type`'s argument
  type `AB`.
  Since the type relation of `ValidSubType <: Type` is the opposite direction
  as `AB :> A`, a function argument type is said to be contravariant.

In this post I have focused on the idea of substitution for a function call,
but the property of substitution applies to arrays and generics too.
The
[Wikipedia article on covariance and contravariance](https://en.wikipedia.org/wiki/Covariance_and_contravariance_(computer_science))
has more information.
