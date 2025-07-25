In this chapter, we will focus on setting up our medallion layers in our lakehouse. Let’s start with the **Bronze layer lakehouse** in **Microsoft Fabric**. The Bronze layer is where raw data is ingested and stored without any transformations. This step is foundational for building the data lakehouse architecture, as the Bronze layer will hold unprocessed data that will be transformed in subsequent Silver and Gold layers.

Here, we will only cover the process of **creating the Bronze lakehouse**. Loading data into the lakehouse will be covered in the following sections of this chapter.

  

The **Bronze layer** is the raw data storage tier in a data lakehouse architecture. It stores data in its original format (CSV, JSON, Parquet, etc.) as it is ingested from various sources. The purpose of the Bronze layer is to ensure that all raw data is stored securely and in an organized manner, ready for future processing.

Key features of the Bronze layer:

- **Raw data storage**: Data is stored in its original form, without any cleansing or transformation
- **High-volume ingestion**: Designed to handle large volumes of data from multiple sources
- **Immutable**: Data is stored in its raw state, preserving the full history of changes
- **Data integrity**: The Bronze layer ensures that data is stored securely and consistently for future processing

### **Step-by-Step Guide to Creating the Bronze Layer Lakehouse**

**1. Log in to Microsoft Fabric**

- Start by logging in to the **[Microsoft Fabric portal](https://fabric.microsoft.com/)** using your Azure credentials.

**2. Navigate to the Data Lakehouse Section**

- In the Fabric portal, navigate to **Lakehouse** under the **Workspaces** section.
- This is where you will create and manage the Bronze layer for the data lakehouse.

**3. Create the Bronze Lakehouse**

- Click on **Create Lakehouse** and select the option to create a new lakehouse specifically for the Bronze layer.
- Name the lakehouse **BronzeLayer** to clearly identify its purpose as the raw data storage layer.

[![](https://media.licdn.com/dms/image/v2/D4D0DAQHNUXKSdPJ2Fw/learning-article-inline-scale_1000_2000/learning-article-inline-scale_1000_2000/0/1732559861429?e=1752163200&v=beta&t=EjpLb-WVBDnWkU-FU3kW5M-kovl2yTuIpZz25BIMt8k)](https://media.licdn.com/dms/image/v2/D4D0DAQHNUXKSdPJ2Fw/learning-article-inline-scale_1000_2000/learning-article-inline-scale_1000_2000/0/1732559861429?e=1752163200&v=beta&t=EjpLb-WVBDnWkU-FU3kW5M-kovl2yTuIpZz25BIMt8k)

Let’s move forward and prepare to load data into the lakehouse!