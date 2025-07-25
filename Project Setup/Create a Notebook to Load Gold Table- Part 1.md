We have now arrived at the culmination of our data engineering efforts: the Gold layer. This is where we transform our clean, structured Silver layer data into high-value, aggregated assets that directly answer business questions. This notebook will create a `daily_sales` summary table, a foundational metric for almost any business.

### The "Why": The Business Value of an Aggregated Gold Table

The Gold layer is designed for speed, simplicity, and direct business consumption. While the Silver layer holds every individual transaction, a business analyst or executive often just needs to know, "How did we do yesterday?".

- **Performance:** A Power BI report querying a Gold table with 365 rows (one for each day) will be thousands of times faster than one that has to scan millions of individual transactions in the Silver layer and aggregate them on the fly.
- **Simplicity:** We pre-calculate the key metrics. This means business users can simply drag-and-drop `daily_total_sales` into a report without writing complex aggregation logic themselves.
- **Single Source of Truth for Metrics:** By defining the calculation for `daily_total_sales` here, we ensure everyone in the organization uses the exact same number. It prevents discrepancies where one report calculates sales one way and another report calculates it differently.

This notebook takes our granular Silver data and "rolls it up" into a powerful, business-ready summary.

---

### Prerequisites

- An existing **Silver Lakehouse** (`LH_Silver`) containing the clean `silver_orders` table.
- An existing **Gold Lakehouse** (`LH_Gold`) to store the final aggregated table.

---

### Step 1, 2, & 3: Navigate and Create the Notebook

1. **Log in to Microsoft Fabric**.
2. **Navigate to your** `**LH_Gold**` **Lakehouse** via your workspace.
3. **Create a New Notebook**:
    - Click **Open notebook > New notebook**.
    - Name it descriptively: `Ntbk_Gold_DailySales_Summary`.

---

### Notebook Implementation

We will explore two methods to create this Gold table.

### Method 1: The Simple Full Refresh (As described in the prompt)

This method is straightforward and effective for smaller datasets or when a full daily recalculation is acceptable. It completely replaces the Gold table every time it runs.

### Cell 1: Create or Replace the Gold Table with Aggregated Data

```Python
# Cell 1: This single command reads from Silver, aggregates, and writes to Gold, replacing the table if it exists.

spark.sql("""
    CREATE OR REPLACE TABLE LH_Gold.gold_daily_sales
    USING DELTA
    AS
    SELECT
        transaction_date,
        SUM(total_amount) AS daily_total_sales
    FROM
        LH_Silver.silver_orders
    GROUP BY
        transaction_date
""")

print("Successfully created or replaced the gold_daily_sales table.")

```

**Detailed Explanation:**

- `**CREATE OR REPLACE TABLE**`: This is a powerful, atomic command.
    - If `gold_daily_sales` does not exist, it will be created.
    - If it _does_ exist, it will be completely dropped and recreated with the new data from the `SELECT` statement. This is a full-refresh pattern.
- `**USING DELTA**`: While optional (Delta is the default), it's good practice to be explicit that we are creating a Delta Lake table.
- `**AS SELECT ...**`: The schema of the new `gold_daily_sales` table is automatically inferred from the output of this `SELECT` query. It will have two columns: `transaction_date` and `daily_total_sales`.
- `**FROM LH_Silver.silver_orders**`: We specify the full path to our clean source data in the Silver Lakehouse.
- `**GROUP BY transaction_date**`: This is the core of the aggregation. It tells Spark to collect all rows for each unique `transaction_date` into a single group.
- `**SUM(total_amount) AS daily_total_sales**`: For each group (i.e., for each day), this function calculates the sum of the `total_amount` and gives the resulting column a user-friendly name, `daily_total_sales`.

### Cell 2: Verify the Gold Table

```Python
# Cell 2: Verify the contents of the newly created Gold table.

print("Verifying the data in the Gold table...")
spark.sql("""
    SELECT *
    FROM LH_Gold.gold_daily_sales
    ORDER BY transaction_date DESC
    LIMIT 10
""").show()
```

**Detailed Explanation:**

