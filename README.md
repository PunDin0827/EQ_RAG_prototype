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

---

## 2. 功能特色 

- **RAG 問答**  
  - 文件切分 → 產生 MetaData → 向量化 (Embedding) → 寫入 ChromaDB → 檢索 → Qwen 生成回答。  

- **Hybrid Retrieval：向量＋關鍵字＋MetaData**  
  - 以向量相似度搭配 **BM25** 關鍵字分數。  
  - 透過 **MetaData Filter** 限縮設備型號、Alarm Code 等欄位，提升檢索精度。  

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
nodes rerank
    ↓
Top-k relevant chunks
    ↓
[Qwen 生成回答]
    ↓
回傳維修步驟 / 建議
