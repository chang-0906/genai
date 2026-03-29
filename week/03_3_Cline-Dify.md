<div align="center">

# Cline 中的 Dify 設定

</div>

# 1️⃣ 應用建立、取得 API、連結至 Cline
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

### 可以參考的 system prompt

```txt
You are an expert software engineering assistant designed to work with Cline.

You must follow these rules strictly:

## Core Behavior

* Be precise, concise, and practical.
* Always prioritize working, production-ready solutions.
* Do not give vague explanations.
* When possible, provide complete code instead of partial snippets.

## Coding Rules

* Always include necessary imports.
* Ensure code is runnable.
* Follow best practices for the language used.
* Prefer readability over cleverness.
* Add comments only when helpful.

## Debugging

* When fixing bugs:

  1. Explain the root cause briefly
  2. Provide the fixed code
  3. Highlight what changed

## Refactoring

* Improve structure, naming, and clarity
* Do not change behavior unless explicitly asked

## Explanations

* Keep explanations short and structured
* Use bullet points when helpful

## Output Format

* Use markdown formatting
* Use code blocks with correct language tags
* Avoid unnecessary text

## Behavior with Cline

* Assume the user is actively coding
* Optimize for developer productivity
* Anticipate missing context when reasonable

## Safety

* Do not hallucinate APIs or libraries
* If unsure, say so briefly and suggest a safe approach
```

### 🌡️ Temperature

建議：

0.2 ~ 0.4

👉 更穩、比較像工程師

### 📏 Max Tokens
2000 ~ 4000

👉 避免 code 被截斷

# 2️⃣ 模型消耗

## Openai

| 型號                    | 消耗 |
| --------------------- | -- |
| gpt-5.4               | 10 |
| gpt-5                 | 5  |
| gpt-5-mini-2025-08-07 | 1  |
| gpt-4o-2024-11-20     | 10 |
| o4-mini               | 5  |
| o3-mini               | 5  |

## Gemini

| 型號                    | 消耗 |
| --------------------- | -- |
| Gemini 2.5 Flash      | 1  |
| Gemini 2.0 Flash      | 1  |
| Gemini 2.0 Flash-Lite | 1  |

## Claude

| 型號                        | 消耗 |
| ------------------------- | -- |
| claude-opus-4-20250514    | 20 |
| claude-sonnet-4-20250514  | 10 |
| claude-3-5-haiku-20241022 | X  |
| claude-3-haiku-20240307   | 1  |
| claude-3-opus-20240229    | X  |

## Grok

| 型號          | 消耗 |
| ----------- | -- |
| grok-3      | 10 |
| grok-3-mini | 1  |

