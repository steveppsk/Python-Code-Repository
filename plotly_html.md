明白！如果你的儀表板是動態擴充的（例如後期會隨時加入第三張、第四張圖表），寫死高度（如 `height=600`）或強制限制最大寬度確實會失去彈性，日後維護會很麻煩。

要做到像 Tableau 那種「不管加幾多張圖，版面都會自動偵測並適應螢幕/容器大小（Auto-fit）」的效果，我們不能依賴固定像素，而是要利用 **Plotly 的內置響應式引擎 (`autosize=True`)** 搭配 **CSS Grid 或 Flexbox 的彈性網格排版**。

這樣做的好處是：當你以後增加新圖表時，只需丟進網格容器裡，網頁會自動幫你排版，不用手動調校尺寸。

以下是實現**全自動動態適應版面**的完整方案：

### 1. Plotly 圖表設定：開啟純自動延展

在 Plotly 中，不要給固定的 `height`，而是讓它吃滿父層容器的寬高：

```python
# 長條圖自動適應設定
fig_bar.update_layout(
    autosize=True,
    height=None, # 清除固定高度，讓 CSS 控制
    margin=dict(t=40, b=20, l=20, r=20),
    xaxis_title="醫院名稱",
    yaxis_title="病床總數",
    xaxis_tickangle=-45,
    hovermode="x unified"
)

# 圓餅圖自動適應設定
fig_pie.update_layout(
    autosize=True,
    height=None,
    margin=dict(t=40, b=20, l=20, r=20),
)
```

### 2. HTML/CSS 網格系統 (Dashboard Grid Layout)

我們利用 CSS 的 Grid（網格）或 Flexbox，建立一個自動流動的儀表板架構。這樣未來就算加入更多圖表，它也會自動排列。

請使用以下這套升級版的完整 Python 程式碼：

```python
import pandas as pd
import plotly.express as px

# ----------------- 1. 資料讀取與處理 -----------------
df = pd.read_csv(r"xxxxxxx\hosp_beds.csv")
df = df[~df['Hospital'].str.contains('小計|合計', na=False)]

hospital_beds = df.groupby('Hospital')['Hospital Beds'].sum().reset_index()
hospital_beds = hospital_beds.sort_values(by='Hospital Beds', ascending=False)

# ----------------- 2. 繪製長條圖 -----------------
fig_bar = px.bar(
    hospital_beds, x='Hospital', y='Hospital Beds', title='香港各醫院病床數量分佈',
    text_auto=True, color='Hospital Beds', color_continuous_scale='Blues'
)
# 設定自動延展 (關鍵)
fig_bar.update_layout(
    autosize=True,
    margin=dict(t=50, b=30, l=30, r=30),
    xaxis_title="醫院名稱", yaxis_title="病床總數", 
    xaxis_tickangle=-45, hovermode="x unified"
)

# ----------------- 3. 繪製圓餅圖 -----------------
top5 = hospital_beds.head(5)
others_sum = hospital_beds.iloc[5:]['Hospital Beds'].sum()
others_df = pd.DataFrame({'Hospital': ['其他'], 'Hospital Beds': [others_sum]})
pie_data = pd.concat([top5, others_df])

fig_pie = px.pie(
    pie_data, names='Hospital', values='Hospital Beds', title='香港前五大醫院病床數佔比',
    color_discrete_sequence=px.colors.qualitative.Pastel
)
# 設定自動延展並優化互動
fig_pie.update_traces(
    textposition='inside', textinfo='percent+label',
    hovertemplate="<b>%{label}</b><br>病床總數: %{value} 張<br>佔比: %{percent}<extra></extra>"
)
fig_pie.update_layout(
    autosize=True,
    margin=dict(t=50, b=30, l=30, r=30)
)

# ----------------- 4. 轉換 HTML -----------------
html_bar = fig_bar.to_html(full_html=False, include_plotlyjs='cdn')
html_pie = fig_pie.to_html(full_html=False, include_plotlyjs=False)

# ----------------- 5. 具備「自動適應與擴充」功能的網頁模板 -----------------
html_template = f"""
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>香港醫院病床數量動態儀表板</title>
    <!-- 引入 Plotly 內置的響應式 JS 支援，確保隨視窗大小縮放 -->
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    <style>
        body {{
            font-family: 'Microsoft JhengHei', Arial, sans-serif; 
            background-color: #f4f7f6; 
            margin: 0;
            padding: 20px;
        }}
        h1 {{
            text-align: center; 
            color: #333;
            margin-bottom: 25px;
        }}
        /* 
          類似 Tableau 的自動響應儀表板網格 (Dashboard Grid)
          當螢幕夠大時，它可以自動排版；後期加圖表時會順著往下或往右流動
        */
        .dashboard-grid {{
            display: grid;
            grid-template-columns: 1fr; /* 預設單欄向下排列，也可以改成 auto-fit 做左右並排 */
            gap: 20px;
            max-width: 1600px;
            margin: 0 auto;
        }}
        /* 圖表卡片容器：強制設定高度，內部的 Plotly 會自動填滿這個高度 */
        .chart-card {{
            background-color: white; 
            padding: 15px; 
            border-radius: 10px; 
            box-shadow: 0 4px 8px rgba(0,0,0,0.1);
            width: 100%;
            box-sizing: border-box;
            height: 550px; /* 統一卡片高度，或可依圖表性質調整 */
            display: flex;
            flex-direction: column;
        }}
        /* 讓 Plotly 內部 SVG 完美撐滿卡片容器 */
        .chart-card > div {{
            width: 100% !important;
            height: 100% !important;
        }}
    </style>
</head>
<body>
    <h1>香港醫院病床數量分析儀表板</h1>
    
    <div class="dashboard-grid">
        <!-- 圖表 1 -->
        <div class="chart-card">
            {html_bar}
        </div>
        
        <!-- 圖表 2：以後想加圖表 3、圖表 4，直接複製這段 <div class="chart-card"> 即可 -->
        <div class="chart-card">
            {html_pie}
        </div>
    </div>

    <!-- 自動縮放觸發腳本：確保瀏覽器拉動或更改大小時計算正確比例 -->
    <script>
        window.addEventListener('resize', function(){{
            var plots = document.querySelectorAll('.plotly-graph-div');
            plots.forEach(function(p){{
                Plotly.Plots.resize(p);
            }});
        }});
    </script>
</body>
</html>
"""

output_path = r"D:\Z-Works\Python_Project\Web_scraping\hospital_dashboard.html"
with open(output_path, "w", encoding="utf-8") as f:
    f.write(html_template)

print(f"✅ 自動適應動態版面 HTML 已更新: {output_path}")

```

### 為什麼這樣做能達到類似 Tableau 的效果？

1. **容器與圖表分離**：我們利用 CSS 控制 `.chart-card` 的高度（例如 `550px`），而 Plotly 設定 `autosize=True`。這代表圖表會**像橡皮筋一樣自動縮放**來貼合容器。
2. **支援未來無限擴充**：後期如果你要加第三張圖，只需要在 `.dashboard-grid` 裡面多加一個 `<div class="chart-card">新圖表</div>`，網頁就會自動幫你往下排版。
3. **動態螢幕偵測 (`window.addEventListener('resize')`)**：最底下的 JavaScript 會即時監聽你的瀏覽器視窗大小。不管你縮放視窗、全螢幕或用不同大小的螢幕打開，圖表線條和圓餅圖都會自動重新計算比例，不會變形或出現過多白邊。
