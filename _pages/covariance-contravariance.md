---
layout: default
title: Covariance and Contravariance
---

The concept of covariance and contravariance can be difficult to grasp or remember. This is how I remember it.

## Types

Here is a type that has a name and age:

```
{ name, age }
```

Here is a type that has an x, y, and z:

```
{ x, y, z }
```

## Subtypes

More complex types are subtypes of simpler types. This means that some type B is a subtype of A:

```
B < A
```

Or, using the object notation from before:

```
{ x, y, z } < { x, y } < { x } < { }
```

## Subtype example

Time for code!

```
var in: { x, y }
var out: { x, y } = foo(in)
print(out.x)
print(out.y)
```

We start with a variable `in`, which is a type that has an `x` and a `y`. We pass `in` into `foo` and get some result that happens to have the same type. Then we do some stuff with `out`.

`foo` might look like this:

```
fun foo(in: { x, y }):
  print(in.x)
  print(in.y)
  return { in.x+1, in.y+1 }
```

`foo` demands that it is given something with `x` and `y` and promises to return something else that has `x` and `y`.

Can we change the type of `in`? Or the return type of `foo`? Sure! You can always pass in a subtype or return a subtype.

```
fun foo(in: { x, y }):
  print(in.x)
  print(in.y)
  return { in.x+1, in.y+1, a }

var in: { x, y, z }
var out: { x, y } = foo(in)
print(out.x)
print(out.y)
```

## Covariance/Contravariance Example

Covariance and contravariance are properties of a function. They describe what you can and can't do if you want to substitute a function.

Will this work?

```
fun foo(in: { x, y }):
  print(in.x)
  print(in.y)
  return { in.x+1, in.y+1 }

fun bar(in: { x, y, z }):
  print(in.x)
  print(in.y)
  print(in.z)
  return { in.x+1, in.y+1 }

var in: { x, y }
// var out: { x, y } = foo(in)
var out: { x, y } = bar(in)
print(out.x)
print(out.y)
```
