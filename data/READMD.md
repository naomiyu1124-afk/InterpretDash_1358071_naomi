# 🩺 醫資大數據分析專案：EBM 模型開發與異常診斷紀錄

## 1. 資料預處理挑戰 (Data Cleaning Challenges)

### ❌ 問題一：中位數補值型別衝突 (TypeError)
* **現象**：執行 `SimpleImputer(strategy='median')` 時報錯 `could not convert string to float: 'Male'`。
* **原因**：Heart 或 Stroke 資料集中包含類別文字（Categorical Data），中位數運算僅支援數值。
* **對策**：
    1.  實施 **資料型別分離**：數字欄位用 `Median`，類別欄位用 `Most Frequent` (眾數)。
    2.  針對 Heart 資料集，先將 `?` 替換為 `np.nan`，並執行 `pd.to_numeric(errors='coerce')` 強制轉型。

---

## 2. 模型解釋性深挖 (Interpretability Analysis)

### ❌ 問題二：Shape Function 繪圖維度不匹配
* **現象**：手動提取 `ebm_global.data()` 繪圖時，X 軸（切點）比 Y 軸（分數）多一個。
* **原因**：EBM 以區間邊界（Edges）儲存連續變數，區間數比邊界數少 1。
* **對策**：
    * 撰寫 **維度對齊邏輯**：`if len(x) == len(y) + 1: x = x[:-1]`。
    * 改用 `plt.step(where='post')` 繪製，呈現 EBM 特有的階梯函數 (Step Function)。

### 🛡️ 數據清理與品質保證 (Data Cleaning & QA Pipeline)
為了建立穩健的機器學習模型，本專案開發了整合式的自動化清洗腳本：
1. **隱性缺失揭露**：全面掃描並標準化 `Unknown`, `?`, `NAN` 等 6 種偽裝空值。
2. **自動型態校正 (Type Recovery)**：針對醫療數據中常見的型態誤判執行遍歷檢查，確保所有生理特徵回歸數值計算格式。
3. **即時熱圖驗證 (Real-time Heatmap)**：在補值程序後立即生成 **Seaborn Heatmap**，透過視覺化手段確保全資料集缺失率降為 0%。
**結果**：成功消除了 `Heart` 與 `Stroke` 資料集中的數據噪音，為後續 Random Forest 的 AUC 評估提供了高品質的基礎數據。


### 🕵️ 預測誤差之臨床歸因 (Error Attribution)
進一步針對 6 例漏診 (FN) 與 13 例誤報 (FP) 進行個案審計：


* **漏診風險分析 (FN Risk)**：發現模型在處理「低齡且生理指標正常」之患者時存在判斷盲點。這揭示了單一特徵（如年齡）在權重分配中可能過於主導，掩蓋了微小的臨床異常。
* **預處理影響證明**：部分誤判個案與原始數據中 `ca` (血管阻塞數) 的缺失值高度重合。這證明了「中位數補值」雖然能讓模型運行，但可能在邊界個案中引入人為的「健康偏誤」。
* **臨床建議**：未來部署應調降分類閾值（Threshold），以犧牲部分特異度為代價，換取更高的臨床安全性（減少漏診）。


---

## 3. 模型效能對比 (Model Benchmarking)

為了驗證 **EBM (Explainable Boosting Machine)** 的實力，針對三個醫學資料集進行了 Glassbox 模型 。

### 📊 AUC 效能對比表

| 資料集 | EBM AUC (模型 1) | Randon forset (模型 2) | Logistic Regression (模型 3) |
| :--- | :--- | :--- | :--- |
| **Heart** |0.9121 |0.8617 | 0.9008|
| **Stroke** | 0.8435| 0.7969|0.8401 |
| **Diabetes** | 0.8315| 0.7881|0.8230 |

> **註記**：EBM 通常在處理非線性醫療特徵（如年齡與 BMI 的交互作用）時表現最優。

### 🛠️ 異常排除：解決 "at least one array or dtype is required"
* **錯誤分析**：當資料集所有欄位皆被誤判為 `object` 時，`num_cols` 為空，導致 `SimpleImputer` 因無輸入數據而報錯。
* **修正對策**：
  1. 在偵測欄位型態前，預先對 `bmi` 等關鍵連續變數執行 `pd.to_numeric`。
  2. 增加 **條件判斷式 (If-statement)**，確保欄位清單長度大於 0 才執行 `fit_transform`。
* **成效**：提升了 Pipeline 的容錯率，確保不同規格的醫療資料集都能順利通過預處理。

---

## 4. 關鍵技術點總結 (Key Takeaways)

* **模型透明度 (Transparency)**：透過 `explain_global` 與 `explain_local`，醫療人員能理解 AI 判斷的依據。
* **數據稀疏性 (Data Sparsity)**：當特定類別樣本數過少時，AI 的解釋可能會失真，必須人工介入審查。
* **穩健的 Pipeline**：處理醫療髒數據（Dirty Data）時，`replace('?', np.nan)` 是不可或缺的步驟。

---

## 📚 專業術語對照 (Technical Glossary)

| 術語 | 中文解釋 |
| :--- | :--- |
| **Log-odds Score** | 對數勝算分數（EBM 的風險權重） |
| **Piecewise Constant** | 分段常數（階梯圖的數學本質） |
| **Generalization** | 泛化能力（模型對新病人的適應力） |