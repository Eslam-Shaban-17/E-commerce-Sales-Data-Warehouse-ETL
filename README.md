# 🛒 E-Commerce Sales Data Warehouse (ETL + Star Schema)

> **Centralized Data Warehouse and ETL System for Brazilian E-Commerce Dataset**

---

## 🧠 Business Problem

An online retail company faced challenges in managing and analyzing its sales data:

- ❌ Scattered data across multiple CSV files
- ❌ No historical tracking of prices or customer changes
- ❌ Manual reports taking days to prepare
- ❌ Data inconsistencies leading to wrong business decisions

---

## 🚀 Solution

Developed a **centralized Data Warehouse** and automated **ETL pipelines** that:

✅ Consolidate **100,000+ orders** from **9 different sources**  
✅ Track historical changes for customers and products (SCD Type 2)  
✅ Automate daily incremental loads  
✅ Reduce reporting time **from days to seconds**  
✅ Maintain **99.9% data accuracy** through quality checks

---

## 💼 Business Impact

📊 **Sales Analysis** – Identify top products and categories in seconds  
📈 **Trend Detection** – Reveal seasonal and monthly revenue patterns  
👥 **Customer Insights** – Segment customers by region and behavior  
🚚 **Operational Efficiency** – Monitor delivery and shipping KPIs

---

## 🏗️ Technical Architecture

```

CSV Files → Staging Layer → Data Warehouse (Star Schema) → Power BI Reports
(Extract & Validate)        (Transform + Load)             (Visualization)

```

---

## 🌟 Data Warehouse Design (Star Schema)

| Table Type     | Tables                                                              | Description                        |
| -------------- | ------------------------------------------------------------------- | ---------------------------------- |
| **Fact**       | `FactSales`                                                         | 112,650 transactions               |
| **Dimensions** | `DimDate`, `DimCustomer`, `DimProduct`, `DimSeller`, `DimGeography` | SCD Type 2 for historical tracking |

**Highlights:**

- `DimCustomer`: Tracks address and region changes
- `DimProduct`: Tracks price and category changes
- `DimDate`: Enables time-series and trend analysis

---

## 🛠️ Technologies Used

| Component       | Technology                                  |
| --------------- | ------------------------------------------- |
| Database        | SQL Server 2022 (Developer Edition)         |
| ETL Tool        | SSIS (SQL Server Integration Services)      |
| Data Source     | Olist Brazilian E-Commerce Dataset (Kaggle) |
| Version Control | Git & GitHub                                |
| Reporting       | Power BI                                    |

---

## ⚙️ Key Features Implemented

### 🧩 1. Slowly Changing Dimensions (SCD Type 2)

Tracks historical changes in product prices and customer locations:

```sql
ProductKey | ProductName | Price | EffectiveDate | ExpirationDate | IsCurrent
-----------|--------------|-------|----------------|----------------|-----------
1001       | Laptop       | 800   | 2024-01-01     | 2024-06-30     | 0
1001       | Laptop       | 750   | 2024-07-01     | 9999-12-31     | 1
```

---

### ⚡ 2. Incremental Loading

Only new or changed records are processed — reducing load time by **80%**

- Full Load: ~15 minutes
- Incremental Load: ~3 minutes

---

### 🧹 3. Data Quality Framework

Ensures reliability and trust in reporting:

- Null value detection & cleaning
- Referential integrity validation
- Duplicate prevention logic
- Error logging and recovery framework

---

### 🧾 4. ETL Control & Monitoring

Each ETL execution tracks:

- Start & End timestamps
- Rows processed per table
- Success/Failure status
- Error messages & logs

---

## 💡 Business Questions Answered

**Q1.** What are the top 5 product categories by revenue?

```sql
SELECT TOP 5
    p.ProductCategory,
    SUM(f.TotalAmount) AS Revenue,
    COUNT(DISTINCT f.OrderKey) AS Orders
FROM FactSales f
JOIN DimProduct p ON f.ProductKey = p.ProductKey
WHERE p.IsCurrent = 1
GROUP BY p.ProductCategory
ORDER BY Revenue DESC;
```

➡️ _Result:_ Health & Beauty, Watches, Bed/Bath/Table, Sports/Leisure, Computers

---

**Q2.** What’s our monthly revenue trend?

