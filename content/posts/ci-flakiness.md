---
title: Stabilize CI Network Flakiness
---

We run integration tests in our CI/CD pipeline.
For a while, it was common for a C# integration test to fail with this message:
```
System.Net.Http.HttpRequestException: An error occurred while sending the request.
 ---> System.IO.IOException: The response ended prematurely.
```

The network failure was transient and you could usually re-run the tests to get them to pass.
As the number of tests grew, the chances of hitting a network hiccup went up, and eventually
most programmers were clicking to re-run tests on most PRs - sometimes more than once!

I decided to address the flakiness so that programmers could spend less time waiting for pipelines
to re-run.
Adding retries to the integration test HTTP client for network hiccups seemed like the right solution,
so I wrote an HTTP server to intentionally cause `The response ended prematurely.` errors
and ensure that my fix for the integration test runner worked.

This is the script I wrote:
```
const http = require('http');

let counter = 0;

const server = http.createServer((req, res) => {
  counter++;
  const action = counter % 10 === 0 ? 'end' : 'destroy';
  console.log(`connection ${counter} -> ${action}`);

  res.write('any response');
  if (action === 'end') {
    res.end();
  } else {
    // abruptly stop the connection, trigger responses ended prematurely
    res.socket.destroy();
  }
});

const port = 8080;
server.listen(port);
```

If you built your own `HttpClient` like we did for integration tests
[this documentation demonstrates how to add retries to your client](https://learn.microsoft.com/en-us/dotnet/fundamentals/networking/http/httpclient-guidelines#resilience-with-static-clients).
[This documentation shows how to add resiliency pipelines to DI for other uses in a .NET application](https://learn.microsoft.com/en-us/dotnet/core/resilience/).

Prior to implementing the resiliency pipeline, we had CI failures on the majority of PRs.
Afterwards, `The response ended prematurely` failures went to nearly zero
and only a couple of PRs per week across all programmers had to be rerun for transient network
issues.
