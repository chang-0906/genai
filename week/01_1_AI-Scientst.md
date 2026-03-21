<div align="center">

# AI Scientst：全自動化的 AI 科學研究系統

</div>

> AI-Scientist 是由 Sakana AI 開發的一個開源專案，目標是讓 大型語言模型（LLM）自主完成完整的科學研究流程——從產生想法到撰寫論文，再到同行評審，全部自動化。

## 1️⃣ 完整的研究流程

```txt
1. Idea Generation（產生想法）
        ↓
2. Literature Search（文獻搜尋）
        ↓
3. Experiment Design & Execution（實驗設計與執行）
        ↓
4. Result Analysis（結果分析）
        ↓
5. Paper Writing（論文撰寫）
        ↓
6. Automated Peer Review（自動化同行評審）
        ↓
7. Iteration / Refinement（反覆迭代改進）
```

## 📦 各階段詳細說明

### 1. Idea Generation（構思研究想法）
- LLM 基於給定的研究模板（template），自動產生**新穎的研究假說**
- 系統會用 Semantic Scholar API 搜索相關文獻，確認想法的新穎性
- 會對想法進行**評分和排序**（依據新穎性、可行性、有趣程度等）

### 2. Experiment Execution（實驗執行）
- 系統會自動**修改 Python 程式碼**來實現實驗
- 使用 `aider`（一個 AI 編程助手）來進行程式碼編輯
- 自動執行實驗程式碼，收集結果數據
- 若執行出錯，會嘗試 **自動 debug**

### 3. Paper Writing（論文撰寫）
- 基於 LaTeX 模板，自動生成一篇**完整的學術論文**
- 包含：摘要、引言、方法、實驗結果、結論等完整結構
- 自動插入**圖表**和**引用文獻**
- 使用 `latexmk` 編譯成 PDF

### 4. Automated Peer Review（自動化審稿）
- 使用 LLM 模擬 **學術期刊的同行評審過程**
- 從多個維度評分：新穎性、正確性、清晰度、重要性等
- 給出 1-10 分的評分以及接受/拒絕的建議
- 該評審模型已驗證與人類審稿者有相近水準的準確率

---

## 🏗️ 技術架構

```
┌─────────────────────────────────────────┐
│              AI Scientist               │
├─────────────────────────────────────────┤
│                                         │
│   LLM Backend（可選用）：               │
│   ├── Claude (Anthropic)                │
│   ├── GPT-4o (OpenAI)                   │
│   ├── DeepSeek-Coder                    │
│   └── Llama / 開源模型 (via vLLM/OLLAMA)│
│                                         │
│   工具鏈：                               │
│   ├── aider（AI 程式碼編輯）             │
│   ├── Semantic Scholar API（文獻搜尋）   │
│   ├── LaTeX（論文排版）                  │
│   └── Python（實驗執行環境）             │
│                                         │
│   研究模板（Templates）：                 │
│   ├── NanoGPT（語言模型研究）            │
│   ├── 2D Diffusion（擴散模型研究）       │
│   └── Grokking（泛化現象研究）           │
│                                         │
└─────────────────────────────────────────┘
```

## 2️⃣ 運行

- ✅ 本地運行
- ✅ 半雲端運行：本地操作 + 雲端運算資源
- ✅ 全雲端運行：利用 github codespace 或 Google Cloud CLI + 雲端運算資源（如 colab pro / Hugging Face / LightMadal
https://github.com/codespaces
