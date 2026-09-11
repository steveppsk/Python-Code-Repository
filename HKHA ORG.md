以下是醫院管理局（HA）在 `www.ha.org.hk/opendata` 上發布的主要開放數據集列表。這些數據集大多同時提供 JSON 和 XLSX 格式。

### 📋 醫院管理局開放數據集列表

| 數據集名稱 | 描述 / 用途 | JSON 連結 |
|-----|-----|---|
| **醫院/機構/專科門診診所/家庭醫學診所目錄** | 提供醫管局轄下各醫院聯網的醫院、機構、專科門診及家庭醫學診所清單。 | `https://www.ha.org.hk/opendata/facility-hosp.json` |
| **普通科門診診所目錄** | 提供醫管局轄下普通科門診診所的清單。 | `https://www.ha.org.hk/opendata/facility-gop.json` |
| **專科門診診所目錄** | 提供醫管局轄下專科門診診所的清單。 | `https://www.ha.org.hk/opendata/facility-sop.json` |
| **急症室等候時間** | 提供各公立醫院急症室的實時預計等候時間，每 15 分鐘更新。 | `https://www.ha.org.hk/opendata/aed/aedwtdata-tc.json` |
| **住院及日間住院病人服務量** | 提供醫院病床數目、病人出院人次及死亡人數等統計數據。 | `https://www.ha.org.hk/opendata/hosp-bed-tc.json` |
| **日間及社康服務量** | 提供急症室就診人次、專科門診就診人次等日間及社區服務統計數據。 | `https://www.ha.org.hk/opendata/ae-attnd-sc.json` |
| **服務需求高峰期重點數據** | 提供服務需求高峰期（如流感季節）的公立醫院關鍵統計數據，如急症室首次就診人次、內科病房佔用率等。 | `https://www.ha.org.hk/opendata/pas_report/Daily_Services_Statistics/Daily_Services_Statistics_EN.json` |
| **人手狀況 - 等同全職人員數目** | 提供醫管局按職員組別劃分的等同全職人員數目數據。 | `https://www.ha.org.hk/opendata/manpower-position-by-clusters-sc.json` |
| **醫院管理局的主要財務信息** | 提供醫管局的主要收入及支出項目數據。 | `https://www.ha.org.hk/opendata/key-financial-information-HA-en.json` |
| **中醫診所及教研中心就診數據** | 提供中醫診所暨教研中心的就診人次數據。 | `https://www.ha.org.hk/opendata/cmctr/cmctr-attnd-en.json` |
| **選定內窺鏡程序的統計數字** | 提供醫管局選定內窺鏡程序的服務量統計數據。 | `https://www.ha.org.hk/opendata/endo-en.json` |
| **專職醫療(門診)就診人次** | 提供按專職醫療部門及診所劃分的就診人次數據。 | `https://www.ha.org.hk/opendata/ahop-attnd-en.xlsx` |

### 📌 補充說明

*   **數據格式**：上述數據集大多提供 **JSON** 和 **XLSX** 兩種格式，JSON 檔案適合程式化讀取，XLSX 檔案則適合直接開啟檢視。
*   **多語言支援**：部分數據集提供繁體中文（`-tc`）、簡體中文（`-sc`）和英文（`-en`）版本，你可以根據需要替換連結中的語言代碼。
*   **更新頻率**：不同數據集的更新頻率各異，例如急症室等候時間每 15 分鐘更新，而醫院目錄則為每季更新。
*   **數據來源**：這些開放數據同時在 **DATA.GOV.HK** 平台上發布，你可以透過該平台獲取更完整的數據集描述和更新記錄。

如果你需要某個特定數據集的 Python 讀取範例，可以隨時提出。

當你拿到一個全新的 API 卻不知道它的內容和資料格式時，不用擔心，這是所有開發者都會遇到的第一關。

根據香港政府資料一線通（DATA.GOV.HK）的紀錄，你手上的這個 API (`hosp-bed-tc.json`) 是香港醫院管理局提供的開放數據，內容為「按專科及聯網/醫院劃分的醫院病床數目」的繁體中文版。

要拆解並處理這個 API，你可以按照以下三個步驟來進行：

### 第一步：預覽與觀察結構（Format）

在寫程式之前，我們通常會先「肉眼觀察」它的資料長什麼樣子。

1. **直接用瀏覽器開啟**：把 `[https://www.ha.org.hk/opendata/hosp-bed-tc.json](https://www.ha.org.hk/opendata/hosp-bed-tc.json)` 貼到 Chrome、Edge 或 Safari 的網址列直接打開。
2. **使用美化工具**：如果你看到的只是一堆密密麻麻擠在一起的文字，你可以把那些文字全部複製，貼到線上工具如 **[JSONFormatter](https://jsonformatter.org/)** 中。它會幫你把資料排版成整齊的樹狀結構，讓你一眼就能看出它包含哪些欄位（例如：年份、醫院名稱、專科名稱、病床數量等）。

### 第二步：使用程式拉取並解析資料

知道資料大概長怎樣後，就可以用程式把它抓下來。以下是用 Python 處理這個 API 的基礎寫法。這段程式碼的核心任務是「把資料載下來，並且印出第一筆資料的格式」，幫助你掌握所有的鍵值（Keys）：

```python
import requests
import json

# 你手上的 API 網址
url = "https://www.ha.org.hk/opendata/hosp-bed-tc.json"

try:
    # 1. 向 API 發送請求拿取資料
    response = requests.get(url)
    response.raise_for_status()  # 確保請求成功 (沒有遇到 404 等錯誤)
    
    # 2. 將回傳的文字轉換為 Python 可讀的 JSON (字典或串列)
    data = response.json()
    
    print(f"✅ 成功取得資料！總共有 {len(data)} 筆記錄。\n")
    print("-" * 40)
    
    # 3. 抓出「第 1 筆」資料來印出，觀察它提供哪些欄位
    if len(data) > 0:
        print("第一筆資料的完整結構 (Format)：")
        # indent=4 會讓排版變整齊，ensure_ascii=False 確保中文字能正常顯示
        print(json.dumps(data[0], indent=4, ensure_ascii=False))

except Exception as e:
    print(f"❌ 發生錯誤：{e}")

```

### 第三步：決定如何輸出（Output）

當你執行了上面的程式，看到了具體的欄位名稱後，你就可以決定要怎麼 Output。最常見的做法是**轉換成 Excel / CSV 檔案**，方便非技術人員閱讀，或是用來做後續的數據分析。

你可以利用 Python 的 `pandas` 套件，只要兩行程式碼就能轉出 CSV：

```python
import pandas as pd

# 假設你已經取得了上方的 data
df = pd.DataFrame(data)

# 匯出成 CSV 檔案，encoding='utf-8-sig' 可以避免用 Excel 打開時中文字變成亂碼
df.to_csv('hosp_beds.csv', index=False, encoding='utf-8-sig')

print("資料已成功匯出至 hosp_beds.csv！")

```

總結處理流程：先看懂長相 ➡️ 寫程式抓取 ➡️ 轉存成需要的格式（CSV/資料庫/網頁顯示）。

