# MediCare Outpatient Visits Analysis (2025)

**End-to-End Healthcare Data Analytics & Operational Strategy**

**Author:** Bhargava Shanmukha ([@lowkey999-netizen](https://github.com/lowkey999-netizen))  
**Course:** Python for Full Stack Data Science with AI & Generative AI · Naresh IT  
**Lead Trainer:** Ajit Byru  

---

## Executive Summary

As the data analyst for **MediCare Outpatient Clinic**, I analyzed **2,423 patient visit records from 2025** to help the clinic administrator optimize operations, staffing, and billing transparency for 2026.

By cleaning and profiling the raw data, conducting targeted exploratory data analysis, and examining consultation efficiency, this study identified two major operational bottlenecks:

1. **Severe Front-of-Week Congestion:** Monday is the busiest day clinic-wide (495 visits), with General Medicine carrying nearly a third of all clinic volume (792 visits) and generating an alarming 90th percentile wait time of **49 minutes**.
2. **Revenue Skew from Surgical Procedures:** Routine consultation revenues were virtually stagnant across 2025 (~₹1.4L–₹1.8L/month). Apparent revenue spikes were driven entirely by a handful of high-cost ₹50,000 procedures in Cardiology and Orthopedics, demonstrating that median fees (not means) must be used for public reporting.

---

## Data-Quality Log (7 Cleaning Decisions)

The raw export (`clinic_visits_2025.csv`, 2,423 rows × 13 columns) contained significant real-world data hygiene issues. All fixes followed strict analytical governance and are documented in [`data_quality_log.xlsx`](./data_quality_log.xlsx):

| # | Problem Identified | Rows Affected | Fix Applied | Defense & Rationale |
| --- | --- | :---: | --- | --- |
| **1** | **Exact duplicate records** | 23 | `df.drop_duplicates()` | Duplicate rows falsely inflate visit volume and revenue. |
| **2** | **Mixed date formats** (`/`, `-`, text months) | 2,423 | Regex pattern masks + 3 explicit `pd.to_datetime` parsers | The naive `dayfirst=True` parser erroneously misread ISO dates, creating 83 phantom Sunday visits (clinic is closed Sundays). Explicit masking resolved 100% of dates with zero Sundays. |
| **3** | **Department name inconsistencies** | 27 | `.str.strip().str.title().replace('Ent', 'ENT')` | Inconsistent casing and whitespace created 17 apparent departments instead of the actual 6, which would break all downstream aggregations. |
| **4** | **Impossible patient ages** (412, 199) | 2 | Dropped rows (`df['age'] <= 110`) | Extreme values with no verifiable patient history to repair them. Retaining them would heavily bias age correlation statistics. |
| **5** | **Negative wait times** (-37, -33, etc.) | 5 | Repaired with `df['wait_minutes'].abs()` | The absolute values (21–37 min) fall right within the normal 25th–75th percentile wait distribution, pointing to a clerical minus-sign entry error rather than unrecoverable noise. |
| **6** | **Fee outliers** (₹50,000 visits) | 13 | Kept intact; tagged with `is_procedure` | Flagged by the 1.5×IQR rule, but verified as legitimate surgical/cardiac procedures rather than billing glitches. |
| **7** | **Missing consultation fees** | 120 | Imputed with per-department median | Overall median would over-impute inexpensive departments (General Medicine) and under-impute expensive ones (Cardiology). Median avoids distortion from procedure fees. |

* **Final Clean Dataset:** **2,398 rows × 17 columns** (saved as [`clinic_visits_clean.csv`](./clinic_visits_clean.csv) with zero missing values).

---

## Key Findings & Business Insights

### 1. Peak Demand & Staffing (Q1)

* **General Medicine** is by far the highest-volume department (792 visits, ~33% of clinic load), followed by Cardiology (384) and Orthopedics (375).
* **Monday is the peak weekday** (495 visits) before tapering toward Saturday (302 visits).
* *Takeaway:* Front-desk triage and general consultation coverage must be weighted toward Monday mornings.

### 2. Wait Time Disparities (Q2)

* General Medicine patients suffer both the highest median wait (**38 minutes**) and the worst 90th percentile wait (**49 minutes**).
* In contrast, specialized departments like Dermatology manage far tighter queues (median 18 min, p90 29 min).

### 3. Reporting Integrity: Mean vs. Median Fees (Q3)

* In Cardiology, the mean fee is **₹2,019.27**, but the median fee is only **₹1,260.00**.
* In Orthopedics, the mean fee is **₹1,972.75**, while the median is **₹1,060.00**.
* *Takeaway:* Thirteen ₹50,000 surgical procedures artificially inflate the average fee by ₹750–₹900. The **median** fee must be published in the annual report to accurately represent typical outpatient costs.

### 4. Patient Age vs. Consultation Duration (Q4)

* A strong positive linear correlation exists between patient age and consultation length (**$r = 0.77$**).
* Regression slope indicates consultation length increases by **~2.1 minutes for every decade of age** ($0.21$ min/year), starting from a baseline intercept of ~8.6 minutes.

### 5. Revenue Trend: Organic Growth vs. One-Off Spikes (Q5)

* Routine consultation revenue was stable throughout 2025 (between ₹1.4L and ₹1.8L per month).
* Pronounced revenue spikes in January (₹3.62L) and September (₹2.67L) were entirely caused by clustered surgical procedures, not clinic growth.

### 6. Own Investigation: Consultation Efficiency (Section 4)

* Calculated the **Wait-to-Consult Ratio** ($\text{Wait Minutes} / \text{Consult Minutes}$) across all departments.
* **General Medicine is the least efficient department**, forcing patients to wait an average of **2.3 minutes in the waiting room for every 1 minute spent with the physician**.
* *Limitation:* To confirm if this is operational inefficiency or clinical necessity, triage records are needed to distinguish walk-in emergencies from scheduled appointments.

---

## Strategic Recommendation for 2026

> **To the Clinic Administrator:**  
> Our analysis identifies **General Medicine** as MediCare's primary operational bottleneck. Patients wait an average of 38 minutes—with 10% waiting 49 minutes or longer—and spend 2.3 minutes waiting for every single minute of consultation. Compounding this, clinic visits surge to their weekly peak on **Mondays (495 visits)**.  
>
> For 2026, I recommend deploying a dedicated triage nurse and opening a second General Medicine consultation room specifically on **Monday mornings (8:00 AM – 1:00 PM)** to absorb the front-of-week volume spike without increasing overhead on quieter days like Thursday and Saturday.  
>
> Before making permanent staffing investments, I recommend tracking hourly check-in timestamps to map the exact intra-day queue formation.

---

## Analytical Methodology & Edge-Case Decisions

### Why keep ₹50,000 procedure fees despite the 1.5×IQR outlier flag?

Statistical outlier rules (like the 1.5×IQR fence) flag distributional skewness, not data corruption. In a multi-specialty outpatient hospital, ₹50,000 charges correspond to valid surgical and cardiac procedures performed in Cardiology and Orthopedics. Discarding these rows would artificially deflate total clinic revenue and misrepresent clinical throughput. Instead of dropping legitimate records, we preserved them in the dataset and established an explicit boolean indicator (`is_procedure = fee >= 50000`) to separate routine visit economics from procedure-based billing.

### Root cause and remediation for the Sunday parsing anomaly

Automated date parsers configured with `dayfirst=True` failed silently on standard ISO strings (`YYYY-MM-DD`), misinterpreting dates such as `2025-11-12` as December 11 instead of November 12. Because MediCare is strictly closed on Sundays, extracting day-of-week frequencies revealed 83 phantom Sunday visits—providing definitive proof of silent parsing distortion. The solution was to partition the column into three discrete regular-expression masks corresponding to each format pattern (`DD/MM/YYYY`, `DD-Mon-YYYY`, and `YYYY-MM-DD`) and parse each slice with strict, explicit format strings, achieving zero unparsed records and exactly zero Sunday visits.

### Rationale for specialty-level median fee imputation

Consultation fees are governed by medical specialty rather than hospital-wide averages. A routine General Medicine visit has a median fee of ₹520, whereas specialized Cardiology consultations command a median of ₹1,260. Imputing with a clinic-wide global metric would systematically over-impute inexpensive general consultations and under-impute specialized visits. Furthermore, because specialized departments include high-value surgical procedures, department means are distorted (Cardiology mean: ₹2,019 vs. median: ₹1,260). Using the group-level median (`transform("median")`) ensures imputed values reflect the central tendency of the appropriate peer group without contamination from extreme outliers.

### Metric selection for clinic reporting: Mean vs. Median

When reporting typical consultation costs for the annual report, the median fee should be published rather than the arithmetic mean. In departments with procedure volume (Cardiology and Orthopedics), the arithmetic mean is upwardly skewed by ₹750 to ₹900 due to extreme ₹50,000 charges. Presenting the mean fee of ₹2,019 in Cardiology would misrepresent typical patient out-of-pocket costs. The median of ₹1,260 reflects the true 50th percentile baseline that incoming patients can expect.

### Limitations and future data enrichment

The data cleaning strategy rejected two records with impossible patient ages (412 and 199) because age was an essential regressor for consultation modeling and no secondary demographic fields existed to infer the true values. While dropping was the only methodologically defensible choice within the isolated dataset, access to a Master Patient Index linking `patient_id` to external health records or national identity databases would allow deterministic recovery of corrupted timestamps and patient ages, eliminating data loss altogether.

---

## Repository Structure

``` text

clinic_visits_analysis/
├── clinic_visits_2025.csv              # Raw visit records (~2,400 rows)
├── clinic_visits_clean.csv             # Cleaned production dataset (2,398 rows)
├── clinic_visits_analysis_student.ipynb # End-to-end analysis notebook
├── data_quality_log.xlsx               # 7-point data quality & governance log
├── requirements.txt                    # Project dependencies
└── README.md                           # Project showcase & methodology
```

### Reproducibility

To run this analysis locally:

```bash
pip install -r requirements.txt
jupyter notebook clinic_visits_analysis_student.ipynb
```
