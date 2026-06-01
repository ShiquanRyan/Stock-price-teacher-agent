# 🤖 AI 股市領航員 (Stock Market Mentor Agent) 技術流程規範書

本專案實作一個基於 **Code-First (代碼主導)** 設計哲學的 AI 股市多代理人 (Multi-Agent) 導讀系統。系統能動態擷取即時金融數據、總體經濟指標與媒體輿情，並透過**「動態條件路由」**與**「雙軌事實查核」**機制，提供使用者深入淺出、且不被媒體噪音誤導的每日股市成因導讀。

本系統專為高併發試用情境設計，全面採用開源技術棧，並導入 Serverless 雲端架構與後端快取攔截機制，兼顧執行穩定度與 Token 成本控制，具備極高的高可用性（High Availability）與系統強健性（Robustness）。

---

## 🛠️ 一、 開源技術棧與環境組態 (Tech Stack & Setup)

本專案堅持「全開源生態系」原則，除基礎雲端運算與大語言模型 API 外，其餘組件皆採用開源且免費的解決方案。

### 1. 核心技術選型
* **Agent 核心框架**：`langgraph` (用於構建有向圖狀態機，精準控管資料流)
* **LLM 驅動（可自由替換）**：`langchain-google-genai` (Gemini API) 或 `langchain-openai` (OpenAI API)
* **後端 API 服務**：`fastapi` + `uvicorn` (異步高效能 Web 框架，支援高併發 Websocket / HTTP)
* **快取與記憶體資料庫**：`redis` (開源高可用記憶體資料庫)
* **數據與資料爬蟲**：`yfinance` (金融與財報數據)、`beautifulsoup4` + `requests` (輿情爬蟲)
* **前端互動介面**：`streamlit` (純 Python 響應式前端)

### 2. 本地開發環境配置
#### 步驟 A：複製本專案並建立虛擬環境
* 透過 Git 工具將遠端儲存庫複製至本地，並使用 Python 內建的 venv 模組建立隔離之虛擬環境，隨後依據作業系統類型啟動環境。

#### 步驟 B：安裝依賴套件
* 在專案根目錄下配置依賴宣告清單，包含自動化圖形狀態機套件、大模型整合介面套件、非同步網頁框架套件、記憶體資料庫驅動套件、金融與網絡爬蟲套件以及環境變數解析套件，並透過終端機執行封裝安裝。

#### 步驟 C：環境變數配置 (`.env`)
* 在專案根目錄下手動建立環境組態檔案，明確宣告包含大語言模型金鑰、外部快取資料庫存取端點、服務監聽連接埠以及生產環境識別標籤之配置項目。

---

## 🔄 二、 系統架構與資料流流程圖 (Architecture Diagram)

以下為完整的端到端處理流程圖。包含「前端攔截層」、「後端核心 LangGraph 狀態機」與「多智能體事實查核」的詳細動態路由。
```mermaid
graph TD
    %% 樣式定義
    classDef user fill:#ececff,stroke:#9393ff,stroke-width:2px;
    classDef cache fill:#fff2cc,stroke:#d6b656,stroke-width:2px;
    classDef graphNode fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px;
    classDef agent fill:#d5e8d4,stroke:#82b366,stroke-width:2px;
    classDef endNode fill:#f8cecc,stroke:#b85450,stroke-width:2px;

    %% 流程節點
    Start([使用者輸入股票代號]) --> FastAPI{FastAPI 後端入口}
    class Start user;
    
    %% 快取層
    FastAPI -->|1. 查詢快取| CheckCache{Redis 快取是否存在?}
    class CheckCache cache;
    CheckCache -->|頁面命中 Cache Hit| ReturnCache[0.01 秒閃電回傳分析報告]
    class ReturnCache endNode;
    ReturnCache --> RenderStreamlit[Streamlit 前端美化呈現]
    
    %% LangGraph 核心層
    CheckCache -->|頁面未命中 Cache Miss| LangGraph[初始化 LangGraph 狀態機]
    class LangGraph graphNode;
    LangGraph --> Node1[節點: 常態監測工具 <br> 擷取大盤/總經/VIX/媒體新聞]
    class Node1 graphNode;
    
    %% 動態路由
    Node1 --> ConditionalEdge{決策邊: 文本關鍵字掃描 <br> 是否提及財報/法說會?}
    class ConditionalEdge graphNode;
    
    ConditionalEdge -->|是 Yes| Node2[節點: 深度文件爬蟲 <br> 下載官方財報與法說會逐字稿]
    class Node2 graphNode;
    ConditionalEdge -->|否 No| Agent1
    
    %% 多智能體協作層
    Node2 --> Agent1[Agent 1: 新聞主編 <br> 媒體文本降維與去噪]
    class Agent1 agent;
    Agent1 --> Agent2[Agent 2: 事實查核大師 <br> 雙軌交叉驗證/偏見過濾]
    class Agent2 agent;
    Agent2 --> Agent3[Agent 3: 財經小老師 <br> 白話導讀/催化劑預警/自我覺察機制]
    class Agent3 agent;
    
    %% 交付層
    Agent3 --> WriteCache[回寫 Redis 快取 <br> 並設定 TTL 生存時間]
    class WriteCache cache;
    WriteCache --> ReturnNew[輸出最新結構化 JSON]
    ReturnNew --> RenderStreamlit
    RenderStreamlit --> End([結束導讀體驗])
    class End endNode;
