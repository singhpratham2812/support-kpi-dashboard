# Support Team KPI Dashboard

![Excel](https://img.shields.io/badge/Tool-Microsoft%20Excel-217346?style=flat&logo=microsoftexcel&logoColor=white)
![Dataset](https://img.shields.io/badge/Dataset-Kaggle%20Support%20Tickets-0C447C?style=flat)
![Domain](https://img.shields.io/badge/Domain-Operations%20Analytics-378ADD?style=flat)
![Agents](https://img.shields.io/badge/Team%20Size-11%20Agents-7F77DD?style=flat)
![Status](https://img.shields.io/badge/Status-Completed-639922?style=flat)

---

## Overview

Support teams generate thousands of tickets monthly — but without a structured performance view, ops managers are flying blind. They can't spot which agents need coaching, which SLAs are slipping, or whether a volume spike is driving a backlog.

This project builds a **monthly operational KPI dashboard** for an 11-agent support team, transforming raw ticket data into a single-screen performance view with agent-level drill-down and interactive month filtering.

> **Real-world context:** This mirrors the reporting framework I owned as Customer Experience Lead at Noon, where I managed performance analytics for a 11 person support team tracking CSAT, SLA, AHT, and resolution metrics on a monthly basis.

---

## Problem Statement

Given 8,469 support tickets with status, priority, channel, and satisfaction data, can we:
1. Track 6 core operational KPIs at both team and agent level?
2. Identify high and low performing agents from the same dataset?
3. Spot monthly trends in CSAT, SLA compliance, and pending backlogs?
4. Deliver an interactive dashboard that filters by month and agent?

---

## Dataset

| Attribute | Detail |
|-----------|--------|
| Name | Customer Support Ticket Dataset |
| Source | [Kaggle — suraj520](https://www.kaggle.com/datasets/suraj520/customer-support-ticket-dataset) |
| Rows | 8,469 tickets |
| Granularity | Monthly |
| Team size | 11 agents (see note below) |
| License | Public / Open |

### Key columns used

| Column | Description |
|--------|-------------|
| `Ticket ID` | Unique ticket identifier |
| `Ticket Status` | Open / Closed / Pending Customer Response |
| `Ticket Priority` | Critical / High / Medium / Low |
| `First Response Time` | Datetime of first agent response |
| `Time to Resolution` | Datetime ticket was resolved |
| `Customer Satisfaction Rating` | 1–5 CSAT score |
| `Ticket Type` | Billing / Technical / Product / Cancellation |

### Data quality decisions

**No agent column:** The dataset has no native `Agent_ID` or `Agent_name`  field. A simulated assignment was engineered using a formula that creates realistic productivity variance across 11 agents.

```excel=CHOOSE(
RANDBETWEEN(1,20),
"Aisha","Aisha","Aisha","Aisha",
"Ravi","Ravi","Ravi",
"Sara","Sara","Sara",
"Mohit","Mohit",
"Priya","Priya",
"Arjun",
"Neha","Rahul","Divya","Karan","Meera","Sanjay"
)
```

**No ticket creation timestamp:** The dataset includes `Date of Purchase` (product purchase date) and `First Response Time` (response datetime) — but the gap between them ranges from days to years, making DOP unreliable as a ticket creation proxy. AHT was therefore excluded from the KPI set and replaced with Ticket Volume.

**Timestamp reliability for AHT:** `First Response Time` and `Time to Resolution` are both reliable datetimes on the same ticket lifecycle — however, due to inconsistent gaps in some rows, AHT was dropped in favour of clean, reliable KPIs. This is documented as a data limitation in the dashboard.

---

## KPI Definitions

| # | KPI | Definition | Formula Approach |
|---|-----|-----------|-----------------|
| 1 | CSAT Score | Average satisfaction rating (1–5 scale) | `AVERAGEIF` on rating column |
| 2 | SLA Adherence % | % of tickets not breaching SLA threshold | Critical/High not-Closed flagged as breach |
| 3 | FCR % | % of tickets resolved on first contact | `Ticket Status = "Closed"` |
| 4 | Pending Rate | % of tickets awaiting customer response | `Ticket Status = "Pending Customer Response"` |
| 5 | Avg Utilization % | Avg ticket load vs capacity per agent/month | Tickets per agent ÷ 160 capacity |
| 6 | Ticket Volume | Total tickets handled | `COUNTA` of Ticket ID |

### SLA Breach Logic

SLA compliance was derived from priority + status since reliable response timestamps were unavailable:

| Priority | SLA Breach Condition |
|----------|---------------------|
| Critical | Any status other than Closed |
| High | Status = Open |
| Medium | No breach flagged |
| Low | No breach flagged |

```excel
=IF(OR(
  AND([@[Ticket Priority]]="Critical",[@[Ticket Status]]<>"Closed"),
  AND([@[Ticket Priority]]="High",[@[Ticket Status]]="Open")
),"Yes","No")
```

---

## Engineered Columns

All calculated columns added to the raw dataset:

| Column | Formula | Purpose |
|--------|---------|---------|
| `Month` | `=TEXT([@[First Response Time]],"MMM-YYYY")` | Monthly grouping |
| `Month_Num` | `=MONTH([@[First Response Time]])` | Numeric sort order |
| `SLA_Breach` | Priority + Status logic (above) | SLA flag |
| `FCR_Flag` | `=IF([@[Ticket Status]]="Closed","Yes","No")` | Resolution flag |
| `Pending_Flag` | `=IF([@[Ticket Status]]="Pending Customer Response","Yes","No")` | Backlog flag |
| `Agent_name` |  Weighted Random Selector formula - RANDBETWEEN(1, 20) & CHOOSE(index_num, value1, value2, ...) | Agent assignment |
| `Utilization_Pct` | `=COUNTIFS([Agent_ID],[@Agent_ID],[Month_Num],[@Month_Num])/160` | Load metric |

---

## Dashboard Components

> ![Support Team Dashboard_page-0001](https://github.com/user-attachments/assets/48a86dfd-b7d9-4d87-af60-2b0b7dfe2e27)


### KPI Cards (top row)
Six metric cards:

| KPI | Value |
|-----|-------|
| CSAT Score | 3.4 / 5.0 |
| SLA Adherence % | 75% |
| FCR % | 33% |
| Avg Utilization | 53% |
| Pending Rate | 34% |
| Ticket Volume | 8,469 |

### Charts
- **Monthly CSAT Trend** — Line chart with 3.5 target reference line
- **SLA Adherence by Month** — Column chart, red/green conditional coloring at 80% threshold
- **Agent Productivity** — Horizontal bar chart sorted by ticket volume, top 3 highlighted
- **Ticket Volume vs Pending Rate** — Combo chart with dual axes showing volume-backlog correlation

### Agent Performance Table
11-agent table with per-agent CSAT, SLA%, FCR%, and Ticket Volume. Conditional formatting applied:
- CSAT < 3.35 → coaching needed
- CSAT ≥ 3.50 → top performer
- SLA% < 70% → 
- SLA% ≥ 90% → 

---

## Key Findings

**CSAT of 3.4/5.0** indicates moderate satisfaction. Top agent (Arjun at 3.58) outperforms the bottom agent (Karan at 3.23) by 10.8% — a meaningful gap that translates to real customer experience differences at scale.

**33% FCR rate** reflects that two-thirds of tickets remain open or pending at any given snapshot — typical for a dataset captured mid-lifecycle rather than at closure.

**34% Pending Rate** closely mirrors the FCR gap, confirming that most unresolved tickets are sitting in "Pending Customer Response" rather than actively being worked.

**Volume-backlog correlation** visible in the combo chart — months with higher ticket volumes show a corresponding rise in pending rate, suggesting the team operates near capacity and volume spikes directly impact resolution quality.

**Agent productivity variance** ranges from ~370 tickets (Arjun) to ~1,670 tickets (Aisha) per month across the simulated distribution, enabling meaningful performance comparison in the agent chart.

---

## Tools & Techniques

- **Microsoft Excel 2016** — full build including pivot tables, charts, slicers
- **Formulas** — COUNTIF, COUNTIFS, AVERAGEIF, AVERAGEIFS, SUMPRODUCT, COUNTA, CHOOSE, RANDBETWEEN, CHOOSE, WEEKNUM, TEXT, IF, AND, OR
- **Features** — Pivot Tables, Slicers (Report Connections), Conditional Formatting, Combo Charts (dual axis), Data Tables

---

## Limitations

| Limitation | Impact | Decision |
|-----------|--------|----------|
| No native Agent_ID column | Can't measure true agent performance | Simulated using weighted MOD formula — documented |
| No ticket creation timestamp | Can't calculate true response time or AHT | Dropped AHT, used Status-based SLA proxy |
| Date of Purchase ≠ Ticket Created | DOP gap ranges from days to years | DOP excluded from all time calculations |
| FCR based on status snapshot | May undercount true resolutions | Noted as data limitation in dashboard |

---

## Learnings

- Engineering agent assignment with MOD-weighted distribution to create realistic productivity variance rather than even distribution
- Deriving SLA compliance from ticket priority + status when timestamp data is unreliable — a real-world data quality decision analysts face regularly
- Dropping AHT after identifying unreliable timestamp gaps — knowing when NOT to use data is as important as knowing how to use it
- Building combo charts with dual axes to surface the volume–backlog correlation
- Connecting slicers to multiple pivot tables via Report Connections for a fully interactive dashboard

---

## Roadmap

This project is part of a 5-project analytics portfolio:

| # | Project | Tools | Status |
|---|---------|-------|--------|
| 1 | E-Commerce Refund Abuse Tracker | Excel | ✅ Complete |
| 2 | Support Team KPI Dashboard | Excel | ✅ Complete |
| 3 | Fraud Detection SQL Case Study | SQL | 📅 Planned |
| 4 | Edtech Cohort Analysis | Power BI | 📅 Planned |
| 5 | ATO Pattern Detection | Python + SQL | 📅 Planned |

---

## Author

**Pratham Singh**

- Email: pratham.singh2800@gmail.com
- LinkedIn: *https://www.linkedin.com/in/prathamsingh9996/*
- Dataset: [Kaggle — suraj520/customer-support-ticket-dataset](https://www.kaggle.com/datasets/suraj520/customer-support-ticket-dataset)
