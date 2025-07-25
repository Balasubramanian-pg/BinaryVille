Following the successful creation of our cleaned customer table, we now turn our attention to another vital dataset: product data. This guide provides a detailed, cell-by-cell explanation for building a PySpark notebook that transforms raw product data from the Bronze Lakehouse, applies critical cleansing and enrichment rules, and loads it into a `silver_products` table.

### The "Why": The Importance of Clean Product Data

Clean product data is the backbone of e-commerce analytics, supply chain management, and sales reporting. The raw data often contains inconsistencies that can skew analysis. This notebook's purpose is to rectify these issues.

- **Data Cleansing:** We correct logical errors like negative prices or stock quantities, which are physically impossible and would corrupt calculations.
- **Standardization:** We enforce business rules, such as clamping product ratings to a standard 0-5 scale, ensuring consistency across the dataset.
- **Business Enrichment:** We create new, high-value attributes that don't exist in the source system. `price_category` and `stock_status` provide immediate analytical value for merchandising and inventory management.
- **Idempotent and Incremental:** Just like our customer notebook, this process is designed to be efficient and reliable, only processing new data and safely handling re-runs.

---

### Step-by-Step Guide with Detailed Explanations

### Prerequisites

- A **Bronze Lakehouse** (e.g., `LH_Bronze`) containing the raw `product` table. This table must have an `ingestion_timestamp` column.
- An existing **Silver Lakehouse** (e.g., `LH_Silver`).

---

### Step 1, 2, & 3: Navigate and Create the Notebook

1. **Log in to Microsoft Fabric**.
2. **Navigate to your** `**LH_Silver**` **Lakehouse** via your workspace.
3. **Create a New Notebook**:
    - Click **Open notebook > New notebook**.
    - Name it descriptively: `Ntbk_Bronze_to_Silver_Products`.

---

### Notebook Implementation: A Cell-by-Cell Breakdown

### Cell 1: Define the Target Silver Table Schema

```Python
# Cell 1: Define the structure of our clean product table in the Silver Lakehouse.

spark.sql("""
    CREATE TABLE IF NOT EXISTS LH_Silver.silver_products (
        product_id STRING,
        name STRING,
        category STRING,
        brand STRING,
        price DECIMAL(10, 2), -- Use Decimal for precision in monetary values
        stock_quantity INT,
        rating DOUBLE,
        is_active BOOLEAN,
        price_category STRING,
        stock_status STRING,
        last_updated TIMESTAMP
    )
    USING DELTA
""")
```

**Detailed Explanation:**

- `**CREATE TABLE IF NOT EXISTS**`: This idempotent command ensures the script won't fail on subsequent runs if the table is already in place.
- `**LH_Silver.silver_products**`: We explicitly define the target location, preventing any ambiguity about where the table will live.
- **Schema Definition**: We are creating a well-defined schema for our `silver_products` table. This schema-on-write approach guarantees that only data conforming to these types and structures can be inserted.
    - `price DECIMAL(10, 2)`: We use the `DECIMAL` data type for the `price` column. This is a crucial best practice for financial data to avoid floating-point inaccuracies that can occur with `DOUBLE` or `FLOAT`.
    - `price_category`, `stock_status`: These are the new, enriched columns we will calculate during the transformation.
    - `last_updated`: This audit column is essential for our incremental loading strategy.
- `**USING DELTA**`: We explicitly create a Delta Lake table to leverage its transactional capabilities, especially the `MERGE` command.

---

### Cell 2: Get the Last Processed Timestamp for Incremental Loading

