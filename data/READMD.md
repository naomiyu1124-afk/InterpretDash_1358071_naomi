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

### 🔍 重大發現：Stroke 資料集的 $N=1$ 陷阱
* **觀察**：`gender` 特徵在 `Other` 類別出現劇烈負分 (約 -0.42)。
* **診斷**：透過 `value_counts()` 發現 `Other` 樣本數僅有 **1 筆**。
* **結論**：此為 **小樣本偏差 (Small Sample Bias)** 導致的 **過度擬合 (Overfitting)**。在醫療場景中，此分數不具臨床代表性，應視為 **統計假象 (Statistical Artifact)**。

---

## 3. 模型效能對比 (Model Benchmarking)

為了驗證 **EBM (Explainable Boosting Machine)** 的實力，針對三個醫學資料集進行了 Glassbox 模型 PK。

### 📊 AUC 效能對比表

| 資料集 | EBM AUC (模型 1) | Decision Tree (模型 2) | Logistic Regression (模型 3) |
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