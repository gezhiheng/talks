---
theme: seriph
title: 浏览器内 AI Web 应用
info: |
  In-browser AI Web APP
class: text-center
transition: slide-left
mdc: true
drawings:
  persist: false
---

# 浏览器内 AI Web 应用

In-browser AI Web APP

<div class="absolute bottom-4 right-4 text-sm opacity-50">
  分享人: 葛智恒
</div>

<!-- Presenter Notes:
开场：
- 现状引入：目前主流 AI 应用的架构（Client-Server 模式）及其痛点。
- 什么是 In-browser AI：完全在客户端（浏览器）运行推理。
- 为什么是现在？硬件、标准、模型的发展。
-->

---
layout: center
---

# 现状：Client-Server 架构的痛点

<div class="flex flex-col mt-8 px-10 max-w-3xl mx-auto text-left">
  <v-click>
    <div class="flex items-center gap-4 p-4 border border-red-400/20 rounded-xl bg-red-500/5 shadow-sm">
      <div class="i-carbon-timer text-3xl text-red-500" />
      <div class="flex flex-col">
        <span class="text-lg font-bold">高延迟 (Latency)</span>
        <span class="text-sm opacity-60">网络请求往返，实时性受限</span>
      </div>
    </div>
  </v-click>

  <v-click>
    <div class="flex justify-center py-2">
      <div class="i-carbon-arrow-down text-2xl opacity-30" />
    </div>
    <div class="flex items-center gap-4 p-4 border border-yellow-400/20 rounded-xl bg-yellow-500/5 shadow-sm">
      <div class="i-carbon-currency text-3xl text-yellow-500" />
      <div class="flex flex-col">
        <span class="text-lg font-bold">高昂成本 (Cost)</span>
        <span class="text-sm opacity-60">GPU 服务器费用高昂，难以规模化</span>
      </div>
    </div>
  </v-click>

  <v-click>
    <div class="flex justify-center py-2">
      <div class="i-carbon-arrow-down text-2xl opacity-30" />
    </div>
    <div class="flex items-center gap-4 p-4 border border-gray-400/20 rounded-xl bg-gray-500/5 shadow-sm">
      <div class="i-carbon-security text-3xl text-gray-500" />
      <div class="flex flex-col">
        <span class="text-lg font-bold">隐私担忧 (Privacy)</span>
        <span class="text-sm opacity-60">用户数据必须上传云端</span>
      </div>
    </div>
  </v-click>
</div>

---
layout: center
---

# 什么是 In-browser AI?

<div class="text-2xl font-bold text-center mt-10 mb-10">
  完全在<span class="text-blue-500">客户端（浏览器）</span>运行推理<br/>无需后端 API 支持
</div>

<div class="grid grid-cols-3 gap-6 px-4">
  <v-click>
    <div class="flex flex-col items-center p-6 border border-blue-500/20 rounded-xl bg-blue-500/5">
      <div class="i-carbon-chip text-4xl text-blue-500 mb-4" />
      <h3 class="text-lg font-bold">硬件 (Hardware)</h3>
      <p class="text-sm opacity-75 text-center">消费级设备 GPU 性能过剩</p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-6 border border-green-500/20 rounded-xl bg-green-500/5">
      <div class="i-carbon-code-reference text-4xl text-green-500 mb-4" />
      <h3 class="text-lg font-bold">标准 (Standards)</h3>
      <p class="text-sm opacity-75 text-center">WASM & WebGPU 成熟</p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-6 border border-purple-500/20 rounded-xl bg-purple-500/5">
      <div class="i-carbon-model text-4xl text-purple-500 mb-4" />
      <h3 class="text-lg font-bold">模型 (Models)</h3>
      <p class="text-sm opacity-75 text-center">模型小型化与蒸馏 (Llama 3 8B, Phi-3)</p>
    </div>
  </v-click>
</div>

---
layout: two-cols-header
transition: slide-up
---

# 核心技术栈 (The Stack)

::left::

<div class="flex flex-col gap-6 mt-8 mr-4">
  <div class="p-4 border border-gray-500/20 rounded-lg">
    <h3 class="text-xl font-bold mb-2 flex items-center gap-2">
      <div class="i-carbon-calculation text-blue-500" /> 算力层
    </h3>
    <ul class="list-disc pl-6 space-y-2">
      <li>
        <b>WebAssembly (WASM)</b>
        <div class="text-sm opacity-60">CPU 推理的基础，广泛兼容</div>
      </li>
      <li>
        <b>WebGPU</b>
        <div class="text-sm opacity-60 text-green-500 font-bold">Game Changer</div>
        <div class="text-sm opacity-60">直接调用底层 GPU 并行计算，性能远超 WebGL</div>
      </li>
    </ul>
  </div>
</div>

::right::

<div class="flex flex-col gap-6 mt-8 ml-4">
  <div class="p-4 border border-gray-500/20 rounded-lg">
    <h3 class="text-xl font-bold mb-2 flex items-center gap-2">
      <div class="i-carbon-settings text-purple-500" /> 推理引擎
    </h3>
    <ul class="list-disc pl-6 space-y-2">
      <li><b>Transformers.js</b>: Hugging Face Web 版</li>
      <li><b>ONNX Runtime Web</b>: 微软跨平台引擎</li>
      <li><b>TensorFlow.js</b>: 老牌库，生态成熟</li>
      <li><b>WebLLM</b>: 专注 LLM 浏览器优化</li>
    </ul>
  </div>
</div>

---
layout: center
---

