# Telecom Customer Churn Analytics | Executive Dashboard (Power BI)

A single-page Power BI dashboard that shows **who is leaving a telecom company, where the revenue loss comes from, and what a retention programme could save**.

- **Data:** IBM Telco customer churn sample, 7,043 customers, one table called `Customers`.
- **Tool:** Power BI Desktop (project file `ChurnDashboard.pbip`).
- **Level:** beginner to early intermediate (slicers, KPI cards, DAX measures, a what-if simulator).

> This README describes the dashboard as it appears in your screenshot. The measure formulas in section 6 are the ones the project was built with. If you edited any measure, check it in the Fields pane (**Data > Customers**) and update the table.

---

## 1. Business problem
The company loses customers every month. Management wants to know:
1. How many customers churn (leave), and how many are retained?
2. How much revenue is lost, now and over a year?
3. Which contracts, internet services, payment methods, tenure groups and add-on services are linked to churn?
4. What could a retention programme save?

## 2. Page layout (what is on the dashboard)

| Area | Visuals | Purpose |
|---|---|---|
| **Title bar** | "TELECOM CUSTOMER CHURN ANALYTICS \| EXECUTIVE DASHBOARD" | Page title (see the typo note in section 9) |
| **Slicers (7)** | Contract, InternetService, TenureBand, SeniorLabel, PaymentMethod, **Reduction**, **Cost** | Filter every visual. The last two drive the what-if simulator. |
| **KPI cards (8)** | Churn Rate, Retention Rate, Customer Lifetime Value, Customers, MRR Lost, Annualized Revenue Lost, Net Benefit (Yr 1), Annual Revenue Kept | Headline numbers |
| **Trend / profile** | Line chart: Churn Rate by tenure. Column chart: Churn Rate by TenureBand. Pie: Churn Rate by Contract | When and in which contract customers leave |
| **Drivers** | Column: Churn Rate by PaymentMethod. Bar: by TechSupport. Pie: by OnlineSecurity | Which behaviours and services go with churn |
| **Revenue** | Bar: MRR Lost by Contract. Column: MRR Lost by InternetService | Where the money is lost |
| **Detail table** | Contract, InternetService, TenureBand, Customers, Churn Rate, MRR Lost, with a Total row | Find the riskiest segments (sort by Churn Rate) |
| **Text boxes** | "Retention recommendations" and "Key insights" | Plain-English story |

**How to use:** pick values in the slicers or click any bar or slice. All other visuals filter to match (cross-filtering). Click again to clear.

## 3. The dataset

| Column | Meaning |
|---|---|
| `customerID` | Unique customer code |
| `gender`, `SeniorCitizen`, `Partner`, `Dependents` | Profile |
| `tenure` | Months with the company |
| `PhoneService`, `MultipleLines` | Phone services |
| `InternetService` | DSL, Fiber optic, No |
| `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies` | Add-ons |
| `Contract` | Month-to-month, One year, Two year |
| `PaperlessBilling`, `PaymentMethod` | Billing |
| `MonthlyCharges`, `TotalCharges` | Monthly bill and total billed so far |
| `Churn` | Yes if the customer left |

**Columns added in cleaning:** `Churn_Flag` (1 = left, 0 = stayed), `TenureBand` (0-6 mo ... 61-72 mo), `TenureBandSort` (1-7, so bands sort correctly), `SeniorLabel` (Senior / Non-senior).

**Cleaning done:** `TotalCharges` had 11 blank values (all brand-new customers with tenure 0), set to 0; data types fixed; no duplicate customer IDs.

## 4. Data model
- One fact table `Customers` (7,043 rows). No relationships are needed.
- Two small **what-if tables** created with `GENERATESERIES`: `ReductionPct[Reduction]` (5% to 50%) and `CostPerSave[Cost]` ($20 to $400). They feed the **Reduction** and **Cost** slicers.
- `TenureBand` is sorted by `TenureBandSort`.

## 5. How the numbers work (filter context)
A **measure** recalculates every time you click. When you click "Month-to-month", Power BI first filters the `Customers` table to those customers, then evaluates every measure on only those rows. That is why one measure, such as Churn Rate, shows a different value for every bar, and why the cards change when you use a slicer.

## 6. Measures (DAX)

**Core KPIs (the first five cards)**

| Measure | DAX | Plain words |
|---|---|---|
| Customers | `COUNTROWS(Customers)` | Customers in the current filter |
| Churned Customers | `CALCULATE([Customers], Customers[Churn_Flag] = 1)` | Only those who left |
| Churn Rate | `DIVIDE([Churned Customers], [Customers])` | Left / all |
| Retention Rate | `1 - [Churn Rate]` | Stayed / all |
| Customer Lifetime Value | `DIVIDE(SUM(Customers[TotalCharges]), [Customers])` | Average total billed per customer |
| MRR Lost | `CALCULATE(SUM(Customers[MonthlyCharges]), Customers[Churn_Flag] = 1)` | Monthly recurring revenue of churned customers |
| Annualized Revenue Lost | `[MRR Lost] * 12` | MRR lost for a full year |

