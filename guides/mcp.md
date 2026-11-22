# MCP server for Austrian geodata

Source: https://docs.mapx.at/guides/mcp/

> Connect claude.ai, ChatGPT, Claude Code or Cursor to mapx: 21 read-only tools for search, routing, transit, elevation, traffic and weather in Austria.

The mapx MCP server gives an AI assistant 21 tools for Austrian geodata: address search, routes, public transport, heights, traffic and weather. It uses the [Model Context Protocol](https://modelcontextprotocol.io/) (MCP), so any MCP client can connect. All tools are read-only, the data covers Austria only and comes from official open data, and every answer names its source.

| Property | Value |
|---|---|
| Endpoint | `https://mcp.mapx.at/mcp` |
| Transport | Streamable HTTP, `POST` only, stateless |
| Protocol versions | `2026-07-28` and older ones from `2024-11-05` |
| Authentication | Either sign in with your mapx account (OAuth 2.1 with PKCE, just enter the address) or a token of type "AI assistant (MCP)", prefix `mx_mcp_`, in the header `Authorization: Bearer …` |

## Set up

There are two ways. Signing in with your account is the simpler one: you only enter the address and do not copy a token. The token way works with any MCP client.

### Sign in with your account

1. Add `https://mcp.mapx.at/mcp` as an MCP server in your application (see below).
2. The assistant sends you to account.mapx.at. Sign in with your e-mail address; without an account you create one after entering the code.
3. The consent page shows the application, the redirect target, your account and what the access allows: read-only tools only, calls count towards your usage, no access to your account or your other tokens. Confirm the connection.
4. Ask a question that needs Austrian data, for example "How do I get by bike from Melk station to Melk Abbey, and how long does it take?".

The connection then appears at [account.mapx.at](https://account.mapx.at/) → **Tokens** as "connected via &lt;host&gt;". Each application can hold up to five connections at the same time, for example browser, desktop and phone. The connection does not expire on its own.

These applications are approved for signing in with your account:

- **claude.ai** (web, desktop, mobile): under **Connectors** in the settings, add a custom connector and enter the address. The exact menu path depends on your plan and changes over time; it is described in [Anthropic's help](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp). On Team and Enterprise plans an owner adds the connector first.
- **Claude Code:** `claude mcp add --transport http mapx https://mcp.mapx.at/mcp`, then enter `/mcp` in Claude Code and start the sign-in for mapx.
- **ChatGPT:** connectors or apps with your own MCP address, available depending on plan and settings (sometimes only in developer mode). Steps and availability are in [OpenAI's help](https://help.openai.com/en/articles/12584461-developer-mode-and-mcp-apps-in-chatgpt).

Signing in with your account is limited to known applications. For all other clients use the token way; further applications can be enabled on request (support@mapx.at).

### With a token

1. Create a token at [account.mapx.at/tokens/new?kind=mcp](https://account.mapx.at/tokens/new?kind=mcp). Choose the type **AI assistant (MCP)**, give it a name and optionally an expiry date. The token is shown once; copy it then.
2. Add the server to your client (see below) and paste the token where the examples say `YOUR_MCP_TOKEN`.
3. Ask a question that needs Austrian data.

The token works only at the MCP server. It has no domain or IP binding, can have an expiry date, and you can block or rotate it at any time in your account.

#### Clients for the token way

These clients take a fixed header, so the token goes straight into the configuration.

**Claude Code**

```bash
claude mcp add --transport http mapx https://mcp.mapx.at/mcp --header "Authorization: Bearer YOUR_MCP_TOKEN"
```

**Cursor**

In `mcp.json`:

```json
{
  "mcpServers": {
    "mapx": {
      "url": "https://mcp.mapx.at/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

**VS Code**

In `.vscode/mcp.json`:

```json
{
  "servers": {
    "mapx": {
      "type": "http",
      "url": "https://mcp.mapx.at/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

**Gemini CLI**

In `settings.json`:

```json
{
  "mcpServers": {
    "mapx": {
      "httpUrl": "https://mcp.mapx.at/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

**Anthropic API**

The [MCP connector](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector) of the Messages API is in beta; the beta header and model name below are those of its documentation at the time of writing, so check them there.

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2026-10-21" \
  -H "anthropic-beta: mcp-client-2025-11-20" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 1000,
    "messages": [{"role": "user", "content": "How long does a truck take from Vienna to Linz?"}],
    "mcp_servers": [
      { "type": "url", "url": "https://mcp.mapx.at/mcp", "name": "mapx", "authorization_token": "YOUR_MCP_TOKEN" }
    ],
    "tools": [ { "type": "mcp_toolset", "mcp_server_name": "mapx" } ]
  }'
```

**OpenAI API**

The [remote MCP tool](https://developers.openai.com/api/docs/guides/tools-connectors-mcp) of the Responses API. OpenAI asks for approval before data is sent to the server unless you set `require_approval`. The `authorization` value is not stored, so send it with every request.

```bash
curl https://api.openai.com/v1/responses \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "YOUR_MODEL",
    "input": "How long does a truck take from Vienna to Linz?",
    "tools": [
      { "type": "mcp", "server_label": "mapx", "server_url": "https://mcp.mapx.at/mcp", "authorization": "YOUR_MCP_TOKEN" }
    ]
  }'
```

## Disconnect

At [account.mapx.at](https://account.mapx.at/) → **Tokens** every connection made through your account is listed as "connected via &lt;host&gt;". **Disconnect** ends it; **Block** rejects it immediately and can be undone. The assistant then gets `401` and asks you to sign in again when needed.

## For developers

The MCP server follows MCP authorization (OAuth 2.1):

- Protected resource metadata: `https://mcp.mapx.at/.well-known/oauth-protected-resource/mcp`
- Authorization server: `https://account.mapx.at` (metadata at `/.well-known/oauth-authorization-server`)
- Authorization code with PKCE (`S256`); public clients identify themselves through Client ID Metadata Documents, there is no dynamic registration.
- Scope `mcp`. The access token does not expire and there is no refresh token; revocation happens in the account.

Limits, counting and tools are the same for both ways.

## Tools

The assistant picks the tools itself. Coordinates are `{lon, lat}` in WGS84; `geocode` turns a place name into coordinates for the other tools.

| Group | Tool | Purpose | Main inputs |
|---|---|---|---|
| Search | `geocode` | Coordinates for an address, street, place, postcode, POI or road kilometer | `query`, `near`, `types`, `limit` |
| Search | `reverse_geocode` | Nearest address or place for a coordinate | `lon`, `lat`, `radius_m`, `types` |
| Search | `validate_address` | Check an address against the official register and normalize it | `address` or `street`, `housenumber`, `postcode`, `place` |
| Search | `search_places` | Points of interest by category or name | `category`, `query`, `near`, `near_place`, `radius_m`, `limit` |
| Search | `list_place_categories` | Category ids usable in `search_places` | none |
| Routing | `route` | Route by car, truck, bike or on foot with distance, time and instructions | `profile`, `waypoints` (2 to 25), `exclude`, `truck` |
| Routing | `isochrone` | Area reachable within given minutes or meters | `profile`, `lon`, `lat`, `minutes`, `meters`, `depart_at` |
| Routing | `travel_matrix` | Travel times between 2 to 25 points | `profile`, `points`, `sources`, `destinations` |
| Routing | `optimize_stops` | Fastest order of 2 to 25 stops | `profile`, `stops`, `roundtrip`, `start`, `end` |
| Public transport | `transit_route` | Connections between two points from the scheduled timetable | `from`, `to`, `depart_at`, `arrive_by`, `access`, `connections` |
| Public transport | `departures` | Next scheduled departures at a stop or around a point | `stop_id` or `near`, `time`, `limit` |
| Public transport | `find_stops` | Stops near a point or in a box, with `stop_id` | `near`, `radius_m`, `bbox`, `limit` |
| Elevation | `elevation` | Height above sea level (terrain or surface model) | `points`, `model` |
| Elevation | `elevation_profile` | Ascent, descent, lowest and highest point along a line | `path` (2 to 200 points), `samples`, `model` |
| Traffic | `traffic_incidents` | Accidents, congestion, roadworks and closures | `bbox`, `road`, `road_class`, `types`, `planned` |
| Traffic | `speed_limits` | Statutory speed limits at a point, in a box or along a path | `at`, `bbox`, `path`, `buffer_m` |
| Weather | `weather` | Current observations, hourly forecast or nowcast | `lon`, `lat`, `kind`, `hours` |
| Weather | `weather_warnings` | Official weather warnings | `at`, `bbox`, `type`, `min_level`, `active_now` |
| Weather | `climate` | Long-term climate facts, solar potential | `lon`, `lat`, `sections` |
| Weather | `air_quality` | Air quality index and pollutants | `lon`, `lat`, `hours` |
| Map | `static_map` | Map image with pins and a line | `center`, `zoom`, `pins`, `line`, `style`, `width`, `height` |

`search_places` with free text and `near` searches only within `radius_m` (500 to 100000 meters, default 25000), sorts by distance and says so explicitly when it has to fall back to matches farther away. A category search always returns the nearest first and ignores `radius_m`. Instead of the coordinate `near` you can pass a place name or address as `near_place` (for example "Melk"); the server resolves it with one additional API call, takes the first match and states in the result which place it searched around and which other matches there were. The place search returns name, type, coordinates and distance, but no opening hours, phone numbers or reviews.

Times in the answer text are Austrian local time with the offset, for example `2026-11-18T08:00+01:00`; this applies to weather, warnings, air quality and traffic reports. Public transport times are local time as well. The structured part of the result (`structuredContent`) keeps the timestamps of the API.

Routing profiles are `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb` and `walking`; `isochrone` also accepts `transit`.

## Example questions

- "Where is Herrengasse 13 in 1010 Vienna, and where is the nearest fuel station?"
- "How long does a truck with 40 t take from Vienna to Linz, and are there closures on the way?"
- "Which parts of Melk can I reach within 15 minutes on foot from the station?"
- "What are the next departures at Melk Bahnhof?"
- "How high is the Grossglockner area, and what does the profile of my hiking path look like?"
- "Will it rain in Innsbruck in the next three hours?"
- "Plan the best order for these five delivery addresses and show them on a map."

## Limits and counting

Every tool call counts like an API call and appears in your usage view as MCP. Some tools make several API calls, for example `search_places` with `near_place` (one extra geocoding call) or free-text search with `near` (up to two when nothing lies within the radius). The protocol messages themselves (each request to `/mcp`) are listed separately as product `mcp`; they do not use up the per-minute limit. The per-minute limit of your token (6000 by default) and the zone limits of the API apply, over MCP per token instead of per IP address ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)). The MCP endpoint itself accepts 10 requests per second per IP address (burst 40) and request bodies up to 256 KB. Per token, at most 4 calls run at the same time; further ones wait or fail with a waiting time. If a limit is exceeded, the assistant gets an error message with a waiting time.

## Errors

The assistant receives the error as text and can react to it. Error codes and causes are listed in [Errors and limits](https://docs.mapx.at/api/errors.md), in particular [`mcp-token-only`](https://docs.mapx.at/api/errors.md#mcp-token-only) and [`mcp-token-required`](https://docs.mapx.at/api/errors.md#mcp-token-required).

## Limitations

- Austria only; data comes from official open data, and each answer names the source.
- All tools only read; nothing is changed or stored.
- Public transport answers come from the scheduled timetable, from the scheduled timetable.
- Climate, weather, warnings and air quality come from GeoSphere Austria, traffic reports from ASFINAG and EVIS.AT.
- Long results are shortened.

## Privacy

Your questions and the results pass through the provider of your assistant. mapx only sees the tool calls, like ordinary API calls.

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `401` | No or invalid token. With sign-in through your account, this makes the assistant start the sign-in; after you disconnect or block the connection it reports `401` and asks again. With the token way, check the `Authorization: Bearer` header and that the token was not deleted or expired; the MCP server takes the token only in this header, not as `access_token` in the URL. |
| Tools are missing, or the assistant calls a tool that no longer exists (`Unknown tool`) | Clients keep the tool list from when the connection was made. After mapx changes its tools, disconnect the mapx connector in your client and connect it again (in claude.ai under Settings → Connectors), then start a new chat. |
| Error page "Application not allowed" | The application is not on the list of approved applications. Use the token way or write to support@mapx.at. |
| `403` `mcp-token-required` | A public (`mx_pk_`) or secret (`mx_sk_`) token was used at the MCP server. Create a token of type "AI assistant (MCP)". |
| `403` `mcp-token-only` | An `mx_mcp_` token was used for a direct API call. Use a secret token (`mx_sk_`) there. |
| `429` | A limit was exceeded. Wait the time stated in the message and make fewer calls in parallel. |
| `403` for a browser client | The `Origin` is not allowed. Browser clients must come from an allowed site; use a desktop client or the API. |

## Next

- [Access and limits](https://docs.mapx.at/start/access-and-limits.md)
- [Errors and limits](https://docs.mapx.at/api/errors.md)
