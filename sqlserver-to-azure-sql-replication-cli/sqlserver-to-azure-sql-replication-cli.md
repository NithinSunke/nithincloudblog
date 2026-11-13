# Transactional Replication: SQL Server VM to Azure SQL Database — CLI Runbook

Nov 5, 2026 · @Nithin Sunke

## Overview

The SQL Server VM is both **publisher** and **distributor**, and Azure SQL Database is a **push subscriber**. Three SQL Agent jobs on the VM move the data: the Snapshot Agent makes the initial copy, the Log Reader Agent reads committed changes from the transaction log, and the Distribution Agent pushes both to Azure over port 1433.

Azure SQL Database can only be a subscriber. It cannot publish or distribute, and it only accepts **push** subscriptions.

**Tools used**

- **Azure CLI (`az`)** for the Azure resources, run from your laptop or Cloud Shell.
- **sqlcmd** on the VM to run every T-SQL script, for example `sqlcmd -S localhost -E -C -i 03_distribution.sql`. Here `-E` means Windows auth and `-C` trusts the local server certificate.
- **PowerShell** on the VM for service and network checks.

**Lab values used throughout** (replace with your own)

| Placeholder | Example | Meaning |
| --- | --- | --- |
| `LabDB` | LabDB | Source database on the VM, and the target database name in Azure |
| `Pub_LabDB` | Pub\_LabDB | Publication name |
| `<srv>` | sqlrepl-lab-01 | Azure SQL logical server name (globally unique) |
| `<srv>.database.windows.net` | sqlrepl-lab-01.database.windows.net | Subscriber name used in replication |
| `sqladmin` / `<SqlPwd>` | sqladmin | Azure SQL admin login (SQL auth) |
| `<AgentAccount>` / `<AgentPwd>` | .\\repladmin | Windows account the replication agents run as on the VM |
| `C:\ReplData` | C:\\ReplData | Snapshot folder on the VM |

## Step 1 — Create the Azure side (Azure CLI)

This step creates the subscriber: a logical server, a firewall rule that lets the VM in, and an **empty** target database. The snapshot will create the tables, so don't create any schema here.

```bash
# 1. Sign in and pick the subscription
az login
az account set --subscription "<subscription-name-or-id>"

# 2. Variables
RG=rg-repl-lab
LOC=centralindia
SRV=sqlrepl-lab-01            # must be globally unique
DB=LabDB
ADMIN=sqladmin
SQLPWD='<StrongPassword>'
VM_IP=<public-IP-of-the-VM>

# 3. Resource group
az group create --name $RG --location $LOC

# 4. Logical SQL server (holds logins and firewall rules)
az sql server create --resource-group $RG --name $SRV --location $LOC \
  --admin-user $ADMIN --admin-password "$SQLPWD"

# 5. Firewall rule so the VM can reach the server on 1433
az sql server firewall-rule create --resource-group $RG --server $SRV \
  --name AllowReplVM --start-ip-address $VM_IP --end-ip-address $VM_IP

# 6. Empty target database (small tier is fine for a lab)
az sql db create --resource-group $RG --server $SRV --name $DB \
  --service-objective S0

# 7. Confirm
az sql db show --resource-group $RG --server $SRV --name $DB \
  --query "{name:name, status:status, sku:currentSku.name}" -o table
```

**What each part does**

- **Steps 3–4:** the logical server is the endpoint `<srv>.database.windows.net`. The admin login you set here is the SQL-auth login the Distribution Agent will use.
- **Step 5:** Azure SQL blocks all traffic by default. If the VM is in Azure, get its outbound IP with `az vm show -d -g <vm-rg> -n <vm-name> --query publicIps -o tsv`. If it reaches the internet through a NAT gateway, use the NAT gateway's IP instead.
- **Step 6:** S0 is enough for a lab. For a real migration, size the database for the snapshot load, which is the heaviest part.
- **Optional, for Azure VMs only:** connections from inside Azure use *Redirect* mode by default, which also needs outbound ports 11000–11999. In a lab, the simplest fix is to force Proxy mode with `az sql server conn-policy update -g $RG -s $SRV --connection-type Proxy`.

## Step 2 — Check the VM prerequisites

Fix everything in this step before configuring replication. Most replication failures come from one of these checks.

**2a. SQL Server Agent running, snapshot folder ready** (PowerShell as Administrator)

