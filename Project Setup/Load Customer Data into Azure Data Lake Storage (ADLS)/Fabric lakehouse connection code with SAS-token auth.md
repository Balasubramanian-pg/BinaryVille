## 3. **Power BI Fabric Lakehouse or Dataflow Connection (via SAS)**

If you're using **Power BI or Fabric Dataflow Gen2**, here’s how to access your ADLS Gen2 container:

### **Power BI → Data Lake Connector (DFS Endpoint)**

1. Go to **Power BI Desktop** → **Get Data** → **Azure Data Lake Gen2**
2. Use the following **DFS URL**:
    
    ```Plain
    https://ouff.dfs.core.windows.net/landing
    ```
    
3. When prompted for auth:
    - Choose **Shared Access Signature (SAS)**
    - Paste only the SAS token (starts with `?sv=...` or just the part after `?`)
4. You’ll now be able to browse the file system just like folders.

---

## 🔐 Security Best Practices

- **Never expose full SAS in public notebooks or GitHub**
- Store the token in **Key Vault**, or use **Managed Identity** for production workloads
- Rotate SAS tokens regularly (set expiry ≤ 7 days if possible)

---

Would you like a fully integrated **deployment template** (e.g., Bicep or ARM) that provisions:

- ADLS Gen2 with correct config
- Creates filesystem
- Uploads a sample file
- Outputs the abfss path and a working notebook?

Let me know — I’ll build and drop the script for you.