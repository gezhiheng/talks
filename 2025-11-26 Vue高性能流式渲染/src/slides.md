---
theme: seriph
title: Vue高性能流式渲染
info: |
  分享主题：Vue 高性能流式渲染与 `markstream-vue`
class: text-center
transition: slide-left
mdc: true
drawings:
  persist: false
---

# Vue高性能流式渲染

更快、更稳、更流畅的 Markdown 流式输出

<div class="absolute bottom-4 right-4 text-sm opacity-50">
  分享人: 葛智恒
</div>

<!-- Presenter Notes:
开场：为什么今天聊“流式渲染”，侧重真实业务痛点： - 大模型、AI 助手、在线文档编辑器的普及，流式输出变成标配体验。 - 但大部分实现仍停留在“能看就行”，体验和性能问题很多。 - 今天从「模式对比 → 典型问题 → 方案设计 → 实战库」这条线讲清楚。
-->

---
layout: center
---

# 传统「一次性渲染」有什么问题？

<div class="flex flex-col mt-8 px-10 max-w-3xl mx-auto text-left">
  <v-click>
    <div class="flex items-center gap-4 p-4 border border-gray-400/20 rounded-xl bg-gray-500/5 shadow-sm">
      <div class="i-carbon-cloud-download text-3xl text-blue-500" />
      <div class="flex flex-col">
        <span class="text-lg font-bold">1. 等待网络 IO</span>
        <span class="text-sm opacity-60">必须下载完所有数据才能开始处理</span>
      </div>
    </div>
  </v-click>

  <v-click>
    <div class="flex justify-center py-2">
      <div class="i-carbon-arrow-down text-2xl opacity-30" />
    </div>
    <div class="flex items-center gap-4 p-4 border border-gray-400/20 rounded-xl bg-gray-500/5 shadow-sm">
      <div class="i-carbon-code text-3xl text-orange-500" />
      <div class="flex flex-col">
        <span class="text-lg font-bold">2. 等待解析</span>
        <span class="text-sm opacity-60">全量解析 Markdown，主线程长时间阻塞</span>
      </div>
    </div>
  </v-click>

  <v-click>
    <div class="flex justify-center py-2">
      <div class="i-carbon-arrow-down text-2xl opacity-30" />
    </div>
    <div class="flex items-center gap-4 p-4 border border-gray-400/20 rounded-xl bg-gray-500/5 shadow-sm">
      <div class="i-carbon-screen text-3xl text-green-500" />
      <div class="flex flex-col">
        <span class="text-lg font-bold">3. 最后渲染</span>
        <span class="text-sm opacity-60">内容一次性上屏，缺乏渐进感</span>
      </div>
    </div>
  </v-click>
</div>

---
layout: center
---

# 「流式渲染」的优势

<div class="grid grid-cols-2 gap-10 mt-12 px-4">
  <v-click>
    <div class="flex flex-col items-center p-8 border border-blue-500/20 rounded-2xl bg-blue-500/5 shadow-lg transition hover:scale-105">
      <div class="i-carbon-flash text-6xl text-blue-500 mb-6" />
      <h3 class="text-2xl font-bold mb-3">首屏极速响应</h3>
      <p class="text-lg opacity-75 text-center leading-relaxed">
        打破“全量等待”<br/>让用户<span class="text-blue-500 font-bold">毫秒级</span>看到核心内容
      </p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-8 border border-purple-500/20 rounded-2xl bg-purple-500/5 shadow-lg transition hover:scale-105">
      <div class="i-carbon-flow text-6xl text-purple-500 mb-6" />
      <h3 class="text-2xl font-bold mb-3">体验丝滑流畅</h3>
      <p class="text-lg opacity-75 text-center leading-relaxed">
        消除长时间白屏与掉帧<br/>提供 <span class="text-purple-500 font-bold">实时反馈</span>
      </p>
    </div>
  </v-click>
</div>

<!-- Presenter Notes: 强调一点：流式渲染不仅是“更快”，更重要是主观体验——用户感觉页面一直在工作，而不是在“卡”。 -->

---
layout: section
transition: slide-up
---
# 几种「流式」模式对比

---
layout: two-cols-header
transition: slide-up
---

# 模式 1：伪流式 (Append String)

::left::

```ts {all|4|7|all}
// ❌ 简单粗暴，但体验较差
const target = document.getElementById('el')

for await (const chunk of stream) {
  // 仅仅是字符串拼接
  // 无法处理 Markdown 语法
  target.textContent += chunk
}
```

