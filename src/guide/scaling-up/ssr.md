---
aside: deep
---

# Server-Side Rendering (SSR)

## Overview

### What is SSR?

Vue.js is a framework for building client-side applications. By default, Vue components produce and manipulate DOM in the browser as output. However, it is also possible to render the same components into HTML strings on the server, send them directly to the browser, and finally "hydrate" the static markup into a fully interactive app on the client.

A server-rendered Vue.js app can also be considered "isomorphic" or "universal", in the sense that the majority of your app's code runs on both the server **and** the client.

### Why SSR?

Compared to a client-side Single-Page Application (SPA), the advantage of SSR primarily lies in:

- **Better SEO**: the search engine crawlers will directly see the fully rendered page.

  :::tip
  Note that as of now, Google and Bing can index synchronous JavaScript applications just fine. Synchronous being the key word there. If your app starts with a loading spinner, then fetches content via Ajax, the crawler will not wait for you to finish. This means if you have content fetched asynchronously on pages where SEO is important, SSR might be necessary.
  :::

- **Faster time-to-content**: this is more prominent on slow internet or slow devices. Server-rendered markup doesn't need to wait until all JavaScript has been downloaded and executed to be displayed, so your user will see a fully-rendered page sooner. This generally results in improved [Core Web Vitals](https://web.dev/vitals/) metrics, better user experience, and can be critical for applications where time-to-content is directly associated with conversion rate.

- **Unified mental model**: you get to use the same language and the same declarative, component-oriented mental model for developing your entire app, instead of jumping back and forth between a backend templating system and a frontend framework.

There are also some trade-offs to consider when using SSR:

- Development constraints. Browser-specific code can only be used inside certain lifecycle hooks; some external libraries may need special treatment to be able to run in a server-rendered app.

- More involved build setup and deployment requirements. Unlike a fully static SPA that can be deployed on any static file server, a server-rendered app requires an environment where a Node.js server can run.

- More server-side load. Rendering a full app in Node.js is going to be more CPU-intensive than just serving static files, so if you expect high traffic, be prepared for corresponding server load and wisely employ caching strategies.

Before using SSR for your app, the first question you should ask is whether you actually need it. It mostly depends on how important time-to-content is for your app. For example, if you are building an internal dashboard where an extra few hundred milliseconds on initial load doesn't matter that much, SSR would be an overkill. However, in cases where time-to-content is absolutely critical, SSR can help you achieve the best possible initial load performance.

### SSR vs. SSG

**Static Site Generation (SSG)**, also referred to as pre-rendering, is another popular technique for building fast websites. If the data needed to server-render a page is the same for every user, then instead of rendering the page every time a request comes in, we can render it only once, ahead of time, during the build process. Pre-rendered pages are generated and served as static HTML files.

SSG retains the same performance characteristics of SSR apps: it provides great time-to-content performance. At the same time, it is cheaper and easier to deploy than SSR apps because the output is static HTML and assets. The keyword here is **static**: SSG can only be applied to pages consuming static data, i.e. data that is known at build time and does not change between deploys. Every time the data changes, a new deployment is needed.

If you're only investigating SSR to improve the SEO of a handful of marketing pages (e.g. `/`, `/about`, `/contact`, etc), then you probably want SSG instead of SSR. SSG is also great for content-based websites such as documentation sites or blogs. In fact, this website you are reading right now is statically generated using [VitePress](https://vitepress.vuejs.org/), a Vue-powered static site generator.

## Basic Usage

### Rendering an App

Vue's server-rendering API is exposed under `vue/server-renderer`.

Let's take a look at the most bare-bone example of Vue SSR in action. First, create a new directory and run `npm install vue` in it. Then, create an `example.mjs` file:

```js
// example.mjs
// this runs in Node.js on the server.
import { createSSRApp } from 'vue'
import { renderToString } from 'vue/server-renderer'

const app = createSSRApp({
  data: () => ({ msg: 'hello' }),
  template: `<div>{{ msg }}</div>`
})

;(async () => {
  const html = await renderToString(app)
  console.log(html)
})()
```

Then run:

```sh
> node example.mjs
```

...which should print the following:

```html
<div>hello</div>
```

