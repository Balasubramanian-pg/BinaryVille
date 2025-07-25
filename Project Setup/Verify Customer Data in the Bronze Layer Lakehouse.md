Once your pipeline reports a "Succeeded" status, the next crucial step is to perform validation checks. This ensures the data has not only arrived but is also accurate, complete, and usable for downstream processes. Here are four effective methods to verify your `Customer` data in the `LH_Bronze` Lakehouse, ranging from quick visual checks to in-depth programmatic analysis.

### Method 1: The Visual Explorer (Quick Spot-Check)

This is the fastest way to confirm that the table was created and contains data. It's perfect for a quick, initial confirmation.

**Why use this method?**  
It's simple, requires no coding, and provides immediate visual feedback.  

**Steps:**

1. **Navigate to your Bronze Lakehouse:**
    - Log into the **[Microsoft Fabric portal](https://fabric.microsoft.com/)**.
    - From your workspace, select your Bronze Lakehouse (e.g., `LH_Bronze`).
        
        [![](https://media.licdn.com/dms/image/v2/D4E0DAQEBiKsVugIXjw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562181575?e=1753772400&v=beta&t=EU9EPRNJUsYC746x2GISzzJUgxDj7JSnbupPY-RPVAU)](https://media.licdn.com/dms/image/v2/D4E0DAQEBiKsVugIXjw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562181575?e=1753772400&v=beta&t=EU9EPRNJUsYC746x2GISzzJUgxDj7JSnbupPY-RPVAU)
        
2. **Inspect the Table:**
    - In the **Lakehouse Explorer** on the left, you will see two main folders: `Tables` and `Files`. The `Tables` folder contains your managed Delta tables.
    - Expand the **Tables** folder. You should see your newly created `Customer` table. The icon indicates it's a Delta table.
    - Click on the `Customer` table.
3. **Preview the Data:**
    - The central pane will display a preview of the table's contents.
    - **What to look for:**
        - **Presence of Data:** Are there rows and columns?
        - **Column Headers:** Do the column names match your source CSV file's headers?
        - **Data Content:** Do the values in the first few rows look correct and sensible? (e.g., emails look like emails, dates are in a consistent format).

### Method 2: Querying with the SQL Analytics Endpoint (For Analysts and Engineers)

Every Lakehouse has a corresponding SQL Analytics Endpoint, which allows you to query your Delta tables using standard T-SQL. This is a powerful method for performing quantitative checks.

**Why use this method?**  
It uses a familiar language (SQL) and allows for precise, aggregate-level validation that a visual preview cannot provide.  

**Steps:**

1. **Switch to the SQL Endpoint:**
    - While in your Lakehouse view, click the dropdown menu in the top-right corner that says "Lakehouse".
    - Select **SQL analytics endpoint** from the list.
2. **Run Validation Queries:**
    - The view will change to a SQL query editor. Your tables from the Lakehouse will be listed in the Explorer on the left.
    - Open a **New SQL query** and run validation commands.

**Essential Validation Queries:**

- **Check the total row count:** This is the most important check. Compare this number to the row count of your source CSV file.
    
    ```SQL
    SELECT COUNT(*)
    FROM Customer;
    ```
    
- **Preview a random sample of data:**
    
    ```SQL
    SELECT *
    FROM Customer
    LIMIT 100;
    ```
    
- **Check for NULLs in critical columns:** Identify potential data quality issues early.
    
    ```SQL
    -- Check for any customers with a NULL email address
    SELECT COUNT(*)
    FROM Customer
    WHERE Email IS NULL;
    ```
    
- **Check distinct values to understand cardinality:**
    
    ```SQL
    -- See how many unique countries are in the dataset
    SELECT DISTINCT Country
    FROM Customer;
    ```
    

### Method 3: In-Depth Analysis with Spark Notebooks (For Data Engineers and Scientists)

For the most comprehensive and automatable validation, use a Spark Notebook. This gives you the full power of PySpark (or Scala/R) to programmatically inspect the DataFrame.

**Why use this method?**  
It's incredibly flexible, scalable for massive datasets, and allows you to create complex validation rules and automated data quality checks that can be integrated into your pipelines.  

**Steps:**

1. **Open a Notebook:**
    - Navigate back to your `LH_Bronze` Lakehouse view.
    - In the top ribbon, click **Open notebook** > **New notebook**. This creates a notebook that is already connected to your Lakehouse.
2. **Load the Table and Perform Checks:**
    - In a new cell, use Spark to read the Delta table into a DataFrame. Then, use DataFrame methods to validate it.

**Essential Validation Commands (PySpark):**

- **Load the table and count the rows:**
    
    ```Python
    # Load the Customer table into a Spark DataFrame
    df_customer = spark.read.table("Customer")
    
    # Get the total row count
    print(f"Total rows in Customer table: {df_customer.count()}")
    ```
    
- **Inspect the schema:** This is crucial for verifying that data types (e.g., string, integer, timestamp) were inferred correctly.
    
    ```Python
    # Print the schema to check column names and data types
    df_customer.printSchema()
    ```
    
- **Generate summary statistics:** The `.describe()` function provides a statistical overview (count, mean, stddev, min, max) for all numeric columns, which is excellent for spotting anomalies.
    
    ```Python
    # Get descriptive statistics for numeric columns
    df_customer.describe().show()
    ```
    
- **Display sample data:**
    
    ```Python
    # Show the first 20 rows
    df_customer.show()
    ```
    

### Method 4: End-to-End Validation with Power BI (For Business Users and Analysts)

Verifying that the data can be used by downstream reporting tools like Power BI is the ultimate validation. This confirms not just the data's existence but also its accessibility and usability.

**Why use this method?**  
It validates the entire data flow from source to consumption and ensures the data is ready for business intelligence and reporting.  

**Steps:**

1. **Create a Power BI Dataset:**
    - From your `LH_Bronze` Lakehouse view, click the **New Power BI dataset** button in the top ribbon.
2. **Select the Table:**
    - A new screen will appear. Select the `Customer` table and click **Confirm**.
3. **Build a Quick Report:**
    - Fabric will automatically create a Power BI dataset connected to your Lakehouse table.
    - From the dataset page, click **Create a report**.
    - In the Power BI report builder, drag a **Card** visual onto the canvas and add the primary key (e.g., `CustomerID`) to the field well. Change the summarization to **Count**.
    - This card should display the same total row count you found using SQL or Spark. You can also add a **Table** visual to browse the data.

---

### Summary: Which Method Should You Use?

|Method|Ease of Use|Power & Flexibility|Typical User|Best For...|
|---|---|---|---|---|
|**1. Visual Explorer**|★★★★★ (Easiest)|★☆☆☆☆ (Low)|Anyone|Quick, initial confirmation that the load happened.|
|**2. SQL Endpoint**|★★★★☆ (Easy)|★★★☆☆ (Medium)|Data Analyst, BI Developer, Data Engineer|Precise row counts, NULL checks, and aggregate queries.|
|**3. Spark Notebook**|★★☆☆☆ (Requires Code)|★★★★★ (Highest)|Data Engineer, Data Scientist|In-depth profiling, complex validation rules, and automated quality checks.|
|**4. Power BI**|★★★☆☆ (Requires some BI knowledge)|★★★☆☆ (Medium)|BI Developer, Business Analyst|Validating end-to-end accessibility and readiness for reporting.|

For a robust validation process, you should typically use a combination of these methods. Start with a **quick visual check (Method 1)**, perform a **quantitative check with SQL (Method 2)**, and for critical production pipelines, implement **automated checks with a Notebook (Method 3)**.