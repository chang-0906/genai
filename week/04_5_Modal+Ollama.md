<div align="center">

# Modal + Ollama + Github Codespace 自行建立本地運行模型

</div>

# On 雲端

## 1️⃣ 起始

### 步驟 1：先建立一個 GitHub repo

### 步驟 2：建立 Codespace

### 


## 2️⃣ 檔案準備

### `requirements.txt`



### `modal_ollama_gguf.py`

## 3️⃣ 在 Codespace 終端機安裝 Modal CLI

```
pip install -r requirements.txt
```

確認 Modal 的 CLI 有沒有安裝好

```
modal --version
```

### 4️⃣ 準備 Modal 認證

```
modal token new
```

確認有沒有成功

```
modal token info
```

## 5️⃣ 部屬程式

```
modal deploy modal_ollama_gguf.py
```