```powershell
# Default instance = SQLSERVERAGENT; named instance = SQLAgent$<InstanceName>
Get-Service SQLSERVERAGENT
Set-Service SQLSERVERAGENT -StartupType Automatic
Start-Service SQLSERVERAGENT

# Snapshot folder; the agent account needs Modify rights on it
New-Item -ItemType Directory -Path C:\ReplData -Force
icacls C:\ReplData /grant "<AgentAccount>:(OI)(CI)M"

# Can the VM reach Azure SQL on 1433?
Test-NetConnection <srv>.database.windows.net -Port 1433
```

The replication agents run as SQL Agent jobs, so the Agent must be running. `TcpTestSucceeded : True` confirms the firewall rule from Step 1 works. The **Replication** feature must also be installed with SQL Server; if it isn't, add it from SQL Server Setup.

**2b. Version and server name** (`sqlcmd -S localhost -E -C -i 02_checks.sql`)

```sql
-- Version: SQL Server 2016 or later is the safe baseline for an Azure SQL Database subscriber
SELECT SERVERPROPERTY('ProductVersion') AS version,
       SERVERPROPERTY('Edition')        AS edition;

-- These two must match, or distribution setup fails
SELECT @@SERVERNAME AS at_servername,
       SERVERPROPERTY('ServerName') AS actual_name;
```

A mismatch is common on VMs that were cloned or renamed. To fix it, run the following, then restart the SQL Server service:

```sql
EXEC sp_dropserver '<old_name>';
EXEC sp_addserver  '<actual_name>', 'local';
```

**2c. Every table to publish needs a primary key**

```sql
USE LabDB;
SELECT s.name AS schema_name, t.name AS table_name
FROM sys.tables t
JOIN sys.schemas s ON s.schema_id = t.schema_id
WHERE t.is_ms_shipped = 0
  AND NOT EXISTS (SELECT 1 FROM sys.indexes i
                  WHERE i.object_id = t.object_id AND i.is_primary_key = 1);
```

Any table listed here must either get a primary key or be left out of the publication.

**2d. Test the SQL-auth login to the subscriber**

```powershell
sqlcmd -S tcp:<srv>.database.windows.net,1433 -d LabDB -U sqladmin -P "<SqlPwd>" -Q "SELECT DB_NAME(), @@VERSION"
```

This is the same connection the Distribution Agent will make. If it fails here, replication will fail too.

## Step 3 — Configure distribution (T-SQL on the VM)

This step makes the VM its own distributor. It creates the `distribution` database, which stores replicated commands until they are delivered, and registers the VM as a publisher that uses it.

Run it with `sqlcmd -S localhost -E -C -i 03_distribution.sql`:

```sql
USE master;
GO
-- 1. Make this instance its own distributor
DECLARE @srv sysname = @@SERVERNAME;
EXEC sp_adddistributor
     @distributor = @srv,
     @password    = N'<DistributorAdminPwd>';
GO

-- 2. Create the distribution database
EXEC sp_adddistributiondb
     @database          = N'distribution',
     @security_mode     = 1,      -- Windows auth to the distributor
     @min_distretention = 0,
     @max_distretention = 72,     -- hours to keep delivered commands
     @history_retention = 48;
GO

-- 3. Register this instance as a publisher using that distributor
DECLARE @srv sysname = @@SERVERNAME;
EXEC sp_adddistpublisher
     @publisher         = @srv,
     @distribution_db   = N'distribution',
     @security_mode     = 1,
     @working_directory = N'C:\ReplData';   -- snapshot folder
GO

-- 4. Verify
EXEC sp_get_distributor;
EXEC sp_helpdistpublisher;
```

**What each call does**

- **`sp_adddistributor`** marks the instance as a distributor. The password belongs to the internal `distributor_admin` link, which matters only when publisher and distributor are on different servers.
- **`sp_adddistributiondb`** creates the `distribution` database and its cleanup jobs. Retention controls how long commands are kept, in hours.
- **`sp_adddistpublisher`** lets this instance publish through the distributor. `@working_directory` is where the Snapshot Agent writes files. For a push subscription, the Distribution Agent reads them locally on the VM.
- **Verify:** `sp_get_distributor` should show `installed = 1`. `sp_helpdistpublisher` should list your server with `C:\ReplData`.

## Step 4 — Create the publication (T-SQL on the VM)

This step enables `LabDB` for publishing, creates the Log Reader and Snapshot agents, defines the publication, and adds the tables as articles.

