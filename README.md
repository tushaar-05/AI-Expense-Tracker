# 🤖 FinSmart AI — AI-Powered Student Expense Tracker

> **FOAI Group Project** | Department of Computer Science  
> Built with n8n, Google Sheets, DeepSeek AI (via OpenRouter), and Gmail

---

## 👥 Team

| Role | Name |
|------|------|
| **Team Lead** | **Tushar R Singh** |
| VM / Developer | Aman Sharma |
| Developer | Divyam Nailwal |
| Developer | Sai Hrudhay Gubba |

---

## 📌 Project Overview

**FinSmart AI** is an automated personal finance tracking system designed for students. It lets you log expenses through a simple web form, automatically categorizes them using an AI model, stores data in a structured Google Sheets dashboard, and sends intelligent email reports with budget alerts — all without any manual effort.

The system combines an **n8n automation workflow** (for AI processing and email delivery) with a **multi-sheet Excel/Google Sheets dashboard** (for live analytics, charts, and budget tracking).

---

## ✨ Key Features

- **Smart Expense Logging** — Submit expenses via an n8n form; AI auto-categorizes and generates a saving tip per transaction
- **4-Sheet Live Dashboard** — Raw Data, Weekly Analysis, Summary, and Dashboard sheets update automatically as data is added
- **Budget vs. Actual Tracking** — Per-category budget thresholds with ✅ On Track / ❗ Over Budget status indicators
- **Instant Budget Alerts** — After every submission, if any category hits ≥ 80% or 100% of its budget, an alert email fires immediately
- **Daily Summary Email** — Every evening at 8:00 PM, a formatted spending summary is automatically emailed
- **Monthly Report Email** — Full HTML report with category breakdown, payment mode analysis, weekly trends, MoM comparison, and AI tips
- **Weekly Spike Detection** — Automatically flags weeks with abnormal spending spikes (⚠ Spike)
- **Recurring Expense Detection** — Flags descriptions appearing 3+ times in a month

---

## 🗂️ Dashboard — Google Sheets / Excel Structure

The workbook (`Finance_Tracker_With_Charts_5.xlsx`) contains 4 sheets:

### 1. 📋 Raw Data
The source-of-truth table where every transaction is stored. Each row is appended automatically by the n8n workflow after a form submission.

| Column | Description |
|--------|-------------|
| Date | Day of the month (number) |
| Month | Full month name (e.g., April) |
| Year | 4-digit year |
| Category | AI-assigned category |
| Description | User-entered description |
| Amount (₹) | Expense amount in INR |
| Payment Mode | Cash / UPI / Card / Net Banking |
| Week (Month) | Week number within the month (1–5) |
| AI Suggestion | AI-generated saving tip for this expense |

### 2. 📈 Weekly Analysis
Aggregates spending by week of the month and highlights anomalies.

- **KPI row** — This week's total, last week's total, week-on-week change, and spike flag
- **Week-by-Week Breakdown** — Weeks 1–5 with total spend, transaction count, avg per transaction, change vs previous week (₹ and %), and spike indicator
- **Smart Alerts** — Plain-language summaries: current week spike status, highest-spend week, most active week, average weekly spend, and a budget tip

**Sample data (April 2026):**

| Week | Total (₹) | Transactions | Spike? |
|------|-----------|-------------|--------|
| Week 1 | 40 | 2 | — |
| Week 2 | 1,054 | 10 | ⚠ Spike |
| Week 3 | 25,365 | 11 | ⚠ Spike |
| Week 4 | 10 | 1 | ✅ OK |
| **TOTAL** | **26,469** | **24** | — |

### 3. 📊 Summary
A full-year rollup with category and payment mode breakdowns.

- **KPI Bar** — Total Spent, # Transactions, Avg per Transaction, Top Category, Biggest Expense
- **Spending by Category** — Total, % of total, and transaction count per category
- **By Payment Mode** — Total and % split across Cash, UPI, Card, Net Banking
- **Monthly Breakdown** — Month-by-month totals, transaction counts, and avg spend per day for the full year

