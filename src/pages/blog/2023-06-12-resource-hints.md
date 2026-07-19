---
templateKey: "blog-post"
title: "Speed up page loads with Resource Hints"
date: 2023-06-12T15:04:10.000Z
featuredpost: false
description: >-
  Resource hints are declarative instructions in HTML that tell the browser to prepare for future navigation or fetch work early—DNS lookup, TCP/TLS handshake, or downloading a file—before the parser or JavaScript would normally discover the need. Used carefully, dns-prefetch, preconnect, prefetch, preload, and modulepreload can shave meaningful time off critical paths without changing application logic.


tags:
  - Resource hints
  - Performance
  - dns-prefetch
  - preconnect
  - preload
  - Page load performance
---

Every millisecond on the critical path counts. By the time your HTML parser reaches a `<link rel="stylesheet">` or a `<script src="...">`, the browser still has to resolve DNS, open a connection, negotiate TLS, and only then download the file. **Resource hints** let you start some of that work earlier—often while the document is still streaming—so that when the real request arrives, the network is already warm.

They are not a replacement for good bundling, caching, or CDN placement. They are a small, declarative layer you add in `<head>` (or via `Link` response headers) to nudge the browser toward work you already know you will need.

## The hint family

All resource hints share the same mechanism: a `<link>` element (or an HTTP `Link` header) with a specific `rel` value and, for fetch-related hints, an `href` pointing at a URL.

| Hint | What it does | Typical use |
| --- | --- | --- |
| **dns-prefetch** | Resolves the hostname to an IP address | Third-party origins you will call later (analytics, fonts, API) |
| **preconnect** | DNS + TCP + TLS (full connection setup) | Origins where you will fetch several resources soon |
| **prefetch** | Low-priority fetch of a resource for a *future* navigation | Next page in a flow, likely user click |
| **preload** | High-priority fetch of a resource for the *current* navigation | Hero image, critical font, above-the-fold CSS chunk |
| **modulepreload** | Preload for ES modules (also pre-parses the module graph) | Entry or shared chunks in a module-based app |

Think of them on a spectrum from cheapest to most committed:

```
dns-prefetch  →  preconnect  →  prefetch / preload / modulepreload
   (DNS only)     (connection)        (actual download)
```

## dns-prefetch

`dns-prefetch` is the lightest hint. The browser looks up the IP for a hostname and caches the result. It does not open a socket or download anything.

```html
<link rel="dns-prefetch" href="https://fonts.googleapis.com">
<link rel="dns-prefetch" href="https://api.example.com">
```

Use it when you know you will contact an origin later but are not sure exactly when, or when `preconnect` would be wasteful (many origins, only one or two requests each). The cost is small; the win is avoiding DNS latency on the first request to that host.

## preconnect

`preconnect` goes further: DNS lookup, TCP handshake, and TLS negotiation all happen ahead of time. When the first real request to that origin fires, the connection is ready.

```html
<link rel="preconnect" href="https://cdn.example.com" crossorigin>
```

Add `crossorigin` when you will fetch CORS-enabled resources (fonts, images with CORS, fetch/XHR). Without it, the browser may open a connection that cannot be reused for credentialed or CORS requests.

**Do not preconnect to everything.** Each warm connection consumes memory and CPU on both client and server. Limit preconnect to a handful of origins that sit on your critical path—your CDN, your API, a font provider—not every third-party script on the page.

## prefetch

`prefetch` tells the browser to fetch a resource at low priority because it will likely be needed on a **future** navigation, not the current one. The response is stored in the HTTP cache (or discarded if cache policy does not allow storage).

```html
<link rel="prefetch" href="/checkout" as="document">
<link rel="prefetch" href="/assets/chunk-cart.js" as="script">
```

Good fits:

- The next step in a wizard after the user completes the current screen.
- A route bundle the user is likely to open (product detail from a listing page).
- Assets for a page linked prominently in the UI.

Prefetch is speculative. If the user never navigates there, you spent bandwidth for nothing. Keep prefetches tied to strong signals—visible links, recent behavior, or explicit prerender/speculation rules—not blind guesses on every page.

## preload

`preload` is for the **current** navigation. It fetches a resource early with high priority so it is available when the parser or script actually needs it.

```html
<link rel="preload" href="/fonts/inter-var.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/hero.webp" as="image">
<link rel="preload" href="/critical.css" as="style">
```

