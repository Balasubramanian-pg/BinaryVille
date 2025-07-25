With our raw customer data safely landed in the Bronze layer, our next goal is to refine it. The **Silver layer** is where this refinement happens. It acts as the "single source of truth" for business analysts and a reliable source for further aggregation. Before we create the artifact itself, let's understand its critical role.

### The "Why": Understanding the Purpose of the Silver Layer

The Silver layer is arguably the most valuable layer in the Medallion Architecture. It takes the raw, often messy, data from the Bronze layer and transforms it into a clean, conformed, and queryable asset.

Key characteristics of the Silver layer include:

- **Cleaned & Validated:** Data is cleansed of impurities. This includes handling `NULL` values, correcting data types (e.g., converting text dates to proper date formats), and filtering out bad records.
- **Transformed & Enriched:** Business logic is applied. This might involve joining tables (e.g., enriching customer data with sales data), calculating new fields (e.g., `CustomerAge`), or standardizing categorical values (e.g., mapping 'USA', 'U.S.A.', and 'United States' to a single 'USA' value).
- **Conformed Dimensions:** Data is modeled into a more structured, business-oriented format. This often involves creating dimension and fact tables, making the data intuitive for analysts.
- **Query-Optimized:** Data is always stored in the Delta Lake format, providing ACID transactions, time travel, and excellent query performance.

Think of it like this:

- **Bronze:** Raw, uncooked ingredients (flour, eggs, sugar).
- **Silver:** The prepared ingredients, mixed into a consistent cake batter, ready for baking.
- **Gold:** The final, decorated cake, ready to be served (aggregated for a specific business report).

In this section, we are simply creating the "mixing bowl" for our Silver data—the Lakehouse itself.

---

### Prerequisites

- An active **Microsoft Fabric Workspace** with an assigned capacity.
- Appropriate permissions within the workspace (e.g., **Admin**, **Member**, or **Contributor**) to create new artifacts.

---

### Step-by-Step Guide to Creating the Silver Lakehouse

### Step 1: Log in to Fabric and Select Your Workspace

1. Log in to the **[Microsoft Fabric portal](https://fabric.microsoft.com/)** with your credentials.
2. From the navigation pane on the left, click on **Workspaces**.
3. Select the workspace where you created your Bronze layer Lakehouse. Keeping all layers of a project within the same workspace is a common practice for easy management.

### Step 2: Initiate Lakehouse Creation

There are multiple ways to create a new artifact in Fabric. The most direct method is using the `+ New` button.

1. Inside your workspace, click the **+ New** button in the top-left corner.
2. From the dropdown list of artifacts, select **Lakehouse**.

### Step 3: Name and Create the Lakehouse

This is a critical step where we define the identity of our new data store. Naming conventions are crucial for maintaining an organized data platform.

1. A "New lakehouse" dialog box will appear.
2. Enter a descriptive name for your Lakehouse.
    
    [![](https://media.licdn.com/dms/image/v2/D4E0DAQGhyEg4VqqHSg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562571577?e=1753772400&v=beta&t=cvDmhbP0rBArMXf5fpKwjfnuARhRB4GQlyCLAwvBwXs)](https://media.licdn.com/dms/image/v2/D4E0DAQGhyEg4VqqHSg/learning-article-inline-scale_500_1000/learning-article-inline-scale_500_1000/0/1732562571577?e=1753772400&v=beta&t=cvDmhbP0rBArMXf5fpKwjfnuARhRB4GQlyCLAwvBwXs)
    
    - **Best Practice:** Use a consistent naming convention that includes the layer and the artifact type. Instead of just `SilverLayer`, consider a more descriptive name like `LH_Silver` or `Silver_Lakehouse`. This makes it immediately clear in a long list of artifacts that this item is a Lakehouse and it belongs to the Silver tier.
    - **Name:** `LH_Silver`
3. Click **Create**.

Fabric will now provision your new Lakehouse. This process is very fast and typically completes in a few seconds.

### What You've Just Created

Once created, you will be taken to the Lakehouse Explorer view for `LH_Silver`. It will appear empty, but several powerful components have been set up behind the scenes:

- **An Empty Canvas:** You'll see the familiar `Tables` and `Files` folders, both empty. This is your clean slate, ready to be populated with transformed data.
- **Underlying OneLake Storage:** Fabric has automatically created a folder structure for this Lakehouse within your organization's unified data lake, **OneLake**. You don't need to manage this storage account; Fabric handles it for you.
- **An Automatic SQL Analytics Endpoint:** Just like with the Bronze Lakehouse, a fully functional SQL endpoint has been provisioned. Once you load tables into `LH_Silver`, you'll be able to query them instantly using T-SQL.

Your `LH_Silver` Lakehouse is now successfully created and ready.

[![](https://media.licdn.com/dms/image/v2/D4E0DAQE6GgNBccTfxg/learning-article-inline-scale_1000_2000/learning-article-inline-scale_1000_2000/0/1732562556262?e=1753772400&v=beta&t=s8p3gof2wowPZkfASxPXUGRXLcqJADNoDhrpExVS4zU)](https://media.licdn.com/dms/image/v2/D4E0DAQE6GgNBccTfxg/learning-article-inline-scale_1000_2000/learning-article-inline-scale_1000_2000/0/1732562556262?e=1753772400&v=beta&t=s8p3gof2wowPZkfASxPXUGRXLcqJADNoDhrpExVS4zU)

### Conclusion and Next Steps

You have now successfully provisioned the container for your clean, structured data. This empty Silver Lakehouse serves as the destination for our upcoming data transformation tasks.

**What's next?**

The next logical step is to build the ETL (Extract, Transform, Load) process that:

1. **Extracts** the raw `Customer` data from the `LH_Bronze` Lakehouse.
2. **Transforms** it by applying cleaning rules and business logic (e.g., using a Spark Notebook).
3. **Loads** the resulting clean DataFrame as a new `Customer_Silver` table into the `LH_Silver` Lakehouse you just created.