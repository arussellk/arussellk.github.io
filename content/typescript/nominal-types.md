---
title: Nominal Types
---

TypeScript uses structural subtyping.
This means that any type that fits the shape of a different type is compatible.
When using nominal types, on the other hand, two types are considered distinct and incompatible types
even if they have the same underlying representation.

Here is a reasonably readable and usable way to have a form of nominal types in TypeScript:

```ts
type FooId = string & { __tag: 'FooId' }
type BarId = string & { __tag: 'BarId' }

const isFooId = (fooId: string): fooId is FooId => /^[0-9a-f]{8}$/.test(fooId)
const isBarId = (barId: string): barId is BarId => barId.startsWith('BAR_')

const process = (fooId: FooId, barId: BarId) => { /* ... */ }

process('asdf', 'fdsa') // Error

const maybeFooId = 'deadbeef'
const maybeBarId = 'BAR_bar'
if (isFooId(maybeFooId) && isBarId(maybeBarId)) {
  process(maybeBarId, maybeFooId) // Error

  process(maybeFooId, maybeBarId)
}
```
