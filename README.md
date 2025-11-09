# Qwen + RAG：設備維修問答機器人

一個結合 **Qwen 大語言模型** 與 **Retrieval-Augmented Generation (RAG)**  的設備維修助理，  
協助工程師依據設備型號、Alarm Code 與關鍵字，快速查找故障原因與排除步驟。  

---

## 1. 專案簡介 

- 目標：整合 **維修手冊、SOP、保修紀錄、常見故障整理表** 的資訊當作資料庫，讓工程師不用翻文件，只要對一個對話式助理提問，就能查到維修手冊、SOP 與保修紀錄中的相關內容。
- 典型使用情境：輸入「設備 A / Alarm 1050 無法復歸」，系統自動檢索相關段落並用自然語言彙整處理流程。
- 支援在 **無外網** 下執行（可搭配本地模型或內網部署的 Qwen 服務）。

---

## 2. 功能特色 

- **RAG 問答**  
  - 文件切分 → 產生 MetaData → 向量化 (Embedding) → 寫入 ChromaDB → 檢索 → Qwen 生成回答。  
  - Retrieval pipeline: document chunking → metadata → embeddings → ChromaDB → hybrid search → generation.

- **Hybrid Retrieval：向量＋關鍵字＋MetaData**  
  - 以向量相似度 (cosine similarity) 為主，搭配 **BM25** 關鍵字分數。  
  - 透過 **MetaData Filter** 限縮設備型號、Alarm Code、文件類型等欄位，提升檢索精度。  

- **對話記憶 (Conversation Memory)**  
  - 紀錄最近多輪對話，支援「延續同一台設備」的追問與補充說明。  
  - 例：先問「A 機台 Alarm 2030」，下一句問「那如果重開後還是一樣？」依舊能理解上下文。

- **可擴充的知識庫 (Extensible Knowledge Base)**  
  - 新增 Markdown / PDF / TXT 後，可透過匯入腳本重新向量化並寫入 ChromaDB。  
  - 支援 incremental update，而不需要重建整個資料庫。

---

## 3. 系統架構 (Architecture)

```text
使用者問題
    ↓
[Query Preprocessing]
    ↓
[Hybrid Retrieval]
  - Vector search @ ChromaDB
  - BM25 keyword search
  - Metadata filter (model, alarm_code, doc_type)
    ↓
Top-k relevant chunks
    ↓
[Qwen 生成回答 (LLM)]
    ↓
回傳維修步驟 / 建議
