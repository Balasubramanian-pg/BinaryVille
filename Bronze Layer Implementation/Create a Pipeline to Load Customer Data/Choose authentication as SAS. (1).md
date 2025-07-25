### What is a **Filesystem** in Azure Data Lake Storage Gen2?

In **ADLS Gen2**, a **filesystem** is the **top-level container** for your data — similar to a **container** in Blob Storage.

---

### 🔁 Mapping: Blob Storage vs. ADLS Gen2

|Blob Storage Term|ADLS Gen2 Term|
|---|---|
|Container|Filesystem|
|Blob|File|
|Directory|Directory|

So if you're used to Blob Storage, just remember:

> Container = Filesystem in ADLS Gen2.

---

### 📌 Where to Find Filesystem Name

You can get it in the Azure portal:

1. Go to your **Storage Account**
2. Go to **Containers** (same UI, but behind the scenes they behave as filesystems when Hierarchical Namespace is ON)
3. The name you see in the list (e.g., `rawdata`, `financefiles`, etc.) is your **filesystem name**

---

### ✅ Example URI Using Filesystem

Assume:

- Storage account: `mystorageaccount`
- Filesystem: `rawdata`
- File path: `sales/2024/data.csv`

Then your ADLS Gen2 path is:

```Plain
https://mystorageaccount.dfs.core.windows.net/rawdata/sales/2024/data.csv
```

Or in **Spark/Power BI**:

```Plain
abfss://rawdata@mystorageaccount.dfs.core.windows.net/sales/2024/data.csv
```

---

If you tell me your storage account name and container name, I’ll format the exact `dfs.core` and `abfss://` URL for you.