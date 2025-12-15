### 📅 分享大纲：In-browser AI - 重新定义 Web 应用边界

这份大纲按照 **“趋势 -> 技术 -> 优劣势 -> 演示 -> 展望”** 的逻辑编排，适合 30-45 分钟的分享。

#### 1. 开场与背景 (The Shift)

- **现状引入：** 目前主流 AI 应用的架构（Client-Server 模式）及其痛点（高延迟、高昂的 GPU 服务器成本、数据隐私担忧）。
- **什么是 In-browser AI：** 定义——完全在客户端（浏览器）运行推理，无需后端 API 支持。
- **为什么是现在？** 三大驱动力：
  - **硬件：** 消费级设备 GPU 性能过剩。
  - **标准：** WebAssembly (WASM) 和 WebGPU 的成熟。
  - **模型：** 模型的小型化与蒸馏技术 (Distillation, Quantization) 爆发 (如 Llama 3 8B, Phi-3, Gemma)。

#### 2. 核心技术栈 (The Stack)

- **算力层：**
  - **WebAssembly (WASM):** CPU 推理的基础。
  - **WebGPU:** 真正的 Game Changer，允许浏览器直接调用底层 GPU 并行计算能力（比 WebGL 快得多）。
- **推理引擎/框架：**
  - **Transformers.js:** Hugging Face 的 Web 版，让 JS 直接运行 PyTorch 模型。
  - **ONNX Runtime Web:** 微软推出的跨平台高性能推理引擎。
  - **TensorFlow.js:** 老牌库，生态成熟。
  - **WebLLM:** 专门针对大语言模型在浏览器运行的优化库。

#### 3. 优劣势分析 (Trade-offs)

- **优势 (Pros):**
  - **隐私 (Privacy):** 数据不出本地，GDPR 合规神器。
  - **成本 (Cost):** $0 服务器推理成本，利用用户端算力。
  - **体验 (UX):** 零网络延迟，离线可用。
- **挑战 (Cons):**
  - **首屏加载 (Cold Start):** 需要下载模型权重（几十 MB 到几 GB）。
  - **设备兼容性:** 用户的显卡参差不齐。
  - **功耗:** 可能会让用户笔记本发热或耗电。

#### 4. 应用场景与案例 (Use Cases)

- 实时图像/视频处理 (背景移除、滤镜)。
- 本地文档分析 (隐私敏感的 PDF 总结)。
- 辅助输入 (实时语法纠错、代码补全)。

#### 5. Live Demo 环节 (The Show)

- *（见下文推荐方向）*

#### 6. 总结与未来 (Future)

- WebGPU 正式版普及率。
- Hybrid AI 架构（端云结合：小模型在端，大模型在云）。