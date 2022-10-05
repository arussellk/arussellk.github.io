---
title: Nominal Types
---

```
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
