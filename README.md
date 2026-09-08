# webview-anchor-probe

A single-page probe for how a browser or embedded web view places popovers, and what
CSS anchor positioning it actually has.

Open `index.html`, hover the column headers of a 50-column table, and scroll sideways.
Radio buttons switch between three placement mechanisms for the same popup, and a live
readout gives the header's rect, the popup's rect, the scroll offset, the computed
`top`/`left`, and whether the two line up.

| mode | how the popup is placed |
|---|---|
| `anchor + position-area` | `anchor-name` / `position-anchor`, `position-area: block-end span-inline-end`, `position-try-*` |
| `anchor() longhands` | `top: anchor(bottom); left: anchor(left)` |
| `measured` | `getBoundingClientRect()` on the header, written to custom properties the stylesheet uses as insets, re-run on scroll and resize |

The **environment** button (and a console log on load) reports the engine, whether it is
a browser or an embedded web view, and a yes/no list of the CSS features involved.

## Why

macOS ships CSS anchor positioning as an experimental WebKit feature that is **off by
default in `WKWebView`** and cannot be enabled from the page. Any app embedding a web
view — Electrobun, Tauri, a SwiftUI wrapper — therefore renders anchor-positioned
popovers at their *static* position, uncorrected for scroll: over the toolbar, or off
the right edge of the window once a wide table is scrolled. Enabling the feature via
`WKPreferences._setEnabled(_:forFeature:)` is not a fix either — `position-area` is
still unimplemented, and the `anchor()` that does exist resolves to document
coordinates, ignoring scroll.

Measured on macOS 15.7.9, AppleWebKit 621.3.11: of the three modes, only `measured`
places the popup correctly, at any scroll offset, in a bare `WKWebView`.

## Usage

ES modules are not needed, but `file://` works fine:

```
open index.html
```

Or serve it [locally](http://localhost:8000/), to test from another device:

```
python3 -m http.server 8000
```

To drive it headlessly, the page exposes `window.ENV()` and `window.PROBE(columnIndex)`,
the latter returning a promise of the readout after scrolling that header into view and
opening the popup on it.
