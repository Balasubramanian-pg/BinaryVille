This guide provides a detailed, step-by-step walkthrough for creating an automated data pipeline in Microsoft Fabric. The goal is to ingest raw customer data from an Azure Data Lake Storage (ADLS) Gen2 account into a Bronze layer Lakehouse. This pipeline forms the foundational step in a modern data platform, ensuring that raw data is captured reliably and efficiently.

### Understanding the Architecture: The "Why"

Before we build, let's understand the concepts:

- **Medallion Architecture:** We are building the first stage of a Medallion Architecture (Bronze, Silver, Gold).
    - **Bronze Layer:** This is our raw data landing zone. The data here is a direct, unfiltered, and immutable copy of the source systems. We store it in its original format (or a highly efficient open format like Delta Lake) to maintain a complete historical record and for reprocessing if needed.
- **Microsoft Fabric Lakehouse:** A Lakehouse combines the scalability and flexibility of a data lake with the performance and transactional consistency of a data warehouse. By landing our data in a Lakehouse table, we are actually creating a **Delta Table**, which provides features like ACID transactions, time travel, and schema enforcement right out of the box.
- **Data Factory in Fabric:** This is Fabric's cloud-based data integration service for creating, scheduling, and orchestrating data movement and transformation workflows (pipelines).

### Prerequisites

Ensure you have the following resources and permissions configured before you begin.

- **Microsoft Fabric Workspace:** An active Fabric workspace with a capacity (e.g., F2, F64, or a Trial) assigned to it.
- **Customer Data in ADLS Gen2:**
    - Raw customer data in CSV format is available in your Azure Data Lake Storage Gen2 account.
    - Example Path: `https://yourstorageaccount.dfs.core.windows.net/raw-data/customers/customer_data_20240101.csv`
- **Bronze Layer Lakehouse:**
    - You have already created a Lakehouse in your Fabric workspace to serve as the Bronze layer. Let's assume it's named `LH_Bronze`.
- **Permissions:**
    - You need a **Member**, **Contributor**, or **Admin** role in the Fabric workspace to create pipelines and write to the Lakehouse.
    - You need permissions to read from the ADLS Gen2 account. The easiest way to get started is with a **SAS (Shared Access Signature) token**. For production, a Service Principal is recommended.

---

### Step-by-Step Pipeline Creation

### Step 1: Log in to Microsoft Fabric and Navigate to Data Factory

