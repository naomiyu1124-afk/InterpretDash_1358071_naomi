# 醫學資訊與可解釋人工智慧 實驗分析報告

## Q1：EBM AUC 效能差異分析 (Performance Comparison)

**實驗結果摘要：**
* **Heart AUC:** 0.9121
* **Stroke AUC:** 0.8395
* **Diabetes AUC:** 0.8315 (最低)

**原因分析 (Reasoning):**
我認為影響 **Diabetes (糖尿病)** 表現最顯著的因素是 **Data Quality & Feature Relevance (資料品質與特徵相關性)**。

* **Predictive Power (預測強度):** 心臟病資料集（Heart）包含如 `ST segment` 或 `Thalassemia` 等具備極高臨床診斷價值的特徵。相比之下，糖尿病的特徵如 `BMI` 或 `Blood Pressure` 在健康人群中也極為常見，這類 **Overlapping Features** 增加了模型區分的難度。
* **Class Imbalance (正例比例):** 若資料集中糖尿病正例比例過低，EBM 在捕捉特定族群的非線性規律時會受到限制，導致整體 AUC 略低於其他資料集。

---

## Q2：GAM Changer 中的 `asthma` Shape Function 描述


---

## Q3：黑箱模型能發現氣喘悖論嗎？ (Can Black-box Models Detect the Paradox?)

**結論：它能「學習」到規律，但無法主動「揭露」問題。**

**深度解析 (Analysis):**

1.  **Implicit Learning (隱性學習):**
    Black-box models (如神經網路或隨機森林) 的確能「發現」統計上的相關性。在 Caruana 的論文中，黑箱模型的 AUC 往往很高，正是因為它們精準地捕捉到了「氣喘 = 低風險」這個統計模式。

2.  **Lack of Transparency (缺乏透明度):**
    雖然模型學到了這個規律，但它將其隱藏在數百萬個參數中，不會以圖形或公式的形式告知醫療人員。人類無法主動看出模型到底是根據「正確的病理學」還是「錯誤的統計偏誤」進行判斷。

3.  **The Danger of Trust (信任的危險):**
    因為黑箱模型無法揭露 (Exposure) 內部邏輯，人類專家無法在模型部署前攔截這個錯誤。若直接應用於臨床，將導致高風險的氣喘患者被誤判為低風險，這正是 **Glass-box models (如 EBM)** 存在的必要性。

> **Summary:** Black-box models **adopt** the paradox to maximize accuracy, while EBM **exposes** the paradox for human intervention.
