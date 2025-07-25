Loading orders data into Azure Data Lake Storage (ADLS) is a critical task in establishing the data lakehouse architecture for Binaryville. This ensures that all orders information is securely stored and readily available for further processing and analytics.

## Prerequisites

Before beginning the upload process, ensure the following prerequisites are met:

1. **Orders Data File**:
    - Ensure that the orders data file (e.g., `transactions.snappy.parquet`) is available and prepared for upload. This file typically comes from the POS (Point of Sale) system and contains essential orders information such as order IDs, product details, quantities, timestamps, and more. A sample file can be found in the exercise folder.
2. **Access to ADLS**:
    - Verify that you have the necessary permissions to upload files to the designated ADLS account, including write access to the storage account and containers.

## Step-by-Step Instructions

### Step 1: Log in to Azure Portal

Begin by logging in to the Azure portal using your Azure credentials:

1. Open your web browser.
2. Navigate to the [Azure Portal](https://portal.azure.com/).
3. Enter your Azure credentials (email and password) to log in.
    - If your organization uses multi-factor authentication (MFA), complete the additional verification steps.

### Step 2: Navigate to the ADLS Account

After logging in, locate your ADLS account to prepare for data upload:

1. In the Azure portal, locate the search bar at the top of the page.
2. Type the name of your ADLS account (e.g., “{name_of_your_adls_account}”) into the search bar and press Enter.
3. Select the ADLS account from the search results to navigate to its overview page.

### Step 3: Use an Existing Container

Upload the orders data file into the designated container:

1. In the ADLS account overview page, navigate to the **Containers** section under the **Data storage** category.
2. Locate and click on the existing container named “landing” where the raw orders data will be uploaded.
3. Within the container, click on the **Upload** button to initiate the upload process.
4. In the upload dialog box, locate and select the orders data file (e.g., `transactions.snappy.parquet`) from your local machine.
5. Click **Upload** to begin transferring the file to the ADLS container.
    - Monitor the progress to ensure the file uploads successfully.

### Step 4: Verify the Upload

After the file is uploaded, verify its presence and integrity:

1. Navigate back to the container where you uploaded the orders data file.
2. Confirm that the file appears in the list of blobs within the container.
3. Click on the file to view its properties, ensuring that the file size, format, and contents match what you uploaded.

![[image 8.png|image 8.png]]

## Conclusion

By following these detailed steps, you have successfully loaded orders data into Azure Data Lake Storage. This data is now securely stored and ready for further processing and analysis within your data lakehouse architecture.

**Additional Tips:**

- Regularly back up important orders data files to prevent any data loss.
- Ensure that access permissions are correctly configured to protect sensitive orders information.
- Monitor upload processes for large files and verify data integrity post-upload.

Effectively loading orders data into ADLS ensures that Binaryville’s orders information is efficiently managed, contributing to robust analytics and decision-making processes.

If you encounter any issues or have questions, refer to the Azure documentation or reach out to your organization’s IT support for assistance.