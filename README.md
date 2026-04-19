# DSA3050A - Advanced Power BI End-Semester Examination
### Healthcare Analytics: Medical Appointment No-Show Intelligence System

---

> **Student:** Hermela Seltanu Gizaw (670446)
> 
> **Instructor:** Prof. Austin Owur Odera
> 
> **Semester:** Spring 2026
> 
> **Institution:** United States International University - Africa

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Dataset Description](#3-dataset-description)
4. [Tools Used](#4-tools-used)
5. [Project Architecture - Snowflake Schema](#5-project-architecture--snowflake-schema)
6. [Steps Followed](#6-steps-followed)
   - [Part A: Data Acquisition](#part-a-data-acquisition-and-understanding)
   - [Part B: Data Cleaning & Power Query](#part-b-data-cleaning-and-transformation)
   - [Part C: Data Modeling](#part-c-data-modeling)
   - [Part D: DAX Measures & Calculated Columns](#part-d-dax-measures-and-calculated-columns)
   - [Part E: Dashboard Design](#part-e-dashboard-design-and-visualization)
   - [Part F: Insights & Recommendations](#part-f-analytical-insights-and-business-recommendations)
   - [Part G: GitHub Repository](#part-g-github-repository)
7. [Dashboard Features](#7-dashboard-features)
8. [Key DAX Measures](#8-key-dax-measures)
9. [Key Insights](#9-key-insights)
10. [Challenges Encountered](#10-challenges-encountered)
11. [Conclusion](#11-conclusion)
12. [Repository Structure](#12-repository-structure)
13. [Live Links](#13-live-links)

---

## 1. Project Overview

This project is a complete end-to-end Business Intelligence solution built in Microsoft Power BI as part of the DSA 3050A Advanced Practical Examination. The solution was developed in the **Healthcare Analytics** domain, using a real-world dataset of over 110,000 medical appointment records from Brazilian public health clinics.

The system goes beyond descriptive reporting. It integrates external socio-economic data (Brazilian Cities GDP and HDI) into the data model through a **Snowflake Schema**, enabling cross-dimensional analysis that links patient behaviour to regional economic conditions. The result is an interactive, stakeholder-ready dashboard that allows clinic managers, healthcare administrators, and policymakers to diagnose the root causes of patient non-attendance - and act on them.

**Key metrics delivered by this system:**

| Metric | Value |
|---|---|
| Total Appointments Analysed | 110,527 |
| Dataset Variables | 14 |
| Schema Type | Snowflake (5 tables) |
| Dashboard Pages | 3 |
| DAX Measures Created | 8+ |
| Power Query Transformation Steps | 8+ |
| Neighbourhoods Covered | 81 |

---

## 2. Problem Statement

> *"In healthcare systems across Brazil and the developing world, patient non-attendance (No-Show) creates operational inefficiencies, underutilisation of expensive medical equipment, and increased wait times for other patients. This project analyses 110,527 medical appointments to identify the key demographic, behavioural, and socio-economic drivers of no-shows. By understanding these patterns, healthcare providers can implement targeted interventions - such as optimised SMS reminders, demographic-specific engagement strategies, and mobile clinic deployments in low-GDP regions - to reduce the no-show rate and improve the equity of healthcare access."*

### Business Questions

This project was designed to answer the following core analytical questions:

1. What is the overall no-show rate, and how does it vary across the 81 neighbourhoods?
2. Does booking lead time (the gap between scheduling and appointment date) predict whether a patient will attend?
3. Is the current SMS reminder system effective, and is its impact diminishing over time?
4. Do socio-economic indicators (GDP per capita, HDI) correlate with regional no-show rates?
5. Which patient demographics - gender, age group, scholarship status - are the highest-risk segments?

---

## 3. Dataset Description

| Attribute | Detail |
|---|---|
| **Source** | [Kaggle - Medical Appointment No Shows](https://www.kaggle.com/datasets/joniarroba/noshowappointments) |
| **Original File** | `KaggleV2-May-2016.csv` |
| **Total Records** | 110,527 medical appointments |
| **Time Period** | April – June 2016 |
| **Location** | Vitória, Espírito Santo, Brazil |
| **Variables** | 14 (8 categorical, 6 numerical, 1 date) |
| **Target Variable** | `No-show` - whether the patient attended (Attended / Missed) |
| **External Dataset** | IBGE Brazilian Cities Dataset (GDP per Capita, HDI, Cars per capita) |

### Core Variables

| Variable | Type | Description |
|---|---|---|
| `AppointmentID` | Text (ID) | Unique identifier for each scheduled visit |
| `PatientId` | Text (ID) | Unique identifier per patient |
| `ScheduledDay` | DateTime | Date and time the appointment was booked |
| `AppointmentDay` | Date | Date of the actual medical appointment |
| `Age` | Integer | Patient age in years |
| `Gender` | Categorical | Male (M) or Female (F) |
| `Neighbourhood` | Categorical | Clinic location (81 distinct areas) |
| `Scholarship` | Binary (0/1) | Enrolled in Bolsa Família social welfare program |
| `Hipertension` | Binary (0/1) | Patient has hypertension diagnosis |
| `Diabetes` | Binary (0/1) | Patient has diabetes diagnosis |
| `Alcoholism` | Binary (0/1) | Patient has alcoholism on record |
| `Handcap` | Integer (0–4) | Degree of registered disability |
| `SMS_received` | Binary (0/1) | Whether a reminder SMS was sent |
| `No-show` | Categorical | **Target variable** - "Yes" = missed, "No" = attended |

---

## 4. Tools Used

| Tool | Version / Details | Purpose |
|---|---|---|
| **Microsoft Power BI Desktop** | Latest (2026) | Primary BI development environment |
| **Power Query (M Language)** | Built into Power BI | Data cleaning & ETL transformation |
| **DAX (Data Analysis Expressions)** | Built into Power BI | Measures, calculated columns, time intelligence |
| **Power BI Service** | Cloud (app.powerbi.com) | Publishing and sharing the live dashboard |
| **Microsoft Excel** | - | Initial data inspection and external dataset preparation |
| **GitHub** | - | Version control and project documentation |
| **Kaggle** | - | Primary dataset source |
| **IBGE Open Data** | Brazilian Government | External socio-economic dimension data |

---

## 5. Project Architecture - Snowflake Schema

This project implements a **Snowflake Schema** rather than a simple Star Schema. The key distinction is that the location dimension is extended with an external economic data table, enabling macro-level analysis linking neighbourhood-level clinic data to city-level prosperity indicators.

```
                        ┌─────────────────────┐
                        │    Dim_Date          │
                        │  (Calendar Table)    │
                        │  PK: Date            │
                        └──────────┬──────────┘
                                   │ 1:*
                        ┌──────────▼──────────┐
   ┌──────────────┐     │                     │     ┌────────────────────┐
   │  Dim_Patient │ 1:* │  Fact_Appointments  │ *:1 │   Dim_Location     │
   │  PK:PatientId│◄────│  FK: PatientId      │────►│  PK: Neighbourhood │
   │  Gender      │     │  FK: AppointmentDate│     │  City              │
   │  Age/AgeGroup│     │  FK: Neighbourhood  │     └─────────┬──────────┘
   │  Scholarship │     │  WaitDays           │               │ *:1
   │  Health Flags│     │  SMS_received       │     ┌─────────▼──────────┐
   └──────────────┘     │  No-show (Status)   │     │  Dim_SocioEconomic │
                        └─────────────────────┘     │  PK: City          │
                                                     │  GDP_CAPITA        │
                                                     │  IDHM (HDI)        │
                                                     │  Cars              │
                                                     └────────────────────┘
```

**Relationship types:** All relationships are One-to-Many (1:*) with Single Cross-Filter Direction flowing from dimension tables into the fact table. This ensures correct filter context propagation for all DAX measures.

---

## 6. Steps Followed

### Part A: Data Acquisition and Understanding

1. Downloaded the `KaggleV2-May-2016.csv` dataset from Kaggle.
2. Conducted an initial data profiling review: 110,527 rows, 14 columns, no header issues.
3. Identified the business domain: **Healthcare Analytics** - specifically patient no-show behaviour.
4. Documented key variables, their types, and their analytical roles.
5. Sourced the IBGE Brazilian Cities dataset to enable socio-economic enrichment.
6. Assessed dataset suitability: sufficient scale (>9,000 rows), multiple related dimensions, date fields for time intelligence, and a binary target variable suitable for rate analysis.

---

### Part B: Data Cleaning and Transformation

All transformations were performed in **Power Query (M Language)** inside Power BI Desktop. A minimum of 8 meaningful transformation steps were applied and documented.

#### Transformation Summary

| Step | Column(s) | Issue | Action Taken | Justification |
|---|---|---|---|---|
| **1** | `PatientId`, `AppointmentID` | Imported as numeric - caused scientific notation | Converted to **Text** type | Prevents aggregation of identifier fields; preserves ID integrity |
| **2** | `ScheduledDay` | DateTime format imported incorrectly | Changed to **Date/Time** type | Required for WaitDays calculated column |
| **3** | `AppointmentDay` | Date format inconsistent | Changed to **Date** type | Needed for relationship with Dim_Date |
| **4** | `Age` | Contained anomalous negative values (age = –1) | Filtered rows where Age ≥ 0 | Removes 1 invalid record; ensures demographic analysis is sound |
| **5** | `No-show` column | Values: "Yes" / "No" - counterintuitive | Replaced: "Yes" → **"Missed"**, "No" → **"Attended"** | Produces executive-ready, self-explanatory labels on all charts |
| **6** | `WaitDays` | Did not exist in raw data | Created **Custom Column**: `= Duration.Days([AppointmentDay] - [ScheduledDay])` | Core metric for lead-time analysis |
| **7** | `WaitTimeCategory` | Did not exist in raw data | Created **Conditional Column**: Same Day (0 days), Short (1–7 days), Long (>7 days) | Enables categorical behavioral segmentation |
| **8** | `Dim_Patient` table | Patient data embedded in flat file | Used **Reference** + **Remove Duplicates** on PatientId to build a separate patient dimension | Normalises the schema; creates a valid 1:* relationship |
| **9** | Date parts | No Year/Month/Day columns existed | Extracted Year, Month Number, Month Name, Day Name from AppointmentDay | Enables time intelligence and chronological sorting |
| **10** | `Dim_Location` table | Neighbourhood embedded in flat file | Isolated Neighbourhood + City into a new reference table | Bridge table for snowflake extension to socio-economic data |

---

### Part C: Data Modeling

The model was built as a **Snowflake Schema** with 5 tables: 1 Fact table and 4 Dimension tables.

#### Table Roles

| Table | Type | Primary Key | Role |
|---|---|---|---|
| `Fact_Appointments` | Fact | AppointmentID | Central hub - stores all metrics and foreign keys |
| `Dim_Patient` | Dimension | PatientId | Demographic filtering: gender, age, scholarship, health flags |
| `Dim_Date` | Dimension | Date | Time intelligence - YTD, MTD, prior period comparisons |
| `Dim_Location` | Dimension | Neighbourhood | Bridge table linking clinics to economic data |
| `Dim_SocioEconomic` | Dimension (Snowflake) | City | External GDP and HDI data for macro-level analysis |

#### Relationships

| From | To | Cardinality | Filter Direction |
|---|---|---|---|
| `Dim_Date[Date]` | `Fact_Appointments[AppointmentDate]` | 1:* | Single (Dim → Fact) |
| `Dim_Patient[PatientId]` | `Fact_Appointments[PatientId]` | 1:* | Single (Dim → Fact) |
| `Dim_Location[Neighbourhood]` | `Fact_Appointments[Neighbourhood]` | 1:* | Single (Dim → Fact) |
| `Dim_SocioEconomic[City]` | `Dim_Location[City]` | 1:* | Single (Dim → Dim) |

The **Dim_Date** table was created via a DAX `CALENDAR()` expression to ensure a continuous, unbroken date range - a requirement for reliable time intelligence functions such as `TOTALMTD`, `DATEADD`, and `SAMEPERIODLASTYEAR`.

---

### Part D: DAX Measures and Calculated Columns

#### Calculated Columns (created in Fact_Appointments)

| Column | DAX Formula | Purpose |
|---|---|---|
| `WaitDays` | `DATEDIFF(Fact_Appointments[ScheduledDay], Fact_Appointments[AppointmentDay], DAY)` | Numeric lead time between booking and visit |
| `WaitTimeCategory` | `IF([WaitDays] = 0, "Same Day", IF([WaitDays] <= 7, "Short", "Long"))` | Categorical segmentation for behavioral analysis |

#### DAX Measures

| # | Measure Name | DAX Pattern Used | Business Purpose |
|---|---|---|---|
| 1 | `Total Appointments` | `COUNT()` | Capacity baseline - total demand for healthcare services |
| 2 | `No-Show Rate %` | `DIVIDE(CALCULATE(COUNT, No-show="Missed"), [Total Appointments])` | Primary KPI - fair comparison across clinics of different sizes |
| 3 | `Average Wait Days` | `AVERAGE()` | Lead-time indicator - identifies if long queues drive no-shows |
| 4 | `MTD Appointments` | `TOTALMTD()` | Real-time monthly tracking vs. capacity targets |
| 5 | `PM No-Show Rate` | `CALCULATE() + DATEADD()` | Prior-month baseline for trend and intervention analysis |
| 6 | `Neighbourhood Rank` | `RANKX()` | Resource allocation - identifies the worst-performing clinic areas |
| 7 | `High GDP No-Show Rate` | `CALCULATE() + FILTER()` on GDP_CAPITA | Socio-economic context - tests if wealth predicts attendance |
| 8 | `SMS Impact Variance` | Subtraction of filtered rates | ROI calculation - proves or disproves the SMS program's effectiveness |

---

### Part E: Dashboard Design and Visualization

The dashboard consists of **3 interactive report pages**, built with a consistent blue-teal professional theme.

#### Page 1 - Executive Summary
Designed for senior management and clinic directors who need an instant situational overview.

- **KPI Cards:** Total Appointments, No-Show Rate %, Average Wait Days
- **Trend Line Chart:** Monthly no-show rate over time
- **Treemap:** No-show volume by Neighbourhood (reveals geographic concentration at a glance)
- **Gauge Visual:** Current no-show rate vs. target threshold
- **Slicers:** Gender, Scholarship Status, WaitTimeCategory

#### Page 2 - Detailed Analysis
Designed for data analysts and operations managers who need to investigate root causes.

- **Decomposition Tree:** AI-powered breakdown of no-show drivers (e.g., Long Wait → Male → No SMS)
- **Key Influencers Visual:** Machine-learning visual identifying the top factors predicting "Missed" vs. "Attended"
- **Matrix Table:** Neighbourhood × WaitTimeCategory cross-tab with drill-down
- **Scatter Plot:** GDP per Capita vs. No-Show Rate % by neighbourhood (with tooltip showing HDI)

#### Page 3 - Insights and Performance Monitoring
Designed for operational review meetings and policy planning.

- **Area Chart (Time Intelligence):** Current vs. prior-month no-show rate - highlights the gap created by interventions
- **Funnel Chart:** Patient journey from booking → confirmation → attendance (visualises process loss)
- **Top/Bottom 5 Table:** Best and worst-performing neighbourhoods by no-show rate
- **Conditional Formatting:** Red/green indicators on key metrics for immediate anomaly detection

---

### Part F: Analytical Insights and Business Recommendations

*(See full write-up in the PDF report - summarised below)*

#### Top 5 Insights

1. **Lead-Time is the strongest predictor of no-shows.** Appointments categorised as "Long" (>7 days wait) have a ~32% no-show rate, versus ~5% for "Same Day" bookings.
2. **Low-GDP neighbourhoods consistently underperform.** Areas with below-average GDP per capita cluster in the bottom 20 of the neighbourhood ranking, suggesting economic barriers to attendance.
3. **The Scholarship Paradox.** Patients enrolled in the Bolsa Família program are *more* likely to miss appointments - government financial aid alone does not remove structural access barriers.
4. **Gender-based behavioral difference.** Male patients have a higher no-show rate than female patients, particularly for long-term appointments - a demographic-specific risk segment.
5. **Diminishing SMS returns.** Despite sustained SMS volume in May–June, the no-show rate did not decline proportionally, suggesting the current one-size-fits-all reminder strategy is losing effectiveness.

#### 3 Actionable Recommendations

| # | Recommendation | Targets |
|---|---|---|
| **1** | **Staggered Confirmation Workflow:** Standard SMS for short bookings; mandatory phone call 48h before long bookings; release unconfirmed slots to a waitlist | Reduces 32% "Long Wait" no-show rate |
| **2** | **Mobile Clinics for Low-GDP Regions:** Deploy pop-up health units in the bottom-ranked neighbourhoods (e.g., Ilhas Oceânicas) to remove transport barriers | Addresses economic attendance gap |
| **3** | **Demographic-Specific Outreach:** For men - efficiency-focused messaging ("Your wait today is under 10 minutes"). For scholarship recipients - partner with community leaders to prioritise health during grant payout periods | Reduces gender and scholarship no-show rates |

---

### Part G: GitHub Repository

- **Repository:** [HermelaDev/DSA3050A-Advanced-PowerBI-Exam](https://github.com/HermelaDev/DSA3050A-Advanced-PowerBI-Exam)
- **Power BI Published Report:** [View Live Dashboard](https://app.powerbi.com/view?r=eyJrIjoiZTRmZjA2Y2QtOTE4ZC00MGQ1LWEwYTctYTFlNzhiYjdkNjY1IiwidCI6IjE2ZDgzZWU2LTI1NGEtNDY5ZC1hNmNjLTU0ZTJjYTIzMTNlNyIsImMiOjh9)

---

## 7. Dashboard Features

| Feature | Description |
|---|---|
| **Dynamic Slicers** | Filter entire dashboard by Gender, Scholarship, WaitTimeCategory, Month |
| **Cross-Filtering** | Clicking any visual updates all others on the page |
| **Drill-Down** | Matrix table supports neighbourhood → wait-time drill-down |
| **Tooltips** | Scatter plot tooltips show exact GDP, HDI, and no-show rate on hover |
| **AI Visuals** | Key Influencers visual allows toggle between "Attended" and "Missed" outcomes |
| **Time Intelligence** | Area chart compares current vs. prior month using DAX DATEADD |
| **Conditional Formatting** | Red/amber/green indicators on KPI cards and ranking table |
| **Consistent Theme** | Blue-teal professional healthcare colour palette throughout |

---

## 8. Key DAX Measures

```dax
-- 1. Total Appointments
Total Appointments = COUNT(Fact_Appointments[AppointmentID])

-- 2. No-Show Rate %
No-Show Rate % =
DIVIDE(
    CALCULATE([Total Appointments], Fact_Appointments[No-show] = "Missed"),
    [Total Appointments],
    0
)

-- 3. Average Wait Days
Average Wait Days = AVERAGE(Fact_Appointments[WaitDays])

-- 4. Month-to-Date Appointments
MTD Appointments = TOTALMTD([Total Appointments], Dim_Date[Date])

-- 5. Prior Month No-Show Rate
PM No-Show Rate =
CALCULATE(
    [No-Show Rate %],
    DATEADD(Dim_Date[Date], -1, MONTH)
)

-- 6. Neighbourhood Rank
Neighbourhood Rank =
RANKX(
    ALL(Dim_Location[Neighbourhood]),
    [No-Show Rate %],
    ,
    DESC,
    Dense
)

-- 7. High GDP No-Show Rate
High GDP No-Show Rate =
CALCULATE(
    [No-Show Rate %],
    FILTER(Dim_SocioEconomic, Dim_SocioEconomic[GDP_CAPITA] > 20000)
)

-- 8. SMS Impact Variance
SMS Impact Variance =
CALCULATE([No-Show Rate %], Fact_Appointments[SMS_received] = 1)
    - [No-Show Rate %]
```

---

## 9. Key Insights

| # | Insight | Supporting Visual |
|---|---|---|
| 1 | "Long Wait" appointments (>7 days) have a **32% no-show rate** vs. ~5% for same-day bookings | Decomposition Tree, Funnel Chart |
| 2 | Low-GDP neighbourhoods dominate the **bottom 20** of the neighbourhood ranking | Scatter Plot, Bottom 5 Table |
| 3 | Scholarship recipients are **more likely to miss** appointments than non-recipients | Key Influencers Visual |
| 4 | Male patients show **higher no-show rates** on long-term bookings vs. female patients | Stacked Bar Chart + WaitTimeCategory filter |
| 5 | SMS reminders show **diminishing returns** in May–June despite high send volume | Time Intelligence Area Chart |

---

## 10. Challenges Encountered

| Challenge | How It Was Resolved |
|---|---|
| **Scientific notation in PatientId** | Converted to Text type in Power Query before any other transformation to preserve ID integrity |
| **Invalid Age values (age = –1)** | Applied a row filter in Power Query (Age ≥ 0) to remove the 1 anomalous record |
| **Counterintuitive No-show column labels** | "Yes" (confusingly meaning "did not attend") was relabeled to "Missed"; "No" was relabeled to "Attended" |
| **Flat file had no relational structure** | Used the Reference feature in Power Query to create Dim_Patient and Dim_Location from the main table |
| **No external economic data in the base dataset** | Sourced and merged the IBGE Brazilian Cities dataset via Neighbourhood → City bridge key |
| **Continuous Date table required for Time Intelligence** | Created a dedicated Dim_Date using DAX CALENDAR() to guarantee no date gaps |
| **81 neighbourhoods made visuals cluttered** | Used Top N filters and RANKX-based tables to surface only the most analytically relevant locations |

---

## 11. Conclusion

This project successfully delivered a full-stack Business Intelligence solution that transforms a flat CSV file of medical appointments into an interactive, multi-layered analytical system. By constructing a Snowflake Schema - extending the model with external socio-economic data - the dashboard moves beyond simple attendance counting to reveal the structural, demographic, and economic forces driving patient no-show behaviour.

The three recommendations (staggered confirmation workflow, mobile clinics for low-GDP areas, and demographic-specific outreach) are directly grounded in the data and provide a measurable, actionable roadmap for any healthcare administrator looking to improve service efficiency and health equity.

This examination demonstrates proficiency in the complete BI lifecycle: data acquisition → cleaning → modeling → DAX development → dashboard design → insight generation → professional documentation.

---

## 12. Repository Structure

```
DSA3050A-Advanced-PowerBI-Exam/
│
├── data/
│   ├── KaggleV2-May-2016.csv          # Raw Kaggle appointment dataset
│   └── brazil_cities_ibge.csv         # External socio-economic data (IBGE)
│
├── powerbi/
│   └── DSA3050A_Hermela_670446.pbix   # Power BI project file
│
├── screenshots/
│   ├── 01_raw_data/                   # Raw data previews
│   ├── 02_power_query/                # Before & after transformation screenshots
│   ├── 03_model_view/                 # Relationship diagram screenshot
│   ├── 04_dax_measures/               # DAX formula screenshots
│   └── 05_dashboard_pages/            # All 3 dashboard page screenshots
│
├── report/
│   └── DSA3050A_Hermela_670446_Report.pdf   # Full PDF submission
│
├── README.md                          # This file
└── LICENSE
```

---

## 13. Live Links

| Resource | Link |
|---|---|
| 📊 **Live Power BI Dashboard** | [View on Power BI Service](https://app.powerbi.com/view?r=eyJrIjoiZTRmZjA2Y2QtOTE4ZC00MGQ1LWEwYTctYTFlNzhiYjdkNjY1IiwidCI6IjE2ZDgzZWU2LTI1NGEtNDY5ZC1hNmNjLTU0ZTJjYTIzMTNlNyIsImMiOjh9) |
| 💻 **GitHub Repository** | [HermelaDev/DSA3050A-Advanced-PowerBI-Exam](https://github.com/HermelaDev/DSA3050A-Advanced-PowerBI-Exam) |
| 📦 **Dataset Source** | [Kaggle - Medical Appointment No Shows](https://www.kaggle.com/datasets/joniarroba/noshowappointments) |

---

*DSA 3050A End-Semester Examination - Spring 2026 - United States International University Africa*
