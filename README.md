# Qwen + RAG：設備維修問答機器人

- 結合檢索增強的維修助理，用於故障排除與維修指南。
- 資料來源：維修手冊、SOP、保修紀錄、常見故障（陸續增量匯入）。  
- 文件切分 → MetaData → Embedding → ChromaDB → 檢索 → Qwen 生成回答。  
- 向量相似度、MetaDataFilter、BM25 綜合評分取 nodes。  
- 支援上下文記憶功能。  
