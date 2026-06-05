# 📞 Call Analysis Dashboard
### B2B Sales Call Intelligence with Power BI

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-12%20Measures-blue?style=for-the-badge)

---

## 📌 Project Overview

A **B2B sales call tracking and lead intelligence dashboard** built in Power BI to monitor outbound calling activity, connectivity performance, DMC (Decision Maker Connect) rates, and appointment conversions across companies, sectors, and regions.

> **Designed to answer:** *How many calls are we making? Who are we actually reaching? Which leads are converting to appointments — and how does this week compare to last month?*

---

## 🖼️ Dashboard Preview

> 📸 *Add your dashboard screenshots here*

| Overview | Call Performance |
|---|---|
| `![Overview](Overview.png)` | `![Calls](Calls.png)` |

---

## 📂 Data Model

> 📸 *Add your data model screenshot here*

`![Data Model](Data_Model.png)`

### Schema

```
┌──────────────┐         ┌─────────────┐
│   Calendar   │────────▶│ Worksheet   │
│  (Date Dim)  │         │    Data     │
└──────────────┘         │  (Fact)     │
                         └─────────────┘
```

| Table | Type | Key Columns |
|---|---|---|
| **Worksheet Data** | Fact | `Sl.No`, `Company`, `Date`, `Lead`, `Disposition`, `Connectivity Status`, `DMC`, `Region`, `Sector`, `DM`, `Department`, `Designation`, `Lead Stage`, `Lead Temperature` |
| **Calendar** | Date Dimension | `Date`, `Day`, `Day of Week`, `Month Name`, `Month Number`, `Quarter`, `Week Number`, `Year`, `Year Month`, `Year Quarter`, `Is Weekend` |

**Relationship:** Calendar `[Date]` → Worksheet Data `[Date]` (one-to-many), enabling all time intelligence measures across the call log.

---

## 🔄 Data Loading & Transformation

### Source
Raw worksheet data (Excel/CSV) containing outbound call records with lead, company, and disposition details.

### Power Query Steps

| Step | Transformation |
|---|---|
| Promoted headers | First row used as column headers |
| Changed column types | `Date`, `Lead Generation Date` set to Date; `Sl.No` set to whole number |
| Verified key fields | `Connectivity Status`, `DMC`, `Disposition` validated as text categories |
| Loaded to model | Single flat fact table with all call-level attributes |

### Calendar Table
Built in DAX using `CALENDARAUTO()` and extended with time intelligence columns:

```dax
Calendar =
ADDCOLUMNS(
    CALENDARAUTO(),
    "Year",               YEAR([Date]),
    "Month Number",       MONTH([Date]),
    "Month Name",         FORMAT([Date], "MMMM"),
    "Quarter",            "Q" & QUARTER([Date]),
    "Week Number",        WEEKNUM([Date]),
    "Day",                DAY([Date]),
    "Day of Week",        FORMAT([Date], "DDDD"),
    "Day of Week Number", WEEKDAY([Date], 2),
    "Is Weekend",         WEEKDAY([Date], 2) >= 6,
    "Year Month",         FORMAT([Date], "YYYY MMM"),
    "Year Month Number",  YEAR([Date]) * 100 + MONTH([Date]),
    "Year Quarter",       FORMAT([Date], "YYYY") & " Q" & QUARTER([Date])
)
```

---

## 📊 Key DAX Measures

All 12 measures are stored in a dedicated `_Measures` table, organised into logical groups.

---

### 📋 Core Activity Metrics

**Total Dials Made** — counts every unique call attempt (row-level) in the dataset.
```dax
Total Dials Made = DISTINCTCOUNT('Worksheet Data'[Sl.No])
```

**Unique Companies** — number of distinct companies targeted across all calls.
```dax
Unique Companies = DISTINCTCOUNT('Worksheet Data'[Comapny])
```

**Average Calls Made** — dials per company, showing calling intensity per target.
```dax
Average Calls Made = DIVIDE([Total Dials Made], [Unique Companies])
```

---

### 📡 Connectivity & Reach

**Connected Attempts** — filters total dials to only those where the call was answered.
```dax
Connected Attempts =
CALCULATE(
    [Total Dails Made],
    'Worksheet Data'[Connectivity Status] = "Connected"
)
```

**Connected Companies** — distinct companies where at least one call was connected.
```dax
Connected Companies =
CALCULATE(
    [Unique Companies],
    'Worksheet Data'[Connectivity Status] = "Connected"
)
```

> These two measures together tell you both *volume* (how many calls got through) and *reach* (how many unique companies you actually spoke with).

---

