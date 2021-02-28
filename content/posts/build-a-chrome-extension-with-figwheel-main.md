---
title: Waste plastic to diesel fuel demo
date: 2021-02-27T15:54:31+00:00
lastmod: 2021-02-27T15:54:31+00:00
author: biomassives
avatar: "/me/yy.jpg"
cover: https://raw.githubusercontent.com/biomassives/peacewater_search_scdhub_solutions_library/main/static/img/50bef46befeec85cab9d532fdc23d811.jpg
categories: []
tags: []

---
Bring a smooth experience to the development of Chrome Extension.

27<!--more-->

## Before starting

Below is the screenshot from

## Setup

Assuming you have [lein](https://leiningen.org/) installed, then open [https://rigsomelight.com/figwheel-main-template/](https://rigsomelight.com/figwheel-main-template/) and copy the `new` command to your shell:

```sh
lein new figwheel-main hello-world.core -- --reagent
```

> Note: for a simple setup, we won't use `+npm-bundle` in the options but use cljsjs packages.

This command will create a dir named `hello-world.core` and add a minimal [Reagent](https://reagent-project.github.io/) application into it.

Then we can run:

```sh
lein fig:build
```

to bootstrap the dev environment. After build, a new tab will be opened automatically and the repl will also be launched.

We can test it by run below in repl:

```clojure
(js/alert "Am I connected?")
```

If you see an alert opened, then our preparations are complete.

## Extension manifest

To develop a chrome extension, we need to create a `manifest.json` in the `resources/public` dir:

```sh
touch resources/public/manifest.json
```

You can view [Manifest file format](https://developer.chrome.com/docs/extensions/mv2/manifest/#manifest_version) for more details. This time we will fill the content below:

```json
{
  "name": "Hello World",
  "version": "0.1.0",
  "manifest_version": 2,
  "browser_action": {
    "default_popup": "index.html"
  }
}
```

Then we can go to the next step.

## Load unpacked

Now we can put it into extensions, open `chrome://extensions` and click **Load unpacked** to select `resources/public` folder:

![Load unpacked](/img/build-a-chrome-extension-with-figwheel-main/load-unpacked.png)

Everything looks normal, but when you click the extension icon in the extensions bar, some errors will occur:

![Errors](/img/build-a-chrome-extension-with-figwheel-main/errors.png)

This is the first point we need to pay attention to: [Chrome Apps Content Security Policy](https://developer.chrome.com/docs/apps/contentSecurityPolicy/).

Since we are in a development environment, Figwheel Main will insert some inline scripts (relate to its own functionality) into the document. You can view the `dev-main.js`:

```js
if (typeof goog == 'undefined') document.write('<script src="/cljs-out/dev/goog/base.js"></script>')
document.write('<script src="/cljs-out/dev/goog/deps.js"></script>')
document.write('<script src="/cljs-out/dev/cljs_deps.js"></script>')
document.write(
  '<script>if (typeof goog == "undefined") console.warn("ClojureScript could not load :main, did you forget to specify :asset-path?");</script>'
)
document.write('<script>goog.require("figwheel.core");</script>')
document.write('<script>goog.require("figwheel.main");</script>')
document.write('<script>goog.require("figwheel.repl.preload");</script>')
document.write('<script>goog.require("devtools.preload");</script>')
document.write('<script>goog.require("figwheel.main.system_exit");</script>')
document.write('<script>goog.require("figwheel.main.css_reload");</script>')
document.write('<script>goog.require("process.env");</script>')
document.write('<script>goog.require("hello_world.core");</script>')
```

This violates this rule:

> You can't use inline scripting in your Chrome App pages. The restriction bans both <script> blocks and event handlers (<button onclick="...">).

Refer to the errors above, to solve this problem, we need to set `content_security_policy` field in the `manifest.json`:

```json
{
  "content_security_policy": "script-src 'self' 'unsafe-eval' 'sha256-xxx' 'sha256-xxx' 'sha256-xxx'; object-src 'self'"
}
```

The errors contain all `sha256-xxx` which need to be filled in the policy field. This is a little bit cumbersome. You may think why we can't use the `unsafe-inline`?
Because chrome ignores this keyword especially, even the errors tell you can 😢.

After finishing this, edit the code and you will see that your code has completed the hot reload 😎.

## Production

After you finish your application, you still need to do something before bundle it.

By developing the chrome extension, you need to use `chrome` API to do somethings, like save and sync the user storage, etc. We need to tell the closure compiler `chrome` is the [externs](https://developers.google.com/closure/compiler/docs/externs-and-exports) we used.

There are two files we need to download: [https://github.com/google/closure-compiler/blob/master/contrib/externs/chrome.js](https://github.com/google/closure-compiler/blob/master/contrib/externs/chrome.js) and [https://github.com/google/closure-compiler/blob/master/contrib/externs/chrome_extensions.js](https://github.com/google/closure-compiler/blob/master/contrib/externs/chrome_extensions.js).

Put them into `externs` folder and edit `dev.cljs.edn` like below content:

    {:main hello-world.core
     :externs ["externs/chrome.js" "externs/chrome_extensions.js"]}

Then run:

```sh
lein fig:min
```

to build the production code.

## Conclusion

These are all steps to build a chrome extension with Figwheel Main.

If you are looking for a real-world example as a reference, the [GitHub Colorful Contributions](https://github.com/g1eny0ung/github-colorful-contributions-graph) is what you want.

Thanks for reading. Happy coding with Figwheel Main!

References:

* [ClojureScript](https://clojurescript.org/)
* [Figwheel](https://figwheel.org/)  -what is it? this is a tibdbit left in the theme we are adopting for peacewater_search

![](/uploads/screenshot_2021-02-27-google-closure-compiler.png)