::right::

<div class="flex flex-col gap-4 mt-10 ml-4">
  <v-clicks>
    <div class="flex items-center gap-2 text-green-500">
      <div class="i-carbon-checkmark-outline" />
      <span>实现极简，首屏极快</span>
    </div>
    <div class="flex items-center gap-2 text-yellow-500">
      <div class="i-carbon-warning" />
      <span>丢失 Markdown 格式 (粗体/链接)</span>
    </div>
    <div class="flex items-center gap-2 text-yellow-500">
      <div class="i-carbon-warning" />
      <span>代码块/表格无法正确渲染</span>
    </div>
  </v-clicks>
</div>

---
layout: two-cols-header
transition: slide-up
---

# 模式 2：全量累积 & 重解析 & 重建 DOM

::left::

```ts {all|4-5|7-8|10-11|all}
// ⚠️ 传统做法：解析 + 渲染都是全量
let accumulated = ''

for await (const chunk of stream) {
  accumulated += chunk

  // 每次都从头解析整个文档
  const html = md.render(accumulated)

  // 每次都销毁并重建整块 DOM
  el.innerHTML = sanitize(html)
}
```

::right::

<div class="flex flex-col gap-4 mt-10 ml-4">
  <v-clicks>
    <div class="flex items-center gap-2 text-green-500">
      <div class="i-carbon-checkmark-outline" />
      <span>支持完整 Markdown 语法</span>
    </div>
    <div class="flex items-center gap-2 text-yellow-500">
      <div class="i-carbon-warning" />
      <span>O(n²) 复杂度：长文卡顿严重</span>
    </div>
    <div class="flex items-center gap-2 text-yellow-500">
      <div class="i-carbon-warning" />
      <span>频繁 GC 与 DOM 重排 (闪烁)</span>
    </div>
    <div class="flex items-center gap-2 text-yellow-500">
      <div class="i-carbon-warning" />
      <span>用户选区/焦点容易丢失</span>
    </div>
  </v-clicks>
</div>

---
layout: two-cols-header
transition: slide-up
---

# 模式 3：全量 AST & 渲染层增量

::left::

```ts {all|4-5|7-8|10-11|all}
// ✅ 解析全量，但渲染做了精细优化
let content = ''

for await (const chunk of stream) {
  content += chunk

  // 1. 每次都把完整 content 解析成 AST
  const nodes = parseMarkdownToStructure(content, md)

  // 2. 交给 Vue 组件做渲染层优化
  render(<MarkdownRender :nodes="nodes" />)
}
```

::right::

<div class="flex flex-col gap-4 ml-4">
  <v-clicks depth="2">
    <div class="flex items-center gap-2 text-xl text-purple-600">
      <div class="i-carbon-cube text-3xl" />
      <span>AST 驱动的组件化渲染</span>
    </div>
    <div class="flex items-center gap-2 text-green-500">
      <div class="i-carbon-checkmark-outline" />
      <span>始终保持「完整 Markdown 语义」</span>
    </div>
    <div class="flex items-center gap-2 text-green-500">
      <div class="i-carbon-checkmark-outline" />
      <span>渲染层做增量优化</span>
    </div>
    <div class="flex items-center gap-2 text-green-500">
      <div class="i-carbon-checkmark-outline" />
      <span>按需加载组件</span>
    </div>

  <div class="p-3 bg-orange-500/10 rounded-lg border border-orange-500/20">
      <span v-mark.circle.orange="10">markstream-vue</span> 采用此策略
    </div>

  </v-clicks>
</div>

---

# 库简介：markstream-vue

https://github.com/Simon-He95/markstream-vue

<v-click>
针对 Vue 3 的高性能、流式友好型 Markdown 渲染组件
</v-click>

