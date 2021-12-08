# 利用组合复用逻辑 {#reusing-logic-with-composables}

### 保持响应性 {#retaining-reactivity}

当我们想要使用一个很大的响应式对象上的一小部分属性时，可能立即先想到的是使用解构。然而解构出的属性会丢失与原代理的响应性链接。

```js
const state = reactive({
  count: 0
  // ... 许多其他属性
})

// `count` 一旦被解构就不再具有响应性
// 因为此时他只是一个 number 类型的值
const { count } = state
```

你可以使用 [`toRef()`](/api/reactivity-utilities.html#toref) 方法依靠响应式对象中的某个属性创建一个 ref：

```js
import { toRef } from 'vue'

const count = toRef(state, 'count')

state.count++
console.log(count.value) // 1
```
