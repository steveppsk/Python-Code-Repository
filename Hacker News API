### Hacker News (HN) 主要提供兩套公開的 API 介面，皆無需認證即可使用，但它們的用途和資料結構完全不同。

### 🔥 官方 Firebase API (用於獲取即時資料)

這是 HN 官方的後端 API，由 Firebase 驅動，提供近乎即時的資料。它的特點是**只返回項目 ID**，你需要再用這些 ID 去獲取具體內容。

| 端點 (Endpoint) | 用途 (Description) | 回傳格式 |
| :--- | :--- | :--- |
| `/v0/maxitem.json` | 取得當前最大的項目 ID，可用於抓取所有歷史資料 | 整數 |
| `/v0/topstories.json` | 取得首頁頭條故事的 ID 列表（最多 500 個） | ID 陣列 |
| `/v0/newstories.json` | 取得最新故事的 ID 列表（最多 500 個） | ID 陣列 |
| `/v0/beststories.json` | 取得最佳故事的 ID 列表 | ID 陣列 |
| `/v0/askstories.json` | 取得 Ask HN 故事的 ID 列表 | ID 陣列 |
| `/v0/showstories.json` | 取得 Show HN 故事的 ID 列表 | ID 陣列 |
| `/v0/jobstories.json` | 取得招聘職缺的 ID 列表 | ID 陣列 |
| `/v0/item/{id}.json` | 取得單一項目（故事、評論、招聘等）的詳細內容 | JSON 物件 |
| `/v0/user/{username}.json` | 取得使用者個人資料 | JSON 物件 |

**使用範例：**
```bash
# 獲取首頁前 30 個故事的 ID
curl https://hacker-news.firebaseio.com/v0/topstories.json

# 獲取 ID 為 8863 的項目詳細資訊
curl https://hacker-news.firebaseio.com/v0/item/8863.json?print=pretty
```

### 🔍 Algolia 搜尋 API (用於全文檢索)

這是 HN 官方搜尋功能（hn.algolia.com）背後的 API，提供強大的全文搜尋和過濾功能。它**直接返回完整的資料物件**，非常適合用來搜尋和分析文章。

| 端點 (Endpoint) | 用途 (Description) | 範例參數 |
| :--- | :--- | :--- |
| `/api/v1/search` | 全文搜尋故事、評論等 | `?query=python&tags=story` |
| `/api/v1/search_by_date` | 按時間排序的搜尋 | `?query=react&tags=story` |
| `/api/v1/items/{id}` | 取得單一項目（與官方 API 類似） | `/api/v1/items/8863` |
| `/api/v1/users/{username}` | 取得使用者資料 | `/api/v1/users/pg` |

**常用過濾參數：**
*   **`tags`**：可組合過濾，如 `story`, `comment`, `poll`, `show_hn`, `ask_hn`。
*   **`numericFilters`**：數值過濾，如 `points>100`, `num_comments>50`, `created_at_i>1609459200`。
*   **`author_USERNAME`**：按作者過濾。
*   **`story_ID`**：取得某個故事下的所有評論。

**使用範例：**
```bash
# 搜尋「python」相關且分數超過 100 的故事
curl "https://hn.algolia.com/api/v1/search?query=python&tags=story&numericFilters=points>100"
```

### ⚖️ 兩套 API 對比與選擇建議

| 特性 | 官方 Firebase API | Algolia 搜尋 API |
| :--- | :--- | :--- |
| **主要用途** | 獲取即時、最新的故事列表和項目詳情 | 進行全文搜尋、過濾和歷史資料檢索 |
| **資料回傳** | 先回傳 ID 列表，需二次請求取得內容 | 直接回傳完整 JSON 物件 |
| **速率限制** | 寬鬆，無需認證 | 每小時 10,000 次請求（無需金鑰） |
| **適合場景** | 建立即時看板、抓取首頁最新文章 | 搜尋特定關鍵字、分析歷史趨勢、研究特定作者 |

### 🛠️ 第三方工具與生態系

除了上述兩套核心 API，社群也基於它們開發了許多便利的工具：
*   **客戶端庫 (Client Libraries)**：如 `hn-client` (TypeScript)、`GoHN` (Go) 等，封裝了 API 呼叫。
*   **MCP 伺服器**：讓 AI 代理（如 Claude）能直接與 HN 互動，例如 `hn-mcp`、`rawveg/hacker-news-mcp`。
*   **付費替代方案**：如 **Apify** 提供現成的爬蟲 Actor，可直接輸出結構化資料，適合需要批量處理或自訂欄位的場景。

總結來說，**官方 Firebase API** 適合獲取即時列表，而 **Algolia API** 則是你需要搜尋和分析時的首選。兩者結合使用，就能滿足絕大多數與 Hacker News 互動的需求。
