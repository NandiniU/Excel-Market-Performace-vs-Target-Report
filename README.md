## Excel-Market-Performace-vs-Target-Report

The **Market Performance vs Target Report** empowers AtliQ Technologies to evaluate its 2021 sales performance against pre-established targets across distributor markets. Leveraging a Power Pivot model that unifies customer, market, product, monthly sales, and target data, the report calculates gaps and percent-achievement measures, and presents results via interactive PivotTables, slicers, and trend charts. By highlighting over- and under-performing regions, this analysis guides strategic channel adjustments, target refinements, and resource allocation to drive revenue growth and optimize distributor partnerships.

---
## 📷 Screenshot
![SS](https://github.com/user-attachments/assets/c969d4f3-90a2-460d-b7a5-e297958ca823)

---

### 📊 Report Overview

The **Market Performance vs Target Report** is an Excel-based analysis designed to compare AtliQ’s actual 2021 sales against predefined net-sales targets across its distributor markets. 

* **Benchmarking**: Calculates gaps between actual NetSales and 2021 targets by market
* **Performance Metrics**: Derives percentage achievement and highlights over-/under-performing regions
* **Interactive Analysis**: Uses PivotTables, slicers (Region, Division), and line charts for trend visualization
* **Data-Driven Decisions**: Empowers AtliQ to adjust channel strategies, reallocate resources, and refine target-setting practices based on clear insights.

---

### Repository Structure

```
├── Final Report.xlsx                  # Main report file
├── Report PDF.pdf                     # PDF export of the report
├── Insights.pptx                      # Key insights of the report
├── data/                              # Source tables in CSV format in a zip file
│   ├── dim_customer.csv
│   ├── dim_market.csv
│   ├── dim_product.csv
│   ├── fact_sales_monthly.csv
│   └── ns_targets_2021.csv
└── README.md                          # This file
```

---

### 📂 Source File Descriptions

Built on a Power Pivot data model that integrates `dim_customer`, `dim_market`, `dim_product`, `fact_sales_monthly`, and `ns_targets_2021`, this report enables:
* **dim\_customer.csv**: Customer master table with `customer_code`, `product_code`, `Qty`, and `net_sales_amount` for buyer profiling.
* **dim\_market.csv**: Market dimension listing `market`, `region`, and `division` to segment sales territories.
* **dim\_product.csv**: Product dimension defining `product_code`, `product_name`, and category hierarchy.
* **fact\_sales\_monthly.csv**: Monthly sales fact table recording `date`, `customer_code`, `product_code`, `Qty`, and `net_sales_amount`.
* **ns\_targets\_2021.csv**: Target table of 2021 net-sales goals by `market` and `date` (`ns_target`).

---

### ⚙️ Workflow Steps (Market Performance vs Target Report)
**Before creating the report:** Perform ETL on the source CSVs (`dim_customer.csv`, `dim_market.csv`, `dim_product.csv`, `fact_sales_monthly.csv`): extract data, remove duplicates/wrong/empty values, apply transformations, and load into the data model (create connections only).


1. **Collect transformed data**

   * In Power Pivot, click **Manage**.
   * Import fields: Net Sales, Year, Division, Country, Region from the parsed tables (derive missing fields as needed).
2. **Prepare fields**

   * Net Sales exists in the fact table.
   * Extract Year from the date column using a formula.
   * Pull Division, Country, and Region from `dim_market`.
3. **Connect data model**

   * In Power Pivot’s Diagram View, position the fact table centrally (star schema).
   * Create relationships:

     * fact\_sales\_monthly\[customer\_code] → dim\_customer\[customer\_code]
     * fact\_sales\_monthly\[product\_code] → dim\_product\[product\_code]
     * dim\_customer\[market] → dim\_market\[market]
4. **Build `dim_date` table**

   * In Power Query, create a new blank query with:

     ```powerquery
     ={Number.From(#date(2018,1,1))..Number.From(#date(2018,12,31))}
     ```
   * Convert the list to a table, change the column type to Date, and rename to `Date`.
   * Add custom column `FY Month`: `Date.AddMonths([Date], 4)` (AtliQ’s FY is Sep–Aug).
   * Add custom column `FY Year`: `Date.Year([FY Month])`.
5. **Insert PivotTable**

   * From the data model, insert a PivotTable.
   * Set filters: Region, Market, Division.
   * Set Rows: Customer.
6. **Create FY measures**
    * Open Power Query, load the fact table, and add the FY column from `dim_date` using the formula:

      ```powerquery
      =RELATED(dim_date[FY])
      ```
   * Click **Close & Load** to persist the new FY column in the model.
   * Return to the PivotTable sheet and define a measure for 2019:

      ```DAX
      NetSales_2019 := CALCULATE([Net Sales].dim_date[FY] = "2019")
      ```

      * Format as US \$ currency with two decimals.
   * Repeat steps 3 for `NetSales_2020` and `NetSales_2021`, adjusting the FY filter accordingly.


7. **Import & Prep Targets**

   * Download **ns\_target\_2021.csv**.
   * Cleanse (remove duplicates/invalids) and apply any transforms in Power Query, then load it into the data model (connection only).

8. **Connect Target Table**

   * In Power Pivot → Manage → Diagram View:

     * Link `ns_target_2021[market]` → `dim_market[market]`
     * Link `ns_target_2021[date]` → `dim_date[Date]`

9. **Measure: Target**

   ```DAX
   Target := SUM(ns_target_2021[ns_target])
   ```

   * Format as US \$ currency (2 decimals).

10. **Measure: 2021-Target**

    ```DAX
    2021-Target := [NetSales_2021] - [Target_2021]
    ```

    * Format as US \$ (2 decimals).
    * In the PivotTable’s Value Field Settings, set custom number format to `0.0,,"M"`.

11. **Measure: %**

    ```DAX
    % := DIVIDE([2021 - Target], [target21], 0)
    ```

    * Format as percentage (1 decimal).
    * Once these measures are in place, you can remove the raw `ns_target_2021` column from the model.

12. **Build PivotTable**

    * Insert a PivotTable from the data model.
    * Filters: Region, Division.
    * Rows: Market.
    * Values: `2020`, `2021`, `2021 - Target`, `%`.

13. **Finalize Formatting**

    * Apply conditional formatting to highlight over-/under-performance.
    * Adjust fonts, borders, and slicer styles for clarity.


> **Note:** Row and column names may differ; adjust to align with your mock-up.
---