```Python
# Cell 2: Find our starting point by getting the high-water mark from the last run.

# Query the Silver table for the most recent 'last_updated' timestamp.
try:
    last_processed_df = spark.sql("SELECT MAX(last_updated) as last_processed FROM LH_Silver.silver_products")
    last_processed_timestamp = last_processed_df.collect()[0]['last_processed']
    if last_processed_timestamp is None:
        # This handles the case where the table exists but is empty.
        raise ValueError("Timestamp is None")
except Exception as e:
    # If the query fails or the timestamp is None (first run or empty table), set a default historical date.
    print(f"Could not retrieve last processed timestamp. Defaulting to historical load. Error: {e}")
    last_processed_timestamp = "1900-01-01T00:00:00.000+00:00"

print(f"Processing product data ingested after: {last_processed_timestamp}")
```

**Detailed Explanation:**

- **The Goal**: To efficiently process only new or updated product records since the last successful run.
- `**try...except**` **Block**: This is a more robust way to handle the "first run" scenario. The `spark.sql` command could fail if the table doesn't exist (though our first cell prevents this), or `last_processed_df.collect()` could fail if the DataFrame is empty. This structure gracefully catches any error and defaults to the historical timestamp, making the script more resilient.
- **Logic**: The logic is identical to the customer notebook: find the maximum `last_updated` value from the target table. If it's not found, use a very old date to ensure all data is processed.

---

### Cell 3: Load New Data from the Bronze Layer

```Python
# Cell 3: Read only the new product data from the Bronze layer into a temporary view.

# Lazily read the full Bronze product table
bronze_products_df = spark.read.table("LH_Bronze.product")

# Filter for new records based on ingestion_timestamp
incremental_bronze_df = bronze_products_df.filter(f"ingestion_timestamp > '{last_processed_timestamp}'")

# Create a temporary view for easy SQL querying in the next transformation step
incremental_bronze_df.createOrReplaceTempView("bronze_incremental_products_view")
```

**Detailed Explanation:**

- This cell mirrors the logic from our customer notebook. We read the source Bronze table, apply a filter to isolate only the new records based on our high-water mark, and create a temporary view. This temporary view (`bronze_incremental_products_view`) now represents the batch of data we need to clean and transform.

---

### Cell 4: Apply Transformations and Data Quality Rules

```Python
# Cell 4: Apply all data cleansing and business enrichment rules.

silver_incremental_df = spark.sql("""
    SELECT
        product_id,
        name,
        category,
        brand,

        -- Rule 1 & 2: Price and Stock Normalization
        CASE WHEN price < 0 THEN 0.00 ELSE price END AS price,
        CASE WHEN stock_quantity < 0 THEN 0 ELSE stock_quantity END AS stock_quantity,

        -- Rule 3: Rating Normalization (Clamping)
        CASE
            WHEN rating < 0 THEN 0.0
            WHEN rating > 5 THEN 5.0
            ELSE rating
        END AS rating,

        is_active,

        -- Rule 4: Price Categorization (applied to the already cleaned price)
        CASE
            WHEN (CASE WHEN price < 0 THEN 0.00 ELSE price END) > 1000 THEN 'Premium'
            WHEN (CASE WHEN price < 0 THEN 0.00 ELSE price END) > 100 THEN 'Standard'
            ELSE 'Budget'
        END AS price_category,

        -- Rule 5: Stock Status Calculation (applied to the already cleaned stock_quantity)
        CASE
            WHEN (CASE WHEN stock_quantity < 0 THEN 0 ELSE stock_quantity END) = 0 THEN 'Out of Stock'
            WHEN (CASE WHEN stock_quantity < 0 THEN 0 ELSE stock_quantity END) < 10 THEN 'Low Stock'
            WHEN (CASE WHEN stock_quantity < 0 THEN 0 ELSE stock_quantity END) < 50 THEN 'Moderate Stock'
            ELSE 'Sufficient Stock'
        END AS stock_status,

        CURRENT_TIMESTAMP() AS last_updated

    FROM bronze_incremental_products_view

    -- Basic data quality filter: ensure core identifiers are not null
    WHERE name IS NOT NULL AND category IS NOT A'S NULL
""")

# Create the final temporary view for the MERGE operation
silver_incremental_df.createOrReplaceTempView("silver_incremental_products_upserts")

```

