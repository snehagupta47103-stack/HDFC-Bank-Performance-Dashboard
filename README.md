<h1 align="center">🏦 HDFC Bank Performance Dashboard</h1>

<p align="center">
  <b>An interactive two-page Power BI dashboard covering deposits vs targets, loan portfolio and branch performance</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Tool-Power%20BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" />
  <img src="https://img.shields.io/badge/Transactions-5,00,000-012672?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Branches-500-E53935?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Pages-2-2E7D32?style=for-the-badge" />
</p>

<p align="center"><b>Data as on: 03/11/2026</b></p>

---

## 📌 Table of Contents

1. [Project Overview](#-project-overview)
2. [Dashboard Screenshots](#-dashboard-screenshots)
3. [Page 1: Deposits vs Targets](#-page-1-deposits-vs-targets)
4. [Page 2: Loan Portfolio & Branch Performance](#-page-2-loan-portfolio--branch-performance)
5. [Interactivity](#-interactivity)
6. [Dataset](#-dataset)
7. [Data Preparation](#-data-preparation-power-query)
8. [Data Model](#-data-model)
9. [DAX Measures](#-dax-measures)
10. [Key Observations](#-key-observations)
11. [Business Recommendations](#-business-recommendations)
12. [Design](#-design)
13. [How to Use](#-how-to-use)
14. [Notes](#-notes)
15. [Achievements](#-project-highlights--achievements)
16. [Skills Demonstrated](#-skills-demonstrated)
17. [Future Enhancements](#-future-enhancements)
18. [Author](#-author)

---

## 🎯 Project Overview

This project is an interactive Power BI dashboard for a bank, built on transaction, loan repayment and branch target data. It helps a manager see how deposits compare with targets, how the loan book and EMI collections are moving, and how branches are performing across regions, zones and cities.

The dashboard is built in the style of a bank reporting dashboard, with a branded header, side navigation, dropdown filters and KPI cards.

**Objectives**

- Compare actual deposits with deposit targets
- Track deposit trend over months and by product
- Monitor loan repayments, EMI collection and active loan accounts
- Compare branches by region, zone and city
- Give managers one place to filter and explore the numbers

---

## 🖼️ Dashboard Screenshots

### Page 1: Deposits vs Targets

<img width="962" height="643" alt="Screenshot 2026-11-07 102602" src="https://github.com/user-attachments/assets/f8acc6e9-ee2a-4cd7-bdb4-874ec87e6ceb" />


---

## 📊 Page 1: Deposits vs Targets

Shows how actual deposits compare with the targets set for each branch.

| Section | Details |
|---|---|
| **Slicers (7)** | Financial Year, Quarter, Month, Region, Zone, Branch, Product Type |
| **KPI Cards (4)** | Total Deposits, Deposit Target, Achievement %, YoY Deposit Growth |
| **Column Chart** | Actual Deposits vs Target by month |
| **Line Chart** | Deposit Trend over time |
| **Donut Chart** | Deposits by Product Type (Savings, Current, Salary, Fixed Deposit) |
| **Matrix** | Deposit performance by branch hierarchy: Region, Zone, City, Branch, with Deposits, Target, Variance, Achievement % and YoY Growth |

---

## 🏠 Page 2: Loan Portfolio & Branch Performance

Combines loan portfolio and branch performance views on a single page.

| Section | Details |
|---|---|
| **KPI Cards** | EMI Collection, Active Loan Accounts, Outstanding Balance, Total Transactions |
| **Loan Portfolio** | Loan amount and EMI collection by loan product (Home, Personal, Vehicle, Education, Business) |
| **EMI Collection Trend** | EMI collected over months |
| **Transaction Type Distribution** | Share of each transaction type |
| **Branch Performance** | Branch and region comparison of deposits, loans and transactions |

**Navigation:** A left-hand navigation bar switches between the two pages.

---

## 🔗 Interactivity

- Seven dropdown slicers filter every visual on the page.
- A **Clear Filters** button resets all slicers in one click.
- Clicking a bar, column or donut slice cross-filters the other visuals.
- The branch matrix can be expanded from Region down to the individual Branch.
- Page navigation icons move between pages.

---

## 🗂️ Dataset

The dataset follows a star schema with three fact tables and three dimension tables.

**Files used**

| File | Rows | Used For |
|---|---|---|
| `FactTransactions.csv` | 5,00,000 | Deposits, transactions, trend, product and transaction type |
| `FactLoanRepayments.csv` | 1,50,000 | EMI collection, outstanding balance, active loans |
| `BranchDepositTargets.csv` | 18,000 | Monthly deposit target for each branch |
| `DimBranch.csv` | 500 | Branch, City, Zone, Region |
| `DimDate.csv` | 1,096 | Date, Year, Quarter, Month (01-Jan-2023 to 31-Dec-2025) |
| `DimProduct.csv` | 10 | Product names (4 deposit products, 5 loan products, Credit Card) |

**File not used, and why**

| File | Reason |
|---|---|
| `DimAccount.csv` | Its BranchID matches the branch in the transaction table for only about 0.2% of rows, so joining it would give wrong branch-level numbers. Branch is taken directly from the fact tables instead. |

---

## 🧹 Data Preparation (Power Query)

1. **Headers:** `DimBranch` and `DimProduct` were loaded with generic column names (Column1, Column2). The first row was promoted to headers.
2. **Data types:** checked and set for dates, whole numbers and amount columns.
3. **Financial Year:** a new column in `DimDate` built from the calendar month (April to March).
4. **YearMonth and Month Year:** new columns for correct month sorting and clean month labels.
5. **Target DateKey:** a new column in `BranchDepositTargets` that turns Year and Month into a date key, so targets connect to the date table.
6. **Date table:** `DimDate` was marked as the date table so year-on-year calculations work.

---

## 🧩 Data Model

```
                    DimDate
                   /   |    \
                  /    |     \
   FactTransactions   FactLoanRepayments   BranchDepositTargets
                  \    |     /
                   \   |    /
                   DimBranch        DimProduct
              (Transactions and Loans also link to DimProduct)
```

| Many (Fact) | One (Dimension) |
|---|---|
| FactTransactions (DateKey, BranchID, ProductCode) | DimDate, DimBranch, DimProduct |
| FactLoanRepayments (DateKey, BranchID, ProductCode) | DimDate, DimBranch, DimProduct |
| BranchDepositTargets (BranchID, Target DateKey) | DimBranch, DimDate |

All relationships are many-to-one with single-direction filtering.

---

## 🧮 DAX Measures

```DAX
Total Deposits    = CALCULATE(SUM(FactTransactions[Amount]), FactTransactions[TransactionType] = "Deposit")

Deposit Target    = SUM(BranchDepositTargets[DepositTarget])

Total Deposits Cr = DIVIDE([Total Deposits], 10000000)

Deposit Target Cr = DIVIDE([Deposit Target], 10000000)

Achievement %     = DIVIDE([Total Deposits], [Deposit Target])

Deposits PY       = CALCULATE([Total Deposits], SAMEPERIODLASTYEAR(DimDate[Date]))

Deposit YoY %     = DIVIDE([Total Deposits] - [Deposits PY], [Deposits PY])

Variance Cr       = [Total Deposits Cr] - [Deposit Target Cr]

Deposit Target PY = CALCULATE([Deposit Target], SAMEPERIODLASTYEAR(DimDate[Date]))
```

---

## 🔍 Key Observations

| Area | Observation |
|---|---|
| **Scale of data** | 5,00,000 transactions, 1,50,000 loan repayments, 500 branches and 8 cities |
| **Deposits** | Total deposits across 2023 to 2025 are about ₹ 1,780 Cr |
| **Targets** | Total deposit targets across the same period are about ₹ 49,320 Cr |
| **Achievement** | Actual deposits are only about 3.6% of the targets, so the targets are far above what the data shows |
| **Yearly deposits** | Deposits are very steady each year: about ₹ 587 Cr (2023), ₹ 599 Cr (2024) and ₹ 591 Cr (2025) |
| **Products** | Deposit transactions are spread almost evenly across the products |
| **Loans** | Total EMI collected is about ₹ 383 Cr from about 77,775 loan accounts |

> Update this table with the final numbers from your dashboard before submitting.

---

## 💡 Business Recommendations

1. **Review the deposit targets.** Actual deposits are far below the targets, so the targets may need to be set more realistically for each branch.
2. **Look for growth.** Yearly deposits are almost flat, so campaigns for savings and fixed deposits can help lift them.
3. **Track the gap by branch.** The branch matrix shows the variance for each branch, so low performers can be identified and supported.
4. **Watch EMI collection.** Regular monitoring of EMI collection and active loans helps spot repayment problems early.
5. **Use filters for planning.** Region, zone and branch filters let managers plan actions locally instead of looking only at totals.

---

## 🎨 Design

- Dark blue header with the HDFC Bank logo, page title and "Data as on" date.
- Dark blue left navigation bar with page tiles (the active page is highlighted in red).
- Light grey canvas with white cards and rounded borders.
- Colour palette: blue, red, green and white, in line with the bank's brand colours.

---

## ▶️ How to Use

1. Open the `.pbix` file in Power BI Desktop.
2. Pick a Financial Year (for example 2024-25) from the first dropdown.
3. Use the other dropdowns to filter by Quarter, Month, Region, Zone, Branch or Product.
4. Click **Clear Filters** to reset everything.
5. Use the left navigation to move between the two pages.

---

## 📝 Notes

- The deposit targets in the dataset are much larger than actual deposits, so **Achievement %** is low compared with a typical bank dashboard. The numbers are shown exactly as they come from the data.
- Some items in a typical bank dashboard are not in this dataset, such as Branch Type, NPA %, loan status and state-wise location, so they are not shown.
- "Deposits" here means transactions with the type **Deposit**.
- `DimAccount.csv` was left out because its branch information does not match the transaction data.

---

## 🏆 Project Highlights & Achievements

- ✅ Built a complete 2-page interactive banking dashboard in Power BI
- ✅ Worked with a star schema of 6 tables and over 6,50,000 rows
- ✅ Created 7 dropdown slicers, 4 KPI cards, charts and a branch hierarchy matrix
- ✅ Added a Clear Filters button and page navigation
- ✅ Fixed missing column headers and built a financial-year column
- ✅ Spotted that `DimAccount.csv` would give wrong branch numbers and excluded it
- ✅ Connected targets, deposits and loans through shared date and branch tables
- ✅ Wrote DAX measures for deposits, targets, achievement and year-on-year growth
- ✅ Reported results honestly, including the gap between targets and actual deposits

---

## 🛠️ Skills Demonstrated

![Power Query](https://img.shields.io/badge/Data%20Cleaning-Power%20Query-F2C811?style=flat-square)
![Data Modeling](https://img.shields.io/badge/Data%20Modeling-Star%20Schema-012672?style=flat-square)
![DAX](https://img.shields.io/badge/DAX-Time%20Intelligence-E53935?style=flat-square)
![Visualisation](https://img.shields.io/badge/Data-Visualisation-2E7D32?style=flat-square)
![Dashboard Design](https://img.shields.io/badge/Dashboard-Design-F5821F?style=flat-square)
![Banking Analytics](https://img.shields.io/badge/Banking-Analytics-0F8B8D?style=flat-square)

---

## 🚀 Future Enhancements

- [ ] Add a map of branches by city
- [ ] Add a Key Insights box that updates with the filters
- [ ] Add NPA % and loan status if the data becomes available
- [ ] Add drill-down from Year to Quarter to Month
- [ ] Add a branch ranking page
- [ ] Publish to Power BI Service with scheduled refresh

---

## 👩‍💻 Author

| | |
|---|---|
| **Name** | Sneha Gupta |
| **Course / Class** | [ Data Analyst ] |
| **Date** | 07/11/2026 |
| **GitHub** | [snehagupta47103-stack](https://github.com/snehagupta47103-stack) |

---

## 🙏 Acknowledgement

I would like to thank my teacher for the guidance and for giving me the opportunity to work on this project. It helped me learn how to turn raw banking data into a clear and interactive dashboard.

---

<p align="center"><b>⭐ Thank you for viewing this project! ⭐</b></p>
