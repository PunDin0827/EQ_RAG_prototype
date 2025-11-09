# Qwen + RAG：設備維修問答機器人

一個結合 **Qwen 大語言模型** 與 **Retrieval-Augmented Generation (RAG)**  的設備維修助理，  
協助工程師依據設備型號、Alarm Code 與關鍵字，快速查找故障原因與排除步驟。  

---

## 1. 專案簡介 

- 目標：整合 **維修手冊、SOP、保修紀錄、常見故障整理表** ，只要與問答機器人提問，就能查到維修手冊、SOP 與保修紀錄中的相關內容。

- 典型使用情境：輸入「設備 A / Alarm 1050 無法復歸」，系統自動檢索相關段落並用自然語言彙整處理流程。

- 執行環境：  
  - 使用 `llama.cpp` 載入 **本地 Qwen (gguf)**，  
  - 可在 **無外網** 的環境中執行。
  
- 文件向量與索引使用 **ChromaDB** 儲存，方便後續增量更新。

---

## 2. 功能特色 

- **RAG 問答流程**  
  - 文件前處理：將維修手冊 / SOP 依段落切成複數 `chunk`，並加上設備名稱、Alarm Code 等 MetaData。  
  - 向量化與索引：使用 embedding 模型將 chunk 轉為向量，寫入 **ChromaDB** 以支援相似度搜尋。  
  - 查詢時會把「檢索到的相關內容 + 使用者問題」一起丟給 Qwen，產生維修建議。

- **Hybrid Retrieval：向量＋關鍵字＋MetaData**  
  - 以向量相似度搭配 **BM25** 關鍵字分數。  
  - 對「同時被向量搜尋、關鍵字搜尋、MetaData Filter 命中的段落」給予較高權重，
  - 再做一層簡單的重排序，挑出最有代表性的 Top-k chunks。

- **對話記憶**  
  - 紀錄最近多輪對話，支援「延續同一台設備」的追問與補充說明。  
  - 例：先問「A 機台 Alarm 2030」，下一句問「那如果重開後還是一樣？」依舊能理解上下文。

- **可擴充的知識庫**  
  - 利用 ChromaDB 的增量寫入機制，新增文件時無需重建整個向量資料庫。
  
---

## 3. 系統架構 (Architecture)

```text

[User Query]
    ↓
[Hybrid Retrieval]
  - Vector search @ ChromaDB
  - BM25 keyword search
  - Metadata filter (EQ_name , alarm_code)
    ↓
nodes rerank & Top-k relevant chunks
    ↓
[Qwen 生成回答]
    ↓
回傳維修步驟 / 建議
