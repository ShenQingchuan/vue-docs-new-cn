# Server-Side Rendering API

## renderToString()

> Exported from `vue/server-renderer`

**Signature**

```ts
function renderToString(
  input: App | VNode,
  context?: SSRContext
): Promise<string>
```

**Usage**

```js
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

### Handling Teleports

If the rendered app contains teleports, the teleported content will not be part of the rendered string. Instead, they are exposed under the `teleports` property of the ssr context object:

```js
const ctx = {}
const html = await renderToString(app, ctx)

console.log(ctx.teleports) // { '#teleported': 'teleported content' }
```

## renderToNodeStream()

Renders input as a [Node.js Readable stream](https://nodejs.org/api/stream.html#stream_class_stream_readable).

**Signature**

```ts
function renderToNodeStream(input: App | VNode, context?: SSRContext): Readable
```

**Usage**

```js
// inside a Node.js http handler
renderToNodeStream(app).pipe(res)
```

**Note:** This method is not supported in the ESM build of `vue/server-renderer`, which is decoupled from Node.js environments. Use `pipeToNodeWritable` instead.

## pipeToNodeWritable()

Render and pipe to an existing [Node.js Writable stream](https://nodejs.org/api/stream.html#stream_writable_streams) instance.

**Signature**

```ts
function pipeToNodeWritable(
  input: App | VNode,
  context: SSRContext = {},
  writable: Writable
): void
```

**Usage**

```js
// inside a Node.js http handler
pipeToNodeWritable(app, {}, res)
```

## renderToWebStream()

Renders input as a [Web ReadableStream](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API).

**Signature**

```ts
function renderToWebStream(
  input: App | VNode,
  context?: SSRContext
): ReadableStream
```

**Usage**

```js
// inside an environment with ReadableStream support
return new Response(renderToWebStream(app))
```

**Note:** in environments that do not expose `ReadableStream` constructor in the global scope, `pipeToWebWritable` should be used instead.

## pipeToWebWritable()

Render and pipe to an existing [Web WritableStream](https://developer.mozilla.org/en-US/docs/Web/API/WritableStream) instance.

**Signature**

```ts
function pipeToWebWritable(
  input: App | VNode,
  context: SSRContext = {},
  writable: WritableStream
): void
```

**Usage**

This is typically used in combination with [`TransformStream`](https://developer.mozilla.org/en-US/docs/Web/API/TransformStream):

```js
// TransformStream is available in environments such as CloudFlare workers.
// in Node.js, TransformStream needs to be explicitly imported from 'stream/web'
const { readable, writable } = new TransformStream()
pipeToWebWritable(app, {}, writable)

return new Response(readable)
```

## renderToSimpleStream()

Renders input in streaming mode using a simple readable interface.

**Signature**

```ts
function renderToSimpleStream(
  input: App | VNode,
  context: SSRContext,
  options: SimpleReadable
): SimpleReadable

interface SimpleReadable {
  push(content: string | null): void
  destroy(err: any): void
}
```

**Usage**

```js
let res = ''

renderToSimpleStream(
  app,
  {},
  {
    push(chunk) {
      if (chunk === null) {
        // done
        console(`render complete: ${res}`)
      } else {
        res += chunk
      }
    },
    destroy(err) {
      // error encountered
    }
  }
)
```

## useSSRContext()

**Signature**

```ts
function useSSRContext<T = Record<string, any>>(): T | undefined
```

**Usage**

`useSSRContext()` is used to retrieve the context object passed to `renderToString()` or other server render APIs:

```vue
<script setup>
import { useSSRContext } from 'vue'

// make sure to only call it during SSR
// https://vitejs.dev/guide/ssr.html#conditional-logic
if (import.meta.env.SSR) {
  const ctx = useSSRContext()
  // ...attach properties to the context
}
</script>
```

This can be used to attach information that is needed for rendering the final HTML (e.g. head metadata) to the render context.
