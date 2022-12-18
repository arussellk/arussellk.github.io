---
title: String Union Runtime Validation and Iteration
---

TypeScript types are removed by the compiler while producing JavaScript.
This is fine much of the time within your system, but sometimes you need to validate user input or
requests from external systems.
One way to have both runtime validation and compile time types is to derive a string union type from
an array.

```ts
const foos = [
  'bar',
  'baz',
] as const
type Foo = typeof foos[number]

const isFoo = (x: string): x is Foo => foos.includes(x as Foo)

for (const foo of foos) {
  console.log(foo)
}
```