Run it with `sqlcmd -S localhost -E -C -i 04_publication.sql`:

```sql
-- 1. Enable the database for transactional publishing
USE master;
EXEC sp_replicationdboption
     @dbname = N'LabDB', @optname = N'publish', @value = N'true';
GO

USE LabDB;
GO
-- 2. Log Reader Agent (one per published database)
EXEC sp_addlogreader_agent
     @job_login    = N'<AgentAccount>',
     @job_password = N'<AgentPwd>',
     @publisher_security_mode = 1;
GO

-- 3. The publication
EXEC sp_addpublication
     @publication       = N'Pub_LabDB',
     @description       = N'LabDB to Azure SQL Database',
     @status            = N'active',
     @repl_freq         = N'continuous',  -- transactional
     @sync_method       = N'concurrent',  -- no long table locks during snapshot
     @allow_push        = N'true',
     @allow_pull        = N'false',
     @independent_agent = N'true',
     @immediate_sync    = N'true';
GO

-- 4. Snapshot Agent for this publication (run on demand)
EXEC sp_addpublication_snapshot
     @publication    = N'Pub_LabDB',
     @frequency_type = 1,                -- once / on demand
     @job_login      = N'<AgentAccount>',
     @job_password   = N'<AgentPwd>',
     @publisher_security_mode = 1;
GO

-- 5. Add every user table that has a primary key as an article
DECLARE @sch sysname, @tbl sysname;
DECLARE c CURSOR LOCAL FAST_FORWARD FOR
  SELECT s.name, t.name
  FROM sys.tables t
  JOIN sys.schemas s ON s.schema_id = t.schema_id
  WHERE t.is_ms_shipped = 0
    AND EXISTS (SELECT 1 FROM sys.indexes i
                WHERE i.object_id = t.object_id AND i.is_primary_key = 1);
OPEN c;
FETCH NEXT FROM c INTO @sch, @tbl;
WHILE @@FETCH_STATUS = 0
BEGIN
  DECLARE @art sysname = @sch + N'_' + @tbl;
  EXEC sp_addarticle
       @publication       = N'Pub_LabDB',
       @article           = @art,
       @source_owner      = @sch,
       @source_object     = @tbl,
       @type              = N'logbased',
       @destination_owner = @sch,
       @destination_table = @tbl,
       @pre_creation_cmd  = N'drop';    -- recreate table at subscriber
  FETCH NEXT FROM c INTO @sch, @tbl;
END
CLOSE c; DEALLOCATE c;
GO

-- 6. Verify
EXEC sp_helppublication @publication = N'Pub_LabDB';
EXEC sp_helparticle     @publication = N'Pub_LabDB';
```

**What each part does**

- **`sp_replicationdboption`** flags the database so the Log Reader can read its transaction log.
- **`sp_addlogreader_agent`** creates the job that reads committed transactions from the log and writes them to the `distribution` database. `<AgentAccount>` is the Windows account the job runs as. It needs `db_owner` in `LabDB` and `distribution`, plus rights on `C:\ReplData`.
- **`sp_addpublication`** defines what gets published and how:
  - `continuous` means transactional replication.
  - `concurrent` takes the snapshot without holding table locks for its whole duration.
  - `allow_push` is required, because Azure SQL Database supports only push subscriptions.
- **`sp_addpublication_snapshot`** creates the Snapshot Agent job. It runs on demand, which is all a one-time migration needs.
- **`sp_addarticle`** adds each table. `logbased` means changes come from the log. `@pre_creation_cmd = 'drop'` lets the snapshot drop and recreate the table at the subscriber. The loop skips tables without a primary key, because they can't be published.
- **Objects other than tables** (views, procedures, functions) can be added as articles with `@type = N'view schema only'` or `N'proc schema only'`. You can also deploy them to Azure separately with a script or DACPAC.

## Step 5 — Create the push subscription to Azure SQL Database

This step registers Azure SQL Database as the subscriber, creates the Distribution Agent that pushes data there, and generates the initial snapshot.

Run it with `sqlcmd -S localhost -E -C -i 05_subscription.sql`:

```sql
USE LabDB;
GO
-- 1. Register the subscription (push, all articles)
EXEC sp_addsubscription
     @publication       = N'Pub_LabDB',
     @subscriber        = N'<srv>.database.windows.net',
     @destination_db    = N'LabDB',
     @subscription_type = N'Push',
     @sync_type         = N'automatic',   -- initialize from snapshot
     @article           = N'all',
     @update_mode       = N'read only';
GO

-- 2. Distribution Agent: runs on the VM, connects to Azure with SQL auth
EXEC sp_addpushsubscription_agent
     @publication              = N'Pub_LabDB',
     @subscriber               = N'<srv>.database.windows.net',
     @subscriber_db            = N'LabDB',
     @job_login                = N'<AgentAccount>',
     @job_password             = N'<AgentPwd>',
     @subscriber_security_mode = 0,          -- 0 = SQL Server authentication
     @subscriber_login         = N'sqladmin',
     @subscriber_password      = N'<SqlPwd>',
     @frequency_type           = 64;         -- run continuously
GO

-- 3. Generate the initial snapshot
EXEC sp_startpublication_snapshot @publication = N'Pub_LabDB';
GO

-- 4. Verify
EXEC sp_helpsubscription @publication = N'Pub_LabDB';
```

**What each call does**

- **`sp_addsubscription`** records the Azure database as a subscriber to all articles. `@sync_type = 'automatic'` means the subscriber is initialized from the snapshot, so the tables are created and loaded for you.
- **`sp_addpushsubscription_agent`** creates the Distribution Agent job on the VM:
  - **`@subscriber_security_mode = 0` is the key setting.** Azure SQL Database accepts only SQL authentication from the agent, so this must not be 1.
  - `@job_login` is the Windows account the job runs as on the VM. `@subscriber_login` is what it uses to sign in to Azure.
  - `@frequency_type = 64` keeps the agent running, so changes flow within seconds.
- **`sp_startpublication_snapshot`** runs the Snapshot Agent. It writes schema scripts and BCP data files to `C:\ReplData`. The Distribution Agent then applies them to Azure and starts delivering ongoing changes.

You can also start the agent jobs from the command line with `sqlcmd -S localhost -E -C -Q "EXEC msdb.dbo.sp_start_job @job_name = N'<job name>'"`. Get the job names from `SELECT name FROM msdb.dbo.sysjobs WHERE category_id IN (10,13,15)`, which lists the distribution, log reader and snapshot jobs.

## Step 6 — Monitor the agents and latency

Watch the snapshot finish, then confirm ongoing changes are flowing. Everything below runs on the VM.

**6a. Agent status and latest messages**

```sql
USE distribution;
-- Snapshot Agent: look for "A snapshot of N article(s) was generated"
SELECT TOP 5 time, comments FROM MSsnapshot_history ORDER BY time DESC;

-- Log Reader Agent: transactions read from the log
SELECT TOP 5 time, comments, delivered_commands FROM MSlogreader_history ORDER BY time DESC;

-- Distribution Agent: what has been delivered to Azure
SELECT TOP 5 time, comments, delivered_transactions, delivered_commands
FROM MSdistribution_history ORDER BY time DESC;

-- Any replication errors
SELECT TOP 10 time, error_code, error_text FROM MSrepl_errors ORDER BY time DESC;
```

The snapshot is fully applied when the Distribution Agent history shows *Delivered snapshot from the …* followed by *No replicated transactions are available*.

**6b. Commands still waiting for delivery**

```sql
USE distribution;
EXEC sp_replmonitorsubscriptionpendingcmds
     @publisher         = N'<VM server name>',   -- value of @@SERVERNAME
     @publisher_db      = N'LabDB',
     @publication       = N'Pub_LabDB',
     @subscriber        = N'<srv>.database.windows.net',
     @subscriber_db     = N'LabDB',
     @subscription_type = 0;                      -- 0 = push
```

This returns `pendingcmdcount` and an estimated time to catch up. It should drop to 0 after the snapshot is applied.

**6c. Tracer tokens: end-to-end latency**

```sql
USE LabDB;
-- Post a token into the log
DECLARE @id int;
EXEC sp_posttracertoken @publication = N'Pub_LabDB', @tracer_token_id = @id OUTPUT;
SELECT @id AS tracer_id;
GO
-- A few seconds later: list tokens, then read one token's timings
EXEC sp_helptracertokens @publication = N'Pub_LabDB';
EXEC sp_helptracertokenhistory @publication = N'Pub_LabDB', @tracer_id = <tracer_id>;
```

A tracer token is a marker written into the log. Its history shows how long it took to get from the publisher to the distributor and from the distributor to Azure. When `subscriber_latency` has a value, everything committed before the token has reached Azure. You'll use this again at cutover.