**What-if simulator (the last two cards and the Reduction and Cost slicers)**

| Measure | DAX | Plain words |
|---|---|---|
| Reduction Selected | `SELECTEDVALUE(ReductionPct[Reduction], 0.2)` | The churn reduction you pick (default 20%) |
| Cost Selected | `SELECTEDVALUE(CostPerSave[Cost], 120)` | Cost to retain one customer (default $120) |
| Saved Customers | `[Churned Customers] * [Reduction Selected]` | Customers kept |
| Annual Revenue Kept | `[MRR Lost] * 12 * [Reduction Selected]` | Yearly revenue protected |
| Program Cost | `[Saved Customers] * [Cost Selected]` | One-off cost |
| Net Benefit (Yr 1) | `[Annual Revenue Kept] - [Program Cost]` | Benefit minus cost |

**Check against your screenshot (slicers on All, so defaults of 20% and $120 apply):**
1,869 churned x 20% = 374 customers saved. Annual revenue kept = $139,131 x 12 x 20% = **$334K**. Program cost = 374 x $120 = $45K. Net benefit = **$289K**. These match the cards.

The model also holds extra measures (such as Share of All Churn, MRR at Risk %, Projected CLV and New Churn Rate) that are not shown on the page.

## 7. What the dashboard shows

| Finding | Value |
|---|---|
| Customers / churned | 7,043 / 1,869 |
| Churn rate / retention rate | **26.5% / 73.5%** |
| Monthly revenue lost | **$139,131** (about **$1.67M** a year) |
| Customer lifetime value (average total billed) | **$2,280** |
| Month-to-month churn | **42.7%** (one year 11.3%, two year 2.8%) |
| Month-to-month share of revenue lost | $120.8K of $139.1K (about 87%) |
| Tenure | Churn falls steadily: 52.9% in the first 6 months, 6.6% at 61-72 months |
| Payment method | Electronic check **45.3%**; bank transfer 16.7%, credit card 15.2%, mailed check 19.1% |
| Internet service | Fiber optic **41.9%**, DSL 19.0%, no internet 7.4% |
| Fiber optic revenue lost | **$114.3K** a month (DSL $22.5K, no internet $2.3K) |
| Tech support | No support 41.6% vs 15.2% with it |
| Online security | No security 41.8% vs 14.6% with it |

**What it means:** customers on month-to-month contracts, paying by electronic check, on fiber optic, without security or support add-ons, and in their first year are the biggest risk, and fiber optic customers account for most of the lost revenue.

## 8. Recommendations (from the text box)
1. Move month-to-month customers to 1-2 year plans (price lock, free add-on).
2. Give a bill credit for switching from e-check to auto-pay.
3. Onboarding: welcome call, day-30 check-in, usage tips.
4. Audit fiber quality and test a loyalty discount.
5. Bundle security and tech support free for 3 months (A/B test).
6. Alert on the high-risk profile (month-to-month, e-check, under 12 months); target churn from 26.5% to 22%.

## 9. Limitations and small fixes

**Limitations**
- The data has **no dates**, so the trend uses **tenure** (months with the company) instead of calendar months.
- The findings show **association, not proof of cause**.
- The what-if estimate assumes saved customers keep paying their current monthly charge for a full year. It is an estimate, not a forecast.
- Customer lifetime value here is average billed to date, not a prediction.
- The data is a public sample, so use it for learning and portfolios.

**Small fixes to make in Power BI**
- **Title typo:** "EXECUITVE" should be **EXECUTIVE**. Click the title text box and correct it.
- **"Annualized Revenue Lost" shows $2M** because the card rounds to whole millions. Select the card, then **Format > Callout value > Display units / Value decimal places**, and set decimal places to **2** to show $1.67M.
- **Truncated card labels** ("Customer Lifetime ...", "Annualized Revenue..."): widen the cards or shorten the label text.
- **Pie charts** for Contract and OnlineSecurity work, but bar charts compare values more easily.
- **Table total** uses Indian number grouping ($1,39,131), which follows your regional settings and is fine.

## 10. How to extend it
- Add a **drill-through page** (for example "Customer Detail") by dragging `Contract` into its drill-through well.
- Add a **date table** if you get signup and churn dates, then build a true monthly trend.
- Add a **heatmap** (matrix of Contract by InternetService with conditional formatting on Churn Rate).
- Add a **sort by Churn Rate** default on the detail table.
