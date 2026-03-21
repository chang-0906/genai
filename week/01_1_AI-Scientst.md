<div align="center">

# AI Scientst：全自動化的 AI 科學研究系統

</div>

# 1️⃣ 簡單介紹

> AI-Scientist 是由 Sakana AI 開發的一個開源專案，目標是讓 大型語言模型（LLM）自主完成完整的科學研究流程——從產生想法到撰寫論文，再到同行評審，全部自動化。

## 完整的研究流程

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

# 2️⃣ 運行

- ✅ 本地運行
- ✅ 半雲端運行：本地操作 + 雲端運算資源
- ✅ 全雲端運行：利用 github codespace 或 Google Cloud CLI + 雲端運算資源（如 colab pro / Hugging Face / LightMadal
https://github.com/codespaces

## ✅ 全雲端運行

- Github Repository & Codespace
- OpenAI 或 Anthropic 的 API Key
- Modal

### 🗺️ 整體流程概覽

```
Github Codespace               Modal 雲端
┌──────────┐    modal run     ┌─────────────────┐
│ 你的終端機 │ ──────────────→ │ Container (GPU)  │
│          │                  │ ├─ Python 3.11   │
│ modal CLI │                  │ ├─ LaTeX         │
│          │ ←────────────── │ ├─ AI-Scientist  │
│ 看結果    │    結果回傳      │ └─ 跑實驗+寫論文   │
└──────────┘                  └─────────────────┘
```

### 1. 建立 GitHub Repository

- 取名：ai-scientist-runner
- 選 Public 或 Private 都行
- 勾選 "Add a README file"

### 2. 啟動 Codespaces

在你的 repo 頁面：
1. 點綠色的 "<> Code" 按鈕
2. 切到 "Codespaces" 分頁
3. 點 "Create codespace on main"

### Step 3：在 Codespaces 終端機中安裝 Modal

在下方的 Terminal 中輸入：

```bash
# 安裝 modal
pip install modal

# 登入 Modal（會給你一個網址，在瀏覽器開啟授權）
modal setup
```

會顯示：
```
Please visit this URL to authenticate:
https://modal.com/token/xxxxxxxx

⠋ Waiting for authentication...
✓ Successfully authenticated!
```

**點那個連結，在新分頁授權即可。**

### Step 4：設定 API Key

```bash
# OpenAI
modal secret create openai-secret \
  OPENAI_API_KEY=sk-你的key

# 或 DeepSeek（更便宜）
modal secret create deepseek-secret \
  DEEPSEEK_API_KEY=sk-你的key
```

### Step 5：建立執行腳本

在 Codespaces 的 VS Code 中，新建檔案 `run.py`：

**點左邊檔案列表 → 右鍵 → New File → 命名 `run.py`**

貼上以下內容：

