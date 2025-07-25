Similar to the customer data, let’s now create a pipeline to ingest **Product data** into the **Bronze layer lakehouse** in **Microsoft Fabric**. This pipeline will automate the process of loading raw Product data from various sources into the lakehouse. The data will be stored in its original format and organized for future transformations in the Silver and Gold layers.

### Before creating the pipeline, ensure the following:

- **Product data**: Raw Product data should be available in JSON format in ADLS account
- **Bronze layer lakehouse**: The **Bronze layer lakehouse** has been created in **Microsoft Fabric** (as discussed in the previous chapter)
- **Microsoft Fabric pipelines**: You should have access **data factory in** **Microsoft Fabric** to build and automate the pipeline

  

**1. Log in to Microsoft Fabric**

- Log into the **[Microsoft Fabric portal](https://fabric.microsoft.com/)** using your Azure credentials.

**2. Navigate to Data factory experience**

- Click on Data pipeline.

**3. Create a New Pipeline**

3.1 Give it a name, such as “Ingest_Product_data.”

[![](https://media.licdn.com/dms/image/v2/D4E0DAQHP-Qp8MchOxw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561720627?e=1752166800&v=beta&t=APRU8NJLshYHgchgLhgTFfJlRCJS9LGBC5pzNZpZVEc)](https://media.licdn.com/dms/image/v2/D4E0DAQHP-Qp8MchOxw/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561720627?e=1752166800&v=beta&t=APRU8NJLshYHgchgLhgTFfJlRCJS9LGBC5pzNZpZVEc)

3.2 Add the Copy Activity

- In the pipeline editor, add the Copy Data Activity and configure the properties of it,
- Click on Source-> Connection-> bineryville (ADLS connection created for previous customer pipeline).
- Select the File path up to the JSON file.
- Select the file format as JSON.

[![](https://media.licdn.com/dms/image/v2/D4E0DAQGAsJsV6Ri8eA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561732426?e=1752166800&v=beta&t=ptZ6MO5DqlWHa-pr1t0gZN1ovQKX-ps6TrXo88XLMpc)](https://media.licdn.com/dms/image/v2/D4E0DAQGAsJsV6Ri8eA/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732561732426?e=1752166800&v=beta&t=ptZ6MO5DqlWHa-pr1t0gZN1ovQKX-ps6TrXo88XLMpc)

- Select **Destination tab.**
- Select **Connection -> BronzeLayer.**
- Select the Table option and select **New.**
- Give the name of the table as Product.

3.3 On the top select the run tab and run the pipeline.

- Pipeline executed successfully and load the data in bronzelayer lakehouse.