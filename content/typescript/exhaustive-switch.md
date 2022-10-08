---
title: Exhaustive Switch
---

```ts
type Foo = 'a' | 'b';

const isSpecial = (foo: Foo): boolean => {
    switch (foo) {
        case 'a':
            return true;
    }

    const exhaustiveCheck: 'exhaustiveCheck' = foo;
    throw Error(`unhandled Foo ${exhaustiveCheck}`);
};
```
