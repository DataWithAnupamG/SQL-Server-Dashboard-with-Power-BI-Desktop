# How to Build a SQL Server Dashboard with Power BI Desktop

This guide walks through creating a real-time dashboard in Power BI Desktop that visualizes key metrics from your SQL Server instance. It's based on the original blog by Dan English, adapted and updated for modern usage and customization.

---

## Prerequisites

* Power BI Desktop installed
* Access to a SQL Server instance (Developer or higher edition)
* A SQL login with necessary permissions (VIEW SERVER STATE, access to `msdb`)

---

## Step 1: Connect to SQL Server in Power BI

1. Open **Power BI Desktop**.
2. Go to **Home > Get Data > SQL Server**.
3. Fill in:

   * **Server:** `YOUR_SERVER_NAME` (e.g., `ANUPAM\MSSQLSERVER01`)
   * **Database:** leave blank or enter `master`
4. Expand **Advanced options**.
5. Paste the following T-SQL query:

```sql
SELECT
    @@servername AS ServerName,
    @@version AS SQLVersion,
    sdb.name AS DatabaseName,
    sdb.create_date AS CreateDateTime,
    sdb.compatibility_level AS CompatibilityLevel,
    sdb.collation_name AS CollationName,
    sdb.recovery_model_desc AS RecoveryModel,
    MAX(bus.backup_finish_date) AS LastBackUpTime,
    size.SizeinMB AS DatabaseSize,
    CASE
        WHEN sdb.database_id < 5 THEN 'System'
        ELSE 'User'
    END AS DatabaseType
FROM sys.databases sdb
LEFT OUTER JOIN msdb.dbo.backupset bus
    ON bus.database_name = sdb.name
INNER JOIN (
    SELECT
        DB_NAME(database_id) AS DatabaseName,
        CAST(SUM(size) * 8 / 1024.0 AS DECIMAL(18, 2)) AS SizeinMB
    FROM sys.master_files
    GROUP BY DB_NAME(database_id)
) size ON sdb.name = size.DatabaseName
GROUP BY
    sdb.name, sdb.create_date, sdb.compatibility_level,
    sdb.collation_name, sdb.recovery_model_desc, size.SizeinMB, sdb.database_id;
```

6. Choose **SQL Server Authentication**, and enter your login.
7. Click **OK** and load the data.

---

## Step 2: Transform and Split Data

1. Go to **Transform Data**.
2. In Power Query Editor, right-click the `SQLVersion` column.
3. Select **Split Column > By Delimiter**.
4. Use custom delimiters such as `-`, `(`, `)` or keywords like `on`.
5. Alternatively, add **Custom Columns** using formulas like:

```m
Text.BetweenDelimiters([SQLVersion], "-", "(") // Gets version number
Text.BetweenDelimiters([SQLVersion], ")", "on") // Gets edition
Text.AfterDelimiter([SQLVersion], "on ") // Gets OS info
```

6. Rename and set appropriate data types.
7. Click **Close & Apply**.

---

## Step 3: Design Your Dashboard

1. Add a **SQL Server logo**:

   * Go to **Insert > Image**.
   * Select a PNG file (e.g. official SQL Server logo).

2. Add **Text Boxes** to display:

   * Server Name
   * SQL Version
   * Edition / OS info

3. Add **Visuals**:

   * **Table or Matrix**: Show database name, size, recovery model, last backup
   * **Bar Chart**: Show database sizes by name
   * **Slicer**: Filter by database type (System/User)

---

## Step 4: Publish to Power BI Service (Optional)

1. Click **Publish** from the ribbon.
2. Choose your workspace on PowerBI.com.
3. Open the report in PowerBI Service.
4. Pin visuals to a new **Dashboard**.

To enable refresh:

* Install **Power BI Gateway**.
* Configure scheduled refresh in Power BI settings.

---

## Permissions Note

Your SQL login must have:

```sql
GRANT VIEW SERVER STATE TO [YourLogin];
USE msdb;
GRANT CONNECT TO [YourLogin];
```

---

## Sample Screenshot

![Dashboard Example](https://denglishbi.files.wordpress.com/2015/08/powerbi_sqldashboard.jpg)

---

## Author

Anupam | [GitHub Profile](https://github.com/anupam-yourhandle)

Feel free to fork this and modify it for your own SQL monitoring needs!

---

**Last updated:** May 2025
