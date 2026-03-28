<div align="center">

# VM & Sandbox：隔離環境

</div>

#　1️⃣ 為什麼需要隔離？

1. **AI 可能判斷錯誤** — 刪錯檔、改錯設定
2. **Prompt Injection** — 專案中的 README 或 Issue 內容可能藏惡意指令，AI 讀取後被誘導執行
3. **權限過大** — AI 用的是你的使用者權限，你能做的它都能做

# 2️⃣ 隔離方案比較

| 方案 | 安全性 | 效能 | 設定難度 | 適合誰 |
|------|--------|------|----------|--------|
| **直接在本機跑（無隔離）** | 🔴 最低 | ⭐⭐⭐⭐⭐ | 零 | 快速測試、不怕炸的環境 |
| **Docker 容器** | 🟡 中高 | ⭐⭐⭐⭐ | ⭐⭐ 中 | **大多數開發者最佳選擇** |
| **VS Code Dev Containers** | 🟢 高 | ⭐⭐⭐⭐ | ⭐⭐ 中 | **Cline 使用者首選** |
| **虛擬機（VM）** | 🟢 最高 | ⭐⭐⭐ 較慢 | ⭐⭐⭐ 高 | 極度敏感的專案 |
| **雲端開發環境** | 🟢 高 | ⭐⭐⭐⭐ | ⭐ 低 | 不想自己設定的人 |

---

# 3️⃣ 🏆 我最推薦：VS Code Dev Containers + Cline

因為你要用 Cline，而 Cline 跑在 VS Code 裡，Dev Containers 是最無縫的方案。

### 原理

```
┌─── 你的本機 ─────────────────────────────────┐
│                                               │
│   VS Code ←─────→ Docker Container           │
│   （介面在本機）     （程式碼 + 執行都在容器內） │
│                     ┌──────────────────────┐  │
│   Cline 擴充 ──────→│ /workspace/project   │  │
│   （透過 VS Code     │ Node.js, Python...   │  │
│    連進容器）         │ npm, git...          │  │
│                     │ ❌ 無法碰本機檔案     │  │
│                     │ ❌ 無法碰 ~/.ssh      │  │
│                     │ ❌ 無法碰 ~/.aws      │  │
│                     └──────────────────────┘  │
│                                               │
│   你的個人檔案、密鑰、系統設定 → 完全隔離 🔒    │
└───────────────────────────────────────────────┘
```

### 實作步驟

**Step 1：安裝 Docker**
```bash
# macOS
brew install --cask docker
# 或去 docker.com 下載 Docker Desktop

# Windows
# 下載 Docker Desktop for Windows

# Linux
sudo apt-get install docker.io docker-compose
```

**Step 2：VS Code 安裝 Dev Containers 擴充**
```
VS Code → Extensions → 搜尋 "Dev Containers" → 安裝
```

**Step 3：在專案根目錄建立設定檔**

```
你的專案/
├── .devcontainer/
│   └── devcontainer.json
├── src/
└── package.json
```

`.devcontainer/devcontainer.json`：
```jsonc
{
  "name": "Safe Vibe Coding",
  
  // 基礎環境（Node.js 為例，可換成 Python 等）
  "image": "mcr.microsoft.com/devcontainers/javascript-node:20",
  
  // 容器內安裝的 VS Code 擴充
  "customizations": {
    "vscode": {
      "extensions": [
        "saoudrizwan.claude-dev",   // Cline
        "esbenp.prettier-vscode",   // 選配
        "dbaeumer.vscode-eslint"    // 選配
      ]
    }
  },

  // 容器建立後自動執行
  "postCreateCommand": "npm install",
  
  // 只開放必要的 port
  "forwardPorts": [3000, 5173],
  
  // 安全性設定
  "runArgs": [
    "--network=bridge",           // 限制網路
    "--memory=4g",                // 限制記憶體
    "--cpus=2"                    // 限制 CPU
  ],

  // 容器內的環境變數（API Key 放這裡而非本機）
  "containerEnv": {
    "OPENROUTER_API_KEY": "${localEnv:OPENROUTER_API_KEY}"
  }
}
```

**Step 4：啟動**
```
1. VS Code 打開你的專案資料夾
2. 左下角綠色按鈕 → "Reopen in Container"
3. 等待容器建立（首次約 2-5 分鐘）
4. 完成後你已經在容器內了
5. Cline 也在容器內運作，所有操作都被隔離
```

---

## 4️⃣ 方案 B：純 Docker（適合 Claude Code CLI）

```bash
# 建立 Dockerfile
cat > Dockerfile.vibedev << 'EOF'
FROM node:20-slim

RUN apt-get update && apt-get install -y git curl
RUN npm install -g @anthropic-ai/claude-code

WORKDIR /workspace
EOF

# 建立並進入容器
docker build -f Dockerfile.vibedev -t vibedev .
docker run -it \
  --name vibe-workspace \
  -v $(pwd)/project:/workspace \
  -p 3000:3000 \
  -e OPENROUTER_API_KEY=$OPENROUTER_API_KEY \
  --memory=4g \
  --cpus=2 \
  vibedev bash

# 容器內就可以跑 claude 或任何指令
# AI 只能碰 /workspace 裡的東西
```

### 方案 C：VM（最高安全性）

```bash
# 用 Multipass（最簡單的 VM 方案）
# macOS / Windows / Linux 都支援

# 安裝
brew install multipass   # macOS
# Windows: 從 multipass.run 下載

# 建立 VM
multipass launch --name vibe-dev --cpus 2 --memory 4G --disk 20G

# 進入 VM
multipass shell vibe-dev

# 在 VM 內安裝開發工具
sudo apt update
sudo apt install -y nodejs npm git
npm install -g @anthropic-ai/claude-code

# VS Code Remote SSH 連進 VM 使用 Cline
```

# 5️⃣ 方案 D：雲端環境（零設定）

| 服務 | 免費額度 | 特色 |
|------|---------|------|
| **GitHub Codespaces** | 60 小時/月免費 | 完整 VS Code 環境，可裝 Cline |
| **Gitpod** | 50 小時/月免費 | 自動化開發環境 |
| **Google Cloud Shell** | 免費 | 限制較多但零成本 |

```
GitHub Codespaces 用法：
1. 在 GitHub repo 頁面點 "Code" → "Codespaces" → "+"
2. 會開一個雲端 VS Code
3. 安裝 Cline 擴充 → 設定 OpenRouter
4. 所有程式碼都在 GitHub 的雲端跑，跟你本機完全無關
```

---

## 安全等級 vs 便利性選擇指南

```
你在做什麼？                         推薦隔離等級
──────────────────────────────────────────────────

學習/練習/玩 side project        →   Docker 或 Dev Container
                                     就很夠了 ✅

接案/幫公司寫程式                →   Dev Container + 
                                     不掛載敏感目錄 ✅

處理有 API Keys 的專案           →   Dev Container +
                                     secrets 不進容器 ✅

企業機密/金融/醫療專案           →   VM 或雲端隔離環境 ✅✅

你電腦裡有加密貨幣私鑰           →   拜託用 VM 🙏
```

---

## 不管用哪種隔離，都要做的安全清單

```
✅ 1. 不要把本機的 ~/.ssh、~/.aws、~/.config 掛進去
✅ 2. API Key 用環境變數傳入，不要寫死在檔案裡
✅ 3. 限制容器/VM 的網路存取（不需要的 port 不開）
✅ 4. 專案有 Git，隨時可以 revert
✅ 5. 定期檢查 AI 生成的程式碼有沒有硬編碼密鑰
✅ 6. 用完的容器可以直接砍掉重建（乾淨環境）
```
