# Errors and retries in the SDK

Every helper throws a `MapxError` when the API answers with an error or the network fails. It mirrors the `application/problem+json` document of the [error catalog](/api/errors/).

| Property | Meaning |
|---|---|
| `status` | HTTP status, `0` for a network error |
| `code` | Stable code such as `invalid-parameter`, `not-found`, `rate-limited` |
| `param` | The offending parameter, if known |
| `allowed` | Accepted values for enumerations |
| `detail` | Explanation for this request |
| `retryAfter` | Seconds to wait on `429` |

```js
try {
  await mapx.geocode("Wien", { types: ["village"] });
} catch (e) {
  if (e instanceof mapx.MapxError && e.code === "invalid-parameter") {
    console.log(`${e.param} must be one of ${e.allowed.join(", ")}`);
  } else throw e;
}
```

Missing or rejected tokens arrive as `401` (`missing-token`, `invalid-token`, `token-expired`) or `403` (`token-blocked`, `origin-not-allowed`, …), see [Access and limits](/start/access-and-limits/). Do not retry them.

Branch on `code`, never on the text. Send `Accept-Language: de` through your own `fetch` if you need German messages; the helpers use the browser default.

## Retry with backoff

Retry only `429`, `502`, `503` and network errors. Never retry `400`, `404` or `422`: the same request fails the same way.

```js
async function withRetry(fn, tries = 3) {
  for (let i = 0; ; i++) {
    try { return await fn(); }
    catch (e) {
      const retryable = e instanceof mapx.MapxError && [0, 429, 502, 503].includes(e.status);
      if (!retryable || i === tries - 1) throw e;
      await new Promise((r) => setTimeout(r, (e.retryAfter ?? 2 ** i) * 1000));
    }
  }
}
const m = await withRetry(() => mapx.matrix("driving", [[16.3725, 48.2085], [15.3294, 48.2272]]));
```

## Cancel stale requests

Type-ahead fields should cancel the previous request. A cancelled request throws the browser's `AbortError`, not a `MapxError`:

```js
let ac;
input.oninput = async () => {
  ac?.abort(); ac = new AbortController();
  try { show(await mapx.searchSuggest(input.value, { signal: ac.signal })); }
  catch (e) { if (e.name !== "AbortError") throw e; }
};
```

## Next

- [Errors and limits](/api/errors/) with the rate-limit zones
- [SDK helpers](/guides/sdk/helpers/)

