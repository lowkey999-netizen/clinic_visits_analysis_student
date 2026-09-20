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

## Viva & Technical Defense (Interview Q&A)

### 1. Why did you keep the ₹50,000 fees when the outlier rule flagged them?
>
> Statistical outlier rules (like 1.5×IQR) only flag values that deviate from the normal distribution—they do not mean the data is incorrect. In a hospital, ₹50,000 fees reflect genuine high-cost surgical procedures in Cardiology and Orthopedics. Dropping them would falsely understate actual clinic revenue. Instead of deleting valid data, I preserved them and tagged them with an `is_procedure` boolean flag.

### 2. Your first date parse produced 83 Sunday visits. How did you catch it, and how did you fix it?
>
> I verified the parsed dates by extracting the day of the week (`.dt.day_name()`) and checking frequency counts. Because MediCare is closed on Sundays, finding 83 Sunday visits was immediate proof of a parsing bug. The issue arose because `pd.to_datetime(..., format="mixed", dayfirst=True)` misinterpreted ISO dates like `2025-11-12` as December 11 instead of November 12. I fixed this by splitting the column into three regex masks (`/`, alpha, and ISO) and parsing each format explicitly.

### 3. Why per-department median for missing fees instead of the overall median or the mean?
>
> Different hospital specialties have fundamentally different pricing tiers—Cardiology's typical visit (₹1,260) costs nearly 2.5× General Medicine (₹520). A clinic-wide median would systematically undercharge Cardiology and overcharge General Medicine. Furthermore, we use the median rather than the mean because the ₹50,000 procedure outliers heavily skew the average upward.

### 4. Cardiology's mean fee is ₹2,021 and its median is ₹1,260. Which one goes in the annual report?
>
> The **median (₹1,260)** must be published in the annual report. The mean (₹2,021) is artificially inflated by a tiny fraction of surgical procedures, giving prospective patients a misleading impression of what a typical visit costs. If the administrator wants to report both, they should separate "Routine Consultations (Median: ₹1,260)" from "Specialized Procedures".

### 5. What is one decision in your log you would change if you had more data?
>
> Rejecting the two impossible age records (412 and 199). Because I had no external patient registry, dropping those rows was the only responsible option to avoid corrupting the age-correlation regression. If I had access to a Master Patient Index linking `patient_id` with birth dates or government IDs, I would recover the true ages instead of discarding patient data.

---

## Repository Structure

```
clinic_visits_analysis/
├── clinic_visits_2025.csv              # Raw visit records (~2,400 rows)
├── clinic_visits_clean.csv             # Cleaned production dataset (2,398 rows)
├── clinic_visits_analysis_student.ipynb # End-to-end analysis notebook
├── data_quality_log.xlsx               # 7-point data quality & governance log
├── requirements.txt                    # Project dependencies
└── README.md                           # Project showcase & interview defense
```

### Reproducibility

To run this analysis locally:

```bash
pip install -r requirements.txt
jupyter notebook clinic_visits_analysis_student.ipynb
```