**6d. Agent jobs at a glance**

```sql
SELECT j.name, ja.start_execution_date, ja.stop_execution_date
FROM msdb.dbo.sysjobs j
JOIN msdb.dbo.sysjobactivity ja ON ja.job_id = j.job_id
WHERE j.category_id IN (10,13,15)
  AND ja.session_id = (SELECT MAX(session_id) FROM msdb.dbo.syssessions);
```

The Log Reader and Distribution jobs should show a start time with no stop time, which means they are running continuously.

## Step 7 — Validate the data

Run the same row-count query on both sides and compare, then prove that live changes arrive.

**7a. Row counts per table** (save as `07_rowcounts.sql`)

```sql
SELECT s.name + '.' + t.name AS table_name,
       SUM(p.rows)            AS row_count
FROM sys.tables t
JOIN sys.schemas s    ON s.schema_id = t.schema_id
JOIN sys.partitions p ON p.object_id = t.object_id AND p.index_id IN (0,1)
WHERE t.is_ms_shipped = 0
GROUP BY s.name, t.name
ORDER BY table_name;
```

```powershell
# Source (VM)
sqlcmd -S localhost -E -C -d LabDB -i 07_rowcounts.sql -o src_counts.txt
# Target (Azure SQL Database)
sqlcmd -S tcp:<srv>.database.windows.net,1433 -d LabDB -U sqladmin -P "<SqlPwd>" -i 07_rowcounts.sql -o tgt_counts.txt
# Compare: no output means the counts match
Compare-Object (Get-Content src_counts.txt) (Get-Content tgt_counts.txt)
```

The counts come from `sys.partitions`, so they are fast but approximate on a busy source. Compare them while writes are quiet, or use `COUNT(*)` on the key tables.

**7b. Live-change test**

```sql
-- On the VM
USE LabDB;
INSERT INTO dbo.<SomeTable> (<cols>) VALUES (<values>);
UPDATE dbo.<SomeTable> SET <col> = <new value> WHERE <key> = <id>;
```

```powershell
# A few seconds later, on Azure
sqlcmd -S tcp:<srv>.database.windows.net,1433 -d LabDB -U sqladmin -P "<SqlPwd>" `
  -Q "SELECT * FROM dbo.<SomeTable> WHERE <key> = <id>"
```

If the new and updated rows appear, the Log Reader and Distribution agents are working end to end.

**7c. Built-in row-count validation (optional)**

```sql
USE LabDB;
EXEC sp_publication_validation @publication = N'Pub_LabDB', @rowcount_only = 1;
```

This asks the Distribution Agent to compare row counts at the subscriber. The pass or fail result for each article appears in `MSdistribution_history` (Step 6a).

## Step 8 — Cut over to Azure SQL Database

Downtime equals the time from stopping writes to repointing the app, usually a few minutes. Replication has already copied everything else.

**8a. Stop writes on the source**

Stop the application, or block new writes at the database level:

```sql
USE master;
ALTER DATABASE LabDB SET READ_ONLY WITH ROLLBACK IMMEDIATE;
```

The Log Reader can still read changes that were committed before the switch.

**8b. Confirm nothing is left in flight**

Post a tracer token (Step 6c) and wait for `subscriber_latency` to show a value. Then confirm the pending command count from Step 6b is 0. Finally, re-run the row counts from Step 7a; they should match exactly now.

**8c. Create the database users in Azure** (replication doesn't copy logins or users)

```sql
-- Run on Azure SQL Database (LabDB) as sqladmin
CREATE USER app_user WITH PASSWORD = N'<AppPwd>';   -- contained user, no server login needed
ALTER ROLE db_datareader ADD MEMBER app_user;
ALTER ROLE db_datawriter ADD MEMBER app_user;
-- add any GRANT EXECUTE / custom roles the app had on the source
```

**8d. Fix identity seeds**

```sql
-- On Azure: check each table that has an identity column
DBCC CHECKIDENT ('dbo.<TableName>', NORESEED);
-- If the current identity value is lower than MAX(id), reseed to the max
DBCC CHECKIDENT ('dbo.<TableName>', RESEED);
```

Rows arrive at the subscriber with their original ID values. If the target's identity seed lags behind, the first insert from the app fails with a duplicate key error.

**8e. Other objects to check**

- **Not replicated as table articles:** views, procedures, functions and triggers. Deploy any that you didn't add as schema-only articles.
- **Not supported in Azure SQL Database:** SQL Agent jobs and cross-database queries. Move jobs to Azure Automation, Elastic Jobs or the app itself.
- **Sequences:** set each sequence to restart above its current value with `ALTER SEQUENCE ... RESTART WITH`.

**8f. Repoint the application**

```text
Server=tcp:<srv>.database.windows.net,1433;Database=LabDB;User ID=app_user;Password=<AppPwd>;Encrypt=True;TrustServerCertificate=False;
```

Start the app against Azure and test a write. Keep the VM database read-only until you're sure you won't roll back.

## Step 9 — Clean up

Remove replication in reverse order: subscription, then publication, then distribution. Delete the Azure resources too if this was only a lab.

**9a. Remove replication on the VM**

```sql
-- Dropping the publication writes to LabDB, so make it writable again first
USE master;
ALTER DATABASE LabDB SET READ_WRITE WITH ROLLBACK IMMEDIATE;
GO