### 🎯 DMC — Decision Maker Connect

DMC measures track whether the call reached an actual decision maker — the most critical quality signal in B2B outbound calling.

**DMC Companies** — companies where a decision maker was successfully reached.
```dax
DMC Companies =
CALCULATE(
    [Unique Companies],
    'Worksheet Data'[DMC] = "Yes"
)
```

**DMC Companies — Current Week**
```dax
DMC Companies Current Week =
CALCULATE(
    [DMC Companies],
    DATESINPERIOD('Calendar'[Date], MAX('Calendar'[Date]), -7, DAY)
)
```

**DMC Companies — Current Month**
```dax
DMC Companies Current Month =
CALCULATE(
    [DMC Companies],
    DATESINPERIOD('Calendar'[Date], MAX('Calendar'[Date]), 1, MONTH)
)
```

**DMC Companies — Last 3 Months**
```dax
DMC Companies Last 3 Month =
CALCULATE(
    [DMC Companies],
    DATEADD('Calendar'[Date], -3, MONTH)
)
```

> The week/month/3-month trio lets managers spot DMC rate trends without needing slicers — all three numbers sit on the dashboard simultaneously.

---

### 📅 Appointment Conversion

**Total Appointments** — companies where the call disposition resulted in an appointment being set.
```dax
Total Appointments =
CALCULATE(
    [Unique Companies],
    'Worksheet Data'[Disposition] = "Appointment"
)
```

**Appointment Call Attempts** — total dials made specifically on leads that eventually converted to appointments. Helps calculate the effort-to-conversion ratio.
```dax
Appointment Call Attempts =
IF(
    COUNTROWS(
        FILTER('Worksheet Data', 'Worksheet Data'[Disposition] = "Appointment")
    ) > 0,
    [Total Dails Made]
)
```

**Appointment DMC Connects** — among appointment-setting calls, how many were via a confirmed Decision Maker Connect. The purest quality measure in the model.
```dax
Appointment DMC Connects =
IF(
    COUNTROWS(
        FILTER('Worksheet Data', 'Worksheet Data'[Disposition] = "Appointment")
    ) > 0,
    CALCULATE([Connected Attempts], 'Worksheet Data'[DMC] = "Yes")
)
```

---

## 📈 KPIs & Business Questions Answered

| Business Question | Measure |
|---|---|
| How many calls did we make? | Total Dials Made |
| How many companies did we target? | Unique Companies |
| How often are we actually getting through? | Connected Attempts · Connected Companies |
| Are we reaching decision makers? | DMC Companies |
| How is DMC trending this week vs month? | DMC Current Week · DMC Current Month · DMC Last 3 Months |
| How many leads converted to appointments? | Total Appointments |
| How many calls does it take to book a meeting? | Appointment Call Attempts ÷ Total Appointments |
| Are our appointments coming from DM connects? | Appointment DMC Connects |
| How many calls per company on average? | Average Calls Made |

---

## 🧰 Tools & Technologies

| Tool | Purpose |
|---|---|
| **Power BI Desktop** | Dashboard development and publishing |
| **Power Query (M)** | Data ingestion and type transformation |
| **DAX** | 12 measures across activity, connectivity, DMC, and appointments |
| **Calendar Table** | Custom DAX date table for time intelligence |

---

## 🗂️ Repository Structure

```
call-analysis-dashboard/
│
├── 📊 CallAnalysisDashboard.pbix
│
├── 📁 data/
│   └── Call_Data.xlsx
│
├── 📁 screenshots/
│   ├── dashboard_overview.png
│   └── data_model.png
│
└── README.md
```

---

## 🚀 How to Run

1. Clone this repository
2. Open `CallAnalysisDashboard.pbix` in Power BI Desktop
3. Point the data source to your local `Call_Data.xlsx` if prompted
4. Refresh — all measures and the Calendar table load automatically

---

## 💡 Key Highlights

- Designed a **single flat fact table model** connected to a custom DAX Calendar for clean time intelligence
- Built **DMC trend measures** across three time windows (week / month / 3 months) using `DATESINPERIOD` and `DATEADD` — all visible simultaneously on the dashboard without slicers
- Used **nested `IF` + `COUNTROWS` + `FILTER`** pattern to safely compute appointment-scoped metrics without circular dependencies
- Tracked the full **call-to-appointment funnel**: Dials → Connected → DMC → Appointment

---

## 👤 Author

**[Your Name]**  
[LinkedIn](https://linkedin.com) · [Portfolio](https://yourportfolio.com) · [GitHub](https://github.com)

---

*Built to track and optimise B2B outbound sales calling performance.*
