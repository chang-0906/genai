<div align="center">

# Cline

</div>

# 🚀 Cline + OpenRouter 快速開始（10 分鐘搞定）

### Step 1：取得 OpenRouter API Key
```
1. 去 openrouter.ai 註冊
2. 儲值一點點（$5 就能用很久測試）
3. 複製 API Key
```

### Step 2：安裝 Cline
```
1. 打開 VS Code
2. 左側 Extensions（擴充功能）
3. 搜尋 "Cline"
4. 安裝
```

### Step 3：設定
```
1. 點左側出現的 Cline 圖示
2. Provider 選 "OpenRouter"
3. 貼上你的 API Key
4. 模型選擇建議：

   日常開發 → claude-sonnet-4-20250514（便宜又強）
   複雜架構 → claude-opus-4-20250514（最強但貴）
   省錢測試 → deepseek/deepseek-chat（超便宜）
   免費試玩 → google/gemini-2.0-flash-exp（免費）
```

### Step 4：開始 Vibe Coding
```
打開你的專案資料夾，在 Cline 對話框輸入：

"幫我建立一個 React + Express 的全端 Todo 應用，
 包含新增、刪除、完成功能，使用 SQLite 資料庫"

然後看它自己動手做 ✨
```

# 2️⃣ 去看不同廠商的用量、模型、限制

1. [OpenRouter](https://openrouter.ai/models)
2. [Gemini](https://openrouter.ai/models?max_price=0)
3. [ST](https://huggingface.co/spaces/yoyop1217/try/blob/main/models_list.py)
4. Dify：OpenAI、Anthropic、Gemini；5000/月（按用量）

# 3️⃣ Dify 設定

## ① 建立應用（App）

進 Dify Cloud：

* 點 **Create App**
* 選：
  👉 **Chat App（聊天應用）**

---

## ② 基本設定（關鍵）

進入 App 後，設定這幾個：

### 🔹 Model（模型）

👉 這就是剛剛那句「model 在 Dify 裡選」的意思

---

### 🔹 System Prompt（建議加）

你可以先放一個簡單的，例如：

```text
You are an expert software engineer.
You help write clean, production-ready code.
You can debug, refactor, and explain code clearly.
```

👉 這會直接影響 Cline 的行為（很重要）

---

### 🔹 Memory（可開）

建議開：

* Conversation history ✅

👉 這樣 Cline 才會「記得上下文」

---

## ③ 發布（很多人忘這步）

👉 右上角一定要按：

**Publish**

不然 API 不能用 ❗

---

## ④ 拿 API Key

---

## ✨ 你要填到 Cline 的資料

| Cline 欄位 | 填什麼                      |
| -------- | ------------------------ |
| Base URL | `https://api.dify.ai/v1` |
| API Key  | 剛剛那個 app key             |

## 進階（之後可以做的）

如果你想讓 Cline 更強，可以在 Dify 加：

### 🔧 Tools（工具）

* Web browsing
* Code execution
* API calls

---

### 🔁 Workflow（進階玩法）

比 Chat App 更強，可以：

* 多步推理
* 自動調用工具
* RAG（接知識庫）


