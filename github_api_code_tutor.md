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



### 4.**GITHUB API LINK 匯總**：

以下根據你提供的 JSON 整理成繁體中文表格。  
其中 `{...}` 是 **URI 模板佔位符**，實際呼叫時要替換成真實值。

| 欄位 | Direct Link | 用途 / Description (Used for) |
|---|---|---|
| `current_user_url` | `https://api.github.com/user` | 取得目前認證使用者的資料。 |
| `current_user_authorizations_html_url` | `https://github.com/settings/connections/applications{/client_id}` | 在 GitHub 網頁上管理目前使用者已授權的 OAuth App；可帶 `client_id`。 |
| `authorizations_url` | `https://api.github.com/authorizations` | 管理目前使用者的 OAuth 授權資訊。 |
| `code_search_url` | `https://api.github.com/search/code?q={query}{&page,per_page,sort,order}` | 搜尋程式碼；`{query}` 為搜尋關鍵字。 |
| `commit_search_url` | `https://api.github.com/search/commits?q={query}{&page,per_page,sort,order}` | 搜尋提交記錄。 |
| `emails_url` | `https://api.github.com/user/emails` | 取得目前認證使用者的電子郵件地址。 |
| `emojis_url` | `https://api.github.com/emojis` | 取得 GitHub 所有 emoji 列表。 |
| `events_url` | `https://api.github.com/events` | 取得 GitHub 公開事件時間軸。 |
| `feeds_url` | `https://api.github.com/feeds` | 取得目前使用者可用的 feeds 列表，例如 Atom feeds。 |
| `followers_url` | `https://api.github.com/user/followers` | 取得目前使用者的追蹤者列表。 |
| `following_url` | `https://api.github.com/user/following{/target}` | 取得目前使用者正在追蹤的人；`{/target}` 可指定某個使用者檢查是否被追蹤。 |
| `gists_url` | `https://api.github.com/gists{/gist_id}` | 取得或管理 Gists；`{/gist_id}` 指定某個 Gist。 |
| `hub_url` | `https://api.github.com/hub` | GitHub PubSubHubbub hub 訂閱端點，用於較舊的事件即時推送機制。 |
| `issue_search_url` | `https://api.github.com/search/issues?q={query}{&page,per_page,sort,order}` | 搜尋 Issue 和 Pull Request。 |
| `issues_url` | `https://api.github.com/issues` | 取得目前認證使用者相關的 Issue，通常跨儲存庫。 |
| `keys_url` | `https://api.github.com/user/keys` | 取得目前使用者的 SSH keys。 |
| `label_search_url` | `https://api.github.com/search/labels?q={query}&repository_id={repository_id}{&page,per_page}` | 搜尋標籤；需要指定 `repository_id`。 |
| `notifications_url` | `https://api.github.com/notifications` | 取得目前使用者的通知。 |
| `organization_url` | `https://api.github.com/orgs/{org}` | 取得指定組織的資訊；`{org}` 為組織名稱。 |
| `organization_repositories_url` | `https://api.github.com/orgs/{org}/repos{?type,page,per_page,sort}` | 取得指定組織的儲存庫列表。 |
| `organization_teams_url` | `https://api.github.com/orgs/{org}/teams` | 取得指定組織的團隊列表。 |
| `public_gists_url` | `https://api.github.com/gists/public` | 取得公開 Gists 列表。 |
| `rate_limit_url` | `https://api.github.com/rate_limit` | 查詢目前 API 速率限制狀態。 |
| `repository_url` | `https://api.github.com/repos/{owner}/{repo}` | 取得或管理指定儲存庫；`{owner}` 為擁有者，`{repo}` 為儲存庫名稱。 |
| `repository_search_url` | `https://api.github.com/search/repositories?q={query}{&page,per_page,sort,order}` | 搜尋儲存庫。 |
| `current_user_repositories_url` | `https://api.github.com/user/repos{?type,page,per_page,sort}` | 取得目前認證使用者的儲存庫列表。 |
| `starred_url` | `https://api.github.com/user/starred{/owner}{/repo}` | 取得目前使用者 Star 的儲存庫；可帶 `owner/repo` 檢查某儲存庫是否已 Star。 |
| `starred_gists_url` | `https://api.github.com/gists/starred` | 取得目前使用者 Star 的 Gists。 |
| `topic_search_url` | `https://api.github.com/search/topics?q={query}{&page,per_page}` | 搜尋 GitHub Topics。 |
| `user_url` | `https://api.github.com/users/{user}` | 取得指定使用者的公開資訊；`{user}` 為使用者名稱。 |
| `user_organizations_url` | `https://api.github.com/user/orgs` | 取得目前認證使用者所屬的組織。 |
| `user_repositories_url` | `https://api.github.com/users/{user}/repos{?type,page,per_page,sort}` | 取得指定使用者的儲存庫列表。 |
| `user_search_url` | `https://api.github.com/search/users?q={query}{&page,per_page,sort,order}` | 搜尋使用者。 |

### 常見佔位符說明

| 佔位符 | 含義 |
|---|---|
| `{query}` | 搜尋關鍵字 |
| `{owner}` | 儲存庫擁有者 |
| `{repo}` | 儲存庫名稱 |
| `{org}` | 組織名稱 |
| `{user}` | 使用者名稱 |
| `{gist_id}` | Gist ID |
| `{target}` | 目標使用者名稱 |
| `{client_id}` | OAuth App 的 Client ID |
| `{repository_id}` | 儲存庫 ID |
| `{?type,page,per_page,sort}` | 可選查詢參數，例如 `?type=all&page=1` |
| `{&page,per_page,sort,order}` | 接在已有 `?q=...` 後面的可選查詢參數，例如 `&page=1&per_page=10` |

這些 URL 都來自 GitHub REST API 根端點：`https://api.github.com/`。部分介面需要認證，使用 GitHub Token 可提高速率限制。