```sql
SELECT
    d.Year,
    d.MonthName,
    SUM(f.TotalAmount) AS Revenue,
    COUNT(DISTINCT f.CustomerKey) AS UniqueCustomers
FROM FactSales f
JOIN DimDate d ON f.DateKey = d.DateKey
GROUP BY d.Year, d.Month, d.MonthName
ORDER BY d.Year, d.Month;
```

➡️ _Result:_ Revenue grew **45%** from Jan 2017 → Dec 2018

---

**Q3.** Which states generate the most orders?

```sql
SELECT
    g.State,
    COUNT(DISTINCT f.OrderKey) AS TotalOrders,
    SUM(f.TotalAmount) AS Revenue
FROM FactSales f
JOIN DimCustomer c ON f.CustomerKey = c.CustomerKey
JOIN DimGeography g ON c.GeographyKey = g.GeographyKey
WHERE c.IsCurrent = 1
GROUP BY g.State
ORDER BY TotalOrders DESC;
```

➡️ _Result:_ **São Paulo (SP)** leads with **41,746 orders**

---

## 📈 Project Metrics

| Metric           | Value                      |
| ---------------- | -------------------------- |
| Orders Processed | **112,650**                |
| Unique Customers | **99,441**                 |
| Sellers          | **3,095**                  |
| Unique Products  | **32,951**                 |
| Geo Coordinates  | **1M+**                    |
| ETL Runtime      | **12 minutes (full load)** |

---

## 🔧 ETL Pipeline Overview

### 🧱 **Package 1: Load_Dimensions.dtsx**

Loads all dimensions with SCD Type 2 logic:

- DimDate (2016–2020)
- DimCustomer (location changes)
- DimProduct (price/category changes)
- DimSeller, DimGeography

---

### ⚙️ **Package 2: Load_FactSales.dtsx**

- Extracts and cleans order data
- Looks up dimension surrogate keys
- Calculates derived fields (e.g., `TotalAmount = Price + Freight`)
- Loads `FactSales` with error handling

---

### 🔁 **Package 3: Load_Incremental.dtsx**

- Uses watermark table for delta detection
- Processes only new/updated rows
- Updates ETL control logs

---

## 🧠 Performance Optimizations

✅ Clustered indexes on all PKs
✅ Non-clustered indexes on FKs and date columns
✅ Partitioning by `DateKey` (per year)
✅ SSIS Bulk Insert for high-speed loads
✅ Optimized queries (reduced avg. query time **45s → 2s**)

---

## 🧩 Data Quality Metrics

| Metric       | Value        |
| ------------ | ------------ |
| Completeness | 99.8%        |
| Accuracy     | 99.9%        |
| Consistency  | 100%         |
| Timeliness   | Daily @ 2 AM |

---

## 🗂️ Project Structure

```
📁 01_Documentation/       → Architecture diagrams, data dictionary
📁 02_Source_Data/         → Raw CSV files from Kaggle
📁 03_Database_Scripts/    → DDL, DML, business queries
📁 04_SSIS_Packages/       → ETL packages (.dtsx)
📁 05_ETL_Logs/            → Execution logs
📁 06_Testing/             → Test cases & results
📁 07_Presentations/       → Slides for portfolio/demo
```

---

## 🧭 How to Run

1. **Setup Database**
   Run all scripts in `03_Database_Scripts/` (in order).
2. **Configure SSIS**
   Update connection strings for your environment.
3. **Run ETL**
   Execute `Master_Package.dtsx` for full load.
4. **Automate**
   Schedule incremental loads via **SQL Server Agent**.

---

## 🔮 Future Enhancements

- [ ] Integrate **Azure Data Factory** for orchestration
- [ ] Add **real-time streaming** via Azure Event Hub
- [ ] Build **Power BI dashboards** on top of DW
- [ ] Implement **CDC (Change Data Capture)**
- [ ] Add **data lineage tracking** and audit layer

---

## 👤 Author

**Eslam Shaban**
📊 BI Developer | Data Engineer

📧 **[eslamshaban170@gmail.com](mailto:eslamshaban170@gmail.com)**
💼 [LinkedIn](https://www.linkedin.com/in/eslamshaban7/)
💻 [GitHub](https://github.com/Eslam-Shaban-17)

---

## 📚 Dataset Source

**Brazilian E-Commerce Public Dataset by Olist**
🔗 [Kaggle Link](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

⭐ **If you found this project helpful, please star this repository!**

_Last Updated: October 2025_
