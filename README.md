# flok-join

A minimal GitHub Pages redirect that sends each visitor to a **unique, isolated [Flok](https://flok.cc) session** with a pre-configured setup — designed for live coding workshops and classroom use.

## How it works

Each student clicks the same link. The redirect page generates a random session slug client-side (`Math.random()`), then immediately forwards them to:

```
https://flok.cc/s/<randomSlug>#targets=...&c0=<base64code>
```

Because each browser runs the JS independently, every student lands in their own private session — same target configuration and starter code, different session.

## Setup

1. Fork or clone this repo
2. Enable GitHub Pages: **Settings → Pages → Deploy from branch → `main`**
3. Your redirect URL will be: `https://<your-username>.github.io/flok-join`

No build step. No dependencies. One file.

## Usage in slides

Link to the redirect page with a `#` hash carrying the Flok configuration. Each slide can have a different hash for different exercises:

```markdown
[Open exercise](https://your-username.github.io/flok-join#targets=strudel&c0=BASE64_CODE)
```

The hash is passed through transparently to Flok — the redirect only generates the slug.

## Generating base64 starter code

Flok expects pane code (`c0`–`c7`) encoded in Base64. Generate it in the browser console:

```js
btoa('sound("bd sd").fast(2)')
// → "c291bmQoImJkIHNkIikuZmFzdCgy"
```

Or in the terminal:

```bash
echo -n 'sound("bd sd").fast(2)' | base64
```

Then build the full link:

```
https://your-username.github.io/flok-join#targets=strudel&c0=c291bmQoImJkIHNkIikuZmFzdCgyKQ==
```

## Flok URL parameters

Flok splits parameters between the **hash** (`#`) and the **query string** (`?`).

### Hash parameters — session configuration

| Parameter | Description | Example |
|-----------|-------------|---------|
| `targets` | Language targets, comma-separated (max 8 panes) | `#targets=strudel,hydra` |
| `c0`–`c7` | Base64-encoded starter code per pane | `#c0=BASE64` |
| `code` | Alias for `c0` | `#code=BASE64` |
| `username` | Pre-fill username | `#username=student` |

### Query parameters — display and embedding

| Parameter | Description | Example |
|-----------|-------------|---------|
| `readOnly` | Disable editing; skips username prompt | `?readOnly=true` |
| `bgOpacity` | Background opacity, range 0–1 | `?bgOpacity=0` |
| `noWebEval` | Disable code evaluation for specified web targets (still posts messages to parent window) | `?noWebEval=hydra` or `?noWebEval=*` |
| `hideErrors` | Suppress error display for web targets | `?hideErrors=true` |
| `hideCode` | Hide all code editors — output only | `?hideCode=true` |

For use with this redirect, only **hash parameters** are passed through — the redirect forwards `window.location.hash` directly to Flok. Query parameters (`?`) are not forwarded.

Link examples for slides:

```
https://your-username.github.io/flok-join#targets=strudel&c0=c291bmQoImJkIHNkIikuZmFzdCgyKQ==
```

Multiple panes:

```
https://your-username.github.io/flok-join#targets=strudel,hydra&c0=BASE64_STRUDEL&c1=BASE64_HYDRA
```

To use query parameters like `readOnly` or `hideCode`, hardcode them into `index.html` when building the destination URL:

```js
window.location.replace("https://flok.cc/s/" + slug + config + "?readOnly=true");
```

## Available targets

### Web targets — run directly in the browser, no local install needed

| Target | Language / Tool |
|--------|-----------------|
| `strudel` | [Strudel](https://strudel.cc/) — JavaScript pattern language inspired by TidalCycles |
| `hydra` | [Hydra](https://hydra.ojack.xyz/) — live video synthesis; also supports p5.js via `P()` |
| `mercury` | [Mercury Web](https://www.timohoogland.com/mercury-livecoding/) — browser-based Mercury |
| `p5js` | [p5.js](https://p5js.org/) — creative coding / visual sketching |

### REPL targets — require `flok-repl` and a local install of the language

| Target | Language / Tool | Notes |
|--------|-----------------|-------|
| `tidal` | [TidalCycles](https://tidalcycles.org/) | Requires GHCi + SuperDirt |
| `sclang` | [SuperCollider](https://supercollider.github.io/) | Runs `sclang` process; not supported on Windows |
| `remote_sclang` | SuperCollider (via SC IDE) | Requires [FlokQuark](https://github.com/munshkr/FlokQuark); cross-platform |
| `foxdot` | [FoxDot](https://foxdot.org/) | Requires Python + SuperCollider |
| `renardo` | [Renardo](https://renardo.org/) | Maintained FoxDot fork |
| `sardine` | [Sardine](https://sardine.raphaelforment.fr) | Python-based live coding |
| `dummy` | Any OSC-capable tool | Forwards evaluations as OSC to port 3001 — use for unsupported languages |

## Important: this requires students on their own devices

The random slug is generated in each student's browser. If you are the only one loading the page (e.g. projecting it from your machine), everyone would share a single slug and end up in the same session.

**The intended workflow:**

1. Display the link (or QR code) on your projected slide
2. Each student opens it on their own laptop or phone
3. Every browser independently generates a unique slug → isolated session

## Embedding in Obsidian Slides Extended

Since Obsidian sanitizes `<script>` tags in the editor preview, embedding requires the **"Open in Browser"** workflow or exporting the deck as HTML. In the exported/served HTML, iframes work normally:

```html
<iframe
  src="https://flok.cc/s/my-demo#targets=strudel&c0=BASE64"
  width="100%"
  height="500"
  allow="microphone; camera">
</iframe>
```

Use an iframe for your **instructor demo session** (fixed slug, shown projected). Use the redirect link for **student sessions** (random slug, opened on their devices).

## File structure

```
flok-join/
└── index.html   ← the entire redirect, ~10 lines of JS
```

## License

MIT