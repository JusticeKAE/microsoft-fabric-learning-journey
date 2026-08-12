# 🔍 Lab 02: Discover and Connect to Data in OneLake

![Microsoft Fabric](https://img.shields.io/badge/Microsoft%20Fabric-Analytics%20Engineering-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)
![OneLake](https://img.shields.io/badge/OneLake-Unified%20Data%20Lake-0F9D58?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)
![DP--600](https://img.shields.io/badge/Path-DP--600%20Analytics%20Engineer-blueviolet?style=flat-square)

> Part of my ongoing [Microsoft Fabric Learning Journey](../) — hands-on labs, notes, and mistakes made (and fixed) on the road to DP-600.

---

## 📌 Overview

This lab was all about figuring out how data actually moves around inside Microsoft Fabric — from a raw CSV file sitting in a Lakehouse, to something a business user could point Power BI at and build a report from.

Concretely, I worked through: creating a Lakehouse, loading data into a Delta table, discovering that data through the OneLake Catalog, connecting two Lakehouses together with a **OneLake Shortcut** (without copying a single byte), querying everything with T-SQL through the SQL Analytics Endpoint, and finally wrapping it all in a Semantic Model for reporting.

It's a short lab on paper, but it covers a genuinely important idea in Fabric: **one copy of your data, accessed from anywhere.**

---

## 🎯 What I Learned to Do

- [x] Create and configure a Microsoft Fabric workspace
- [x] Create and manage a Lakehouse
- [x] Load CSV data into a Delta table
- [x] Discover data assets using the OneLake Catalog
- [x] Create a OneLake Shortcut between two Lakehouses
- [x] Query Lakehouse data with the SQL Analytics Endpoint
- [x] Build a Direct Lake Semantic Model
- [x] Understand cross-workspace data access *without* duplicating data

---

## 🧰 Tech Stack

| Tool | Purpose |
|---|---|
| **Microsoft Fabric** | Unified analytics platform |
| **OneLake** | Organization-wide data lake |
| **Lakehouse** | Storage for the sales dataset |
| **Delta Tables** | ACID-compliant table format |
| **SQL Analytics Endpoint** | T-SQL querying over Lakehouse data |
| **Semantic Model (Direct Lake)** | Business-ready reporting layer |
| **Power BI** | Data exploration & visualization |

---

## 🗺️ Architecture

```
sales.csv
   │
   ▼
salesdata Lakehouse
   │
   ▼
sales (Delta Table)
   │
   │  OneLake Shortcut (no data copied!)
   ▼
analytics Lakehouse
   │
   ▼
SQL Analytics Endpoint
   │
   ▼
Sales Analysis (Semantic Model)
   │
   ▼
Explore Data / Power BI Visuals
```

*(Swap this for an actual diagram image once you export one — see the note at the bottom.)*

---

## 🧪 Step-by-Step Walkthrough

### 1. Spin up the workspace

Created a workspace called `Fab_Analytics_Space`, running on a Fabric Trial Capacity so I had access to the full set of Fabric workloads.

`📸 images/workspace-created.png`

### 2. Create the Lakehouse

Created a Lakehouse named `salesdata` and uploaded the sample dataset (`sales.csv`) into its **Files** section.

`📸 images/salesdata-lakehouse.png`

### 3. Load the CSV into a Delta table

Converted the raw CSV into a proper Delta table named `sales`. This isn't just a formatting step — Delta gives you:

- ACID transactions
- Better reliability than flat files
- Native SQL query support
- Built-in data versioning

`📸 images/sales-table.png`

### 4. Go hunting in the OneLake Catalog

Used the OneLake Catalog to search for organizational data assets, found the `salesdata` Lakehouse, and dug into its metadata — owner, workspace, SQL connection string, refresh info. This is basically Fabric's version of "who owns this table and can I trust it," which matters a lot more once you're not the only person in the workspace.

### 5. Build a OneLake Shortcut

Created a second Lakehouse, `analytics`, and pointed a **OneLake Table Shortcut** at `salesdata → Tables → sales`.

> 💡 **The core idea of this whole lab:** a shortcut gives `analytics` access to the *same* underlying `sales` data — no copy, no sync job, no drift. Update it once in `salesdata`, and `analytics` sees the change instantly.

**Why that's a big deal:**
- One source of truth, instead of five slightly-different copies floating around
- Way less storage wasted on duplicate data
- Real-time access — no "wait for the nightly refresh"

`📸 images/onelake-shortcut.png`

### 6. Query it with the SQL Analytics Endpoint

Opened the SQL Analytics Endpoint and ran a couple of analytical queries straight against the Lakehouse — no ETL pipeline required.

**Revenue by product:**
```sql
SELECT
    Item,
    SUM(Quantity * UnitPrice) AS TotalRevenue,
    SUM(Quantity) AS TotalQuantity
FROM sales
GROUP BY Item
ORDER BY TotalRevenue DESC;
```

**Top 5 customers by revenue:**
```sql
SELECT TOP 5
    CustomerName,
    SUM(Quantity) AS TotalQuantity,
    SUM(Quantity * UnitPrice) AS TotalRevenue
FROM sales
GROUP BY CustomerName
ORDER BY TotalRevenue DESC;
```

`📸 images/sql-query-results.png`

### 7. Build the Semantic Model

Created a Direct Lake Semantic Model, `Sales Analysis`, sitting directly on top of the `sales` table — no import mode, no scheduled refresh. This is the layer that turns raw tables into something a business user can actually pick up and build a report from.

`📸 images/semantic-model.png`

### 8. Explore the data

Used the **Explore Data** experience to drop `Item` in as a dimension and `Quantity` as a measure, then poked around with a couple of quick visuals to see product performance at a glance.

`📸 images/explore-data.png`

---

## 🚧 Challenges & How I Solved Them

**"Why can't I shortcut my CSV file?"**
My first instinct was to create a OneLake Shortcut straight from the uploaded `sales.csv`. Turns out shortcuts only work against *tables*, not raw files sitting in the Files section.
**Fix:** loaded the CSV into a Delta table first (`sales`), then the shortcut worked exactly as expected.

**Fabric Copilot wasn't available**
The optional Copilot exercise was a dead end — Fabric returned:
> *"Copilot isn't available in your workspace due to geographic or capacity restrictions."*

Interesting side note: Microsoft 365 Copilot Chat worked fine through my student account, but Fabric Copilot specifically wasn't enabled on the Trial capacity hosting my workspace. Filed away as a "known limitation," not a mistake on my end.

---

## 🧠 Key Concepts, in Plain English

| Concept | What it actually means |
|---|---|
| **OneLake** | A single, organization-wide data lake — every workspace gets a piece of it automatically. |
| **Lakehouse** | The best of a data lake (cheap, flexible storage) and a data warehouse (structured, queryable) in one item. |
| **OneLake Shortcut** | A pointer to data stored elsewhere — you read the original data live, without copying it. |
| **SQL Analytics Endpoint** | Auto-generated SQL layer over your Lakehouse tables, so you can query Delta tables like a normal SQL database. |
| **Semantic Model** | The business-friendly layer on top of raw tables — this is what Power BI actually connects to. |

---

## ✅ Key Takeaways

- OneLake really does act as a single foundation for data across Fabric — you're not stitching together separate storage accounts.
- Shortcuts are the trick that makes "no data duplication" actually true in practice, not just a marketing line.
- The SQL Analytics Endpoint means you get T-SQL access to Lakehouse data without standing up a warehouse.
- Semantic Models are the bridge between "engineer built a table" and "business user built a report" — and Direct Lake mode skips the refresh lag entirely.

---

## 🛠️ Skills Gained

`Microsoft Fabric` · `OneLake Navigation` · `Data Discovery` · `Lakehouse Architecture` · `Delta Tables` · `SQL Analytics` · `Semantic Modeling` · `Data Visualization` · `Cross-Workspace Data Access`

---

## 🚀 Outcome

By the end of this lab I had a working, end-to-end analytics flow inside Fabric:

**CSV → Lakehouse → Delta Table → OneLake Shortcut → SQL Analytics Endpoint → Semantic Model → Data Exploration**

Nothing fancy, but it's the exact pattern real analytics teams use to avoid copy-pasting data between systems — which is honestly the part that clicked for me most in this lab.

---

<details>
<summary>📂 Repo structure note (click to expand)</summary>

```
microsoft-fabric-learning-journey/
├── Lab-01/
├── Lab-02-Discover-and-Connect-to-Data-in-OneLake/
│   ├── README.md
│   └── images/
│       ├── workspace-created.png
│       ├── salesdata-lakehouse.png
│       ├── sales-table.png
│       ├── onelake-shortcut.png
│       ├── sql-query-results.png
│       ├── semantic-model.png
│       └── explore-data.png
└── README.md
```

Drop your screenshots into the `images/` folder using the exact filenames referenced above, and every `📸` placeholder in this file will render as a real image once pushed to GitHub.

</details>
