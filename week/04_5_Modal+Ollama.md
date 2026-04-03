<div align="center">

# Modal + Ollama + Github Codespace 自行建立本地運行模型

</div>

# On 雲端

## 1️⃣ 起始

### 步驟 1：先建立一個 GitHub repo

### 步驟 2：建立 Codespace

## 2️⃣ 檔案準備

### `requirements.txt`

```txt
modal
huggingface_hub>=0.30.0
hf-transfer>=0.1.9
openai>=1.30.0
```

### `modal_ollama_gguf.py`

```python
import os
import subprocess
import time
from pathlib import Path
from textwrap import dedent

import modal

APP_NAME = "ollama-gguf-hf-import"
OLLAMA_VERSION = "0.6.5"
OLLAMA_PORT = 11434

# Modal Volume 會掛載到這個路徑
VOLUME_PATH = "/models"
GGUF_DIR = f"{VOLUME_PATH}/gguf"

# Ollama 自己存模型的地方
OLLAMA_MODELS_DIR = "/root/.ollama/models"

MODEL_SPECS = [
    {
        "name": "llama31-abliterated-q4km",
        "repo_id": "mlabonne/Meta-Llama-3.1-8B-Instruct-abliterated-GGUF",
        "filename": "meta-llama-3.1-8b-instruct-abliterated.Q4_K_M.gguf",
        "temperature": "0.7",
        "num_ctx": "8192",
    },
    {
        "name": "deepseek-r1-qwen14b-abliterated-q4km",
        "repo_id": "QuantFactory/DeepSeek-R1-Distill-Qwen-14B-abliterated-v2-GGUF",
        "filename": "DeepSeek-R1-Distill-Qwen-14B-abliterated-v2.Q4_K_M.gguf",
        "temperature": "0.6",
        "num_ctx": "16384",
    },
]

image = (
    modal.Image.debian_slim(python_version="3.11")
    .apt_install("curl", "ca-certificates", "zstd")
    .pip_install(
        "huggingface_hub>=0.30.0",
        "hf-transfer>=0.1.9",
        "openai>=1.30.0",
    )
    .run_commands(
        f"mkdir -p {OLLAMA_MODELS_DIR}",
        f"OLLAMA_VERSION={OLLAMA_VERSION} curl -fsSL https://ollama.com/install.sh | sh",
        "which ollama",
        "ollama --version",
    )
    .env(
        {
            "OLLAMA_HOST": f"0.0.0.0:{OLLAMA_PORT}",
            "OLLAMA_MODELS": OLLAMA_MODELS_DIR,
            "HF_HUB_ENABLE_HF_TRANSFER": "1",
        }
    )
)

app = modal.App(APP_NAME, image=image)
volume = modal.Volume.from_name("ollama-gguf-hf-volume", create_if_missing=True)


def _run(cmd: list[str], check: bool = True) -> subprocess.CompletedProcess:
    print(">>>", " ".join(cmd))
    return subprocess.run(
        cmd,
        check=check,
        text=True,
        capture_output=True,
    )


def _wait_for_ollama(timeout_s: int = 120) -> None:
    import urllib.request

    deadline = time.time() + timeout_s
    last_error = None

    while time.time() < deadline:
        try:
            with urllib.request.urlopen(f"http://127.0.0.1:{OLLAMA_PORT}/api/tags", timeout=5) as resp:
                if resp.status == 200:
                    print("Ollama is ready.")
                    return
        except Exception as e:
            last_error = e
            time.sleep(2)

    raise RuntimeError(f"Ollama server did not become ready in time. Last error: {last_error}")


def _download_gguf(repo_id: str, filename: str, local_dir: str) -> str:
    from huggingface_hub import hf_hub_download

    path = hf_hub_download(
        repo_id=repo_id,
        filename=filename,
        local_dir=local_dir,
        local_dir_use_symlinks=False,
    )
    print(f"Downloaded or reused: {path}")
    return path


def _write_modelfile(model_name: str, gguf_path: str, temperature: str, num_ctx: str) -> str:
    modelfile_dir = Path(GGUF_DIR) / model_name
    modelfile_dir.mkdir(parents=True, exist_ok=True)
    modelfile_path = modelfile_dir / "Modelfile"

    content = dedent(
        f"""
        FROM {gguf_path}
        PARAMETER temperature {temperature}
        PARAMETER num_ctx {num_ctx}
        """
    ).strip() + "\n"

    modelfile_path.write_text(content, encoding="utf-8")
    print(f"Wrote Modelfile: {modelfile_path}")
    print(content)
    return str(modelfile_path)


@app.cls(
    gpu="A100",
    timeout=60 * 60,
    volumes={VOLUME_PATH: volume},
    scaledown_window=10 * 60,
)
class OllamaGGUFServer:
    ollama_process: subprocess.Popen | None = None

    @modal.enter()
    def setup(self):
        os.makedirs(GGUF_DIR, exist_ok=True)
        os.makedirs(OLLAMA_MODELS_DIR, exist_ok=True)

        print("Starting Ollama server...")
        self.ollama_process = subprocess.Popen(
            ["ollama", "serve"],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.STDOUT,
        )

        _wait_for_ollama()

        changed = False

        for spec in MODEL_SPECS:
            gguf_path = os.path.join(GGUF_DIR, spec["filename"])

            if not os.path.exists(gguf_path):
                print(f"Downloading {spec['repo_id']} / {spec['filename']}")
                _download_gguf(
                    repo_id=spec["repo_id"],
                    filename=spec["filename"],
                    local_dir=GGUF_DIR,
                )
                changed = True
            else:
                print(f"Reusing cached GGUF: {gguf_path}")

            modelfile_path = _write_modelfile(
                model_name=spec["name"],
                gguf_path=gguf_path,
                temperature=spec["temperature"],
                num_ctx=spec["num_ctx"],
            )

            result = _run(["ollama", "create", spec["name"], "-f", modelfile_path])
            print(result.stdout)
            if result.stderr:
                print(result.stderr)

            changed = True

        if changed:
            volume.commit()

        result = _run(["ollama", "list"])
        print(result.stdout)

    @modal.exit()
    def teardown(self):
        if self.ollama_process and self.ollama_process.poll() is None:
            print("Stopping Ollama server...")
            self.ollama_process.terminate()
            try:
                self.ollama_process.wait(timeout=10)
            except subprocess.TimeoutExpired:
                self.ollama_process.kill()
                self.ollama_process.wait()

    @modal.web_server(port=OLLAMA_PORT, startup_timeout=10 * 60)
    def api(self):
        print(f"Ollama API is being served on port {OLLAMA_PORT}")


@app.local_entrypoint()
def main():
    print("Local entrypoint ready.")
```