The `as` attribute is required. It lets the browser set correct priority, apply content security policy, and know how to handle the response. Common values: `script`, `style`, `font`, `image`, `fetch`, `document`.

Preload shines when:

- A resource is discovered late (background image in CSS, font referenced only after stylesheet parse).
- You split code and know the main chunk must run immediately.
- LCP depends on an image or font that would otherwise queue behind less important assets.

**Caveat:** Preloading the wrong file wastes bandwidth and can *hurt* performance by competing with truly critical resources. Preload only what you will use on this page load, and prefer one strong candidate over many weak ones.

## modulepreload

For applications built with native ES modules, `modulepreload` fetches a module and optionally walks its static imports to preload the dependency graph.

```html
<link rel="modulepreload" href="/assets/app.js">
```

Bundlers like Vite and webpack can inject these automatically for entry chunks. Manual hints help when you know which module the user’s path will execute first.

## HTTP Link headers

The same hints work as response headers, which is useful when HTML is generated dynamically or when a CDN can attach hints without changing page markup.

```
Link: <https://cdn.example.com>; rel=preconnect; crossorigin
Link: </assets/main.js>; rel=preload; as=script
```

Headers and `<link>` tags can coexist; avoid duplicating the same hint twice without reason.

## Priority and interaction with the browser

Browsers schedule hinted work alongside normal parsing. Rough priority order (simplified):

1. **preload** (current page, high)—competes with parser-discovered critical resources; use sparingly.
2. Normal document subresources (scripts, stylesheets, sync discoveries).
3. **prefetch** (future navigation, low)—runs when the network is idle.

Because preload is high priority, overusing it can starve other critical downloads. Prefetch is deliberately deprioritized so it should not block the current page, but on slow networks or metered connections it still consumes data the user may never need.

## Choosing the right hint

A practical decision flow:

1. **Same page, need it soon for render or LCP?** → `preload` (or `modulepreload` for ES modules).
2. **Same page, but only DNS or connection setup is the bottleneck?** → `preconnect` (or `dns-prefetch` if connection reuse is unlikely).
3. **Next page or likely next navigation?** → `prefetch` (or speculation rules for more advanced cases).
4. **Many third-party hosts, one request each?** → `dns-prefetch` only.

When in doubt, measure. DevTools Network panel shows initiator type (`preload`, `prefetch`, etc.) and whether early hints actually moved the start time of the dependent request.

## Common mistakes

**Preconnecting to unused origins.** Audit hints after refactors; stale preconnects are pure overhead.

**Preloading assets that are already high in the HTML.** If `<script src="app.js">` is the first tag in `<body>`, a preload for the same URL adds duplicate fetch coordination work without benefit.

**Missing `crossorigin` on fonts.** Font requests are CORS-enabled; preload and preconnect must match the eventual request mode or the browser may fetch twice.

**Prefetching without cache-friendly responses.** Prefetch stores in HTTP cache. If `Cache-Control: no-store` applies, the prefetched copy may be unusable on navigation.

**Treating prefetch as guaranteed.** It is a hint. Slow networks, data saver mode, or memory pressure can cause the browser to skip or evict prefetched resources.

## Beyond classic hints: Speculation Rules

Modern Chromium-based browsers support the [Speculation Rules API](https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API), which supersedes the older `rel=prerender` link hint. You define JSON rules for prefetching or prerendering URLs (including full document prerender for instant navigations). It is more flexible than a single `<link rel="prefetch">` but also easier to misuse; prerendering heavy pages can waste CPU and memory.

Use speculation rules when you have high-confidence next navigations (same-origin, user hover or viewport signals). For simple “maybe they click this link” cases, `prefetch` remains enough.

## Further reading

- [Resource Hints](https://www.w3.org/TR/resource-hints/): W3C specification for dns-prefetch, preconnect, prefetch, and prerender.
- [preload on MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/preload): Syntax, `as` values, and CORS notes.
- [Preconnect to required origins](https://web.dev/uses-rel-preconnect/): Lighthouse guidance and measurement.
- [Prefetching, preloading, prebrowsing](https://css-tricks.com/prefetching-preloading-prebrowsing/): Practical overview of when each hint fits.
- [Speculation Rules API](https://developer.chrome.com/docs/web-platform/prerender-pages): Prerender and prefetch with structured rules.
