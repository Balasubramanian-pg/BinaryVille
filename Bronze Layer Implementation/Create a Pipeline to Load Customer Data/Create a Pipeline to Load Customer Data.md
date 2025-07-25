Now let’s create a pipeline to ingest **customer data** into the **Bronze layer lakehouse** in **Microsoft Fabric**. This pipeline will automate the process of loading raw customer data from various sources into the lakehouse. The data will be stored in its original format and organized for future transformations in the Silver and Gold layers.

Before creating the pipeline, ensure the following:

- **Customer Data**: Raw customer data should be available in CSV format in ADLS account
- **Bronze Layer Lakehouse**: The **Bronze layer lakehouse** has been created in **Microsoft Fabric** (as discussed in the previous chapter)
- **Microsoft Fabric Pipelines**: You should have access to **data factory in** **Microsoft Fabric** to build and automate the pipeline

### **1. Log in to Microsoft Fabric**

- Log into the **[Microsoft Fabric portal](https://fabric.microsoft.com/)** using your Azure credentials.

**2. Navigate to Data factory experience**

- Click on Data pipeline.

**3. Create a New Pipeline**

3.1 Give it a name, such as “Ingest_Customer_data.”

[![](https://media.licdn.com/dms/image/v2/D4E0DAQGVJR5OoJXYrw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561259253?e=1752163200&v=beta&t=-Y29vW2HH_54eI96WEf0VAjXmESBshre0RkzNz8k1ng)](https://media.licdn.com/dms/image/v2/D4E0DAQGVJR5OoJXYrw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561259253?e=1752163200&v=beta&t=-Y29vW2HH_54eI96WEf0VAjXmESBshre0RkzNz8k1ng)

3.2 Add the Copy Activity

- In the pipeline editor, add the Copy Data Activity and configure the properties of it

![[image 9.png|image 9.png]]

- Click on Source -> Connection -> more

![[image 1 3.png|image 1 3.png]]

- Under the data source select **Azure data lake storage gen 2.**

![[image 2 4.png|image 2 4.png]]

- Provide the connection settings for it.

![[image 3 4.png|image 3 4.png]]

- Choose the ADLS account URL (ensure path has dfs.core.windows.net).
- First we have to navigate to shared access tokens and click on generate SAS token and URL

![[image 4 4.png|image 4 4.png]]

- But for this token to work, you have to disable Blob Soft Disable

else you will run into this error

> [!important] An exception occurred: DataSource.Error: AzureDataLakeStorage failed to get contents from '[https://balasubramanian.dfs.core.windows.net/landing](https://balasubramanian.dfs.core.windows.net/landing)'. Status code: 409, description: 'This endpoint does not support BlobStorageEvents or SoftDelete. Please disable these account features if you would like to use this endpoint.'.
> 
> ---
> 
> - Read Me to Resolve
>     
>     This error means that your **Storage Account** has certain **Blob-only features enabled** — specifically:
>     
>     > ❌ BlobStorageEvents or SoftDelete
>     > 
>     > These are **Blob Storage features**, and they are **not compatible with ADLS Gen2 (**`**dfs.core**`**) endpoints**.
>     
>     ---
>     
>     ## Root Cause
>     
>     You're trying to access `https://balasubramanian.dfs.core.windows.net/landing` — a Data Lake Gen2 (DFS) endpoint — but the **underlying storage account is configured for Blob Storage–only features**, which conflict with ADLS Gen2 functionality.
>     
>     This is a **misconfigured account**, and Azure explicitly blocks DFS access when:
>     
>     - **Soft delete** is enabled for blobs
>     - **Blob event publishing** (Storage events to Event Grid) is enabled
>     
>     ---
>     
>     ## Solution
>     
>     You must **disable** the following features in the Azure Portal:
>     
>     ---
>     
>     ### **Step 1: Disable Blob Soft Delete**
>     
>     1. Go to your **Storage Account** in the Azure Portal.
>     
>     ![[image 5 3.png|image 5 3.png]]
>     
>     1. Under **Data protection** (left panel)
>     
>     ![[image 6 2.png|image 6 2.png]]
>     
>     1. Find **Soft delete for blobs** → **Disable**
>     
>     ![[image 7 2.png|image 7 2.png]]
>     
>     1. Click **Save**
>     
>     ---
>     
>     ### **Step 2: Disable Storage Event Publishing (BlobStorageEvents)**
>     
>     1. In the same storage account, go to **Events**
>     
>     ![[image 8 2.png|image 8 2.png]]
>     
>     1. If there are any **event subscriptions** for `Blob Created`, `Blob Deleted`, etc. → **Delete** them
>         - These are Blob-level event triggers that conflict with hierarchical namespace
>     2. Save changes
>     
>     ---
>     
>     ### **Step 3: Confirm Hierarchical Namespace**
>     
>     While you're at it:
>     
>     1. Go to **Configuration**
>     2. Ensure **Hierarchical namespace = Enabled**
>         - If not, **you cannot use ADLS Gen2** features at all — you’ll need to **recreate the storage account** because this setting is immutable.
>     
>     [[What is Hierarchical Naming]]
>     
>     ---
>     
>     ### ❗Alternative (if you cannot disable those features)
>     
>     You can **only use the** `**blob.core.windows.net**` **endpoint** in that case — but this means you’re stuck with **flat namespace** and **cannot use ADLS Gen2 features** like directories, file-level ACLs, or Spark access via `abfss://`.
>     
>     ---
>     
>     ## ✅ Best Practice: Recreate the Storage Account (if needed)
>     
>     If this is a development/test account, the cleanest approach is:
>     
>     - Create a **new storage account**
>     - Enable:
>         - `Hierarchical namespace = Yes`
>         - Avoid enabling soft delete or storage event publishing
>     - Use `dfs.core.windows.net` from the beginning
>     
>     ---
>     
>     Let me know if you want me to generate a deployment script for a correctly configured ADLS Gen2 account (ARM/Bicep/Terraform/CLI).
>     

[[Choose authentication as SAS. (1)]]

[[Provide the SAS token. (1)]]

[![](https://media.licdn.com/dms/image/v2/D4E0DAQEsDKtzTzctmg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561287055?e=1752163200&v=beta&t=1LBEr3WDBlpKvThYyerwim_jz4BDstLiTyfnLvTVeDs)](https://media.licdn.com/dms/image/v2/D4E0DAQEsDKtzTzctmg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561287055?e=1752163200&v=beta&t=1LBEr3WDBlpKvThYyerwim_jz4BDstLiTyfnLvTVeDs)

- Select the File path up to the CSV file.
- Select the file format as delimited text.
- Select **Destination tab.**
- Select **Connection** > **BronzeLayer.**
- Select the Table option and select **New.**
- Give the name of the table as Customer.

3.3 On the top select the run tab and run the pipeline.

- Pipeline executed successfully and load the data in bronzelayer lakehouse.

---