<div class="flex flex-col items-center gap-8 mt-10">
  <div class="grid grid-cols-3 gap-6 w-full max-w-5xl">
  <v-clicks>
    <div class="flex flex-col items-center p-6 border border-gray-200 dark:border-gray-700 rounded-xl bg-gray-50 dark:bg-gray-800/50 shadow-sm hover:scale-105 transition opacity-75">
      <div class="i-carbon-chart-relationship text-4xl text-blue-500 mb-4" />
      <h3 class="text-lg font-bold mb-2">渐进式 Mermaid</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400 text-center">图表随内容流式加载</p>
    </div>
    <div class="flex flex-col items-center p-6 border border-gray-200 dark:border-gray-700 rounded-xl bg-gray-50 dark:bg-gray-800/50 shadow-sm hover:scale-105 transition opacity-75">
      <div class="i-carbon-compare text-4xl text-purple-500 mb-4" />
      <h3 class="text-lg font-bold mb-2">流式 Diff 代码块</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400 text-center">智能代码变更，仅重绘修改行</p>
    </div>
    <div class="flex flex-col items-center p-6 border border-gray-200 dark:border-gray-700 rounded-xl bg-gray-50 dark:bg-gray-800/50 shadow-sm hover:scale-105 transition opacity-75">
      <div class="i-carbon-flash text-4xl text-orange-500 mb-4" />
      <h3 class="text-lg font-bold mb-2">大文档实时预览</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400 text-center">延迟低至毫秒级</p>
    </div>
  </v-clicks>
  </div>
</div>

---

# 快速上手

<div>安装</div>

```bash
pnpm add markstream-vue
# 或
npm install markstream-vue
# 或
yarn add markstream-vue
```

<div>示例</div>

```vue {all|3-4|6,10|all}
<script setup lang="ts">
  import { ref } from 'vue'
  import MarkdownRender from 'markstream-vue'
  import 'markstream-vue/index.css'

  const md = ref('# Hello World')
</script>

<template>
  <MarkdownRender :content="md" />
</template>
```

---
layout: section
transition: slide-up
---

# 演示 Demo...

---
layout: section
transition: slide-up
---

# 深入 markstream-vue

---
layout: two-cols-header
---

# 渲染组件入口

```vue {all|7-8|9-11,16-21|all}
<script setup lang="ts">
import { computed, ref } from 'vue'
import { parseMarkdownToStructure } from 'stream-markdown-parser'
import CodeBlockNodeAsync from './asyncComponent'
import FallbackComponent from './FallbackComponent.vue'

const props = defineProps<NodeRendererProps>()
const parsedNodes = computed(() => props.content ? parseMarkdownToStructure(props.content) : [])
function getNodeComponent(node) {
  return node.type === 'code_block' ? CodeBlockNodeAsync : FallbackComponent
}
</script>

<template>
  <div class="markdown-renderer">
    <component
      v-for="node in parsedNodes"
      :key="node.id"
      :is="getNodeComponent(node)"
      :node="node"
    />
  </div>
</template>
```

<arrow v-click="[4,5]" x1="650" y1="410" x2="535" y2="250" color="rgba(95, 238, 13, 1)" width="2" arrowSize="1" />

---
layout: center
transition: slide-up
---

# 解析AST

```ts {all|2-3|5-7|9-10|all}
export function parseMarkdownToStructure(markdown, md, options) {
  // 使用 markdown-it-ts 将 markdown 字符串解析为token
  const tokens = md.parse(markdown || '', {})

  // 如果提供了预处理方法，则预先处理token
  const pre = options.preTransformTokens
  const transformedTokens = pre ? pre(tokens) : tokens

  // 将令牌处理成AST
  const result = processTokens(transformedTokens, options)

  return result
}
```

---
layout: center
---

# 解析后的AST

```json
[
  {
    "type": "paragraph",
    "content": "This is a paragraph."
  },
  {
    "type": "code_block",
    "language": "javascript",
    "content": "console.log('Hello, world!');"
  },
  {
    "type": "bullet_list",
    "children": [
      {
        "type": "list_item",
        "children": [
          { "type": "paragraph", "content": "Item 1" }
        ]
      }
    ]
  }
]
```

---

# 性能优化策略

<div class="grid grid-cols-3 gap-6 mt-12 px-4">
  <v-click>
    <div class="flex flex-col items-center p-6 border border-green-500/20 rounded-xl bg-green-500/5 shadow-sm hover:scale-105 transition duration-300">
      <div class="i-carbon-layers text-5xl text-green-500 mb-6" />
      <h3 class="text-xl font-bold mb-3">批量渲染</h3>
      <p class="text-sm opacity-75 text-center leading-relaxed">
        合并高频微小更新<br/>减少 DOM Patch 频次<br/>避免主线程抖动
      </p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-6 border border-blue-500/20 rounded-xl bg-blue-500/5 shadow-sm hover:scale-105 transition duration-300">
      <div class="i-carbon-view text-5xl text-blue-500 mb-6" />
      <h3 class="text-xl font-bold mb-3">视口懒加载</h3>
      <p class="text-sm opacity-75 text-center leading-relaxed">
        IntersectionObserver 监控<br/>仅渲染可见区域组件<br/>大幅降低首屏内存
      </p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-6 border border-purple-500/20 rounded-xl bg-purple-500/5 shadow-sm hover:scale-105 transition duration-300">
      <div class="i-carbon-machine-learning-model text-5xl text-purple-500 mb-6" />
      <h3 class="text-xl font-bold mb-3">WebWorker 并行</h3>
      <p class="text-sm opacity-75 text-center leading-relaxed">
        数学公式 / Mermaid 图表<br/>移至后台线程计算<br/>保持 UI 交互流畅
      </p>
    </div>
  </v-click>
