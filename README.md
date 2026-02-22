# 澳門法律問答系統

一個基於 Supabase 的智能法律問答系統，支持搜尋澳門特區公報法律條文並提供 AI 分析。

## 功能特點

- 🔍 **智能搜尋**：自然語言輸入法律問題
- 🤖 **AI 分析**：使用 Kimi/OpenAI API 進行法律分析
- 📚 **法條引用**：顯示相關法律條文及原文連結
- 🕐 **歷史記錄**：本地保存搜尋歷史
- 📱 **響應式設計**：支持桌面及移動設備

## 系統架構

```
┌─────────────┐     ┌─────────────────┐     ┌──────────────┐
│   前端      │────▶│  Supabase Edge  │────▶│   外部 API   │
│  GitHub     │◀────│    Function     │◀────│ (Kimi/OpenAI)│
│  Pages      │     └─────────────────┘     └──────────────┘
└─────────────┘              │
                             ▼
                      ┌──────────────┐
                      │  PostgreSQL  │
                      │   Database   │
                      └──────────────┘
```

## 快速開始

### 第一步：創建 Supabase 項目

1. 訪問 [Supabase](https://supabase.com) 並註冊/登入帳號
2. 點擊 "New Project" 創建新項目
3. 記下項目 URL 和 `anon` key（在 Project Settings > API 中）

### 第二步：設置數據庫

1. 在 Supabase Dashboard 中，進入 **SQL Editor**
2. 新建查詢，貼上 `supabase/migrations/001_initial.sql` 的內容
3. 點擊 **Run** 執行

### 第三步：部署 Edge Function

1. 安裝 Supabase CLI：
```bash
npm install -g supabase
```

2. 登入 Supabase：
```bash
supabase login
```

3. 初始化項目（如未初始化）：
```bash
supabase init
```

4. 部署 Edge Function：
```bash
supabase functions deploy search
```

5. 設置環境變量（可選，用於 AI 功能）：
```bash
# 使用 Kimi API
supabase secrets set KIMI_API_KEY=your_kimi_api_key

# 或使用 OpenAI API
supabase secrets set OPENAI_API_KEY=your_openai_api_key
```

### 第四步：配置前端

1. Fork 或下載本專案到您的 GitHub 帳號
2. 在 `index.html` 中，用戶首次使用時會提示輸入 Supabase 配置
3. 或者直接在代碼中修改默認配置：

```javascript
const CONFIG = {
    supabaseUrl: 'https://your-project.supabase.co',
    supabaseKey: 'your-anon-key',
    kimiKey: 'your-kimi-key' // 可選
};
```

### 第五步：部署到 GitHub Pages

1. 進入 GitHub 專案頁面
2. 點擊 **Settings** > **Pages**
3. Source 選擇 **Deploy from a branch**
4. Branch 選擇 **main** 和 **/(root)**
5. 點擊 **Save**
6. 等待幾分鐘，您的網站將在 `https://your-username.github.io/law-assistant-v2` 上線

## 文件結構

```
law-assistant-v2/
├── index.html                    # 前端主頁面
├── README.md                     # 本文件
└── supabase/
    ├── functions/
    │   └── search/
    │       └── index.ts          # Edge Function 源碼
    └── migrations/
        └── 001_initial.sql       # 數據庫結構
```

## 數據庫結構

### searches 表
存儲搜尋記錄

| 字段 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| query | TEXT | 搜尋內容 |
| status | TEXT | 狀態 (pending/processing/completed/failed) |
| result_count | INTEGER | 結果數量 |
| created_at | TIMESTAMP | 創建時間 |

### laws 表
存儲法律條文

| 字段 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| search_id | UUID | 關聯搜尋記錄 |
| source_type | TEXT | 來源 (bo/dsaj/other) |
| law_number | TEXT | 法律編號 |
| law_title | TEXT | 法律名稱 |
| article_number | TEXT | 條文編號 |
| content | TEXT | 條文內容 |
| url | TEXT | 原文連結 |
| relevance_score | FLOAT | 相關度評分 |
| ai_summary | TEXT | AI 摘要 |

### qa 表
存儲問答記錄

| 字段 | 類型 | 說明 |
|------|------|------|
| id | UUID | 主鍵 |
| search_id | UUID | 關聯搜尋記錄 |
| question | TEXT | 問題 |
| answer | TEXT | 回答 |
| analysis | JSONB | AI 分析結果 |
| cited_laws | UUID[] | 引用的法條 |

## API 端點

### POST /functions/v1/search
執行法律搜尋

**請求：**
```json
{
  "query": "消費者權益"
}
```

**響應：**
```json
{
  "success": true,
  "search_id": "uuid",
  "query": "消費者權益",
  "summary": "摘要內容",
  "answer": "AI 分析回答",
  "laws": [...],
  "law_count": 2
}
```

### GET /functions/v1/history
獲取搜尋歷史

### GET /functions/v1/search/:id
獲取單個搜尋結果詳情

### GET /functions/v1/health
健康檢查

## 自定義開發

### 修改爬蟲邏輯

目前 Edge Function 使用模擬數據。要實現真正的爬蟲，請修改 `supabase/functions/search/index.ts` 中的 `scrapeBO` 和 `scrapeDSAJ` 函數。

### 添加新的數據源

1. 在 `laws` 表中添加新的 `source_type`
2. 在 Edge Function 中實現對應的爬蟲函數
3. 更新前端顯示邏輯

## 注意事項

⚠️ **免責聲明**：本系統僅供參考，不構成法律意見。如需正式法律建議，請諮詢專業律師。

## 技術棧

- **前端**：純 HTML/CSS/JavaScript
- **後端**：Supabase Edge Functions (Deno)
- **數據庫**：PostgreSQL (Supabase)
- **AI**：Kimi / OpenAI API

## 授權

MIT License

## 貢獻

歡迎提交 Issue 和 Pull Request！

## 聯繫

如有問題，請通過 GitHub Issues 聯繫我們。
