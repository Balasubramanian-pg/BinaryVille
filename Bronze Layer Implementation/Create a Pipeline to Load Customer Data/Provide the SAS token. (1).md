You can generate the **SAS (Shared Access Signature) token** from the **Azure Portal** by following the steps below. The steps vary slightly depending on whether you're generating the token for:

- The **storage account** (full access),
- A **container/filesystem**,
- Or a **specific file/folder**.

---

## ✅ How to Generate a SAS Token for ADLS Gen2 (via Azure Portal)

### **Step-by-step (for the storage account):**

1. **Go to the Azure Portal**
    
    [https://portal.azure.com](https://portal.azure.com/)
    
2. **Navigate to your Storage Account**
    - Example: `balasubramanian`
3. In the left menu, scroll to:
    
    **Security + networking → Shared access signature**
    
4. **Configure SAS Permissions:**
    - **Allowed services:** Blob (required for ADLS Gen2)
    - **Allowed resource types:** Service, Container, Object
    - **Allowed permissions:** Read, Write, List, Create, Delete (select based on your use case)
    - **Start/Expiry time:** Set a future expiration time (e.g., 7 days)
    - **Protocol:** HTTPS only
5. Click **Generate SAS and connection string**
6. Copy the **SAS token** (just the query string, starting with `?sv=...`)

---

### 📌 Optional: SAS for a Container / Filesystem Level

1. Go to your storage account
2. Click on **Containers**
3. Click on the specific **container/filesystem** (e.g., `landing`)
4. Click on **Generate SAS** (button at the top bar)
5. Select permissions → Generate

---

## 🔐 Example Usage

If your URL is:

```Plain
https://balasubramanian.dfs.core.windows.net/landing
```

And your SAS token is:

```Plain
?sv=2024-04-01&ss=b&srt=co&sp=rlx&se=2025-07-05T15:00:00Z&st=2025-07-03T09:00:00Z&spr=https&sig=abc123...
```

Then your final full URL becomes:

```Plain
https://balasubramanian.dfs.core.windows.net/landing?<sas_token>
```

Or for Spark:

```Python
abfss://landing@balasubramanian.dfs.core.windows.net/<path>?<sas_token>
```

---

## ❗Security Tip

- **Never hardcode** your SAS token in public code.
- Always **set minimal permissions** and **short expiry** where possible.

---

Let me know if you’re using **Python**, **Power BI**, or **Spark**, and I’ll show you exactly how to pass the token.