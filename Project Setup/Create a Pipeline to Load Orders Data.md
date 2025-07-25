With our customer and product data ingested, let’s now create a pipeline to ingest **Orders data** into the **Bronze layer lakehouse** in **Microsoft Fabric**. This pipeline will automate the process of loading raw Orders data from various sources into the lakehouse. The data will be stored in its original format and organized for future transformations in the Silver and Gold layers.

### Dependencies

Before creating the pipeline, ensure the following:

- **Orders data**: Raw Orders data should be available in parquet format in ADLS account.
- **Bronze layer lakehouse**: The **Bronze layer lakehouse** has been created in **Microsoft Fabric** (as discussed in the previous chapter)
- **Microsoft Fabric pipelines**: You should have access to **data factory in** **Microsoft Fabric** to build and automate the pipeline

### Steps

**1. Log in to Microsoft Fabric**

- Log into the **[Microsoft Fabric portal](https://fabric.microsoft.com/)** using your Azure credentials.

**2. Navigate to Data factory experience**

- Click on Data pipeline.

**3. Create a New Pipeline**

- Give it a name, such as “Ingest_Orders_data.”

[![](https://media.licdn.com/dms/image/v2/D4E0DAQHQDmYkfOrbtg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562013286?e=1753772400&v=beta&t=MQH4nMesQVrm08ibjVvDb42Uxwnd7xGUS8h8gu5aZ3o)](https://media.licdn.com/dms/image/v2/D4E0DAQHQDmYkfOrbtg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562013286?e=1753772400&v=beta&t=MQH4nMesQVrm08ibjVvDb42Uxwnd7xGUS8h8gu5aZ3o)

3.1 Add the Copy Activity

- In the pipeline editor, add the Copy Data Activity and configure the properties of it,
- Click on Source-> Connection-> bineryville (ADLS connection created for previous customer pipeline).
- Select the file path up to the Parquet file.
- Select the file format as Parquet.

[![](https://media.licdn.com/dms/image/v2/D4E0DAQGz6yE1XDFvKA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562034202?e=1753772400&v=beta&t=XUgl_xb-04llTrjfWPdvB29mWpQK72sw4YqFfhdwH_w)](https://media.licdn.com/dms/image/v2/D4E0DAQGz6yE1XDFvKA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562034202?e=1753772400&v=beta&t=XUgl_xb-04llTrjfWPdvB29mWpQK72sw4YqFfhdwH_w)

- Select **Destination tab.**
- Select **Connection** > **BronzeLayer.**
- Select the Table option and select **New.**
- Give the name of the table as Orders.

3.2 On the top select the run tab and run the pipeline.

- Pipeline executed successfully and load the data in bronzelayer lakehouse.

**PreviousNext**