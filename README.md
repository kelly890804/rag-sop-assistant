## 檔案結構
```
RAG SOP網頁小幫手/
├── modules/                 # 模組目錄
│   ├── llm/                 # 語言模型模組
│   │   ├── __init__.py      # 初始化文件
│   │   └── openai_client.py # OpenAI API客戶端
│   ├── rag/                 # RAG模組(新增)
│   │   ├── __init__.py      # 初始化文件
│   │   ├── document_processor.py # 文檔處理器
│   │   ├── vector_store.py  # 向量存儲
│   │   ├── retriever.py     # 檢索器
│   │   └── rag_manager.py   # RAG管理器
│   │   └── cache.py         # 快取
│   │   └── bootstrap.py     # 建立各科室資料庫
│   │   └── ragas_eval.py    # RAG評估
│   ├── agent/               # Agent 模組
│   │   ├── rag_registry.py  # 連接不同科室做RAGManager
│   │   ├── router.py        # 判斷各科室Router
│   └── __init__.py          # 初始化文件
├── jarvis_project/          # Django專案目錄
│   ├── __init__.py          # 初始化文件
│   ├── asgi.py              # ASGI配置
│   ├── settings.py          # Django設定
│   ├── urls.py              # URL配置(更新)
│   ├── views.py             # 視圖函數(更新)
│   └── wsgi.py              # WSGI配置
├── chainlit_app/            # Chainlit前端
│   ├── .chainlit/           # Chainlit配置
│   │   └── config.toml      # 配置檔案
│   ├── app.py               # 前端應用(更新)
│   └── chainlit.md          # 歡迎頁面
├── data/                    # 數據目錄(新增)
│   ├── knowledge_base/      # 知識庫文件夾
│   └── temp/                # 臨時文件夾
│   └── sop_docs/            # 大數據中心SOP PDF檔案
│   └── 會計室SOP             # 會計室SOP PDF檔案
│   └── feedback             #  使用者回饋紀錄（Excel）
├── config.py                # 全局配置文件(更新)
├── manage.py                # Django管理
├── run_jarvis.py            # 統一啟動(更新)
├── .env                     # 環境變量
└── README.md                # 本說明文件
```

## 系統架構
- **Web 介面**:使用Chainlit框架，提供使用者輸入自然語言查詢
- **後端 API (Django)**:負責與前端溝通，查詢處理與 RAG 流程整合
- **RAG Pipeline**:文件前處理與切分、向量化與向量檢索、結合檢索內容生成回應
- **向量資料庫**:使用 Chroma 儲存 SOP 文件向量
- **LLM 推論服務**:透過 Ollama 進行地端大型語言模型推論
- **文件來源**:SOP PDF 文件

## 功能說明
- **多種查詢模式** :
- 🤖 不分科室查詢：系統自動判斷科室並回應，適合不熟悉 SOP 分工的使用者
- 🏢 分科室查詢：針對指定科室 SOP 查詢，適合已知問題來源的使用者

## Prompt 設計
系統依不同階段使用不同提示設計：
- 科室路由：使用固定格式 Prompt，限制模型僅回傳科室代碼或 unknown
- 回答生成：依查詢類型（條列 / 摘要）動態調整指令
- 摘要輔助：限制摘要僅保留與問題最相關資訊，避免上下文冗長


## 多科室 RAG 與路由
- 各科室對應獨立 SOP 知識庫與 RAG Pipeline
- 使用 LLM 自動判斷問題所屬科室並進行路由
- 無法判斷時拒絕生成，避免錯誤檢索
- 架構可彈性擴充新增科室

## 快取
- 對查詢向量、Rerank 分數與 LLM 摘要結果進行快取
- 降低重複推論與計算成本


## RAG 流程設計
- **檢索與生成設計**: 
1. **Hybrid 檢索**
   - 結合語意向量搜尋與 BM25 關鍵字搜尋

2. **候選文件合併與去重**

3. **Reranking**
   - 使用跨編碼模型重新排序候選文件
   - 僅保留高相關內容

4. **相關性門檻控制**
   - 低相關查詢將拒絕生成並回覆「此問題不包含在 SOP 文件中」

5. **查詢類型判斷**
   - 條列型 / 摘要型，自動調整回應格式

6. **上下文封裝與來源標示**
   - 提供可點選的文件與附件來源

7. **LLM 摘要輔助**
   - 精簡上下文後再生成最終回應


## RAG 評估
- 使用 RAGAS 指標進行檢索與回答品質評估
- 用於分析檢索相關性與回答一致性，作為後續優化依據

## 技術細節
- **文檔處理**：使用 LangChain 進行 SOP 文件載入與結構化切分
- **向量嵌入**：使用 sentence-transformers 產生語意向量
- **向量存儲**：使用 Chroma 作為主要向量資料庫，並支援本地持久化
- **Hybrid 檢索**：結合語意向量搜尋與 BM25 關鍵字搜尋，提升檢索完整性
- **Reranking**：使用跨編碼模型重新排序候選文件，提升相關性準確度
- **生成控制**：設定相關性門檻，低相關查詢將拒絕生成以避免錯誤回應
- **上下文增強**：整合檢索結果與使用者問題，並附帶來源資訊供追溯

## 使用情境
本系統設計以院內 SOP 查詢為核心，適用於地端或內網環境部署。
