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
  Presenter: Henry Ge
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
layout: center
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

  <div class="mt-4 p-3 bg-orange-500/10 rounded-lg border border-orange-500/20">
      <span v-mark.circle.orange="10">markstream-vue</span> 采用此策略
    </div>

  </v-clicks>
</div>

---

# 库简介：markstream-vue

---

# 快速上手

```bash
pnpm add markstream-vue
```

```vue {all|3-4|10|all}
<script setup lang="ts">
  import { ref } from 'vue'
  import MarkdownRender from 'markstream-vue'
  import 'markstream-vue/index.css'

  const md = ref(`# Hello World\n\nThis is **bold** and this is *italic*.`)
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

# 设计上的几个核心点

::left::

<v-clicks depth="2">
<div>
  <p class="text-lg font-semibold">增量 Tokenizer</p>
  <ul class="mt-1 text-sm leading-tight">
    <li>按 chunk 输入，无需等待完整字符串</li>
    <li>保持语义块边界稳定，避免代码块 / 列表被截断</li>
  </ul>
</div>

<div>
  <p class="text-lg font-semibold">Block 级渲染</p>
  <ul class="mt-1 text-sm leading-tight">
    <li>每个 Block 单独转为 Vue vnode / 组件</li>
    <li>在 Block 内挂载插槽、动态组件，实现细粒度交互</li>
  </ul>
</div>

<div>
  <p class="text-lg font-semibold">可拓展处理链</p>
  <ul class="mt-1 text-sm leading-tight">
    <li>超链接、图片、表格等均可挂载自定义处理器</li>
    <li>原生支持业务数据映射（如工单 ID → 内部系统链接）</li>
  </ul>
</div>
</v-clicks>

::right::

<v-clicks depth="2">
<div>
  <p class="text-lg font-semibold">性能优先</p>
  <ul class="mt-1 text-sm leading-tight">
    <li>默认避免整体 innerHTML 替换，减少重排</li>
    <li>针对长文与大量代码块预设轻量 Diff 路径</li>
  </ul>
</div>

<div>
  <p class="text-lg font-semibold">Vue 生态友好</p>
  <ul class="mt-1 text-sm leading-tight">
    <li>兼容 Composition API</li>
    <li>无缝接入 Vue 3、Nuxt、VitePress 等框架</li>
  </ul>
</div>

<div>
  <p class="text-lg font-semibold">边缘环境兼容</p>
  <ul class="mt-1 text-sm leading-tight">
    <li>零 Node 内置模块依赖，支持 Workers 场景</li>
    <li>解析逻辑可在 Cloudflare Workers 等边缘环境运行</li>
  </ul>
</div>
</v-clicks>

---

# 我的贡献 (Bug Fix PR)

问题现象：大段代码块后紧接标题时，标题未被正确识别为新 Chunk，造成目录缺失。

根因：Tokenizer 在处理三反引号结束符后未重置 `pendingHeading` 状态。

修复要点：
```diff
 if (inFenceEnd) {
   flushFence()
-  // missing state reset
+  resetHeadingLookahead()
 }
```

结果：目录生成稳定；增量模式下不会合并错误区段。并补充单测覆盖边界。

---

# 我的贡献 (文档改进 PR)

- 增补“流式渲染 3 种 flush 策略”示例（首屏 / 固定大小 / Idle）
- 修正文中对 SSR Hydration 顺序描述错误
- 添加 FAQ：与现有 markdown-it 插件迁移步骤

前后对比：

| 条目 | 之前 | 之后 |
| ---- | ---- | ---- |
| Streaming 示例 | 无 | 三种策略表格 + 代码 |
| SSR 部分 | Hydration 时机不清晰 | 明确“脚本延迟 + 目录二次构建” |
| 插件迁移 | 简略一行 | 分步指南 (适配层、测试) |

---

# 经验总结

- 数据驱动：先量化指标再定优化优先级
- Flush 粒度：过细 => Diff 频繁；过粗 => 首屏慢
- 并行不是银弹：主线程交互与 Worker 传输成本需权衡
- 插件桥接：保持与 markdown-it 生态兼容可以降低迁移成本
- 文档清晰度直接影响采纳速度

---

# Roadmap / 展望

- WebWorker 预解析稳定化 & 回退策略
- 浏览器端缓存 Token 片段 (IndexedDB)
- AST 片段签名用于重复段落去重
- 更丰富的指令：`v-progress`, `v-outline-sync`
- Edge 平台适配：Cloudflare Durable Objects 协同 streaming
- 插件迁移工具：自动生成适配层模板

---
layout: center
class: text-center
---

# Q & A

欢迎提问 · 也欢迎一起贡献！

<div mt-4 text-sm opacity-60>
GitHub: <a href="https://github.com/" target="_blank">markstream-vue</a>
</div>

---
layout: center
class: text-center
---

# Thanks 🙌

<div text-lg>谢谢观看</div>

<PoweredBySlidev mt-10 />