USE LabDB;
GO
-- 1. Drop the subscription (also removes the Distribution Agent job)
EXEC sp_dropsubscription
     @publication    = N'Pub_LabDB',
     @article        = N'all',
     @subscriber     = N'<srv>.database.windows.net',
     @destination_db = N'LabDB';
GO
-- 2. Drop the publication (removes the articles and Snapshot Agent job)
EXEC sp_droppublication @publication = N'Pub_LabDB';
GO
-- 3. Turn off publishing for the database
USE master;
EXEC sp_replicationdboption
     @dbname = N'LabDB', @optname = N'publish', @value = N'false';
GO
-- 4. Remove distribution (drops the distribution database and its jobs)
EXEC sp_dropdistributor @no_checks = 1, @ignore_distributor = 1;
GO
```

If any step fails because metadata is half-removed, use `EXEC sp_removedbreplication N'LabDB';`. It strips all replication objects from the database in one call.

**9b. Clean the Azure database** (only if you plan to keep it)

The Distribution Agent leaves tracking tables in the subscriber database, such as `MSreplication_subscriptions`. Remove them with this command on Azure:

```sql
EXEC sp_subscription_cleanup
     @publisher    = N'<VM server name>',
     @publisher_db = N'LabDB',
     @publication  = N'Pub_LabDB';
```

**9c. Delete the lab resources** (Azure CLI)

```bash
az group delete --name rg-repl-lab --yes --no-wait
```

This deletes the logical server, database and firewall rule together, so you stop paying for them.

## Troubleshooting

Most failures show up in `MSrepl_errors` or the agent history tables (Step 6a). Match the message against this table.

| Symptom or error | Likely cause | Fix |
| --- | --- | --- |
| Distribution Agent: *Login failed* at the subscriber | Agent is using Windows auth to Azure | Recreate the agent with `@subscriber_security_mode = 0` and the SQL login (Step 5) |
| Error 40615: *Cannot open server … client with IP address … is not allowed* | Missing Azure firewall rule | Add the IP shown in the error with `az sql server firewall-rule create` (Step 1) |
| Agent times out connecting, but `Test-NetConnection` to 1433 works (VM in Azure) | Redirect connection policy needs outbound ports 11000–11999 | Open those ports in the NSG, or set the policy to Proxy with `az sql server conn-policy update --connection-type Proxy` |
| Error 18483 / *distributor\_admin is not defined as a remote login* when configuring distribution | `@@SERVERNAME` doesn't match the actual host name | Run `sp_dropserver` then `sp_addserver ... 'local'`, and restart SQL Server (Step 2b) |
| Snapshot Agent: *Access to the path C:\\ReplData is denied* | Agent account lacks rights on the folder | Grant Modify with `icacls` (Step 2a) |
| `sp_addarticle`: *table does not have a primary key* | Transactional replication requires a primary key | Add a primary key, or leave the table out (Step 2c) |
| Source transaction log keeps growing | Log Reader Agent not running; `log_reuse_wait_desc = REPLICATION` | Start the Log Reader job. Check with `SELECT name, log_reuse_wait_desc FROM sys.databases` |
| Agents never start | SQL Server Agent stopped, or a wrong `@job_login` password | Start the Agent service. Fix the credentials with `sp_changelogreader_agent_properties` or `sp_changesubscription` |
| Distribution Agent: errors 2627/2601 duplicate key, or 20598 row not found | Someone changed data directly in the subscriber | Treat the Azure database as read-only until cutover. Reinitialize with `sp_reinitsubscription` |