**Current data snapshot:**

| Metric | Value |
|--------|-------|
| Total Spent | ₹26,469 |
| Transactions | 24 |
| Avg per Transaction | ₹1,103 |
| Top Category | Education |
| Biggest Expense | ₹15,000 (School Fees) |
| Top Payment Mode | Cash (63.0%) |

### 4. 🎯 Dashboard
The primary view for at-a-glance monitoring, combining all metrics in one place.

- **Top KPI Cards** — Total Spent, Transactions, This Month, Top Category, Biggest Transaction
- **Category Breakdown Table** — Spent, % of total, and transaction count per category
- **Monthly Spending Table** — Month-by-month spend, transactions, and daily average
- **Category Budget Thresholds** — Budget vs. actual for all 9 categories with remaining amount, % used, and status

**Budget Status (current):**

| Category | Budget (₹) | Spent (₹) | Used % | Status |
|----------|-----------|----------|--------|--------|
| Food | 10,000 | 2,740 | 27.4% | ✅ On Track |
| Transport | 10,000 | 5,029 | 50.3% | ✅ On Track |
| Housing | 10,000 | 0 | 0% | ✅ On Track |
| Entertainment | 10,000 | 1,500 | 15.0% | ✅ On Track |
| Health | 10,000 | 0 | 0% | ✅ On Track |
| **Education** | **10,000** | **15,000** | **150%** | **❗ Over Budget** |
| Shopping | 10,000 | 0 | 0% | ✅ On Track |
| Utilities | 10,000 | 2,100 | 21.0% | ✅ On Track |
| Other | 10,000 | 0 | 0% | ✅ On Track |
| **TOTAL** | **90,000** | **26,369** | **29.3%** | **✅ Within Budget** |

---

## 🏗️ n8n Workflow Architecture

The automation runs in two parallel pipelines:

### Pipeline 1 — Expense Submission (Form-Triggered)

```
Form Submission
    → AI Categorization (DeepSeek V3.2 via OpenRouter)
    → Format & Parse Data
    → Save to Google Sheets (Raw Data tab)
    → Budget & Recurring Check
    → [IF alerts exist] → Build Alert Email → Send Alert via Gmail
    → [Always] → Read All Data → Compute Analytics → Send Monthly Report via Gmail
```

### Pipeline 2 — Daily Scheduled Check

```
Schedule Trigger (8:00 PM daily, cron: 0 20 * * *)
    → Read All Data from Google Sheets
    → Build Daily Summary Email
    → Send Daily Summary via Gmail
```

---

## 🔧 n8n Nodes Breakdown

| Node | Type | Purpose |
|------|------|---------|
| `On form submission2` | Form Trigger | Collects Date, Amount (₹), Description, Payment Mode |
| `Basic LLM Chain2` | LLM Chain | Sends description + date to AI for categorization |
| `OpenRouter Chat Model2` | OpenRouter | DeepSeek V3.2 model powering the LLM chain |
| `Format Data2` | Code | Parses AI JSON, extracts month/week, merges form data |
| `Save to Google Sheets2` | Google Sheets | Appends formatted row to the `Raw Data` sheet |
| `Budget & Recurring Check1` | Code | Aggregates spending, detects recurring items, generates alerts |
| `Has Budget Alert?1` | IF | Routes to alert email if any category ≥ 80% of budget |
| `Build Alert Email` | Code | Builds styled HTML alert email for over-budget categories |
| `Send Budget Alert Email` | Gmail | Sends the budget alert email |
| `Read Raw Data from Sheets2` | Google Sheets | Reads all rows for the monthly report |
| `Compute Analytics & Build HTML Report2` | Code | Full analytics: KPIs, breakdown, weekly/payment trends, MoM, AI tips |
| `Send Report via Gmail2` | Gmail | Sends the complete monthly HTML report |
| `Log Success2` | Set | Logs status, timestamp, and summary of the sent report |
| `Daily Budget Check (8PM)1` | Schedule Trigger | Fires every day at 20:00 |
| `Read Data for Daily Alert1` | Google Sheets | Reads all rows for the daily summary |
| `Build Daily Summary Email` | Code | Builds compact daily spending summary HTML email |
| `Send Daily Summary Email` | Gmail | Sends the daily summary email |

