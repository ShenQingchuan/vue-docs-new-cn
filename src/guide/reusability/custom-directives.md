# 自定义指令 {#custom-directives}

## 介绍 {#intro}

除了 Vue 内置的一系列指令（比如 `v-model` 或 `v-show`）之外，Vue 还允许你注册自定义的指令。虽然在 Vue 中，最基本的抽象和复用代码的方式是使用组件，但也有一些场景需要去访问底层 DOM 元素，此时自定义指令就非常有用了。我们举一个用来聚焦在某个 input 元素上的指令为例：

<!-- <common-codepen-snippet title="自定义指令基础实例" slug="JjdxaJW" :preview="false" /> -->

页面加载完成后，只要浏览页面时没有点击别处，input 元素会自动聚焦（请注意：`autofocus` attribute 在移动端 Safari 上不可用）。你也可以点击 `重试` 按钮，让 input 元素重新聚焦。

现在让我们看看如何写一个指令来做到这件事情：

```js
const app = Vue.createApp({})
// 注册一个全局可用的自定义指令，名为 `v-focus`
app.directive('focus', {
  // 当绑定的元素在 DOM 中挂载完成后...
  mounted(el) {
    // 聚焦该元素
    el.focus()
  }
})
```

如果你想要在组件局部注册指令，也可以在组件中使用 `directives` 选项：

```js
directives: {
  focus: {
    // 指令定义
    mounted(el) {
      el.focus()
    }
  }
}
```

接下来在模板中，你可以像这样在任意元素上使用 `v-focus` attribute：

```vue-html
<input v-focus />
```

## 钩子函数 {#hook-functions}

一个指令的定义是一个提供几个钩子函数的对象（所有钩子函数都是可选的）：

- `created`：在绑定元素的 attribute 或事件监听器应用之前被调用。有时我们需要一些必须在一般的 ` v-on` 事件监听器之前调用的监听器，这种情况会很有用。

- `beforeMount`：在指令首次绑定到元素并在父组件挂载此元素之前调用。

- `mounted`：在父组件挂载此元素完成后调用。

- `beforeUpdate`：在包含此指令的组件的对应 VNode 更新之前调用

:::tip 注意
我们会在 [之后] (/guide/advanced/render-function.html#the-virtual-dom-tree) 的章节中讨论到 VNode。
:::

- `updated`：在包含此指令的组件的对应的 VNode 及其所有子 VNode 更新完成后调用

- `beforeUnmount`：在父组件卸载此元素之前调用。

- `unmounted`：指令从元素中解绑，父元素卸载该元素之后调用，仅会调用一次。

你可以在 [自定义指令 API](/api/application.html#app-directive) 查阅传给这些钩子函数的参数（比如：`el`，`binding`，`vnode` 和`prevVnode`）。

### 动态指令参数 {#dynamic-directive-arguments}

指令参数可以是动态的，举个例子，在 `v-mydirective:[argument]="value"` 中，`argument` 可以基于组件实例的数据属性而更新！这使得我们能更灵活地在应用中使用该自定义指令。

现在我们想做这样一个指令，让某个元素在页面中变为固定定位（`position: fixed`）。并且我们可以在指令中精确到像素地更新元素垂直方向上的位置，就像这样：

```vue-html
<div id="dynamic-arguments-example" class="demo">
  <p>向下滚动页面 ↓</p>
  <p v-pin="200">我被挂在距离页面顶部 200px 的地方</p>
</div>
```

```js
const app = Vue.createApp({})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.value 就是传给该指令的值，在这个例子中就是 200
    el.style.top = binding.value + 'px'
  }
})

app.mount('#dynamic-arguments-example')
```

这将把元素钉在距离页面顶部 200px 的位置。但是，如果我们遇到需要从左侧而不是顶部固定元素的情况，该怎么办呢？我们可以创建一个动态参数，并基于它来更新组件：

```vue-html
<div id="dynamicexample">
  <h3>向下滚动页面 ↓</h3>
  <p v-pin:[direction]="200">我被挂在距离页面左边 200px 的地方</p>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      direction: 'left'
    }
  }
})

app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    // binding.arg 就是传给指令的参数
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})

app.mount('#dynamic-arguments-example')
```

得到的结果是这样：

<!-- <common-codepen-snippet title="Custom directives: dynamic arguments" slug="YzXgGmv" :preview="false" /> -->

现在我们的自定义指令已经足够灵活，能够支持一些不同的使用场景了。为了使其更具动态性，我们还可以使用动态的绑定值。让我们再创建一个数据属性 `pinPadding` 并将其绑定到  `<input type="range">`。（译者注：例子中的 `_i18n` 是一个假想的翻译函数，表示根据描述方向的英文单词翻译为中文）

```vue-html{4}
<div id="dynamicexample">
  <h2>向下滚动页面 ↓</h2>
  <input type="range" min="0" max="500" v-model="pinPadding">
  <p v-pin:[direction]="pinPadding">我被挂载在距离页面 {{ _i18n(direction, 'zh-CN') || '上方' }} {{ pinPadding + 'px' }} 的地方</p>
</div>
```

```js{5}
const app = Vue.createApp({
  data() {
    return {
      direction: 'right',
      pinPadding: 200
    }
  }
})
```

现在让我们扩展该自定义指令的逻辑，重新计算组件更新时的固定距离:

```js{7-10}
app.directive('pin', {
  mounted(el, binding) {
    el.style.position = 'fixed'
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  },
  updated(el, binding) {
    const s = binding.arg || 'top'
    el.style[s] = binding.value + 'px'
  }
})
```

得到的结果是这样：

<!-- <common-codepen-snippet title="Custom directives: dynamic arguments + dynamic binding" slug="rNOaZpj" :preview="false" /> -->

## 简化形式 {#function-shorthand}

在之前的例子中，你可能只想要 `mounted` 和 `updated` 的行为，而并不关心其他钩子。那么你可以只给指令传一个下面这样的函数：

```js
app.directive('pin', (el, binding) => {
  el.style.position = 'fixed'
  const s = binding.arg || 'top'
  el.style[s] = binding.value + 'px'
})
```

## 对象字面量 {#object-literals}

如果你的指令需要多个值，你可以向它传递一个 JavaScript 对象字面量。请记住，指令也可以接收任何合法的 JavaScript 表达式。

```vue-html
<div v-demo="{ color: '白色', text: '你好!' }"></div>
```

```js
app.directive('demo', (el, binding) => {
  console.log(binding.value.color) // => "白色"
  console.log(binding.value.text) // => "你好!"
})
```

## 在组件上使用 {#usage-on-components}

当在组件上使用自定义指令时，它会始终应用于组件的根节点，和 [透传 attributes](/guide/components/attrs.html) 类似。

```vue-html
<my-component v-demo="test"></my-component>
```

```js
app.component('my-component', {
  template: `
    <div> // v-demo 指令会应用在这里
      <span>My component content</span>
    </div>
  `
})
```

而和 attribute 不同的是，指令不能通过类似 `v-bind="$attrs"` 的方式传递给一个别的元素。

如果组件可能含有多个根节点，指令不会起效、被忽略，还会抛出一个警告。
