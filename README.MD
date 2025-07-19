# 🖼️ Azure Durable Functions Image Metadata Pipeline

[![Azure Functions](https://img.shields.io/badge/Azure-Functions-blue.svg)](https://azure.microsoft.com/services/functions/)
[![Python](https://img.shields.io/badge/Python-3.9+-yellow.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**CST8917 Assignment 1** - A serverless, event-driven image processing pipeline that automatically extracts metadata from uploaded images and stores it in Azure SQL Database.

---

## 🚀 **Quick Start**

1. **Upload** an image (`.jpg`, `.png`, `.gif`) to the Azure Blob Storage container
2. **Watch** as the pipeline automatically processes it
3. **View** the extracted metadata in the Azure SQL Database

## 📋 **Project Overview**

### What it does
This solution creates a fully automated image metadata extraction pipeline using Azure's serverless architecture. When images are uploaded to blob storage, the system automatically:

- ✅ Detects new image uploads via blob triggers
- ✅ Extracts comprehensive metadata (dimensions, size, format)
- ✅ Stores structured data in Azure SQL Database
- ✅ Provides real-time processing with zero server management

### Architecture
```
📁 Blob Storage (images-input) 
    ↓ (trigger)
🔄 Durable Function Orchestrator
    ├── 📊 extract_metadata (Activity)
    └── 💾 store_metadata (Activity)
         ↓
🗄️ Azure SQL Database (ImageMetadata table)
```

---

## 🛠️ **Technical Implementation**

### Core Functions
| Function | Type | Purpose |
|----------|------|---------|
| `image_blob_trigger` | Blob Trigger | Initiates workflow when images are uploaded |
| `image_processing_orchestrator` | Orchestrator | Coordinates the metadata extraction pipeline |
| `extract_metadata` | Activity | Analyzes image properties (size, dimensions, format) |
| `store_metadata` | Activity | Persists metadata to Azure SQL Database |

### Database Schema
```sql
CREATE TABLE ImageMetadata (
    id INT IDENTITY(1,1) PRIMARY KEY,
    file_name NVARCHAR(255),
    file_size_kb DECIMAL(10,2),
    width INT,
    height INT,
    format NVARCHAR(10),
    created_at DATETIME DEFAULT GETDATE()
);
```

---

## 🔗 **Project Resources**

### 📂 Repository & Documentation
- **Source Code**: [`function_app.py`](https://github.com/mishsamir/assignment-submission.git)
- **Demo Video**: [🎥 Watch Pipeline in Action]()
- **ChatGPT Session**: [Development Assistance](https://chatgpt.com/c/686f7383-b690-8001-b753-8d933822e2b6)

### 🔧 Azure Resources
- **Storage Container**: [images-input](https://portal.azure.com/#view/Microsoft_Azure_Storage/ContainerMenuBlade/~/overview/storageAccountId/%2Fsubscriptions%2Fcdb9bdf3-e7ee-43e9-8f6c-ba7327868df1%2FresourceGroups%2FCST8917-rg%2Fproviders%2FMicrosoft.Storage%2FstorageAccounts%2Fcst8917storagexxx/path/images-input/etag/%220x8DDBF8945DA7D4D%22/defaultId//publicAccessVal/None)
- **SQL Database**: [cst8917db Query Interface](https://portal.azure.com/#@AlgonquinLivecom.onmicrosoft.com/resource/subscriptions/cdb9bdf3-e7ee-43e9-8f6c-ba7327868df1/resourceGroups/CST8917-rg/providers/Microsoft.Sql/servers/cst8917sql134/databases/cst8917db/connectionStrings)

---

## ⚙️ **Setup & Deployment**

### Prerequisites
- Azure CLI installed and authenticated
- Python 3.9+ with Azure Functions Core Tools
- Azure subscription with appropriate permissions

### Infrastructure Deployment
Execute these Azure CLI commands to provision the required resources:

#### 1. **Resource Group**
```bash
az group create \
  --name cst8917-rg \
  --location canadacentral
```

#### 2. **Storage Account & Container**
```bash
# Create storage account
az storage account create \
  --name cst8917storagexxx \
  --resource-group cst8917-rg \
  --location canadacentral \
  --sku Standard_LRS

# Create blob container for image uploads
az storage container create \
  --name images-input \
  --account-name cst8917storagexxx
```

#### 3. **Azure SQL Database**
```bash
# Create SQL Server
az sql server create \
  --name cst8917sql134 \
  --resource-group cst8917-rg \
  --location canadacentral \
  --admin-user sqladminuser \
  --admin-password StrongP@ssword123!

# Create database
az sql db create \
  --resource-group cst8917-rg \
  --server cst8917sql134 \
  --name cst8917db \
  --service-objective S0

# Configure firewall for Azure services
az sql server firewall-rule create \
  --resource-group cst8917-rg \
  --server cst8917sql134 \
  --name AllowAzureIPs \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

### Local Development
Create `local.settings.json` with your connection strings:
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "your_storage_connection_string",
    "AZURE_SQL_CONNECTION_STRING": "your_sql_connection_string",
    "FUNCTIONS_WORKER_RUNTIME": "python"
  }
}
```

---

## 🧪 **Testing the Pipeline**

### Supported Image Formats
- **JPEG** (`.jpg`, `.jpeg`)
- **PNG** (`.png`)
- **GIF** (`.gif`)

### Verification Steps
1. **Upload Test Images**: Drop sample images into the `images-input` container
2. **Monitor Function Logs**: Check Azure Functions logs for execution details
3. **Query Database**: Use the provided SQL queries to verify stored metadata

### Database Queries
```sql
-- View all processed images
SELECT * FROM ImageMetadata ORDER BY created_at DESC;

-- Clean up test data
TRUNCATE TABLE ImageMetadata;

-- Check processing statistics
SELECT 
    format,
    COUNT(*) as image_count,
    AVG(file_size_kb) as avg_size_kb,
    AVG(width * height) as avg_pixels
FROM ImageMetadata 
GROUP BY format;
```

---

## 📊 **Database Connection**

**Server**: `cst8917sql134.database.windows.net`  
**Database**: `cst8917db`  
**Username**: `sqladminuser`  
**Password**: `StrongP@ssword123!`

> ⚠️ **Security Note**: In production, use Azure Key Vault for credential management and implement proper access controls.

---

## 🔍 **Monitoring & Troubleshooting**

### Key Metrics to Monitor
- Function execution duration
- Blob trigger latency
- Database connection health
- Error rates and retry attempts

### Common Issues
- **Firewall blocking**: Ensure "Allow Azure services" is enabled
- **Connection timeouts**: Check network connectivity and connection string format
- **Unsupported formats**: Verify image file extensions match supported types

---

## 🏗️ **Architecture Benefits**

- **🔄 Event-Driven**: Automatic processing with zero manual intervention
- **📈 Scalable**: Handles varying workloads automatically
- **💰 Cost-Effective**: Pay-per-execution serverless model
- **🛡️ Reliable**: Built-in retry mechanisms and error handling
- **🔧 Maintainable**: Clean separation of concerns with activity functions

---

## 📚 **Additional Resources**

- [Azure Durable Functions Documentation](https://docs.microsoft.com/azure/azure-functions/durable/)
- [Python Azure Functions Guide](https://docs.microsoft.com/azure/azure-functions/functions-reference-python)
- [Blob Storage Triggers](https://docs.microsoft.com/azure/azure-functions/functions-bindings-storage-blob-trigger)
