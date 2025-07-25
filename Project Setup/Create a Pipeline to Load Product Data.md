  

## Prerequisites

Before proceeding, ensure the following dependencies are met:

- **Product Data Availability**: Raw Product data must be readily available in **JSON format** within an **Azure Data Lake Storage (ADLS) Gen2 account**. This ADLS account will serve as the source for our pipeline.
- **Bronze Layer Lakehouse**: A **Bronze layer lakehouse** should be pre-created in **Microsoft Fabric**. This lakehouse acts as the designated landing zone for the raw Product data.
- **Microsoft Fabric Data Factory Access**: You must have the necessary permissions and access to the **Data Factory experience in Microsoft Fabric** to build, configure, and automate data pipelines.

---

# Steps to Ingest Product Data

## 1. Log in to Microsoft Fabric

Begin by logging into the **Microsoft Fabric portal** using your Azure credentials.

## 2. Navigate to Data Factory Experience

Once logged in, locate and click on the **"Data pipeline"** option within the Microsoft Fabric interface. This will take you to the Data Factory experience, where you can create and manage pipelines.

## 3. Create a New Pipeline

### 3.1 Name Your Pipeline

Initiate the creation of a new data pipeline. Provide a descriptive and clear name, such as **"Ingest_Product_data"**. This name helps in easy identification and management of the pipeline later.

[![](https://media.licdn.com/dms/image/v2/D4E0DAQHP-Qp8MchOxw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561720627?e=1753772400&v=beta&t=JgU4KTf2iy33s5IoZPzT6_abEH_YWQtWeJo3Ay5dpHs)](https://media.licdn.com/dms/image/v2/D4E0DAQHP-Qp8MchOxw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561720627?e=1753772400&v=beta&t=JgU4KTf2iy33s5IoZPzT6_abEH_YWQtWeJo3Ay5dpHs)

### 3.2 Add and Configure the Copy Activity

The core of this ingestion pipeline is the **Copy Data activity**. This activity is responsible for moving data from the source (ADLS) to the destination (Bronze layer lakehouse).

- **Add Copy Activity**: In the pipeline editor canvas, drag and drop or add the **"Copy Data"** activity.
- **Configure Source**:
    - Navigate to the **"Source"** tab within the Copy Data activity's properties.
    - Under **"Connection"**, select your existing ADLS connection. The prompt mentions **"bineryville (ADLS connection created for previous customer pipeline)"**, indicating that you should reuse the ADLS connection established for prior data ingestion tasks.
    - Specify the **"File path"** to the JSON file(s) containing your raw Product data. Ensure the path points directly to the location of the Product JSON files within your ADLS account.
    - Crucially, set the **"File format"** to **"JSON"**. This tells the Copy activity to interpret the source data as JSON, which is essential for correct data ingestion.
- **Configure Destination**:
    - Switch to the **"Destination"** tab within the Copy Data activity's properties.
        
        [![](https://media.licdn.com/dms/image/v2/D4E0DAQGAsJsV6Ri8eA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561732426?e=1753772400&v=beta&t=svDqssAcXjKE515qE8apFKpU95DriFCXuuNInEZKSgQ)](https://media.licdn.com/dms/image/v2/D4E0DAQGAsJsV6Ri8eA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561732426?e=1753772400&v=beta&t=svDqssAcXjKE515qE8apFKpU95DriFCXuuNInEZKSgQ)
        
    - Under **"Connection"**, select your **"BronzeLayer"** lakehouse connection.
    - Choose the **"Table"** option as the data sink type.
    - Click on **"New"** to define a new table within the Bronze layer lakehouse where the Product data will be stored.
    - Provide a meaningful name for this new table, such as **"Product"**. The Copy Data activity will create this table if it doesn't already exist and load the JSON data into it.

### 3.3 Run the Pipeline

Once the Copy Data activity is configured for both source and destination, it's time to execute the pipeline.

- On the top ribbon of the pipeline editor, locate and click the **"Run"** tab.
- Initiate the pipeline run.

Upon successful execution, the pipeline will load the raw Product data from your specified ADLS location into the newly created **"Product"** table within your **Bronze layer lakehouse**. You can monitor the pipeline's progress and status within the Microsoft Fabric interface. A successful run confirms that your raw Product data is now available in the Bronze layer, ready for further processing in the Silver and Gold layers.