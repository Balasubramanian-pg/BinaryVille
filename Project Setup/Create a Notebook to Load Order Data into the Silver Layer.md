With our clean customer and product tables established in the Silver layer, we now focus on the transactional data that links them: orders. This guide provides a detailed, cell-by-cell breakdown for a PySpark notebook that transforms raw order data, applies critical data quality and business logic, and loads it into a `silver_orders` fact table.

### The "Why": The Criticality of Clean Transactional Data

The `orders` table is the heart of most business analytics. It represents the events and transactions that generate revenue and drive business activity. Ensuring its quality is paramount.

- **Connecting the Dots:** This table contains the foreign keys (`customer_id`, `product_id`) that allow us to join our datasets and answer critical business questions like "What are our most popular products?" or "Who are our most valuable customers?".
- **Ensuring Referential Integrity:** By filtering out orders with missing customer or product IDs, we prevent orphaned records that can break queries and lead to inaccurate reporting.
- **Cleansing Financial Data:** Normalizing quantities and total amounts prevents incorrect financial calculations and revenue reports.
- **Enriching with Business State:** Deriving an `order_status` adds immediate, actionable context to each transaction, allowing for analysis of order fulfillment, cancellations, and more.
- **Maintaining an Idempotent, Incremental Pipeline:** We continue our best-practice pattern of creating a reliable and efficient process that can be run on a schedule without corrupting data or reprocessing redundant information.

---

### Step-by-Step Guide with Detailed Explanations

### Prerequisites

- A **Bronze Lakehouse** (e.g., `LH_Bronze`) containing the raw `orders` table with an `ingestion_timestamp` column.
- An existing **Silver Lakehouse** (e.g., `LH_Silver`).

---

### Step 1, 2, & 3: Navigate and Create the Notebook

1. **Log in to Microsoft Fabric**.
2. **Navigate to your** `**LH_Silver**` **Lakehouse**.
3. **Create a New Notebook**:
    - Click **Open notebook > New notebook**.
    - Name it descriptively: `Ntbk_Bronze_to_Silver_Orders`.

---

### Notebook Implementation: A Cell-by-Cell Breakdown

### Cell 1: Define the Target Silver Table Schema

```Python
# Cell 1: Define the structure for our clean, transactional orders table.

spark.sql("""
    CREATE TABLE IF NOT EXISTS LH_Silver.silver_orders (
        order_id STRING,
        customer_id STRING,
        product_id STRING,
        quantity INT,
        total_amount DECIMAL(18, 2), -- Best practice: Use DECIMAL for financial data
        transaction_date DATE,
        order_status STRING,
        last_updated TIMESTAMP
    )
    USING DELTA
""")
```

**Detailed Explanation:**

- `**CREATE TABLE IF NOT EXISTS**`: This makes our script idempotent, preventing errors on re-runs.
- `**LH_Silver.silver_orders**`: We explicitly name the target Lakehouse and table to ensure clarity.
- **Schema Definition**:
    - `total_amount DECIMAL(18, 2)`: We are again using the `DECIMAL` data type, which is the industry standard for storing monetary values. It guarantees precision and avoids the potential rounding errors associated with `DOUBLE`.
    - `transaction_date DATE`: Enforcing a `DATE` type ensures consistency and allows for proper date-based functions and joins.
    - `order_status`: This is our new, derived column for business context.
    - `last_updated`: Our essential audit column for the incremental `MERGE` process.
- `**USING DELTA**`: We explicitly create a Delta Lake table to enable transactional `MERGE` operations.

---

### Cell 2: Get the Last Processed Timestamp for Incremental Loading

```Python
# Cell 2: Determine the high-water mark to process only new data.

try:
    last_processed_df = spark.sql("SELECT MAX(last_updated) as last_processed FROM LH_Silver.silver_orders")
    last_processed_timestamp = last_processed_df.collect()[0]['last_processed']
    if last_processed_timestamp is None:
        raise ValueError("Timestamp is None, indicating an empty or new table.")
except Exception as e:
    # If any error occurs (e.g., table is empty), default to a historical timestamp for a full load.
    print(f"Could not retrieve last processed timestamp. Defaulting to historical load. Error: {e}")
    last_processed_timestamp = "1900-01-01T00:00:00.000+00:00"

print(f"Processing order data ingested after: {last_processed_timestamp}")
```

**Detailed Explanation:**

- This cell follows the same robust `try...except` pattern. It finds the most recent `last_updated` timestamp from the `silver_orders` table. If it succeeds, it uses that timestamp as the starting point. If it fails for any reason (e.g., the table is empty on the first run), it gracefully defaults to a very old timestamp, ensuring all historical data is included in the initial load.

---

### Cell 3: Load New Data from the Bronze Layer

```Python
# Cell 3: Read only the new order records from the Bronze layer.

# Lazily read the full Bronze orders table
bronze_orders_df = spark.read.table("LH_Bronze.orders")

# Filter for new records and create a temporary view for SQL transformations
incremental_bronze_df = bronze_orders_df.filter(f"ingestion_timestamp > '{last_processed_timestamp}'")
incremental_bronze_df.createOrReplaceTempView("bronze_incremental_orders_view")
```

**Detailed Explanation:**

- This standard step reads the source `orders` table from the Bronze layer, filters it to isolate only the new records since the last run, and registers this filtered data as a temporary view (`bronze_incremental_orders_view`) for easy querying in the next step.

