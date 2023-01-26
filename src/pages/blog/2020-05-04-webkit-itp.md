---
templateKey: "blog-post"
title: "WebKit ITP & its impact on Cookies and Web Storage"
date: 2020-05-04T15:04:10.000Z
featuredpost: false
description: >-
  WebKit is the web browser engine used by Web & IOS Safari browsers. Intelligent Tracking Prevention (ITP) is a feature of WebKit created mainly to protect users’ online privacy by changing the way WebKit-based browsers such as Safari handle the first-party cookies. Though it is meant to impose restrictions on tracking domains, it could have an impact on the domain getting blacklisted or it could create application bugs if it is not been taken into consideration


tags:
  - WebKit
  - ITP
  - IOS Safari
  - cookies
---

Web browsers have evolved a lot over the decade and so is browser-based storage facilities. In the Modern web, users don’t need to be online to interact with a web app, there are offline supported applications that leverage browsers storage mechanisms such as Cookies, Session Storage, and Local Storage. Web developers are attracted towards these storages to store data thinking that it is persistent over session or time. It may be true for a few browsers, but WebKit-based browsers such as safari have stringent rules in the form of Intelligent Tracking Policy which could jeopardize the application or user experience if not carefully dealt with!

## Purpose of ITP

As regulations are getting strict against the use of Third party cookies selling user information, Browsers are forced to include strict policies to avoid or restrict third party cookies. Safari blocks third-party cookies by default while other browsers have options to disable them.

Because of that, Trackers are finding alternative approaches to get the user tracking information. One such approach is using First party cookies! ITP is coined by WebKit mainly to mitigate and standardize the use of First party cookies.

## How it works

ITP implements a Machine learning model which assesses all the tracker domain which are tracking user information across different websites. This model is based on the statistics collected by the browser. If the Machine learning classifier identifies a first party cookie as a tracker, it will be blocked unless the site has requested the permission via Storage Access API.

ITP rules are targeted only at Tracking domains and not all domains. However, there is no clear list of classified domains provided and we need to be cautious not to do any activities that would blacklist our domain.

All tracking cookies are managed in isolated storage and it is called Partitioned Cookies.

### Versions

There are Six versions of ITP released so far,

| ITP Version | Webkit version | Key changes |
| :-: | :-: | :-: |
| ITP 1.0 | Safari 12 / IOS 11 | 24 hr period for storing third party cookies |
| ITP 1.1 | Safari 12.1 | Introduced Storage Access API |
| ITP 2.0 | Safari for iOS 12 and macOS Mojave | Block all third party and use 30 day for embedded authorized content. |
| ITP 2.1 | beta releases of iOS 12.2 and Safari 12.1 on macOS High Sierra and Mojave | 7 days cookie expiry |
| ITP 2.2 | beta releases of iOS 12.3 and Safari on macOS Mojave 10.14.5 | Expire cookie on 24 hrs (Only for tracking sites) |
| ITP 2.3 | Safari 13 / iOS 13 | prevent cross-site tracking through referrer and through link decoration. |

**Key points to keep in mind before deciding on storage:**

**a. Cookie expiration set to 7 days**

With ITP 2.1, All persistent cookies set through **document.cookie** would be capped for a seven day expiry. Ie) Safari will not allow the client side cookie to be set for expiry of more than 7 days. If you set it for 10 days, it will revert it to 7 days but it respects any day less than that. In order to persist cookie for a longer period of time, consider setting it as HTTP cookies instead.

**b. LocalStorage capped to 7 days in ITP 2.3**

As the cookie usage is restricted to the maximum, Trackers look forward to other storage options such as LocalStorage in place to store the data. However, ITP 2.3 now puts a 7-day limit on all non-cookie storage data, including **localStorage**. The good news is that if a user interacts with the website within seven days, then the expiry date will be reset, allowing the data to remain in the browser’s local storage for another seven days.

**c. Storage Access API**

Introduced mainly to facilitate the third party embedded contents such as IFRAME which the user might use to share any social content such as FB like or G+. Third party can request access to first party cookies when the user interacted with them,

**d. First Party Bounce Trackers & Tracking collusion protection**

ITP established several rules to track down the domains which steal the user data without consent that includes First party Bouncers and Protecting tracking collusion.

Though most of them are pointed towards third parties and tracking domains. They clearly haven’t defined how they classify a domain to be a tracker and don’t expose a list of classified domains either. So, when we develop any cookie strategy it is better to keep those points in mind and create solutions as if our domains might have been classified. Also, Remember to add fallbacks while relying on data cached on the session or local storage.

**Conclusion**

Though most of them are pointed towards third parties and tracking domains. They clearly haven’t defined how they classify a domain to be a tracker and don’t expose a list of classified domains either. So, when we develop any cookie strategy it is better to keep those points in mind and create solutions as if our domains might have classified.

Few concluding points are

1. Try avoid client side cookies and use a secure, httponly HTTP cookies by default.
2. For Experimentation and Analytics, try moving to a Server side solution.

**References**

1. [https://webkit.org/blog/category/privacy/](https://webkit.org/blog/category/privacy/)
2. [https://webkit.org/blog/8124/introducing-storage-access-api/](https://webkit.org/blog/8124/introducing-storage-access-api/)
3. [https://webkit.org/blog/7675/intelligent-tracking-prevention/](https://webkit.org/blog/7675/intelligent-tracking-prevention/)
