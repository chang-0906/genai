# GPT

```md
# 🧠 Project: AI 培養遊戲（LLM Growth Simulator）

請幫我設計並實作一個 Web App，核心概念是「讓使用者從零開始培養一個 AI」，並最終可以產出可攜帶的模型或知識包。

---

# 🎯 核心理念

這不是一般聊天 AI，而是：

> 一個可以被「培養」的 AI 實體（Agent）

此 AI 應該具備：

* 初始狀態接近「無知」
* 知識需透過學習取得（而不是直接使用預訓練知識）
* 個性與能力會隨時間成長
* 最終可以「匯出並分享」

---

# 🧩 系統架構（分兩階段）

## 🟢 Phase 1（無 GPU）

建立「偽從零開始」但體驗真實：

### 核心機制：RAG + 限制知識來源

AI 必須遵守：

* ❌ 不可使用模型原本知識
* ✅ 只能使用「自身知識庫」內容回答
* ❌ 不知道的東西 → 必須說「我不知道」

---

## 🧠 學習流程

1. 使用者設定 AI 發展方向（例如：哲學型、溫柔、科學導向）
2. AI 會「主動提問」
3. 問題送到「老師 LLM」（由使用者提供 API）
4. 回答被存入知識庫
5. AI 回答時只可引用知識庫

---

## 📦 知識儲存格式（可匯出）

請設計：

### 1. personality.json

包含：

* traits（性格）
* tone（語氣）
* growth_history（成長紀錄）

### 2. knowledge.json / .md

包含：

* 問題
* 回答
* embedding（若有）

---

## 📦 匯出格式

設計一個：

`.ncpack`

內容：

* personality.json
* knowledge.json

用途：
👉 可分享給其他使用者，重建同一個 AI

---

# 🟡 Phase 2（進階，可選）

整合雲端 GPU（Modal）

👉 當知識累積足夠後：

## 「昇華（Ascend）」功能

觸發條件：

* 使用者手動點擊

系統：

1. 收集對話資料
2. 轉換為 instruction dataset
3. 呼叫 Modal API
4. 微調模型（QLoRA）

---

## 🧠 模型設定

* 基礎模型：Llama 3.2 3B
* 輸出：LoRA adapter (.safetensors)
* 可下載並用 Ollama / LM Studio 使用

---

# 🔌 API 設計

## 使用者需提供：

* OpenAI-compatible API key（老師 LLM）
* Modal API key（訓練）

---

# 🧱 技術要求

請使用：

## Frontend

* React / Next.js
* Tailwind UI

## Backend

* Node.js / Python（擇一）
* REST API

## Database

* SQLite / Postgres

## 向量資料庫（可選）

* Chroma / FAISS

---

# 🧠 核心模組

請實作：

## 1️⃣ Agent Engine

* 控制 AI 行為
* 強制知識來源限制

---

## 2️⃣ Learning Engine

* 自動生成問題
* 呼叫老師 LLM
* 儲存知識

---

## 3️⃣ Memory System

* 儲存知識
* 提供檢索（RAG）

---

## 4️⃣ Training Pipeline（Modal）

* dataset builder
* fine-tune script（QLoRA）

---

## 5️⃣ Export / Import System

* 匯出 .ncpack
* 匯入並重建 AI

---

# 🎮 使用流程（UX）

1. 使用者創建 AI
2. 設定個性 / 發展方向
3. AI 開始提問學習
4. 使用者可觀察成長
5. 累積一定知識後：

   * 匯出 AI
   * 或進行「昇華訓練」

---

# ⚠️ 關鍵限制（非常重要）

請確保：

* AI 不可洩漏 base model 知識
* 所有回答必須來自 knowledge DB
* 若無資料 → 回答「我還不知道」

---

# 🎯 輸出要求

請幫我：

1. 設計完整系統架構
2. 建立專案目錄結構
3. 實作 MVP（Phase 1）
4. 提供關鍵程式碼（Agent / RAG / Learning）
5. 設計 API routes
6. 提供 Modal integration 範例（Phase 2）

---

# 🚀 額外加分（如果可以）

* AI 成長數值化（level / curiosity / personality drift）
* 多 AI 社群互動
* 知識可視化

---

請從「專案初始化 + 架構設計」開始，一步一步帶我完成。
```

# Claude

# NurtureAI - 個人化 LLM 培養遊戲平台

## 專案概述

建立一個讓用戶從零開始培養個人化 LLM 的遊戲平台。用戶可以設定 AI 的發展方向，讓 AI 實體透過向外部 LLM「提問學習」來累積知識，最終在知識量足夠後觸發真正的模型微調，產出可攜帶、可分享的模型檔案。

---

## 核心架構

### 三階段設計

```
階段一：知識積累（不需 GPU）
├── 用戶設定 AI 實體的發展方向（如：擅長哲學、溫柔性格、對科學好奇）
├── AI 實體自動向「老師 LLM」提問學習（透過用戶自行串接的 OpenAI-compatible API）
├── 所有學到的知識存入結構化知識庫（RAG 架構）
├── AI 實體回答問題時只能使用知識庫中的內容，不可調用預訓練知識
└── 對話被同步整理為 instruction → response 格式的訓練資料

階段二：日常互動
├── 用戶可以和 AI 實體對話
├── AI 實體只能根據自己知識庫中的內容回答
├── 對於知識庫之外的問題，回答「我不知道」或「我需要去探索才能了解」
└── 持續累積訓練資料

階段三：昇華（模型微調，需 GPU）
├── 用戶手動觸發「昇華」按鈕（系統可提示已達到建議的互動門檻）
├── 後端呼叫 Modal API，在雲端 GPU 上進行 QLoRA 微調
├── 基礎模型：Llama 3.2 3B
├── 產出 LoRA adapter 權重檔（.safetensors）
└── 用戶可下載模型檔案，在本地用 Ollama 或 LM Studio 使用
```

