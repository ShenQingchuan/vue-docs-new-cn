# 生命周期钩子 {#lifecycle-hooks}

每个 Vue 组件实例都需要在创建时走过一系列的初始化步骤，比如设置好数据观察，编译模板，挂载实例到 DOM 并在数据改变时更新 DOM。在此过程中，它还运行称为生命周期钩子的函数，让用户有机会在特定阶段添加自己的代码。

## 注册周期钩子 {#registering-lifecycle-hooks}

举个例子，<span class="composition-api">`onMounted`</span><span class="options-api">`mounted`</span> 钩子可以用来在组件完成初始渲染并创建 DOM 节点后运行代码。

<div class="composition-api">

```vue
<script setup>
import { onMounted } from 'vue'

onMounted(() => {
  console.log(`组件现在已经被挂载。`)
})
</script>
```

</div>
<div class="options-api">

```js
export default {
  mounted() {
    console.log(`组件现在已经被挂载。`)
  }
}
```

</div>

还有其他一些钩子，会在实例生命周期的不同阶段被调用，最常用的是 <span class="composition-api">[`onMounted`](/api/composition-api-lifecycle.html#onmounted)，[`onUpdated`](/api/composition-api-lifecycle.html#onupdated) 和 [`onUnmounted`](/api/composition-api-lifecycle.html#onunmounted)。所有生命周期钩子的完整参考及其用法可以在 [这里](/api/composition-api-lifecycle.html) 看到。</span><span class="options-api">[`mounted`](/api/options-lifecycle.html#mounted)，[`updated`](/api/options-lifecycle.html#updated) 和 [`unmounted`](/api/options-lifecycle.html#unmounted)。所有的钩子都会在相应生命周期阶段，带着当前活跃的组件实例作为上下文被调用。所有生命周期钩子的完整参考及其用法可以在 [这里](/api/composition-api-lifecycle.html)。</span>

<div class="composition-api">

当调用 `onMounted` 时，Vue 会自动将注册的回调函数与当前活动组件实例相关联。这就要求这些钩子在组件设置时同步注册。例如请不要这样做：

```js
setTimeout(() => {
  onMounted(() => {
    // 这将不会正常工作
  })
}, 100)
```

</div>

## 生命周期图示 {#lifecycle-diagram}

下面是实例生命周期的图表。您不需要完全理解当前正在进行的所有事情，但随着您学习和构建更多内容，它将是一个有用的参考。

![Component lifecycle diagram](/images/lifecycle.svg)