[`renderToString()`](/api/ssr.html#rendertostring) takes a Vue app instance and returns a Promise that resolves to the rendered HTML of the app. It is also possible to perform streaming render using [Node.js Stream API](https://nodejs.org/api/stream.html) or [Web Streams API](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API). Check out the [SSR API Reference](/api/ssr.html) for full details.

### Client Hydration

In actual SSR applications, the server-rendered markup is typically embedded in an HTML page like this:

```html{6}
<!DOCTYPE html>
<html>
  <head>...</head>
  <body>
    <div id="app">
      <div>hello</div> <!-- server-rendered content -->
    </div>
  </body>
</html>
```

On the client side, Vue needs to perform the **hydration** step. It creates the same Vue application that was run on the server, matches each component to the DOM nodes it should control, and attaches event listeners so the app becomes interactive.

The only thing different from a client-only app is that we need to use [`createSSRApp()`](/api/application.html#createssrapp) instead of `createApp()`:

```js{2}
// this runs in the browser.
import { createSSRApp } from 'vue'

const app = createSSRApp({
  // ...same app as on server
})

// mounting an SSR app on the client assumes
// the HTML was pre-rendered and will perform
// hydration instead of mounting new DOM nodes.
app.mount('#app')
```

## Higher Level Solutions

While the examples so far are relatively simple, production-ready SSR apps are fullstack projects that involve a lot more than just Vue APIs. We will need to:

- Build the app twice: once for the client, and once for the server.

  :::tip
  Vue components are compiled differently when used for SSR - templates are compiled into string concatenations instead of Virtual DOM render functions for more efficient rendering performance.
  :::

- In the server request handler, render the HTML page with the correct outer shell and app markup, including client-side asset links and resource hints. We may also need to switch between SSR and SSG mode, or even mix both in the same app.

- Manage routing, data fetching, and state management stores in a universal manner.

This is quite advanced and highly dependent on the built toolchain you have chosen to work with. Therefore, we highly recommend going with a higher-level, opinionated solution that abstracts away the complexity for you. Below we will introduce a few recommended SSR solutions in the Vue ecosystem.

### Nuxt

[Nuxt](https://v3.nuxtjs.org/) is a higher-level framework built on top of the Vue ecosystem which provides a streamlined development experience for writing universal Vue applications. Better yet, you can also use it as a static site generator! We highly recommend giving it a try.

### Quasar

[Quasar](https://quasar.dev) is a complete Vue-based solution that allows you to target SPA, SSR, PWA, mobile app, desktop app, and browser extension all using one codebase. It not only handles the build setup, but also provides a full collection of Material Design compliant UI components.

### Vite SSR

Vite provides built-in [support for Vue server-side rendering](https://vitejs.dev/guide/ssr.html), but it is intentionally low-level. If you wish to go directly with Vite, check out [vite-plugin-ssr](https://vite-plugin-ssr.com/), a community plugin that abstracts away many challenging details for you.

You can also find an example Vue + Vite SSR project using manual setup [here](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-vue), which can serve as a base to build upon. Note this is only recommended if you are experienced with SSR / build tools and really want to have complete control over the higher-level architecture.

## Writing SSR-friendly Code

Regardless of your build setup or higher-level framework choice, there are some principles that apply in all Vue SSR applications.

### Reactivity on the Server

During SSR, each request URL maps to a desired state of our application. There is no user interaction and no DOM updates, so reactivity is unnecessary on the server. By default, reactivity is disabled during SSR for better performance.

### Component Lifecycle Hooks

Since there are no dynamic updates, lifecycle hooks such as <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span> or <span class="options-api">`updated`</span><span class="composition-api">`onUpdated`</span> will **NOT** be called during SSR and will only be executed on the client.<span class="options-api"> The only hooks that are called during SSR are `beforeCreate` and `created`</span>

You should avoid code that produces side effects that need cleanup in <span class="options-api">`beforeCreate` and `created`</span><span class="composition-api">`setup()` or the root scope of `<script setup>`</span>. An example of such side effects is setting up timers with `setInterval`. In client-side only code we may setup a timer and then tear it down in <span class="options-api">`beforeUnmount`</span><span class="composition-api">`onBeforeUnmount`</span> or <span class="options-api">`unmounted`</span><span class="composition-api">`onUnmounted`</span>. However, because the unmount hooks will never be called during SSR, the timers will stay around forever. To avoid this, move your side-effect code into <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span> instead.

### Access to Platform-Specific APIs

Universal code cannot assume access to platform-specific APIs, so if your code directly uses browser-only globals like `window` or `document`, they will throw errors when executed in Node.js, and vice-versa.

For tasks shared between server and client but use different platform APIs, it's recommended to wrap the platform-specific implementations inside a universal API, or use libraries that do this for you. For example, you can use [`node-fetch`](https://github.com/node-fetch/node-fetch) to use the same fetch API on both server and client.

For browser-only APIs, the common approach is to lazily access them inside client-only lifecycle hooks such as <span class="options-api">`mounted`</span><span class="composition-api">`onMounted`</span>.

Note that if a 3rd party library is not written with universal usage in mind, it could be tricky to integrate it into an server-rendered app. You _might_ be able to get it working by mocking some of the globals, but it would be hacky and may interfere with the environment detection code of other libraries.

### Cross-Request State Pollution

In the State Management chapter, we introduced a [simple state management pattern using Reactivity APIs](state-management.html#simple-state-management-with-reactivity-api). In an SSR context, this pattern requires some additional adjustments.

The pattern declares shared state as **singletons**. This means there is only once instance of the reactive object throughout the entire lifecycle of our application. This works as expected in a pure client-side Vue application, since the our application code is initialized fresh for each browser page visit.

However, in an SSR context, the application code is typically initialized only once on the server, when the server boots up. In such case, singletons in our application will be shared across multiple requests handled by the server! If we mutate the shared singleton store with data specific to one user, it can be accidentally leaked to a request from another user. We call this **cross-request state pollution.**

To workaround this, we need to create a fresh instance of the application and the shared object on each request. Then, instead of directly importing it in our components, we provide the shared state using [app-level provide](/guide/components/provide-inject.html#app-level-provide) and inject it in components that need it.

State Management libraries like Pinia are designed with this in mind. Consult [Pinia's SSR guide](https://pinia.vuejs.org/ssr/) for more details.

### Hydration Mismatch

If the DOM structure of the pre-rendered HTML does not match the expected output of the client-side app, there will be a hydration mismatch error. In most cases, this is caused by browser's native HTML parsing behavior trying to correct invalid structures in the HTML string. For example, a common gotcha is that [`<div>` cannot be placed inside `<p>`](https://stackoverflow.com/questions/8397852/why-cant-the-p-tag-contain-a-div-tag-inside-it):

```html
<p><div>hi</div></p>
```

If we produce this in our server-rendered HTML, the browser will terminate the first `<p>` when `<div>` is encountered and parse it into the following DOM structure:

```html
<p></p>
<div>hi</div>
<p></p>
```

When Vue encounters a hydration mismatch, it will attempt to automatically recover and adjust the pre-rendered DOM to match the client side state. This will lead to some rendering performance loss due to incorrect nodes being discarded and new nodes being mounted, but in most cases, the app should continue to work as expected. That said, it is still best to eliminate hydration mismatches during development.

### Custom Directives

Since most custom directives involve direct DOM manipulation, they are ignored during SSR.

You can provide a transform function to implement the server-side rendering logic for a custom directive. The function should be passed under the `directiveTransforms` option to `@vue/compiler-dom`.

Example Vite config that provides an empty stub for a custom `v-focus` directive:

```js
import vue from '@vitejs/plugin-vue'

export default {
  plugins: [
    vue({
      template: {
        compilerOptions: {
          directiveTransforms: {
            // an empty array indicates that v-focus
            // does not add any rendered attributes to the element.
            focus: () => ({ props: [] })
          }
        }
      }
    })
  ]
}
```

Writing a proper directive transform requires TypeScript proficiency and knowledge of Vue's compiler API. We plan to add documentation for the compiler API in the future, but for now, you will need to consult the [source code](https://github.com/vuejs/vue-next/blob/master/packages/compiler-core/src/transform.ts#L53-L63) to learn more about it.

:::tip
Currently, the SSR compiler will throw an error if a server-side transform is not found for a custom directive. This behavior will be adjusted to a warning in the future.
:::