1. Log into the **[Microsoft Fabric portal](https://fabric.microsoft.com/)** using your organizational credentials.
2. From the home page, click the persona switcher icon in the bottom-left corner and select **Data Factory**. This will take you to the Data Factory home page where you can create and manage your data integration assets.

### Step 2: Create a New Data Pipeline

1. On the Data Factory home page, click on the **Data pipeline** button.
2. A dialog box will appear. Provide a descriptive name for your pipeline. A good naming convention includes the source, destination, and purpose.
    - **Name:** `Ingest_ADLS_Customers_To_Bronze_LH`
3. Click **Create**.

This will open the pipeline canvas, a visual interface where you can design your data workflow.

### Step 3: Add and Configure the Copy Data Activity

The **Copy Data** activity is the workhorse of Fabric pipelines, specializing in moving data between various sources (sources) and destinations (sinks).

1. In the pipeline editor, under the **Activities** tab, find the **Copy data** activity and drag it onto the canvas. Alternatively, you can select **Copy data** from the "Add pipeline activity" section on the blank canvas.
    
    [![](https://media.licdn.com/dms/image/v2/D4E0DAQETclcr1Z2yNA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561272504?e=1753772400&v=beta&t=y7uD8LwxReOrToa9a2b-Ktr6IcQPblkA0wI9QCMszLQ)](https://media.licdn.com/dms/image/v2/D4E0DAQETclcr1Z2yNA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561272504?e=1753772400&v=beta&t=y7uD8LwxReOrToa9a2b-Ktr6IcQPblkA0wI9QCMszLQ)
    
2. Select the newly added Copy Data activity on the canvas. The configuration pane will appear at the bottom of the screen.

### 3.1 Configure the Source (ADLS Gen2)

This is where you tell the pipeline where to get the data from.

1. In the configuration pane, select the **Source** tab.
2. For **Data store type**, ensure **External** is selected.
3. Click the **+ New** button next to the **Connection** dropdown to create a new connection to your ADLS account.
4. In the "New connection" window, find and select **Azure Data Lake Storage Gen2**. Click **Continue**.
    
    [![](https://media.licdn.com/dms/image/v2/D4E0DAQEsDKtzTzctmg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561287055?e=1753772400&v=beta&t=NeBYPngMHS2K99eoDz2a7w3FLya1dRnKIySIkrMvR24)](https://media.licdn.com/dms/image/v2/D4E0DAQEsDKtzTzctmg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561287055?e=1753772400&v=beta&t=NeBYPngMHS2K99eoDz2a7w3FLya1dRnKIySIkrMvR24)
    
5. Now, fill in the connection settings:
    - **URL:** Enter the DFS endpoint URL of your ADLS account. It should look like `https://<your-storage-account-name>.dfs.core.windows.net`.
    - **Connection name:** A default name is generated, but you can change it to something more meaningful, like `ADLS_Source_Connection`.
    - **Authentication kind:** Select **Shared Access Signature (SAS)** from the dropdown.
    - **SAS token:** Paste the SAS token you generated from your Azure Storage Account. This token grants the pipeline temporary, specific permissions to access your data.
6. Click **Create** to save the connection.
7. Once the connection is created, you need to specify the exact file:
    - **File path:** Click **Browse** and navigate to your customer CSV file (e.g., `raw-data/customers/customer_data_20240101.csv`).
    - **File format:** Select **DelimitedText**. Fabric will often auto-detect this. Click on **Settings** next to the format to ensure properties like `First row as header` are correctly checked.

### 3.2 Configure the Destination (Bronze Lakehouse)

This is where you tell the pipeline where to put the data.

1. Select the **Destination** tab in the configuration pane.
2. For **Data store type**, select **Workspace**.
3. For **Workspace data store type**, select **Lakehouse**.
4. In the **Lakehouse** dropdown, select your Bronze Lakehouse (e.g., `LH_Bronze`).
5. Choose **Tables** as the **Root folder**. This ensures the data is loaded into a managed Delta Table, which is a best practice.
6. For **Table name**, you have two options:
    - **Existing:** If a table named `Customer` already exists and you want to append or overwrite, select it.
    - **New:** Since this is the first run, we'll create the table. Click the **New** radio button and enter the desired table name.
        - **Table name:** `Customer`
7. **Table action:**
    - **Append:** Adds new data to the existing table.
    - **Overwrite:** Deletes all data in the table and replaces it with the new data. For initial loads or full refreshes, **Overwrite** is a good choice.

### Step 4: Run and Monitor the Pipeline

Now that the source and destination are configured, it's time to execute the pipeline.

1. At the top of the pipeline editor, click the **Run** button in the **Home** tab.
2. You can choose to **Save and run** to persist your changes before execution.
3. The pipeline will start executing. You can monitor its progress in the **Output** tab at the bottom of the canvas. Wait for the **Status** to change to **Succeeded**.

### Step 5: Validate the Data Load

A successful pipeline run is great, but you must always verify that the data has loaded correctly.

1. **Navigate to the Lakehouse:** Use the workspace navigator on the left to go back to your `LH_Bronze` Lakehouse.
2. **Check the Table:** In the Lakehouse Explorer, you should now see a new table named `Customer` under the **Tables** section.
3. **Preview the Data:** Click on the `Customer` table. A data preview will load in the main window, showing you the columns and rows that were ingested. Verify that the columns and data types look correct and match your source CSV file.
4. **Query with SQL (Optional but Recommended):** You can switch to the **SQL analytics endpoint** of the Lakehouse using the dropdown in the top-right corner. This allows you to run SQL queries directly against your new table to perform more detailed checks.
    
    ```SQL
    SELECT COUNT(*) FROM Customer;
    SELECT * FROM Customer LIMIT 100;
    ```
    

### Conclusion and Next Steps

Congratulations! You have successfully built and executed a data pipeline in Microsoft Fabric to ingest raw customer data into your Bronze layer. This automated process is reliable, scalable, and forms the first critical step in your data analytics platform.

**What's next?**

- **Scheduling:** Go to the **Schedule** button in the pipeline editor to run this pipeline automatically (e.g., daily, hourly).
- **Parameterization:** Instead of hardcoding file names, you can parameterize the pipeline to dynamically pick up new files based on dates or triggers.
- **Silver Layer:** The next logical step is to create a notebook or another pipeline to read the raw data from the `Customer` table in the Bronze Lakehouse, clean it (e.g., handle nulls, standardize formats), enrich it, and save it as a new table in a **Silver Layer Lakehouse**.