---

### Cell 4: Apply Transformations and Data Quality Rules

This is the most critical step, where we clean, enrich, and validate the order data.

```Python
# Cell 4: Apply all business logic and data cleansing rules using a CTE for readability.

# Using a Common Table Expression (CTE) makes the logic clearer.
# Step 1: Clean the data. Step 2: Enrich the clean data.
silver_incremental_df = spark.sql("""
    WITH CleansedData AS (
        SELECT
            transaction_id as order_id,
            customer_id,
            product_id,
            CAST(transaction_date AS DATE) AS transaction_date,

            -- Rule 1: Normalize quantity and total_amount
            CASE WHEN quantity < 0 THEN 0 ELSE quantity END AS quantity,
            CASE WHEN total_amount < 0 THEN 0.00 ELSE total_amount END AS total_amount

        FROM bronze_incremental_orders_view

        -- Rule 4: Data Quality Checks for referential integrity
        WHERE transaction_date IS NOT NULL
          AND customer_id IS NOT NULL
          AND product_id IS NOT NULL
    )
    -- Now, apply enrichment based on the clean data from the CTE
    SELECT
        order_id,
        customer_id,
        product_id,
        quantity,
        total_amount,
        transaction_date,

        -- Rule 3: Derive Order Status based on the CLEANED quantity and total_amount
        CASE
            WHEN quantity <= 0 OR total_amount <= 0 THEN 'Cancelled'
            ELSE 'Completed'
        END AS order_status,

        CURRENT_TIMESTAMP() AS last_updated

    FROM CleansedData
""")

# Create the final temporary view for the MERGE operation
silver_incremental_df.createOrReplaceTempView("silver_incremental_orders_upserts")
```

**Detailed Explanation:**

- **Using a CTE for Clarity**: This query is structured with a Common Table Expression (`WITH CleansedData AS (...)`). This is a powerful best practice that makes complex transformations much easier to read and debug.
    1. **Inside the** `**CleansedData**` **CTE**:
        - `transaction_id as order_id`: We rename the column to match our Silver layer's business-friendly schema.
        - `CAST(transaction_date AS DATE)`: We enforce the `DATE` data type.
        - **Normalization (**`**CASE WHEN ...**`**)**: We clean the `quantity` and `total_amount` columns, correcting invalid negative values.
        - **Data Quality (**`**WHERE**` **clause)**: This is the most important filter. We discard any record that is missing a `transaction_date`, `customer_id`, or `product_id`. An order without these three keys is analytically useless, so we remove it here to maintain the integrity of our Silver layer.
    2. **In the Final** `**SELECT**`:
        - We select the already-cleaned columns from our `CleansedData` CTE.
        - **Order Status Derivation**: We apply the `CASE` statement to derive the `order_status`. Note how this logic is now applied to the _clean_ `quantity` and `total_amount` columns. The logic has also been slightly simplified: if either quantity or amount is zero or less, we can consider the order `Cancelled` or invalid. Otherwise, it's `Completed`. This is a robust starting point.
        - `CURRENT_TIMESTAMP() AS last_updated`: We stamp each processed record with the current time, which will serve as the high-water mark for the next run.

---

### Cell 5: Merge Data into the Silver Layer (Upsert)

```Python
# Cell 5: Atomically upsert the clean, transformed data into the silver_orders table.

spark.sql("""
    MERGE INTO LH_Silver.silver_orders target
    USING silver_incremental_orders_upserts source
    ON target.order_id = source.order_id

    -- If an order is matched, update its record. This handles corrections or status changes.
    WHEN MATCHED THEN
        UPDATE SET *

    -- If it's a new order, insert it.
    WHEN NOT MATCHED THEN
        INSERT *
""")
```

**Detailed Explanation:**

- The `MERGE` statement efficiently handles the "upsert" logic. It uses the `order_id` to find matching records. If an order from the source already exists in the target (e.g., a data correction was sent), `WHEN MATCHED` updates the record. If the `order_id` is new, `WHEN NOT MATCHED` inserts it. This ensures our `silver_orders` table is always an accurate reflection of the latest information.

---

### Cell 6: Verify and Conclude

```Python
# Cell 6: Run a final verification query to confirm the success of the operation.

print("Order merge operation complete.")
print("Verifying data in silver_orders table:")

# Display the total count
final_count_df = spark.sql("SELECT COUNT(*) as TotalRows FROM LH_Silver.silver_orders")
final_count_df.show()

print("Sample of recently processed orders:")
spark.sql("""
    SELECT *
    FROM LH_Silver.silver_orders
    ORDER BY last_updated DESC
    LIMIT 10
""").show()
```

**Detailed Explanation:**

- Our final step is to verify the result. We get a total count of the table to monitor its growth. More importantly, we query the 10 most recently processed records (`ORDER BY last_updated DESC`). This allows us to immediately inspect the output and confirm that our transformations—especially the derived `order_status` column—were applied correctly.

### Final Thoughts

You have now completed the creation of your Silver layer tables. By building these three notebooks for customers, products, and orders, you have established a solid, reliable, and automated foundation for your data platform. The data in your Silver Lakehouse is now cleansed, enriched, structured, and ready for the final stage: aggregation and modeling in the Gold layer to serve specific business intelligence and analytics use cases.