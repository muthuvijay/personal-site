---
templateKey: "blog-post"
title: "Improve performance with Service Worker"
date: 2020-10-23T15:04:10.000Z
featuredpost: false
description: >-
  Caching is a key aspect of performance optimization in any application. The browser has an in-built cache that helps to cache any web assets such as images, scripts, styles, fonts, etc. Like any other caching, a browser cache can be activated only for the second interaction. What if we have an option to precache the assets so they can be fetched and used for first-time users as well? It is possible with the modern browser capabilities with Service worker and Cache API.


tags:
  - Service Worker
  - Cache API
  - Workbox
  - Page load performance
---

## What is Service worker?

Service Worker acts as a network proxy that runs on a separate thread other than main UI thread. It is a good place to handle the cache using CacheStorage API, Web push notification, offline fallback pages etc. It is very powerful that it could run behind the scene even if the main tab / browser is closed or idle.

![Service worker as proxy](/img/sw-1.png)

## Service Worker Lifecycle:

![Service worker as proxy](/img/sw-3.png)

## Need for Service Worker Cache:

While static assets can be cached at the browser, what is the need for service worker? Well, it has few significant advantage over a browser cache as listed below

1.  Provides more control to the app developers on the third party assets which is not possible in browser cache as it should be set by the producer at the CDN / server level.
2.  Precaching can help to load critical assets much faster.
3.  In general, a cache can be useful only at the second time, But Service worker level cache can be tailored to save first time users because of precaching.
4.  Efficient Cache Strategies.
5.  MFE Cache restrictions.
6.  Helps user with poor n/w connectivity.
7.  Long time caching of static chunks.

## Cache API & IndexedDB

- Storage mechanism for [Request](https://developer.mozilla.org/en-US/docs/Web/API/Request)/ [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response)object pairs.
- IndexedDB is a Web API for storing large amount of data in the browser.
- Cache API and IndexedDB are exposed to window scope as well as SW scope. ![Service worker as proxy](/img/sw-2.png)

## Workbox

Library written by Google which provides a wrapper over native service worker.

#### Advantages:

- Simplifies SW process
- Better Cache Strategies
- Expiration Control via IndexedDB.
- Offers Version Control out of box.
- Helps with Debugging.

#### Pre-Caching

- Serves Critical assets upfront when the page loads.
- Precache Manifest is a collection of URLs and associated versions for each URL.
- Happens at the install stage.
- Can be tailored to serve first time users.
- Workbox has a plugin to deal with webpack builds.

#### Dynamic / Runtime. caching:

- Cache as you go.
- Helps to make future request faster.
- Ideal for MFE as we don’t know the chunks created by the apps at build time.
- Allow other caching strategies.

## Caching Strategies :

There are several strategies workbox provides out of box, but we are not limited only to this,

- Cache Only
- Network Only
- Cache First
- Network First
- Stale While Revalidate

## Cache storage limit

- The amount of storage is dependent upon how much available disk space the customer's device has.
- Not specific to service worker as per spec.
- The common rule is 20% of available space
- iOS Safari has exception as usual.

## Cache Invalidation

- Invalidate cache upon each release.
- Controlled Entries to store.
- 30 days expiry for static files (long cached)
- Will auto purge if quota exceed (LIFO)
- Kill Switch (on a worst case)

## Caveats:

1.  Wont be activated if there is already a SW running unless the user close and reopen the tab
2.  Works only with HTTPS.
3.  Opaque Response make it difficult to cache.
4.  Asynchronous -> Can’t use any sync API such as Localstorage.
5.  No support for IE browser.

## References:

[https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle](https://developers.google.com/web/fundamentals/primers/service-workers/lifecycle)

[https://developers.google.com/web/tools/workbox/modules/workbox-strategies](https://developers.google.com/web/tools/workbox/modules/workbox-strategies)

[https://web.dev/precache-with-workbox/](https://web.dev/precache-with-workbox/)

[https://blog.bitsrc.io/understanding-service-workers-and-caching-strategies-a6c1e1cbde03](https://blog.bitsrc.io/understanding-service-workers-and-caching-strategies-a6c1e1cbde03)
