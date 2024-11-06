# Azure-SQL-Database-Migration-Project
SQL Database Migration from On-Premises to Azure with Near-Zero Downtime

# BookpublisherDistributionDB Migration Project

This project details my complete process for migrating **BookpublisherDistributionDB** from my local machine(SQL Server) to an Azure SQL Database, with each step documented clearly. 

---

## Project Overview

- **Goal**: Migrate **BookpublisherDistributionDB** from an on-premises SQL Server to Azure SQL Database with minimal downtime.
- **Steps**:
  1. Database Assessment with DMA
  2. Azure Environment Setup
  3. Backup Strategy and Data Preparation
  4. Configuring Transaction Log Settings
  5. Migration Execution with Azure Data Migration Service (DMS)

---

## 1. Database Assessment and Compatibility Check Using DMA

The Database Migration Assistant (DMA) tool helps me assess my database for any compatibility or migration issues.

### Step-by-Step Guide:

1. **Download DMA**:
   - Go to [DMA Download Page](https://aka.ms/dma) and install the tool on your local machine.

2. **Launch DMA**:
   - Open DMA and select **New Project**.
   - Name the project `BookpublisherDistributionDB Assessment`.
   - **Project Type**: Choose **Assessment**.
   - **Source Server Type**: Select **SQL Server**.
   - **Target Server Type**: Choose **Azure SQL Database**.

3. **Connect to Source Database**:
   - Click **Connect** and input your on-premises SQL Server details.
   - Choose **Windows Authentication** or **SQL Authentication** as needed.

4. **Add Database for Assessment**:
   - Select **BookpublisherDistributionDB** from the list of databases to include in the assessment.

5. **Run Assessment**:
   - Click **Start Assessment** to analyze compatibility with Azure SQL Database.
   - Review the compatibility report, noting any issues or warnings. Save this report for reference.

---

## 2. Set Up the Azure Environment

I prepared the Azure SQL Database and configured network access for my migration.

### Step-by-Step Guide:

1. **Log in to Azure Portal**:
   - Go to [Azure Portal](https://portal.azure.com/).

2. **Create Resource Group**:
   - In the search bar, type **Resource Groups** and create a new resource group named `sai-db-migration1`.

3. **Create Azure SQL Database**:
   - Go to **Create a Resource** > **Databases** > **SQL Database**.
   - **Database Name**: Enter `BookpublisherDistributionDB`.
   - **Server**: Select **Create New Server**.
     - Set **Server Name** as a unique identifier, e.g., `bookpublisherdistributiondb-migration`.
     - Choose **Region** (e.g., i choose `Central India`).
     - Enable **Allow Azure services and resources to access this server**.

4. **Configure Firewall Settings**:
   - Go to **SQL Server** settings > **Networking** > **Firewall rules**.
   - Add your local IP address for on-premises SQL Server access(make sure it is IPV4).

5. **Review and Create**:
   - Click **Review + Create** and confirm settings. Wait until the deployment completes.

---

## 3. Backup Strategy and Data Preparation

I created full backups of the on-premises database to ensure data integrity and minimize risks during migration.

### Step-by-Step Guide:

1. **Open SQL Server Management Studio (SSMS)**.
   - Connect to your on-premises SQL Server instance.

2. **Full Database Backup**:
   - Right-click `BookpublisherDistributionDB` > **Tasks** > **Back Up...**.
   - In the **Backup Type**, choose **Full**.
   - **Destination**: Choose **Disk** and specify the backup file path (`C:\Backups\BookpublisherDistributionDB_Full.bak`).
   - Click **OK** to start the backup.

3. **Transaction Log Backup**:
   - Go to **Tasks** > **Back Up...** again.
   - Select **Backup Type** as **Transaction Log**.
   - Choose the backup location, e.g., `C:\Backups\BookpublisherDistributionDB_Log.bak`.
   - Run the backup to capture any active transactions.

4. **Verify Backups**:
   - Confirm both backups are present in the specified location.

---

## 4. Configure Transaction Log Settings

Since minimal downtime is essential, I configured transaction logs for incremental backups and set up log shipping for continuous sync.

### Step-by-Step Guide:

1. **Enable Full Recovery Mode**:
   - Right-click `BookpublisherDistributionDB` > **Properties** > **Options**.
   - Set **Recovery Model** to **Full**.
   - Click **OK** to save.

2. **Configure SQL Server Agent** (For Regular Transaction Log Backups):
   - In SSMS, expand **SQL Server Agent** > **Jobs**.
   - Right-click **Jobs** > **New Job** > Name it `LogBackupJob`.
   - **Steps**: Add a new step to run transaction log backups.

     ```sql
     BACKUP LOG BookpublisherDistributionDB 
     TO DISK = 'C:\Backups\BookpublisherDistributionDB_Log.bak' 
     WITH NOINIT;
     ```

3. **Set Job Schedule**:
   - Since SQL Server Express doesn’t support SQL Server Agent jobs, I used Windows Task Scheduler to handle transaction log backups.

4. **Verify Transaction Log Settings**:
   - Confirm the **Full Recovery Mode** and the log backups are running by checking the backup files in `C:\Backups`.

---

## 5. Execute the Migration Using Azure Data Migration Service (DMS)

With the environment and backup strategy set, I configured Azure DMS for an online migration.

### Step-by-Step Guide:

1. **Create Migration Project in DMS**:
   - Go to **Azure Portal** > **Azure Data Migration Service** > **+ New Migration Project**.
   - Name the project `BookpublisherMigration`.
   - Set **Source server type** to **SQL Server** and **Target server type** to **Azure SQL Database**.
   - Choose **Online migration** for minimal downtime.

2. **Source and Target Connection Configuration**:
   - **Source**: Enter on-premises SQL Server details.
   - **Target**: Enter the Azure SQL Database credentials for `BookpublisherDistributionDB`.

3. **Start Migration**:
   - Select **Start Migration** and monitor progress in the DMS dashboard.
   - Wait for DMS to perform the final sync and complete the migration.

4. **Application Cutover**:
   - Update connection strings in any applications to point to the Azure SQL Database.
   - Ensure everything is working in the new environment before decommissioning the old server.

---

## Additional SQL Scripts and Files

### Full Database Backup Script

```sql
BACKUP DATABASE BookpublisherDistributionDB 
TO DISK = 'C:\Backups\BookpublisherDistributionDB_Full.bak';


