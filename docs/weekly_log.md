> **Q1：三個資料集的 EBM AUC 差異，你認為原因是什麼？請引用你實驗中的具體數字來回答。**
> Heart AUC = 0.9121，Stroke AUC = 0.8395，Diabetes AUC = 0.8315。其中 Diabetes 最低。
**原因分析 (Reasoning):**
在三個資料集中，**Diabetes (糖尿病)** 的 AUC 最低。我認為影響最大的因素是 **Data Quality & Feature Relevance (資料品質與特徵相關性)**。

* **Positive Class Ratio (正例比例):** 糖尿病資料集的類別分布通常較不平均，導致模型在捕捉少數類別的特徵時較為困難。
* **Predictive Power:** 相較於 Heart dataset 擁有如心電圖 (ST segment) 等強效預測指標，Diabetes 的特徵（如 BMI、血壓）與目標變數之間的線性或非線性關係較為模糊，存在更多雜訊 (Noise)，因此 EBM 較難達到極高的準確率。> 

> **Q2：在 GAM Changer 裡，`asthma` 的 shape function 長什麼樣？（asthma = 1 時，y 軸方向是什麼？）**
> 

> **Q3：黑箱模型能發現 Caruana 論文中的氣喘悖論嗎？為什麼？**
答案是：它能「學到」規律，但無法主動「揭露」問題。

學習能力 (Learning): 黑箱模型非常擅長捕捉資料中的規律。它會準確地捕捉到「氣喘患者死亡率較低」這個統計事實，並反映在預測結果中。

揭露能力 (Exposure): 黑箱模型缺乏透明度。它將這個悖論隱藏在數百萬個參數中。人類無法從模型結構中看出它到底學到了什麼，因此無法在部署前攔截這個錯誤的醫學邏輯。
>