## 3️⃣ 在 Codespace 終端機安裝 Modal CLI

```cmd
pip install -r requirements.txt
```

確認 Modal 的 CLI 有沒有安裝好

```cmd
modal --version
```

### 4️⃣ 準備 Modal 認證

```cmd
modal token new
```

確認有沒有成功

```cmd
modal token info
```

## 5️⃣ 部屬程式

```cmd
modal deploy modal_ollama_gguf.py
```

## 6️⃣ 確認是否有載入完成

### 看模型有沒有載進去

```cmd
curl https://chang-0906--ollama-gguf-hf-import-ollamaggufserver-api.modal.run/api/tags
```

如果成功，你應該會看到模型清單。

你要找的名字是：

- llama31-abliterated-q4km
- deepseek-r1-qwen14b-abliterated-q4km

### 直接跟模型對話

```cmd
curl https://chang-0906--ollama-gguf-hf-import-ollamaggufserver-api.modal.run/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "llama31-abliterated-q4km",
    "messages": [
      {
        "role": "user",
        "content": "請用繁體中文簡單介紹你自己。"
      }
    ],
    "stream": false
  }'
```

## 7️⃣ 用漂亮一點的前端聊天

### 01 `app.py` （在 Github Codespace / HuggingFace）

> 先 `pip install streamlit openai`

```
import streamlit as st
from openai import OpenAI

client = OpenAI(
    base_url="https://chang-0906--ollama-gguf-hf-import-ollamaggufserver-api.modal.run/v1",
    api_key="ollama",
)

st.title("My Modal LLM Chat")

if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "system", "content": "請用繁體中文回答"}
    ]

for msg in st.session_state.messages[1:]:
    with st.chat_message("user" if msg["role"] == "user" else "assistant"):
        st.markdown(msg["content"])

prompt = st.chat_input("輸入訊息")

if prompt:
    st.session_state.messages.append({"role": "user", "content": prompt})

    with st.chat_message("user"):
        st.markdown(prompt)

    resp = client.chat.completions.create(
        model="llama31-abliterated-q4km:latest",
        messages=st.session_state.messages,
    )

    reply = resp.choices[0].message.content
    st.session_state.messages.append({"role": "assistant", "content": reply})

    with st.chat_message("assistant"):
        st.markdown(reply)
```

### 02  `app.py` + langfuse （在 Github Codespace / HuggingFace）

