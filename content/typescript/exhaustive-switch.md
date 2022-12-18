---
title: Exhaustive Switch
---

Sometimes it is useful to have a compile-time check to make sure that all cases of a union are
represented in code.

If the code in question is at a function boundary with a return type, you can get a compile-time
check, but the error message is rather unclear.
We can do better than this:
```ts
type Foo = 'a' | 'b' | 'c';

const isSpecial = (foo: Foo): boolean => {
                           // ~~~~~~~ Function lacks ending return statement and return type does not include 'undefined'.
    switch (foo) {
        case 'a':
            console.log('got a');
            return true;
        case 'b':
            console.log('got b');
            return false;
    }
};

```

If the code in question does not return a value, you will not get a compile-time error.
This is bad because a future developer can unintentionally miss that they should modify this code
when adding a new `Foo`:
```ts
type Foo = 'a' | 'b' | 'c';

const isSpecial = (foo: Foo): void => {
    switch (foo) {
        case 'a':
            console.log('got a');
            return;
        case 'b':
            console.log('got b');
            return;
    }
};
```

To create a compile-time error so that future developers are made aware of your code,
you can use the `never` type or a string literal.
I prefer the string literal version because the error message from the compiler is more clear.
```ts
type Foo = 'a' | 'b' | 'c';

const isSpecial = (foo: Foo): void => {
    switch (foo) {
        case 'a':
            console.log('got a');
            return;
        case 'b':
            console.log('got b');
            return;
        default:
            const exhaustiveCheckNever: never = foo;
            //    ~~~~~~~~~~~~~~~~~~~~ Type 'string' is not assignable to type 'never'.
            const exhaustiveCheckString: 'exhaustiveCheckString' = foo;
            //    ~~~~~~~~~~~~~~~~~~~~ Type '"c"' is not assignable to type '"exhaustiveCheckString"'.

            throw Error(`unhandled Foo ${exhaustiveCheckNever} ${exhaustiveCheckString}`);
    }
};
```
