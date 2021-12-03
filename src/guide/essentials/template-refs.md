# 模板 ref {#template-refs}

虽然 Vue 的声明性渲染模型为您抽象了大部分对 DOM 的直接操作，但在某些情况下，我们仍然需要直接访问底层 DOM 元素。要实现这一点，我们可以使用特殊的 `ref` attribute：

```vue-html
<input ref="input">
```

`ref` 是一个特殊的 attribute ，和 `v-for` 章节中提到的 `key` 类似。它允许我们在一个特定的 DOM 元素或子组件实例被挂载后，获得对它的直接引用。这可能很有用，比如说在组件挂载时编程式地聚焦到一个 input 元素上，或在一个元素上初始化一个第三方库。

## 访问模板 ref {#accessing-the-refs}

<div class="composition-api">

为了通过组合式 API 获得该模板 ref，我们需要声明一个同名的 ref：

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 声明一个 ref 来存放该元素的引用
// 必须和模板 ref 同名
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>
```

</div>
<div class="options-api">

挂载结束后 ref 都会被暴露在 `this.$refs` 之上：

```vue
<script setup>
export default {
  mounted() {
    this.$refs.input.focus()
  }
}
</script>

<template>
  <input ref="input" />
</template>
```

</div>

注意，你只可以 **在组件挂载后** 才能访问 ref。如果你想在模板中的表达式上访问 `$refs.input`，在初次渲染时会是 `null`。这是因为在初次渲染前这个元素还压根不存在呢！

<div class="composition-api">
如果你正试图观察一个模板 ref 的变化，确保考虑到 ref 的值为 `null` 的情况：

```js
watchEffect(() => {
  if (input.value) {
    input.value.focus()
  } else {
    // 此时还未挂载，或此元素已经被卸载（例如经过 v-if 控制）
  }
})
```

</div>

## 函数式 ref {#function-refs}

除了使用字符串值作名字，`ref` attribute 还可以绑定为一个函数，会在每次组件更新时都被调用。函数接受该元素引用作为第一个参数：

```vue-html
<input :ref="(el) => { /* 可以将 el 赋值给一个属性或 ref */ }">
```

如果你想，也可以使用方法名而非内联函数。

## `v-for` 中的 ref {#refs-inside-v-for}

和 Vue 2 不同，Vue 3 在 `v-for` 中使用时模板 ref 不会自动依附于一个数组。但你仍然可以使用函数式 ref 来做到这一点：

<div class="composition-api">

```vue
<script setup>
import { ref, onBeforeUpdate } from 'vue'

const list = ref([
  /* ... */
])

let itemRefs = []

onBeforeUpdate(() => {
  // 在每次更新时都重设该数组
  itemRefs = []
})
</script>

<template>
  <div v-for="item in list" :ref="(el) => el && itemRefs.push(el)"></div>
</template>
```

</div>
<div class="options-api">

```vue
<script>
export default {
  data() {
    return {
      list: [
        /* ... */
      ],
      itemRefs: []
    }
  },
  beforeUpdate() {
    // 在每次更新时都重设该数组
    this.itemRefs = []
  }
}
</script>

<template>
  <div v-for="item in list" :ref="(el) => el && itemRefs.push(el)"></div>
</template>
```

</div>

## 组件上的 ref {#ref-on-component}

> 这一小节假设你已了解 [组件](/guide/essentials/component-basics) 的相关知识，或者你也可以先跳过这里，之后再回来看。

`ref` 也可以被用在一个子组件上。此时 ref 中引用的是组件实例：

<div class="composition-api">

```vue
<script setup>
import { ref, onMounted } from 'vue'
import Child from './Child.vue'

const child = ref(null)

onMounted(() => {
  // child.value 为 <Child /> 这个组件实例
})
</script>

<template>
  <Child ref="child" />
</template>
```

</div>
<div class="options-api">

```vue
<script>
import Child from './Child.vue'

export default {
  components: {
    Child
  },
  mounted() {
    // this.$refs.child 为 <Child /> 这个组件实例
  }
}
</script>

<template>
  <Child ref="child" />
</template>
```

</div>

如果一个子组件使用的是选项式 API 或没有使用 `<script setup>`，被引用的组件实例和该子组件的 `this` 完全一致，这意味着父组件对子组件的每一个属性和方法都有完全的访问权。这使得这使得在父组件和子组件之间创建紧密耦合的实现细节变得很容易，当然也因此，应该只在绝对需要时才使用组件引用。大多数情况下，你应该首先使用标准的 props 和 emit 接口来实现父子组件交互。

<div class="composition-api">

有一个例外的情况，使用了 `<script setup>` 的组件时 **默认私有** 的：一个父组件无法访问到一个使用了 `<script setup>` 的子组件中的任何东西，除非子组件在其中通过 `defineExpose` 宏显式暴露：

```vue
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

defineExpose({
  a,
  b
})
</script>
```

当父组件通过模板 ref 获取到了该组件的实例，得到的实例类型为 `{ a: number, b: number }`（ref 都会自动解套，和一般的实例一样）。

</div>
<div class="options-api">

`expose` 选项可以用于限制对子组件实例的访问：

```js
export default {
  expose: ['publicData', 'publicMethod'],
  data() {
    return {
      publicData: 'foo',
      privateData: 'bar'
    }
  },
  methods: {
    publicMethod() { /* ... */ },
    privateMethod() { /* ... */ }
  }
}
```

在上面这个例子中，父组件通过模板 ref 访问到子组件实例后，仅能访问 `publicData` 和 `publicMethod`。

</div>