**Detailed Explanation:**

- **Nested** `**CASE**` **Statements**: Notice that the logic for `price_category` and `stock_status` re-applies the cleaning logic (`CASE WHEN ... < 0 THEN 0 ...`). This ensures the categorization is based on the _cleaned_ values, not the original raw values. A cleaner way to write this in a multi-step transformation would be to use a Common Table Expression (CTE).
- **Cleaner Approach with CTEs (Common Table Expressions):**  
    A more readable and maintainable way to write the same logic is by using CTEs to stage the transformations.  
    
    ```SQL
    WITH CleansedData AS (
        SELECT
            product_id, name, category, brand, is_active,
            CASE WHEN price < 0 THEN 0.00 ELSE price END AS price,
            CASE WHEN stock_quantity < 0 THEN 0 ELSE stock_quantity END AS stock_quantity,
            CASE WHEN rating < 0 THEN 0.0 WHEN rating > 5 THEN 5.0 ELSE rating END AS rating
        FROM bronze_incremental_products_view
        WHERE name IS NOT NULL AND category IS NOT NULL
    )
    SELECT
        product_id, name, category, brand, price, stock_quantity, rating, is_active,
        CASE
            WHEN price > 1000 THEN 'Premium'
            WHEN price > 100 THEN 'Standard'
            ELSE 'Budget'
        END AS price_category,
        CASE
            WHEN stock_quantity = 0 THEN 'Out of Stock'
            WHEN stock_quantity < 10 THEN 'Low Stock'
            WHEN stock_quantity < 50 THEN 'Moderate Stock'
            ELSE 'Sufficient Stock'
        END AS stock_status,
        CURRENT_TIMESTAMP() AS last_updated
    FROM CleansedData
    ```
    
    This CTE approach first cleans the data in the `CleansedData` block and then applies the enrichment on the already clean columns in the final `SELECT` statement. It's much easier to debug and understand.
    

---

### Cell 5: Merge Data into the Silver Layer (Upsert)

```Python
# Cell 5: Atomically upsert the clean data into the target Silver table.

spark.sql("""
    MERGE INTO LH_Silver.silver_products target
    USING silver_incremental_products_upserts source
    ON target.product_id = source.product_id

    -- When a product is matched, update all of its attributes.
    WHEN MATCHED THEN
        UPDATE SET *

    -- When it's a new product, insert the new record.
    WHEN NOT MATCHED THEN
        INSERT *
""")
```

**Detailed Explanation:**

- The `MERGE` command works exactly as before. It uses the `product_id` as the unique key to determine whether to perform an `UPDATE` (for existing products) or an `INSERT` (for new products). This single, atomic operation prevents partial writes and ensures data consistency.

---

### Cell 6: Verify and Conclude

```Python
# Cell 6: Run a final verification query to confirm the success of the operation.

print("Product merge operation complete.")
print("Verifying data in silver_products table:")

# Display the total count and a sample of the most recently updated data.
final_count_df = spark.sql("SELECT COUNT(*) FROM LH_Silver.silver_products")
final_count_df.show()

print("Sample of recently updated products:")
spark.sql("""
    SELECT
        product_id,
        name,
        price,
        stock_quantity,
        price_category,
        stock_status,
        last_updated
    FROM LH_Silver.silver_products
    ORDER BY last_updated DESC
    LIMIT 10
""").show()

```

**Detailed Explanation:**

- This final cell provides crucial feedback. By showing the total count, you can confirm the table is growing. By showing the top 10 most recently updated records, you can immediately inspect the transformed columns like `price_category` and `stock_status` to ensure your logic was applied correctly.

### Final Thoughts

By creating this notebook, you have built another essential component of a reliable data platform. You've established a repeatable, automated process for transforming raw product data into a clean, enriched, and analysis-ready asset in your Silver Lakehouse. This sets the stage for more advanced analytics, such as creating aggregated tables in the Gold layer that might join this product data with customer and sales data.