---

## 🤖 AI Integration

The system uses **DeepSeek V3.2** via **OpenRouter** to process each expense on submission:

- Classifies the expense into one of 9 categories
- Extracts the month and year from the submitted date
- Calculates which week of the month it falls in (Days 1–7 = Week 1, 8–14 = Week 2, etc.)
- Generates a short, practical saving suggestion (max 10 words)

The AI always returns a structured JSON response:

```json
{
  "category": "Food",
  "month": "April",
  "year": "2026",
  "week": "3",
  "suggestion": "Cook at home more often"
}
```

**Categories supported:** Food, Transport, Housing, Entertainment, Health, Education, Shopping, Utilities, Other

---

## 📧 Email Reports

All emails are sent to `e25b070743@adypu.edu.in`.

| Report | Trigger | Contents |
|--------|---------|----------|
| **Budget Alert** | Immediately after form submission (if ≥ 80% budget hit) | 🔴 OVER / ⚠️ WARNING per category, top recurring expenses |
| **Daily Summary** | Every day at 8:00 PM | Total spent, category progress bars, alerts |
| **Monthly Report** | After every form submission | KPI cards, budget vs. actual, payment modes, weekly trend, all transactions, AI suggestions |

---

## ⚙️ Budget Configuration

Default monthly budgets are hardcoded in two Code nodes (`Budget & Recurring Check1` and `Compute Analytics & Build HTML Report2`). Edit the `BUDGETS` object to change limits:

```javascript
const BUDGETS = {
  Food: 3000,
  Transport: 1500,
  Housing: 5000,
  Entertainment: 1000,
  Health: 2000,
  Education: 2000,
  Shopping: 2000,
  Utilities: 1500,
  Other: 1000
};
```

The Dashboard sheet uses a separate budget threshold of **₹10,000 per category**, which can be updated directly in the sheet's budget column.

---

## 🚀 Setup Instructions

1. **Import `workflow.json`** into your n8n instance
2. **Connect credentials:**
   - Gmail OAuth2
   - Google Sheets OAuth2
   - OpenRouter API key
3. **Open the Google Sheet** (`Finance_Tracker_With_Charts_5.xlsx`) and ensure the `Raw Data` tab has the correct column headers
4. **Update the Sheet ID** in all Google Sheets nodes to your spreadsheet's ID
5. **Update the recipient email** in all Gmail nodes
6. **Set budget limits** inside the Code nodes and in the Dashboard sheet's budget column
7. **Activate the workflow** — the form webhook and daily 8 PM trigger go live immediately

---

## 🛠️ Tech Stack

| Tool | Usage |
|------|-------|
| **n8n** | Workflow automation (form → AI → Sheets → email) |
| **DeepSeek V3.2** | AI categorization and saving suggestions |
| **OpenRouter** | API gateway for the AI model |
| **Google Sheets / Excel** | 4-sheet live dashboard and persistent data storage |
| **Gmail API** | Email delivery for all 3 report types |
| **JavaScript (Node.js)** | All custom logic in n8n Code nodes |

---

## 📁 Repository Contents

```
/
├── workflow.json                          # n8n workflow export (import into n8n)
├── Finance_Tracker_With_Charts_5.xlsx     # Google Sheets dashboard template
└── README.md                              # This file
```

---

## 📜 License

Built for academic purposes as part of the FOAI course project.  
**FinSmart AI** — Department of Computer Science © 2026