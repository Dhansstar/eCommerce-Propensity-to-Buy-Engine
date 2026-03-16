# 🛒 eCommerce Propensity-to-Buy Engine
**End-to-End Predictive Analytics | Google Analytics & BigQuery Cloud Infrastructure**

<div align="center">

[![Python](https://img.shields.io/badge/Python-3.12+-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Google BigQuery](https://img.shields.io/badge/BigQuery-669DF2?style=for-the-badge&logo=google-cloud&logoColor=white)](https://cloud.google.com/bigquery)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white)](https://scikit-learn.org/)
[![Imbalanced-Learn](https://img.shields.io/badge/SMOTE--NC-Handling_Imbalance-orange?style=for-the-badge)]()

</div>

## 📌 Business Context
Dalam ekosistem e-commerce modern, biaya akuisisi pelanggan (*Customer Acquisition Cost*) sangatlah mahal. Tantangannya adalah **98.45% pengunjung** tidak langsung membeli pada kunjungan pertama. Proyek ini membangun sistem prediksi perilaku untuk mengidentifikasi "Pembeli Potensial" sehingga tim marketing dapat memberikan penawaran produk yang presisi, mengoptimalkan budget iklan, dan meningkatkan tingkat konversi.

## 🎯 Technical Competencies
* **Cloud Data Acquisition:** Eksplorasi dan ekstraksi dataset `web_analytics` langsung dari Google BigQuery.
* **Integrated Pipeline:** Mengimplementasikan transformasi data otomatis (Preprocessing hingga Modeling) menggunakan Scikit-Learn Pipeline.
* **Advanced Classification:** Mengembangkan model Logistic Regression dan Decision Tree untuk memprediksi `will_buy_on_return_visit`.
* **Imbalanced Treatment:** Menangani ketimpangan data ekstrem menggunakan **SMOTE-NC**.
* **Model Optimization:** Melakukan Hyperparameter Tuning untuk mendapatkan *decision boundary* yang paling akurat.

---

---

## 🔬 Exploratory Data Analysis (EDA) Highlights
Analisis mendalam terhadap perilaku user memberikan beberapa temuan kunci:

* **📈 Correlation Insights:** Menggunakan **Korelasi Spearman**, ditemukan bahwa `pageviews` (**+0.14**) dan `time_on_site` (**+0.12**) merupakan prediktor positif terkuat. Semakin dalam eksplorasi user, semakin tinggi niat belinya.
* **🔄 The Bounce Paradox:** User yang melakukan interaksi aktif (`bounces = 0`) memiliki peluang konversi **10x lipat lebih besar** dibandingkan user pasif yang langsung *close* tab.
* **🇨🇦 Regional Deep-Dive:** Di wilayah **Canada**, konversi didominasi mutlak oleh pengguna **Desktop** (189 user) dibandingkan Mobile (11) atau Tablet (1).

---

## ⚙️ Engineering & Modeling Pipeline
Standardisasi industri untuk memastikan model bersifat *scalable* dan *reproducible*:

1.  **🛠️ Preprocessing:** Integrasi `SimpleImputer` untuk data kosong, `Winsorizer` untuk menangani outlier durasi kunjungan, dan `MinMaxScaler` untuk normalisasi fitur.
2.  **🎯 Feature Selection:** Memilih fitur yang memberikan dampak statistik signifikan guna menghindari model yang berat dan *overfit*.
3.  **⚖️ Balancing:** Mengatasi *class imbalance* (1.5% pembeli vs 98.5% non-pembeli) menggunakan teknik **SMOTE** untuk memperkuat deteksi pada kelas minoritas.
4.  **⚡ Tuning:** Mengoptimasi model terbaik melalui pencarian *hyperparameter* untuk mencapai performa maksimal.

---

## 📊 Performance Matrix & Evolution
Evolusi model menunjukkan peningkatan kualitas prediksi seiring dengan pengembangan fitur:

| Stage | Data Source | Best Model | ROC-AUC | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Phase 1** | Dataset Baseline | Decision Tree | `0.7779` | *Decent* |
| **Phase 2** | Dataset Enhanced | Decision Tree | `0.7796` | *Decent* |
| **Phase 3** | **Dataset Enhanced** | **DT (Tuned)** | **`0.8240`** | **Fairly Good** |

> **PROS:** Model mampu menangkap hingga **78%** pembeli potensial (*Recall*).  
> **CONS:** Masih terdapat *False Positive* akibat perilaku browsing user yang sangat mirip di awal sesi.

---

## 🚀 Model Inference: Live Simulation
Pengujian *real-time* dilakukan pada profil user baru untuk memvalidasi performa prediksi:

| User Profile | Key Highlight | Prediction | Confidence Score |
| :--- | :--- | :--- | :--- |
| **User 1** | Canada, Desktop, High Interaction | **✅ AKAN MEMBELI** | **87%** |
| **User 2** | Unknown Location, Low Duration | **❌ TIDAK MEMBELI** | **26%** |

**💡 Summary:** Model berhasil membedakan User 1 sebagai **Pembeli Potensial** dengan tingkat keyakinan tinggi. Tim marketing dapat menggunakan data ini untuk memprioritaskan budget promosi pada User 1 guna memaksimalkan efisiensi biaya.

---

## 🛠️ Data Infrastructure & SQL Extraction
Data diekstraksi melalui dua skema dataset untuk menganalisis efek penambahan fitur terhadap performa model:

### **Dataset 1: Baseline Behavioral**
Fokus pada interaksi fundamental: `bounces` dan `time_on_site`.
### **Dataset 2: Advanced User Profile**
Menambahkan dimensi perilaku dan geografis untuk meningkatkan presisi:
- **eCommerce Progress:** Melacak tahapan aksi user di situs.
- **Engagement Metrics:** `pageviews` dan sumber trafik (`source`, `medium`, `channelGrouping`).
- **Demographics:** Geografi (`country`) dan tipe perangkat (`deviceCategory`).

```sql
-- Advanced Extraction Query Snippet
WITH all_visitor_stats AS (
   SELECT fullvisitorid, MAX(IF(totals.transactions >= 1, 1, 0)) AS will_buy_on_return_visit
   FROM `data-to-insights.ecommerce.web_analytics`
   GROUP BY fullvisitorid
)
SELECT * EXCEPT(unique_session_id) FROM (
   SELECT 
      CONCAT(fullvisitorid, CAST(visitId AS STRING)) AS unique_session_id,
      MAX(CAST(h.eCommerceAction.action_type AS INT64)) AS latest_ecommerce_progress,
      IFNULL(totals.bounces, 0) AS bounces,
      IFNULL(totals.timeOnSite, 0) AS time_on_site,
      totals.pageviews,
      trafficSource.source,
      device.deviceCategory,
      IFNULL(geoNetwork.country, "Unknown") AS country
   FROM `data-to-insights.ecommerce.web_analytics`, UNNEST(hits) AS h
   JOIN all_visitor_stats USING(fullvisitorid)
   WHERE totals.newVisits = 1 AND date BETWEEN '20160801' AND '20170531'
   GROUP BY 1, 2, 3, 4, 5, 6, 7, 8)

---

## 📁 Repository Structure
```text
├── analysis.ipynb                        # Main Analysis & Modeling Pipeline
├── analysis_inf.ipynb                    # Live Inference Simulation
├── dataset_1.csv                         # Raw Data (Baseline)
├── dataset_2.csv                         # Raw Data (Advanced)
├── final_ecommerce_model_tuned.pkl       # Optimized Deployment Model
└── README.md                             # Full Documentation