# 优劣势分析 (Trade-offs)

<div class="grid grid-cols-2 gap-10 mt-8 px-4">
  <div class="flex flex-col gap-4">
    <h3 class="text-xl font-bold text-green-500 text-center">优势 (Pros)</h3>
    <v-click>
      <div class="p-4 border border-green-500/20 rounded-lg bg-green-500/5">
        <div class="font-bold">🔒 隐私 (Privacy)</div>
        <div class="text-sm opacity-75">数据不出本地，GDPR 合规神器</div>
      </div>
    </v-click>
    <v-click>
      <div class="p-4 border border-green-500/20 rounded-lg bg-green-500/5">
        <div class="font-bold">💰 成本 (Cost)</div>
        <div class="text-sm opacity-75">$0 服务器推理成本，利用用户端算力</div>
      </div>
    </v-click>
    <v-click>
      <div class="p-4 border border-green-500/20 rounded-lg bg-green-500/5">
        <div class="font-bold">⚡ 体验 (UX)</div>
        <div class="text-sm opacity-75">零网络延迟，离线可用</div>
      </div>
    </v-click>
  </div>

  <div class="flex flex-col gap-4">
    <h3 class="text-xl font-bold text-red-500 text-center">挑战 (Cons)</h3>
    <v-click>
      <div class="p-4 border border-red-500/20 rounded-lg bg-red-500/5">
        <div class="font-bold">🐢 首屏加载 (Cold Start)</div>
        <div class="text-sm opacity-75">需要下载模型权重 (MB ~ GB)</div>
      </div>
    </v-click>
    <v-click>
      <div class="p-4 border border-red-500/20 rounded-lg bg-red-500/5">
        <div class="font-bold">📱 设备兼容性</div>
        <div class="text-sm opacity-75">用户显卡参差不齐，WebGPU 支持度</div>
      </div>
    </v-click>
    <v-click>
      <div class="p-4 border border-red-500/20 rounded-lg bg-red-500/5">
        <div class="font-bold">🔋 功耗</div>
        <div class="text-sm opacity-75">设备发热，耗电增加</div>
      </div>
    </v-click>
  </div>
</div>

---
layout: section
transition: slide-up
---

# 应用场景与案例

---

# 典型应用场景

<div class="grid grid-cols-3 gap-6 mt-12">
  <v-click>
    <div class="flex flex-col items-center p-6 border border-gray-200 dark:border-gray-700 rounded-xl bg-gray-50 dark:bg-gray-800/50 shadow-sm hover:scale-105 transition">
      <div class="i-carbon-touch-1 text-4xl text-blue-500 mb-4" />
      <h3 class="text-lg font-bold mb-2">手势交互</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400 text-center">
        MediaPipe 驱动<br/>隔空操作 / 游戏控制
      </p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-6 border border-gray-200 dark:border-gray-700 rounded-xl bg-gray-50 dark:bg-gray-800/50 shadow-sm hover:scale-105 transition">
      <div class="i-carbon-microphone text-4xl text-orange-500 mb-4" />
      <h3 class="text-lg font-bold mb-2">离线语音识别</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400 text-center">
        Transformers.js + WebWorker<br/>实时字幕 / 隐私保护
      </p>
    </div>
  </v-click>

  <v-click>
    <div class="flex flex-col items-center p-6 border border-gray-200 dark:border-gray-700 rounded-xl bg-gray-50 dark:bg-gray-800/50 shadow-sm hover:scale-105 transition">
      <div class="i-carbon-face-activated text-4xl text-purple-500 mb-4" />
      <h3 class="text-lg font-bold mb-2">人脸检测</h3>
      <p class="text-sm text-gray-500 dark:text-gray-400 text-center">
        MediaPipe 视觉能力<br/>特效滤镜 / 专注度分析
      </p>
    </div>
  </v-click>
</div>

---
layout: center
class: text-center
---

# Live Demo

<div class="text-6xl mb-4">🚀</div>
<div class="text-xl opacity-75">Mediapipe / Transformers.js 演示</div>

---
layout: center
class: text-center
---

<div class="inline-flex items-center gap-3 px-5 py-3 rounded-xl border border-gray-400/20 bg-gray-500/5 hover:bg-gray-500/10 transition">
  <div class="i-carbon-logo-github text-2xl" />
  <span class="text-lg font-bold">https://github.com/gezhiheng/in-browser-ai-web-app</span>
</div>

---
layout: center
---

# 总结与未来 (Future)

<div class="flex flex-col gap-8 mt-8 max-w-3xl mx-auto">
  <v-click>
    <div class="flex items-center gap-6">
      <div class="i-carbon-chart-line text-4xl text-green-500" />
      <div>
        <h3 class="text-xl font-bold">WebGPU 普及</h3>
        <p class="opacity-75">随着浏览器标准推进，性能瓶颈将进一步消除</p>
      </div>
    </div>
  </v-click>

  <v-click>
    <div class="flex items-center gap-6">
      <div class="i-carbon-cloud-satellite text-4xl text-blue-500" />
      <div>
        <h3 class="text-xl font-bold">Hybrid AI 架构</h3>
        <p class="opacity-75">端云结合：小模型在端处理实时任务，大模型在云处理复杂推理</p>
      </div>
    </div>
  </v-click>
</div>

---
layout: center
class: text-center
---

# Q & A 🔍

---
layout: center
class: text-center
---

# Thanks 🙌

<div text-lg>谢谢观看</div>

<PoweredBySlidev mt-10 />