</div>

---

# 我的贡献

<div class="flex flex-col gap-4 mt-6 max-w-2xl mx-auto">
  <v-click>
    <a
      v-motion
      :initial="{ opacity: 0, y: 50, scale: 0.95 }"
      :enter="{ opacity: 1, y: 0, scale: 1, transition: { type: 'spring', stiffness: 300, damping: 20 } }"
      href="https://github.com/Simon-He95/markstream-vue/pull/141" target="_blank" class="block bg-white dark:bg-[#1e1e1e] border border-gray-200 dark:border-gray-700 rounded-lg p-4 shadow-sm hover:shadow-md transition-shadow no-underline">
      <div class="flex items-center gap-2 text-xs text-gray-500 mb-2">
        <div class="i-carbon-logo-github" />
        <span>Simon-He95/markstream-vue</span>
        <span class="px-2 py-0.5 rounded-full bg-purple-100 text-purple-700 text-[10px] font-bold">Merged</span>
      </div>
      <div class="text-base font-bold text-gray-800 dark:text-gray-200">
        fix(strong-link-parser): token错误拆分
      </div>
      <div class="text-xs text-gray-400 mt-2">
        #141 opened by gezhiheng
      </div>
    </a>
  </v-click>

  <v-click>
    <a
      v-motion
      :initial="{ opacity: 0, y: 50, scale: 0.95 }"
      :enter="{ opacity: 1, y: 0, scale: 1, transition: { type: 'spring', stiffness: 300, damping: 20 } }"
      href="https://github.com/Simon-He95/markstream-vue/pull/142" target="_blank" class="block bg-white dark:bg-[#1e1e1e] border border-gray-200 dark:border-gray-700 rounded-lg p-4 shadow-sm hover:shadow-md transition-shadow no-underline">
      <div class="flex items-center gap-2 text-xs text-gray-500 mb-2">
        <div class="i-carbon-logo-github" />
        <span>Simon-He95/markstream-vue</span>
        <span class="px-2 py-0.5 rounded-full bg-purple-100 text-purple-700 text-[10px] font-bold">Merged</span>
      </div>
      <div class="text-base font-bold text-gray-800 dark:text-gray-200">
        chore(docs): fix index.md links & remove vitepress cache
      </div>
      <div class="text-xs text-gray-400 mt-2">
        #142 opened by gezhiheng
      </div>
    </a>
  </v-click>

  <v-click>
    <a
      v-motion
      :initial="{ opacity: 0, y: 50, scale: 0.95 }"
      :enter="{ opacity: 1, y: 0, scale: 1, transition: { type: 'spring', stiffness: 300, damping: 20 } }"
      href="https://github.com/Simon-He95/markstream-vue/pull/135" target="_blank" class="block bg-white dark:bg-[#1e1e1e] border border-gray-200 dark:border-gray-700 rounded-lg p-4 shadow-sm hover:shadow-md transition-shadow no-underline">
      <div class="flex items-center gap-2 text-xs text-gray-500 mb-2">
        <div class="i-carbon-logo-github" />
        <span>Simon-He95/markstream-vue</span>
        <span class="px-2 py-0.5 rounded-full bg-purple-100 text-purple-700 text-[10px] font-bold">Merged</span>
      </div>
      <div class="text-base font-bold text-gray-800 dark:text-gray-200">
        chore(playground): switch to ESM for Tailwind config
      </div>
      <div class="text-xs text-gray-400 mt-2">
        #135 opened by gezhiheng
      </div>
    </a>
  </v-click>
</div>

---
layout: center
class: text-center
---

# Q & A

欢迎提问 · 也欢迎一起贡献！

<div mt-4 text-sm opacity-60>
GitHub: <a href="https://github.com/Simon-He95/markstream-vue" target="_blank">markstream-vue</a>
</div>

---
layout: center
class: text-center
---

# Thanks 🙌

<div text-lg>谢谢观看</div>

<PoweredBySlidev mt-10 />
