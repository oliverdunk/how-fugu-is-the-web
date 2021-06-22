# Project Fugu 🐡 API Detector

An extension to shine light on the Project Fugu 🐡 APIs web apps try to use.

<img width="1168" alt="Screen Shot 2021-06-22 at 16 45 53" src="https://user-images.githubusercontent.com/145676/122972281-41f10d00-d390-11eb-9d81-36d33146c9b3.png">

## How it works

### API data

The raw data for the different Project Fugu 🐡 APIs is curated in a
[spreadsheet](https://docs.google.com/spreadsheets/d/1ndxh6sl0fSOLHFlMdSiLPqmGPMDVJqkJFYnQ0Hsmhwo/edit?usp=sharing),
which is then turned into JavaScript, so keeping the list of APIs updated is
hopefully a straightforward task.

### API detection

The extension monitors the requests a page makes via the
[`chrome.webRequest.onBeforeRequest.addListener()`](https://developer.chrome.com/docs/extensions/reference/webRequest/#event-onBeforeRequest)
API. Each response body, grouped by JavaScript response bodies and Web App
Manifest response bodies, is then run through a set of regular expressions like
`/navigator\.hid\.requestDevice\s*\(/g` to determine if the code hints at a
Project Fugu 🐡 API potentially being used.

### Browser support detection

Most Project Fugu 🐡 APIs are easily feature-detectable by checking for the
existence of interfaces or properties, for example, as in
`'BarcodeDetector' in window`. Other APIs require a
[`ServiceWorkerRegistration`](https://developer.mozilla.org/en-US/docs/Web/API/ServiceWorkerRegistration),
but luckily the popup window in
[Manifest V3 extensions](https://developer.chrome.com/docs/extensions/mv3/intro/)
uses a service worker, so it can be used via an IIFE that can be run in the
client or the service worker. An example is
`(async () => 'periodicSync' in (await navigator.serviceWorker?.ready || self.registration))()`.
The support categories are listed below:

- ✔️ Supported by your browser.
- 🚫 Not supported by your browser.
- 🤷 Support unknown for your browser. (The only way to know would be
  `user-agent` sniffing.)

### Deep-linking

The extension makes use of [Text Fragment URLs](https://web.dev/text-fragments/)
to deep-link to the occurrence of a detected API, for example
[`https://airhorner.com/scripts/main.min.js#:~:text=navigator.setAppBadge(`](https://airhorner.com/scripts/main.min.js#:~:text=navigator.setAppBadge%28).

## Limitations

- The
  [`chrome.webRequest.onBeforeRequest.addListener()`](https://developer.chrome.com/docs/extensions/reference/webRequest/#event-onBeforeRequest)
  API unfortunately does not "see" requests that are handled by a service worker
  ([crbug.com/766433](https://crbug.com/766433)). The workaround is either to
  open DevTools and check the
  [Bypass for network](https://developer.chrome.com/docs/devtools/progressive-web-apps/#:~:text=bypass%20for%20network)
  checkbox in the Service Worker section of the Application tab, or to
  [Clear storage](https://developer.chrome.com/docs/devtools/progressive-web-apps/#clear-storage)
  in the Storage section of the Application tab.
- The extension does _static_ code analysis, that is, there is no guarantee that
  the app actually uses the code snippet where a Project Fugu 🐡 API was
  detected.
- Heavily minified code will not be detected. For example, if an app minifies
  `navigator.clipboard.write()` to
  `const nav = navigator; nav.clipboard.write()`, the extension will not detect
  this.
- In the initial release, code in inline `<script>` tags is not yet considered.

## License

Apache 2.0.
