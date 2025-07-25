Loading customer data into Azure Data Lake Storage (ADLS) is a crucial step in establishing the data lakehouse architecture for Binaryville. This process ensures that large volumes of raw customer data are securely stored and readily available for further processing and analytics.

## Prerequisites

Before starting, ensure you have the following prerequisites in place:

1. **Customer Data File**:
    - Ensure that the customer data file (e.g., `customer.csv`) is available and ready for upload. This file typically comes from the CRM system and contains essential customer information such as names, contact details, purchase history, and more.
2. **Access to ADLS**:
    - You must have appropriate permissions to upload files to the designated ADLS account. This includes write access to the storage account and containers.

## Step-by-Step Instructions

### Step 1: Log in to Azure Portal

To access your Azure Data Lake Storage account, you need to log in to the Azure portal using your Azure credentials:

1. Open your web browser.
2. Navigate to the [Azure Portal](https://portal.azure.com/).
3. Enter your Azure credentials (email and password) to log in.
    - If your organization uses multi-factor authentication (MFA), complete the additional verification steps.

### Step 2: Navigate to the ADLS Account

Once logged in, locate your Azure Data Lake Storage account by following these steps:

1. In the Azure portal, locate the search bar at the top of the page.
2. Type the name of your ADLS account (e.g., “{name_of_your_adls_account}”) into the search bar and press Enter.
3. Select the ADLS account from the search results to navigate to its overview page.

## Follow Along: Loading Customer Data

To successfully load customer data into ADLS, follow the steps below:

### Step 1: Upload Customer Data File

With your ADLS account open, the next step is to upload the customer data file into the designated container. Follow these steps:

1. In the ADLS account overview page, navigate to the **Containers** section under the **Data storage** category.
2. Locate and click on the container where you want to upload the customer data (e.g., “landing” container).
3. Within the container, click on the **Upload** button to start the upload process.
4. In the upload dialog box, browse and select the customer data file (e.g., `customer.csv`) from your local machine.
5. Click **Upload** to begin uploading the file to the ADLS container.
    - Monitor the upload progress to ensure the file is successfully transferred.

### Step 2: Verify the Uploaded Data

Once the file upload is complete, verify that the customer data file is correctly stored in the ADLS container:

1. Navigate back to the container where you uploaded the customer data file.
2. Verify that the file appears in the list of blobs within the container.
3. Optionally, you can click on the file to view its properties and ensure its integrity.

![[image 7.png|image 7.png]]

- Select the ADLS account that will hold the customer data.

![[Verify the Storage Account.png|Verify the Storage Account.png]]

- Once inside the account, navigate to the **Containers** section, where your data will be stored.
    - See the problem with this is, once the hierarchial namespace is disabled it is just a blob storage and not a true Gen2 lake

![[image 2 3.png|image 2 3.png]]

[[Here is a checklist]]

![[image 3 3.png|image 3 3.png]]

> [!important] **Also check:**
> 
> [[A template to automate container-filesystem creation]]
> 
> [[A ready-made Spark read-write code snippet]]
> 
> [[Fabric lakehouse connection code with SAS-token auth]]

**3. Use an Existing Container**

- Select the existing **landing** container to upload the raw customer data.

![[image 4 3.png|image 4 3.png]]

- Locate the **customer data file** on your local machine (such as `customer.csv`) and upload.

**4. Verify the Upload**

- After the file is uploaded, verify the file’s presence.
- Check that the file size, format, and contents match what you uploaded.

![[image 5 2.png|image 5 2.png]]

---

### Conclusion

By following these steps, you have successfully loaded customer data into Azure Data Lake Storage. This data is now securely stored and ready for further processing and transformation in your data analytics pipeline.

**Additional Tips:**

- Regularly back up important customer data files to prevent data loss.
- Ensure that access permissions are correctly set to protect sensitive customer information.
- Monitor upload processes for large files and verify data integrity post-upload.

By effectively managing and loading customer data into ADLS, Binaryville can leverage its data lakehouse architecture for robust analytics and decision-making processes.

If you encounter any issues or have questions, refer to the Azure documentation or reach out to your organization’s IT support for assistance.