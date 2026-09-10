這是一份為你整理的完整對話總結，以及最終為你量身打造的 Streamlit 程式碼。

### 📝 整個對話的總結 (Chat Summary)

在這次對話中，我們經歷了從「靜態 HTML」到「動態互動儀表板」的架構轉變，並解決了多個技術痛點：

1. **初始階段 (Python 生成 HTML)**：
   * 你最初使用 Plotly 的 `to_html()` 生成靜態 HTML，並嘗試用純 JavaScript (`plotly_click`) 來實現點擊圓餅圖下鑽 (Drill-down) 到專科明細。
   * **遇到的問題**：數據加總錯誤（顯示只有 33 張床）、圖表出現亂碼數字、UI 排版重疊（「返回」按鈕蓋住標題）。
   * **解決方案**：發現數據庫包含多個年度，導致數據被重複加總。透過 `Financial Year` 過濾出最新年份，並加入更嚴謹的數據清洗（過濾小計/合計、處理逗號）。同時修復了 Plotly Express 的 Hover 顯示 Bug。

2. **架構討論 (Tableau vs. n8n vs. Streamlit)**：
   * 你意識到寫 HTML/JS 維護成本太高，詢問了 Tableau 和 n8n。
   * **結論**：Tableau 雖然視覺化強，但嵌入 HTML 只是 iframe，無法與你的 Python 數據流互動；n8n 是流程自動化工具，圖表能力弱。**Streamlit** 才是最適合你的方案，因為它可以直接用 Python 寫互動邏輯，無需手寫 HTML/JS。

3. **Streamlit 遷移與除錯**：
   * 你將代碼轉移到 Streamlit，但遇到「網頁一片空白」。
   * **除錯過程**：
     * 發現你使用了 `python xxx.py` 執行，而非 `streamlit run xxx.py`。
     * 發現你將檔案命名為 `streamlit.py`，導致 Python 循環導入 (Circular Import) 報錯。
     * 提醒你使用 `Ctrl + C` 來關閉正在運行的 Localhost 伺服器。
   * 最終我們決定**從最簡單的圓餅圖開始**，先確保環境正常，再逐步加入互動功能。

---

### 💻 最終 Streamlit 程式碼 (app.py)

請將以下代碼存成 `app.py`（**千萬不要叫 `streamlit.py`**），並在 Terminal 執行 `streamlit run app.py`。

```python
import streamlit as st
import pandas as pd
import plotly.express as px

# 1. 頁面設定
st.set_page_config(page_title="醫院床位簡單分析", layout="wide")
st.title("🏥 香港醫院床位數量 - 簡單圓餅圖")

# 2. 讀取與清洗數據
@st.cache_data
def load_data():
    # 請確保此路徑正確
    csv_path = r"D:\Z-Works\Python_Project\Web_scraping\hosp_beds.csv"
    df = pd.read_csv(csv_path)
    
    # 確保 Hospital Beds 係數字 (移除逗號並轉為 numeric)
    df['Hospital Beds'] = pd.to_numeric(
        df['Hospital Beds'].astype(str).str.replace(',', '').str.strip(), 
        errors='coerce'
    ).fillna(0)
    
    # 過濾走「合計」同「小計」
    df = df[~df['Hospital'].astype(str).str.contains('合計|小計', na=False)]
    
    return df

df = load_data()

# 3. 年份選擇器 (Sidebar)
st.sidebar.header("📅 篩選條件")
# 將年份降序排列 (最新年份在最前)
available_years = sorted(df['Financial Year'].unique(), reverse=True)

# 預設揀 2024，如果冇 2024 就揀最新嗰年
default_index = available_years.index('2024') if '2024' in available_years else 0
selected_year = st.sidebar.selectbox("請選擇年份：", available_years, index=default_index)

# 4. 根據所選年份過濾數據
df_year = df[df['Financial Year'] == selected_year].copy()

# 5. 按醫院加總床位，並排序
hospital_total = df_year.groupby('Hospital')['Hospital Beds'].sum().reset_index()
hospital_total = hospital_total.sort_values(by='Hospital Beds', ascending=False)

# 6. 處理 Top 5 與 OTHERS
top5_names = hospital_total.head(5)['Hospital'].tolist()
hospital_total['Group'] = hospital_total['Hospital'].apply(
    lambda x: x if x in top5_names else 'OTHERS'
)

# 將 OTHERS 嘅床位加埋一齊
grouped = hospital_total.groupby('Group', as_index=False)['Hospital Beds'].sum()

# 為咗令圓餅圖嘅排序好睇啲，將 OTHERS 擺喺最後
grouped['sort_order'] = grouped['Group'].apply(lambda x: 1 if x == 'OTHERS' else 0)
grouped = grouped.sort_values(by=['sort_order', 'Hospital Beds'], ascending=[True, False])

# 7. 畫圓餅圖
fig_pie = px.pie(
    grouped, 
    names='Group', 
    values='Hospital Beds',
    title=f'{selected_year} 年度 香港各醫院床位數佔比 (Top 5 + OTHERS)',
    color_discrete_sequence=px.colors.qualitative.Pastel,
    hole=0.35 # 甜甜圈圖
)

fig_pie.update_traces(
    textposition='inside', 
    textinfo='percent+label',
    hovertemplate="<b>%{label}</b><br>床位總數: %{value} 張<br>佔比: %{percent}<extra></extra>"
)

fig_pie.update_layout(
    height=600,
    margin=dict(t=80, b=40, l=40, r=40)
)

# 8. 顯示圖表 (使用 width='stretch' 以符合新版 Streamlit 規範)
st.plotly_chart(fig_pie, use_container_width=True)

# 9. 顯示數據預覽 (方便你對數)
with st.expander("🔍 點擊查看數據預覽"):
    st.write(f"**{selected_year} 年度 Top 5 醫院：**")
    st.dataframe(hospital_total.head(5)[['Hospital', 'Hospital Beds']])
    st.write(f"**OTHERS 包含嘅醫院數量：** {len(hospital_total) - 5} 間")
    st.write(f"**{selected_year} 年度總床位數：** {int(grouped['Hospital Beds'].sum())} 張")
```

### 🚀 執行與關閉步驟：
1. **執行**：在 Terminal 輸入 `streamlit run app.py`。
2. **瀏覽**：打開瀏覽器 `http://localhost:8501`。
3. **關閉**：回到 Terminal，按下 `Ctrl + C` 即可強制停止伺服器。

### 💡 下一步建議：
當你確認這個簡單的圓餅圖能正常顯示，且點開「數據預覽」看到數字合理（例如總床位數是幾千張，而不是 33 張）之後，我們就可以在這個基礎上，安全地加入「點擊圓餅圖下鑽到專科」的功能。
