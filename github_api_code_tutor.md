# 解析：GitHub 專案搜尋 Python 程式碼

這份文件主要解析一段透過 GitHub 官方 API 搜尋開源專案（Repositories），並列出星數（Stars）最高前 5 個結果的 Python 程式碼。

## 1. 原始程式碼

```python
import requests

def search_github_repos(keyword):
    url = 'https://api.github.com/search/repositories'
    params = {'q': keyword, 'sort': 'stars', 'order': 'desc'}
    headers = {'Accept': 'application/vnd.github.v3+json'}
    
    try:
        response = requests.get(url, params=params, headers=headers, timeout=10)
        response.raise_for_status()
        data = response.json()
        
        print(f"找到 {data['total_count']} 個結果\n")
        for repo in data['items'][:5]:
            print(f"名稱: {repo['name']}")
            print(f"描述: {repo['description']}")
            print(f"Stars: {repo['stargazers_count']}")
            print(f"網址: {repo['html_url']}")
            print("-" * 40)
            
    except requests.exceptions.RequestException as e:
        print(f"請求失敗: {e}")

if __name__ == "__main__":
    search_github_repos("requests")
```

## 2. 逐步解析

### 2.1 設定 API 請求條件
* **`url`**：指定 GitHub 搜尋 API 的端點網址。
* **`params`**：設定搜尋的詳細條件：
  * `'q': keyword`：想要搜尋的關鍵字。
  * `'sort': 'stars'`：依照「星星數（Stars）」進行排序。
  * `'order': 'desc'`：排序方式為「降冪（descending）」，讓星星數最多的排在最前面。
* **`headers`**：告訴 GitHub 伺服器，我們希望接收 GitHub API v3 版本的 JSON 格式資料，這是確保 API 穩定回傳的良好習慣。

### 2.2 發送請求與錯誤處理
程式碼使用了 `try...except` 區塊，用來確保網路不穩時程式不會直接崩潰退出：
* **`requests.get(...)`**：向 GitHub 發送 GET 請求，並設定 `timeout=10`（如果 10 秒內沒有回應就強制中斷，避免程式永遠卡住）。
* **`response.raise_for_status()`**：檢查 HTTP 回應狀態碼。如果是 4xx（如找不到網頁）或 5xx（伺服器錯誤），這行會主動拋出例外錯誤，讓程式進入 `except` 區塊處理。
* **`except ... as e`**：如果發生網路斷線、超時或上述的 HTTP 錯誤，會在此攔截並印出「請求失敗」的錯誤訊息，而不會顯示一長串難懂的錯誤追蹤（Traceback）。

### 2.3 解析與顯示資料
* **`data = response.json()`**：將 GitHub 回傳的 JSON 格式字串自動轉換成 Python 的字典（Dictionary）結構，方便後續讀取。
* **`data['total_count']`**：取得並印出符合該關鍵字的專案總數。
* **`data['items'][:5]`**：`items` 陣列包含了所有搜尋結果的清單，`[:5]` 是 Python 的切片（Slicing）語法，代表**只取出前 5 筆資料**。
* **`for repo in ...:`**：透過迴圈將這 5 個專案的名稱 (`name`)、描述 (`description`)、星星數 (`stargazers_count`) 以及專案網址 (`html_url`) 逐一排版並印出。

### 2.4 主程式執行區塊
* **`if __name__ == "__main__":`**：這是 Python 的標準起手式，用來檢查這份腳本是作為主程式被「直接執行」，還是被當作模組 `import` 到其他檔案中。只有在直接執行時，下方的程式碼才會運作。
* **`search_github_repos("requests")`**：實際呼叫我們定義好的函式，並傳入關鍵字 `"requests"` 進行搜尋。

## 3. 總結
當執行這支程式時，它會自動向 GitHub 搜尋名稱或內容包含 "requests" 的專案，按照星星數由高到低排序，最終在終端機畫面上印出總共找到的結果數量，以及最熱門前 5 大專案的詳細資訊。這是一個非常標準且實用的串接 RESTful API 的範例程式碼。



| API 类型 | Link (full link) | Description (Used for) |
|---|---|---|
| REST API | https://docs.github.com/zh/rest?apiVersion=2022-11-28 | 用于标准 HTTP CRUD：仓库、Issue、PR、用户、组织、Actions、Releases 等。 |
| GraphQL API | https://docs.github.com/zh/graphql | 用于精确查询复杂/嵌套数据；单一端点，一次请求取多个关联资源。 |
| Webhooks | https://docs.github.com/zh/webhooks | 用于订阅 GitHub 事件；事件发生时主动推送 HTTP POST 到你的服务器。 |
| Search API (REST) | https://docs.github.com/zh/rest/search/search?apiVersion=2022-11-28#search-repositories | 用于搜索仓库、代码、Issue、PR、用户、主题等；例如搜索仓库接口。 |
| Authentication | https://docs.github.com/zh/authentication | 用于 PAT、OAuth App、GitHub App 认证；认证后提升速率限制和访问权限。 |
| Rate Limits | https://docs.github.com/zh/rest/using-the-rest-api/rate-limits-for-the-rest-api | 用于查看 REST API 速率限制、配额和重置时间。 |


备注：把链接里的 `/zh/` 改成 `/en/` 就是英文版。例如：  
https://docs.github.com/en/rest?apiVersion=2022-11-28

