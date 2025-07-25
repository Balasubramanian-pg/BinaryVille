## 2. **Spark Code to Read/Write with abfss:// (SAS Token Auth)**

### **Spark Read/Write Example with SAS Token**

Assuming you already created a file system named `landing`, and uploaded a file at `landing/raw/hotel_reviews_dataset.json`:

```Python
# Define SAS token and config
spark.conf.set(
  "fs.azure.account.auth.type.ouff.dfs.core.windows.net",
  "SAS"
)
spark.conf.set(
  "fs.azure.sas.token.provider.type.ouff.dfs.core.windows.net",
  "org.apache.hadoop.fs.azurebfs.sas.FixedSASTokenProvider"
)
spark.conf.set(
  "fs.azure.sas.fixed.token.ouff.dfs.core.windows.net",
  "<PASTE_YOUR_SAS_TOKEN_HERE_WITHOUT_QUESTION_MARK>"
)

# Read JSON file
df = spark.read.json("abfss://landing@ouff.dfs.core.windows.net/raw/hotel_reviews_dataset.json")

# Show schema or data
df.printSchema()
df.show()
```

### **Write Example**

```Python
df.write.mode("overwrite").parquet("abfss://landing@ouff.dfs.core.windows.net/processed/hotels")
```

---