```python
# run.py
import modal

# ============================================================
# 環境定義
# ============================================================
image = (
    modal.Image.debian_slim(python_version="3.11")
    .apt_install(
        "git", "wget", "curl",
        # LaTeX 相關
        "texlive", "texlive-latex-extra",
        "texlive-fonts-recommended", "texlive-fonts-extra",
        "texlive-publishers", "texlive-science",
        "texlive-bibtex-extra", "latexmk",
    )
    .run_commands(
        # Clone AI-Scientist
        "git clone https://github.com/SakanaAI/AI-Scientist.git /root/AI-Scientist",
        # 安裝 Python 依賴
        "cd /root/AI-Scientist && pip install -r requirements.txt",
        # 安裝 aider
        "pip install aider-chat",
    )
)

app = modal.App(name="ai-scientist", image=image)

# 持久化儲存
volume = modal.Volume.from_name("ai-scientist-results", create_if_missing=True)


# ============================================================
# 主要執行函數
# ============================================================
@app.function(
    gpu="T4",
    timeout=5 * 3600,
    secrets=[modal.Secret.from_name("openai-secret")],
    volumes={"/results": volume},
)
def run_experiment(model: str, experiment: str, num_ideas: int):
    import subprocess, os, shutil
    from datetime import datetime

    os.chdir("/root/AI-Scientist")

    print(f"🚀 Model: {model} | Experiment: {experiment} | Ideas: {num_ideas}")

    process = subprocess.Popen(
        [
            "python", "launch_scientist.py",
            "--model", model,
            "--experiment", experiment,
            "--num-ideas", str(num_ideas),
        ],
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT,
        text=True,
        bufsize=1,
    )

    for line in process.stdout:
        print(line, end="")
    process.wait()

    # 儲存結果
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    result_dir = f"/results/{experiment}_{timestamp}"
    src = f"/root/AI-Scientist/templates/{experiment}"
    if os.path.exists(src):
        shutil.copytree(src, result_dir, dirs_exist_ok=True)
        print(f"\n📁 Saved to: {result_dir}")
    volume.commit()

    return f"✅ Done! → {result_dir}"


# ============================================================
# 查看結果
# ============================================================
@app.function(volumes={"/results": volume})
def list_results():
    import os
    for item in sorted(os.listdir("/results")):
        print(f"📁 {item}")
        for root, dirs, files in os.walk(f"/results/{item}"):
            for f in files:
                if f.endswith(".pdf"):
                    rel = os.path.join(root, f)
                    size_mb = os.path.getsize(rel) / 1024 / 1024
                    print(f"   📄 {f} ({size_mb:.1f} MB)")


# ============================================================
# 下載 PDF 到本機
# ============================================================
@app.function(volumes={"/results": volume})
def download_pdfs():
    import os, base64
    pdfs = []
    for root, dirs, files in os.walk("/results"):
        for f in files:
            if f.endswith(".pdf"):
                path = os.path.join(root, f)
                with open(path, "rb") as fh:
                    pdfs.append({"name": f, "data": base64.b64encode(fh.read()).decode()})
                print(f"📄 Found: {path}")
    return pdfs


# ============================================================
# 入口
# ============================================================
@app.local_entrypoint()
def main(
    action: str = "run",
    model: str = "gpt-4o-2024-05-13",
    experiment: str = "2d_diffusion",
    num_ideas: int = 2,
):
    if action == "run":
        print("🔬 Starting AI-Scientist...")
        result = run_experiment.remote(model, experiment, num_ideas)
        print(result)

    elif action == "list":
        list_results.remote()

    elif action == "download":
        import base64, os
        pdfs = download_pdfs.remote()
        os.makedirs("downloaded_papers", exist_ok=True)
        for pdf in pdfs:
            path = f"downloaded_papers/{pdf['name']}"
            with open(path, "wb") as f:
                f.write(base64.b64decode(pdf["data"]))
            print(f"💾 Downloaded: {path}")
```

### Step 6：開始跑！

在 Codespaces 的 Terminal 中：

```bash
# ==========================================
# 🚀 跑實驗（第一次會花 10-15 分鐘建 Image）
# ==========================================

# 最基本：2d_diffusion + GPT-4o + 2 個 ideas
modal run run.py

# 用 DeepSeek 省錢版（改 secret 和 model）
modal run run.py \
  --model "deepseek-coder-v2-0724" \
  --experiment "2d_diffusion" \
  --num-ideas 1

# 跑 nanoGPT 實驗
modal run run.py \
  --experiment "nanoGPT" \
  --num-ideas 1
```

### Step 7：查看結果

```bash
# 列出所有跑完的結果
modal run run.py --action list

# 下載 PDF 到 Codespaces
modal run run.py --action download

# 在 VS Code 左邊的檔案列表中，
# 你會看到 downloaded_papers/ 資料夾
# 點開 PDF 就能直接在瀏覽器中預覽！
```

> [!WARNING]
> 目前很順的到了結尾，但是最後卻沒有 PDF 檔案（！）
