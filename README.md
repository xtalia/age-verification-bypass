# Age Verification Bypass (Userscript) — Enhanced fork

Port of the Firefox add-on [helloyanis/age-verification-bypass](https://github.com/helloyanis/age-verification-bypass) to a userscript (Violentmonkey, Tampermonkey, Greasemonkey), enhanced by [Hermes Agent](https://hermes-agent.nousresearch.com) (Nous Research).

Upstream port by [LucianoSkx](https://github.com/LucianoSkx). Fork maintained by [xtalia](https://github.com/xtalia).

## Installation

1. Install [Violentmonkey](https://violentmonkey.github.io/) (recommended), [Tampermonkey](https://www.tampermonkey.net/), or [Greasemonkey](https://www.greasespot.net/)
2. Click [install](https://raw.githubusercontent.com/xtalia/age-verification-bypass/main/age-verification-bypass.user.js)
3. Confirm installation

## What's different in 2.0.0

The 1.x port patched only `window.fetch`. That misses three large classes of traffic:

| 1.x problem | 2.0.0 fix |
|---|---|
| Only `fetch` was intercepted | One rule table drives **both `fetch` and `XMLHttpRequest`**; XHR instances get their `responseText`/`response` accessors overridden so the page reads the rewritten body regardless of listener order |
| SDKs loaded via `<script src>` were never seen by the interceptor | **Global traps** (`AgeCheckerConfig`, `AGEGO`, `Veriff`, `veriffSDK`) fire the "accepted" callback the moment the page assigns its config |
| `window` patching broke under userscript sandboxes | Runs in the **page world** (`@grant none` + `unsafeWindow` fallback) |
| Rewritten responses kept the original `content-length`/`content-encoding` | Those headers are **scrubbed** before the body is replaced |
| ageverif stub depended on `document.currentScript` (null when injected) | Falls back to scanning for the `checker.js` script tag |
| Coverage limited to a fixed site list | Adds a **generic age-gate sweep** — removes overlay-only age gates on any site and restores scrolling |
| Brazilian Portuguese UI strings | Tor hints localised to English |

## Supported Services

- **[AgeChecker.net](https://agechecker.net/demo)** — Full bypass (unless the site does a server-side double-check)
- **[AgeGO](https://agego.com)** — Basic + advanced integration; server-to-server mode (may fail if site does additional checks)
- **[AgeVerif.com](https://demo.ageverif.com/)** — Basic and advanced integrations (not oAuth2 flow)
- **[AliExpress](https://aliexpress.com/)** — "For adults" items (removes blur/modal/overlays, including suggested products)
- **[Bluesky](https://bsky.app)** — Sensitive posts without login (automod + self-labelled posts); media revealed by clicking "Show"
- **[Reddit](https://reddit.com)** — NSFW communities (works best logged out; consider [redlib](https://redlib.catsarch.com/) for a fully private Reddit frontend)
- **[SpankBang](https://spankbang.com)** — View videos even when logged out (removes blur/overlay and neutralizes the age verification modal)
- **[Veriff](https://veriff.com)** — Works on only a few sites (don't expect it to work everywhere)
- **[x.com / Twitter](https://x.com)** — Unblurs sensitive posts in single post view (`TweetResultByRestId`, `TweetDetail`) and profile timelines (`UserOriginalsTimeline`, `UserTweetsAndReplies`); requires being logged in (still BETA upstream)
- **[Cosxplay](https://cosxplay.com)** — Blocks the age verification script (`age.js`)
- **[AngeloGodsHack](https://angelogodshackxxx.com)** — Removes the age gate modal
- **[rule34.xxx](https://rule34.xxx)** — Geographical IP block — shows a Tor Browser hint (no direct bypass, same as upstream)
- **[xHamster](https://xhamster.com)** — Geographical IP block — shows a Tor Browser hint (no direct bypass, same as upstream)
- **Any other site** — generic age-gate sweep (conservative: overlay-positioned elements only)

## How It Works

Three methods:

### Rewrite Server Response
Intercepts `fetch` and `XMLHttpRequest` calls that would create the age verification popup and replaces the body with code that automatically sends the "verification approved" callback. Example: Bluesky, AgeChecker, Veriff.

### Trap SDK Globals
SDKs shipped as `<script src>` never touch `fetch`, so their config globals are intercepted with accessors the moment the page assigns them. Example: AgeChecker, AgeGO, Veriff.

### Hide and Remove DOM Elements
Removes popups, blurs, and overlays added when a page is marked NSFW, plus a generic sweep for age gates nobody has written a rule for. Example: AliExpress, Reddit, Cosxplay.

**No data is collected.** There is no tracking of which sites you visit.

## Updates

The script checks for updates automatically via `@updateURL`/`@downloadURL` pointing to this repository.

## Tests

```bash
node test/templates.test.js
```

The suite runs the userscript inside a stubbed page environment (`node:vm`) and asserts that the fetch wrapper rewrites a matching response, leaves unrelated responses byte-identical, and that the config traps fire the `accepted` callback.

## Credits

- Original: [helloyanis](https://github.com/helloyanis) — [Firefox add-on](https://github.com/helloyanis/age-verification-bypass)
- Port: [LucianoSkx](https://github.com/LucianoSkx) — [userscript port](https://github.com/LucianoSkx/age-verification-bypass)
- Enhancement: **Hermes Agent / Nous Research** — interception engine (fetch + XHR + SDK traps), header scrubbing, generic age-gate sweep

## License

MIT
