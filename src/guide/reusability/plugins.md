# 插件 {#plugins}

插件是一种能为 Vue 添加全局功能的工具代码。它可以一个拥有 `install()` 方法的对象，或者是一个 `function`。

插件没有严格定义的作用域，但是插件发挥作用的常见场景主要包括以下几种：

1. 添加一些全局方法或属性，例如 [vue-custom-element](https://github.com/karol-f/vue-custom-element)。

2. 添加一种到多种全局资源：指令/过滤器/过渡等等。例如 [vue-touch](https://github.com/vuejs/vue-touch))。

3. 通过全局混入（mixin）添加一些组件选项。例如 [vue-router](https://github.com/vuejs/vue-router))。

4. 向 `config.globalProperties` 上添加一些全局实例方法。

5. 一个提供一整套自己的 API 的第三方库，同时也可能会囊括上述部分。例如 [vue-router](https://github.com/vuejs/vue-router))。

## 编写一个插件 {#writing-a-plugin}

为了更好地理解如何构建 Vue.js 插件，我们可以试着做一个简单的 `i18n` 插件来学习。

当插件被添加到应用中时，若其为一个对象，则会调用 `install` 方法，若其为一个 `function`，则会调用该函数。在这两种场景中，都会接收到两个参数。`app` 是 Vue 的 `createApp` 方法返回的结果，第二个则是用户传入的选项。

让我们从设置插件对象开始。建议在一个单独的文件中创建并导出它，以保证更好的管理逻辑，如下所示：

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    // 插件代码在这里书写
  }
}
```

我们想让整个应用程序有一个按 key 名翻译文本内容的函数，因此我们将它暴露在 `config.globalProperties` 上。

这个函数接受一个 `key` 字符串作参数，用来在用户提供的翻译字典中查找对应语言的文本。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = (key) => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }
  }
}
```

我们假设用户在使用插件时传入了一个 `options` 对象作参数，包含了所有要翻译的 key 名。而 `$translate` 函数会接收 `greetings.hello` 这样的索引，并据此在用户提供的翻译字典中取出相应语言的文本，在这里是 `Bonjour!`。

Ex:

```js
greetings: {
  hello: 'Bonjour!',
}
```

我们还可以使用 `provide` 来供给一个函数或 attribute 给插件的用户。举个例子，我们可以让应用内全局可以访问 `options` 参数，以便能够在各处都能使用这个翻译字典对象。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = (key) => {
      return key.split('.').reduce((o, i) => {
        if (o) return o[i]
      }, options)
    }

    app.provide('i18n', options)
  }
}
```

插件用户可以在它们的组件中用 `inject('i18n')` 来注入并访问该对象。

此外，因为我们可以访问到 `app` 对象，因此像 `mixin` 和 `directive` 这样的功能都在插件代码中可用。要了解 `createApp` 这个方法返回的应用实例的更多细节，请参阅 [应用 API 文档](/api/application.html)。

```js
// plugins/i18n.js
export default {
  install: (app, options) => {
    app.config.globalProperties.$translate = (key) => {
      return key.split('.')
        .reduce((o, i) => { if (o) return o[i] }, options)
    }

    app.provide('i18n', options)

    app.directive('my-directive', {
      mounted (el, binding, vnode, oldVnode) {
        // 执行一些逻辑 ...
      }
      ...
    })

    app.mixin({
      created() {
        // 执行一些逻辑 ...
      }
      ...
    })
  }
}
```

## 使用插件 {#using-a-plugin}

在通过 `createApp()` 初始化完一个 Vue 应用之后，你可以通过 `use()` 方法为其添加一个插件。

我们可以将 [上面的示例](#writing-a-plugin) 中制作的插件导出为 `i18nPlugin` 并添加进应用中。

`use()` 方法接收两个参数，第一个是要安装的插件，即我们的 `i18nPlugin` 对象。

它同时也会自动帮你避免重复注册该插件，因此多次对同一个插件调用该方法也只相当于执行了一次。

第二个参数是可选的，并取决于安装的插件本身。例如这里的 `i18nPlugin`，它需要是一个翻译字典对象。

:::info
如果你正在使用像 `Vuex` 或者 `Vue Router` 这样的第三方库，请先查阅它们的文档，了解每个插件需要在安装时传入什么内容作为这里的第二个参数。
:::

```js
import { createApp } from 'vue'
import Root from './App.vue'
import i18nPlugin from './plugins/i18n'

const app = createApp(Root)
const i18nStrings = {
  greetings: {
    hi: 'Hallo!'
  }
}

app.use(i18nPlugin, i18nStrings)
app.mount('#app')
```

你还可以查看 [awesome-vue](https://github.com/vuejs/awesome-vue#components--libraries) 这个列表，这是一个由社区贡献的插件和第三方库大合集。
