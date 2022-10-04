---
title: String Union Extras
---

```
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