- This query reads from our new Gold table and shows the 10 most recent days of sales. This is a perfect quick check to ensure the aggregation worked and the data looks reasonable.

---

### Method 2: Best Practice - An Incremental Approach Using `MERGE`

The full-refresh method is simple, but it's inefficient. Every day, it rescans the _entire_ `silver_orders` table. If you have years of data, this is wasteful. A more advanced, scalable, and cost-effective method is to only calculate the totals for new or changed days.

### Cell 1: Create the Gold Table (If Not Exists)

```Python
# Cell 1: Define the target table schema explicitly. This only runs once.

spark.sql("""
    CREATE TABLE IF NOT EXISTS LH_Gold.gold_daily_sales (
        transaction_date DATE,
        daily_total_sales DECIMAL(18, 2)
    )
    USING DELTA
""")
```

- We use `CREATE TABLE IF NOT EXISTS` to ensure we don't accidentally drop our table. We also explicitly define the schema, which is a more robust practice than relying on inference.

### Cell 2: Find the Last Date to Process

```Python
# Cell 2: Find the most recent date in our Gold table to know where to start the incremental aggregation.

try:
    last_processed_date_df = spark.sql("SELECT MAX(transaction_date) FROM LH_Gold.gold_daily_sales")
    last_processed_date = last_processed_date_df.collect()[0][0]
    if last_processed_date is None:
        raise ValueError("Date is None")
except Exception:
    # If table is empty, we need a very early start date
    last_processed_date = '1900-01-01'

print(f"Last processed date in Gold table: {last_processed_date}")
```

- This finds the "high-water mark" in our Gold table. We will re-process data from this date onwards to catch any late-arriving data for that day, plus all new days.

### Cell 3: Aggregate Only the Incremental Data from Silver

```Python
# Cell 3: Aggregate ONLY the new/recent data from the Silver layer.

spark.sql(f"""
    CREATE OR REPLACE TEMP VIEW daily_sales_incremental_upserts AS
    SELECT
        transaction_date,
        SUM(total_amount) AS daily_total_sales
    FROM
        LH_Silver.silver_orders
    WHERE
        transaction_date >= '{last_processed_date}' -- This filter is the key to efficiency!
    GROUP BY
        transaction_date
""")
```

- The `WHERE transaction_date >= ...` clause is the critical optimization. Instead of scanning the whole `silver_orders` table, we only scan the data for the most recent days. This is vastly more efficient.

### Cell 4: Merge the Incremental Aggregates into the Gold Table

```Python
# Cell 4: Use the MERGE command to upsert the new daily totals.

spark.sql("""
    MERGE INTO LH_Gold.gold_daily_sales target
    USING daily_sales_incremental_upserts source
    ON target.transaction_date = source.transaction_date

    -- If the date already exists, update its total sales amount.
    WHEN MATCHED THEN
        UPDATE SET target.daily_total_sales = source.daily_total_sales

    -- If it's a new date, insert the new record.
    WHEN NOT MATCHED THEN
        INSERT (transaction_date, daily_total_sales)
        VALUES (source.transaction_date, source.daily_total_sales)
""")

print("Incremental merge into gold_daily_sales is complete.")
```

- The `MERGE` command intelligently updates existing daily totals (in case of late data) and inserts new ones. This is the most robust and scalable pattern for maintaining aggregate tables.

### Summary: Which Method to Choose?

|Feature|Method 1: Simple Refresh|Method 2: Incremental Merge|
|---|---|---|
|**Simplicity**|★★★★★ (Very Easy)|★★★☆☆ (More Complex)|
|**Performance**|★☆☆☆☆ (Slow on large data)|★★★★★ (Very Fast)|
|**Cost**|High (scans all source data)|Low (scans only new data)|
|**Best For...**|Small datasets, prototypes, or when daily full recalculation is a requirement.|Production environments, large fact tables, and cost/performance optimization.|

### Final Thoughts

You have successfully built a Gold layer table, transforming clean, granular data into a powerful, business-ready metric. The `gold_daily_sales` table is now the ideal source for a Power BI report tracking daily performance. By understanding both the simple refresh and the advanced incremental merge patterns, you are equipped to build scalable and efficient data models that deliver immense value to the business.