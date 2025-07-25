Here you go - **all 3 resources**, crafted for professional deployment in a modern lakehouse setup using Azure Storage with hierarchical namespace (ADLS Gen2):

---

## **Automation Script to Create Container (Filesystem)**

You can use Azure CLI, ARM, or Bicep. Here’s the **Azure CLI version** — quick and scriptable:

### **Azure CLI (Run in Azure Cloud Shell or local CLI)**

```Shell
# Variables — customize as needed
STORAGE_ACCOUNT="ouff"
RESOURCE_GROUP="Binaryville"
CONTAINER_NAME="landing"

# Create a container (ADLS Gen2 filesystem)
az storage container create \
  --account-name $STORAGE_ACCOUNT \
  --name $CONTAINER_NAME \
  --auth-mode login \
  --public-access off
```

> ⚠️ Requires that you're logged in with az login or using a Service Principal with sufficient RBAC.

If you want ARM or Bicep template as a JSON/YAML file, I can provide that too.

---