# Fehler und Wiederholungen im SDK

MapxError trägt code, status, param und erlaubte Werte aus der problem+json-Antwort; nur wiederholen, was sich lohnt.

Jeder Helfer wirft einen `MapxError`, wenn die API einen Fehler meldet oder das Netz ausfällt. Er bildet das `application/problem+json`-Dokument des [Fehlerkatalogs](/de/api/errors/) ab.

| Eigenschaft | Bedeutung |
|---|---|
| `status` | HTTP-Status, `0` bei Netzwerkfehler |
| `code` | Stabiler Code wie `invalid-parameter`, `not-found`, `rate-limited` |
| `param` | Der fehlerhafte Parameter, falls bekannt |
| `allowed` | Erlaubte Werte bei Aufzählungen |
| `detail` | Erklärung zu dieser Anfrage |
| `retryAfter` | Wartezeit in Sekunden bei `429` |

```js
try {
  await mapx.geocode("Wien", { types: ["village"] });
} catch (e) {
  if (e instanceof mapx.MapxError && e.code === "invalid-parameter") {
    console.log(`${e.param} muss eines von ${e.allowed.join(", ")} sein`);
  } else throw e;
}
```

Auf `code` verzweigen, nie auf den Text. Deutsche Meldungen gibt es mit `Accept-Language: de` über einen eigenen `fetch`; die Helfer verwenden den Browser-Standard.

## Wiederholen mit Backoff

Nur `429`, `502`, `503` und Netzwerkfehler wiederholen. `400`, `404` und `422` nie wiederholen: dieselbe Anfrage scheitert genauso.

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

## Veraltete Anfragen abbrechen

Eingabefelder mit Vorschlägen sollten die vorige Anfrage abbrechen. Eine abgebrochene Anfrage wirft den `AbortError` des Browsers, keinen `MapxError`:

```js
let ac;
input.oninput = async () => {
  ac?.abort(); ac = new AbortController();
  try { show(await mapx.searchSuggest(input.value, { signal: ac.signal })); }
  catch (e) { if (e.name !== "AbortError") throw e; }
};
```

## Weiter

- [Fehler und Limits](/de/api/errors/) mit den Rate-Limit-Zonen
- [SDK-Helfer](/de/guides/sdk/helpers/)