```
import os
import streamlit as st
from langfuse.openai import openai

# 先在環境變數裡設定：
# LANGFUSE_PUBLIC_KEY=pk-lf-...
# LANGFUSE_SECRET_KEY=sk-lf-...
# LANGFUSE_BASE_URL=https://cloud.langfuse.com
# 如果你用美國區，則改成 https://us.cloud.langfuse.com

st.set_page_config(page_title="My Modal LLM Chat", page_icon="💬")
st.title("My Modal LLM Chat")

# 這裡依然是你的 Modal / Ollama OpenAI-compatible endpoint
client = openai.OpenAI(
    base_url="https://chang-0906--ollama-gguf-hf-import-ollamaggufserver-api.modal.run/v1",
    api_key="ollama",
)

if "messages" not in st.session_state:
    st.session_state.messages = [
        {"role": "system", "content": "請一律用繁體中文回答。"}
    ]

if "session_id" not in st.session_state:
    st.session_state.session_id = "streamlit-demo-session"

if "user_id" not in st.session_state:
    st.session_state.user_id = "demo-user"

for msg in st.session_state.messages[1:]:
    with st.chat_message("user" if msg["role"] == "user" else "assistant"):
        st.markdown(msg["content"])

prompt = st.chat_input("輸入訊息...")

if prompt:
    st.session_state.messages.append({"role": "user", "content": prompt})

    with st.chat_message("user"):
        st.markdown(prompt)

    resp = client.chat.completions.create(
        name="modal-ollama-chat",
        model="llama31-abliterated-q4km:latest",
        messages=st.session_state.messages,
        metadata={
            "app": "streamlit-modal-chat",
            "provider": "modal-ollama",
        },
        extra_body={
            # 這些不是 OpenAI 官方欄位，而是 Langfuse wrapper 會讀的追蹤資訊
            "langfuse_user_id": st.session_state.user_id,
            "langfuse_session_id": st.session_state.session_id,
            "langfuse_tags": ["streamlit", "modal", "ollama"],
        },
    )

    reply = resp.choices[0].message.content
    st.session_state.messages.append({"role": "assistant", "content": reply})

    with st.chat_message("assistant"):
        st.markdown(reply)
```

在環境中設定 langfuse 密鑰（在 Terminal 輸入）：

```cmd
export LANGFUSE_PUBLIC_KEY="pk-lf-..."
export LANGFUSE_SECRET_KEY="sk-lf-..."
export LANGFUSE_BASE_URL="https://cloud.langfuse.com"
```

運行：

```
streamlit run app.py
```

### 03 用 Python 運行（Modal / Jupyter notebook）

安裝套件：

```
!pip install streamlit openai

"""
# 如果是用 Modal Notebook
%uv pip install streamlit openai
"""
```

運行的基礎：

```
from openai import OpenAI

client = OpenAI(
    base_url="https://chang-0906--ollama-gguf-hf-import-ollamaggufserver-api.modal.run/v1",
    api_key="ollama",
)

messages = [
    {"role": "system", "content": "請一律用繁體中文回答。"},
]

def chat(user_input):
    messages.append({"role": "user", "content": user_input})
    resp = client.chat.completions.create(
        model="llama31-abliterated-q4km:latest",
        messages=messages,
    )
    reply = resp.choices[0].message.content
    messages.append({"role": "assistant", "content": reply})
    return reply
```

一般聊天介面：

```
chat("你好，你是誰？")
```

漂亮一點的聊天介面：

```
!pip install ipywidgets
```

```
import ipywidgets as widgets
from IPython.display import display, HTML

input_box = widgets.Text(
    placeholder="輸入訊息...",
    layout=widgets.Layout(width='80%')
)

output_area = widgets.Output()

display(output_area, input_box)

def handle_submit(change):
    user_input = change["new"]
    if not user_input:
        return

    input_box.value = ""

    messages.append({"role": "user", "content": user_input})

    resp = client.chat.completions.create(
        model="llama31-abliterated-q4km:latest",
        messages=messages,
    )

    reply = resp.choices[0].message.content
    messages.append({"role": "assistant", "content": reply})

    with output_area:
        display(HTML(f"""
        <div style="text-align:right; margin:10px">
            <b>你:</b> {user_input}
        </div>
        <div style="text-align:left; margin:10px">
            <b>AI:</b> {reply}
        </div>
        """))

input_box.observe(handle_submit, names='value')
```

### 04 用 Python 運行 + LangFuse（Modal / Jupyter notebook）

運行的基礎：（另放程式碼）