### 技術架構圖

```
用戶操作遊戲介面（Web App）
    ↓
後端 API Server
    ├── 知識學習系統
    │     ├── 用戶提供 OpenAI-compatible API endpoint + key
    │     ├── AI 實體根據「發展方向」自動生成問題
    │     ├── 向老師 LLM 提問
    │     └── 回答存入知識庫（向量 DB 或 JSON/MD 檔案）
    │
    ├── 對話系統（RAG）
    │     ├── 用戶提問 → 先查知識庫
    │     ├── 只用知識庫內容生成回答
    │     └── 知識庫外的問題 → 回覆「不知道」
    │
    └── 昇華系統（微調觸發）
          ├── 用戶提供 Modal API key
          ├── 呼叫 Modal 雲端 GPU（A100）
          ├── 載入 Llama 3.2 3B 基礎模型
          ├── 使用累積的訓練資料進行 QLoRA 微調
          ├── 儲存 LoRA adapter 權重
          └── 提供下載連結
```

---

## 功能需求

### 1. 用戶 API 串接設定

- 用戶自行提供 OpenAI-compatible API 的 endpoint 和 API key（用於 AI 實體的學習來源）
- 用戶自行提供 Modal API key（用於模型微調時的雲端 GPU）
- 所有 API 費用由用戶自己的帳號承擔，開發者不負擔
- 設定頁面需清楚說明每個 API 的用途

### 2. AI 實體管理

- 用戶可以建立多個 AI 實體
- 每個實體有獨立的：
  - 個性設定（personality.json）
  - 發展方向（用戶設定的培養目標）
  - 知識庫（memories.md 或 knowledge.json）
  - 成長數值與歷程記錄
  - 累積的訓練資料

### 3. 知識學習系統

- 用戶設定「發展方向」後，AI 實體自動生成相關問題
- 透過用戶串接的 API 向老師 LLM 提問
- 老師的回答被結構化存入知識庫
- 知識庫支援向量搜尋（RAG），回答時優先檢索相關知識
- AI 實體嚴格只能使用知識庫中的知識，不可使用底層模型的預訓練知識

### 4. 匯出 / 匯入系統

- 匯出格式為自定義封裝（如 `.ncpack`），包含：
  - `personality.json`（個性、數值、成長歷程）
  - `knowledge.json` 或 `memories.md`（所有學習過的知識點）
  - 訓練資料檔案
  - 如果已昇華：LoRA adapter 權重檔（.safetensors）
- 其他人匯入後，用同樣的 RAG 邏輯加載，可復刻相同「性格 + 知識 + 記憶」的 AI 實體
- 如果包含模型權重，可直接在本地載入使用

### 5. 昇華系統（模型微調）

- 基礎模型：Llama 3.2 3B
- 微調方法：QLoRA
- 觸發方式：用戶手動點擊「昇華」按鈕
- 系統在達到建議互動門檻時提示用戶（如：已累積 500+ 條訓練資料）
- 預估費用：每次 $0.5–2 美元（3B 模型）
- 預估時間：5–20 分鐘
- 產出：LoRA adapter 檔案（.safetensors），可下載

### 6. 多語言支援

- 介面支援中文和英文切換
- 在用戶開始使用前顯示使用介紹/教學頁面（中英雙語）

---

## 技術選型建議

| 項目 | 建議技術 |
|------|----------|
| 前端 | React / Next.js |
| 後端 | Python（FastAPI 或 Flask） |
| 知識庫儲存 | JSON 檔案 + 向量資料庫（ChromaDB 或 FAISS） |
| RAG 框架 | LangChain 或 LlamaIndex |
| 模型微調 | Modal + PEFT/QLoRA + Transformers |
| 基礎模型 | Llama 3.2 3B |
| 外部 LLM 串接 | OpenAI-compatible API（用戶自行提供） |
| 雲端 GPU | Modal（用戶自行提供 API key） |

---

## 重要設計原則

1. **真實培養感**：AI 實體不是「假裝不知道」，而是真的只能使用知識庫中的內容。知識是增量累積的，不是預設全知。
2. **費用透明**：所有 API 費用由用戶自己承擔，系統需清楚顯示每個操作的預估費用。
3. **可攜帶性**：培養出的 AI 實體（知識庫 + 個性設定 + 模型權重）必須可以匯出、分享、在其他環境使用。
4. **漸進式體驗**：從不需 GPU 的知識積累開始，到需要 GPU 的真正微調，讓用戶可以按自己的節奏和預算推進。

---

## 開發優先順序

1. **P0**：基礎架構 + 用戶 API 串接設定 + AI 實體建立
2. **P0**：知識學習系統（AI 實體向老師 LLM 提問 + 知識庫儲存）
3. **P0**：RAG 對話系統（嚴格限制只用知識庫回答）
4. **P1**：匯出/匯入系統
5. **P1**：昇華系統（Modal 串接 + QLoRA 微調）
6. **P2**：多語言介面 + 使用教學
7. **P2**：成長數值可視化 + 遊戲化元素
