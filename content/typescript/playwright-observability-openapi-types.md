---
title: Using Playwright Observability for API Tests with Types from openapi
---

[Full working example.](https://github.com/arussellk/playwright-openapi)

Suppose you have a strongly-typed fetch api layer for your application
and you want a suite of api tests.
Playwright has good test discoverability and observability
(e.g., test timing and request/response body),
but directly calling your typed `fetch` within Playwright's
Node.js process is _not_ recorded by Playwright's network tab.
If you use [`request` as suggested by Playwright's docs](https://playwright.dev/docs/api-testing),
the request is captured by playwright, but you will not get the benefit of a strongly-typed
api layer.

With a little bit of code, you can have a strongly-typed api and record the request/response
in Playwright. The key is to realize that Typescript needs to interact with your types while
Playwright needs to issue the request.

Key ideas:
1. Create an openapi client with a custom fetch implementation
2. Make the custom fetch implementation use Playwright

[The important code](https://github.com/arussellk/playwright-openapi/blob/99a141d95f2b0f0e44e508b5bc7b594546054536/tests/example.spec.ts#L14)
make a series of transformations like this:
* TypeScript and developers see request types from openapi's fetch
    * Transform fetch to Playwright request model
        * Playwright makes the request, records activity, and gets a response
    * Transform Playwright response to fetch response
* TypeScript and developers see response types from openapi's fetch

<img alt="Screenshot of Playwright with captured fetch request/response" src="/images/playwright-openapi.png" width="100%" />


![Screenshot of Playwright with captured fetch request/response](/images/playwright-openapi.png)
