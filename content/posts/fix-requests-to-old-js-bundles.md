---
title: Fixing requests to old JS bundles
---

In my frontend application, I sometimes got `SyntaxError: Unexpected token '<'`
errors while loading the homepage. How? Why only sometimes?

I looked in Datadog logs and could see that the homepage html response looked normal,
but some of the following requests to JS files were all the same byte length.
(e.g., `https://example.com/` looked fine,
but `https://example.com/_build/abc123.js`
and `https://example.com/_build/def456.js`
were suspiciously the same byte length.)
I know that for many frontend single-page app (SPA) systems, if an HTTP request path
does not match a file path, the server will respond with the homepage HTML so that the JS
SPA code can handle the rest of the app setup and routing.
Knowing this was enough for me to put together a theory of what was happening and test it.

---

Suppose the following:
- There are multiple k8s pods hosting this frontend site.
- The `abc123` portion of `https://example.com/_build/abc123.js` changes for each build.
- The site is deployed. Specifically, k8s pods with the old build `A` are stopping and new pods with
  build `B` are starting.

My theory was that this sequence of events happens:
1. The browser fetches the homepage from build `A`
2. Some version `A` pods stop, some version `B` pods start
3. The browser makes `N` follow-up requests for `/_build/A_1.js`, `/_build/A_2.js`, ..., `/_build/A_N.js`.
4. The request for `/_build/A_n.js` reaches a pod hosting version `B`. The pod checks its
   filesystem, does not find `/_build/A_n.js` and replies with version `B` homepage HTML.
5. The browser page crashes with `SyntaxError: Unexpected token '<'`.

---

I was pretty confident this was the problem, but I wanted to put together some code to exercise the
issue so that we could easily check whether a proposed fix worked well. I threw together a script
to exercise the root issue: requests to JS file references seen in version `A` go to a pod that has
content from version `B` and get an HTML response.

```js
while (true) {
  await collectAndReport();

  const barrangeDelayMs = 300;
  await sleep(barrageDelayMs);
}

function collectAndReport() {
  const bodyText = await getHomePage();
  const timestamp = new Date().toISOString();

  const scripts = Array.from(bodyText.matchAll(/<script src="(.*?)"/g));
  const srcPaths = matchs.map(([match, captureGroup]) => captureGroup);

  const fetches = srcPaths.map(async (p) => {
    const resp = await fetch('https://example.com' + p);
    const text = await resp.text();
    return [p, text];
  });
  const srcPathAndTexts = (await Promise.allSettled(fetches)).map(x => x.value);

  const anyHtml = srcPathAndTexts.some(([_, text]) => text.startsWith('<'));

  const preamble =
    `Barrage started at ${timestamp}: ${anyHtml ? 'has some HTML': 'is all JS'}`;
  console.log(preamble);
  if (anyHtml) {
    for (const [p, text] of srcPathAndTexts) {
      const isHtml = text.startsWith('<');
      console.log('  ', p, 'is HTML?', isHtml);
    }
  }
}

async function sleep(ms) {
  return new Promise((resolve) => {
    setTimeout(resolve, ms);
  });
}
```

Running this script during a deployment would give output like this.
Before and after a deployment, all of the script responses were JS.
Just as pods started to switch from version `A` to `B`, we would see a few different cases:
1. The initial HTML was version `A`, but at least one JS routed to a pod with version `B`
2. The initial HTML was version `B`, but at least one JS routed to a pod with version `A`
3. The initial HTML and all JS routed to pod(s) with the same version

```
Barrage started at <t1>: is all JS
Barrage started at <t2>: is all JS
Barrage started at <t3>: has some HTML
  /_build/A_1.js is HTML? true
  /_build/A_2.js is HTML? false
  /_build/A_3.js is HTML? false
Barrage started at <t4>: is all JS
Barrage started at <t5>: has some HTML
  /_build/B_1.js is HTML? false
  /_build/B_2.js is HTML? true
  /_build/B_3.js is HTML? false
Barrage started at <t6>: is all JS
Barrage started at <t7>: is all JS
```

Using the script, we found that site visitors could encounter a page crash for about 90s during a
deployment (the deploy time of 30s times three replicas deployed sequentially).
The fix we chose was to introduce a kind of sticky session to encourage the system to route requests
to a pod that has the right version of content.
[ingress-nginx session affinity](https://github.com/kubernetes/ingress-nginx/blob/dbb11b92ddb77bee9c35e462479129354c84f939/docs/examples/affinity/cookie/README.md?plain=1#L11)

After deploying the fix, the script demonstrated that users could still encoutner a page crash in a
short time window with probability `p <= 1/num_pods`. The page crash could still happen if
nginx assigned a session affinity cookie for a pod immediately before killing the pod.
The short time window was the network round-trip from nginx to the browser and back.
Blue/green deployments would eliminate this issue by letting the system know it was _about_ to kill
a pod and be smarter about assigning sticky sessions.

---

After making this fix, I found that users could also run into a related issue due to trying to load
per-route JS bundles well after a deploy has concluded
(aka chunking or [lazy loading](https://nextjs.org/docs/pages/guides/lazy-loading).
The root problem of making a request for content for a "stale" script that can only be served by a
specific version of pods is the same problem as before.
Hotswapping script references or keeping the previous application version scripts available in a new
deployment could alleviate this issue, but there are tradeoffs and complexity with each idea.
