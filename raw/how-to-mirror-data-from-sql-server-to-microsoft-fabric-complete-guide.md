---
url: https://www.red-gate.com/simple-talk/databases/sql-server/how-to-mirror-data-from-sql-server-to-microsoft-fabric-complete-guide/
date_fetched: 2026-09-13
---

**Mirroring in Microsoft Fabric is useful when you need to make SQL Server data available in Fabric. Perhaps in OneLake, for example, if you need to use the data for reporting, analytics, notebooks, or another Fabric workload.**

**However, it’s not suitable for  everything, as I’ll explain in this complete guide to mirroring data from SQL Server to Microsoft Fabric.**

Before we get started, a note about compatibility. Since SQL Server 2005, there’s been a type of database mirroring totally unrelated to Microsoft Fabric mirroring. I wish these types of terms weren’t so overloaded but, in this article, when I say *mirroring*, I’m talking *only* about Fabric mirroring.

## Can I use Microsoft Fabric mirroring for my pre-2025 version of SQL Server?

**The way Microsoft Fabric mirroring works has changed in SQL Server 2025 compared to earlier versions. **

For earlier versions that are supported (SQL Server 2016 to SQL Server 2022), mirroring uses Change Data Capture (CDC). The source database *must* support CDC as the mirroring setup either uses CDC that’s already enabled, *or* helps configure it as part of the setup process.

On Windows, Fabric mirroring is supported in Standard, Enterprise, and Developer editions. It’s also supported on Linux, but only from SQL Server 2017 CU18 onwards.

Mirroring also works with a variety of hosted environments. You can work with mirroring for SQL Server on-premises, SQL Server in either Azure virtual machines, and even non-Azure clouds like AWS and Google Cloud Platform (GCP).

## Should I use Microsoft Fabric mirroring for SQL Server data?

**Microsoft Fabric mirroring works well in near real-time analytical replication scenarios. The insert, update, and delete replication latency is low, keeping the data close to the current state. Mirroring is a good fit when you or your organization wants current or near-current analytical copies without building a more traditional batch pipeline.**

**However, mirroring is not the best choice for  every SQL Server integration need. **

For a start, mirroring doesn’t guarantee 100% fresh data in the way that readable synchronous replicas in availability groups do. Instead, it’s designed to reduce ETL (extract, transform, load) complexity for the right workloads.*

So, you’ll be better off taking another approach if the main requirement is heavy reshaping before the data becomes useful, the source configuration is unsupported, or the solution depends on features that mirroring doesn’t support.

** Data transformation still needs to happen somewhere in the system. Mirroring fits when the immediate need is simply to replicate operational data into Fabric and make it available quickly*.

## How does Fabric mirroring work for SQL Server 2025?

**SQL Server 2025 uses a different replication mechanism for mirroring compared to earlier versions. This means design, setup, and troubleshooting assumptions need to change as well.** **The key technical change is that SQL Server 2025 uses the Fabric mirroring change feed feature. **

This alters which source features are required, which are incompatible, and which troubleshooting tools you use to solve potential replication issues.

I do think this functionality is better (and this is a better direction for mirroring overall), but there is currently one key limitation: SQL Server 2025 mirroring is currently only supported for on-premises instances. I’m sure this will change over time but, for now, it’s not ideal.

**For example, mirroring is not currently supported for SQL Server 2025 instances running in an Azure Virtual Machine, and there’s no Linux support  at all.**

With this change, you’ll also use different change-feed-specific dynamic management views (DMVs) and stored procedures to the ones used in previous versions.

### Fast, reliable and consistent SQL Server development…

## What do I need for Fabric mirroring for SQL Server 2025?

**Mirroring for SQL Server 2025 requires Azure Arc – and the server  must be connected to it, so you need Azure Extension for SQL Server as well. **

You can either set it up during installation of SQL Server or add it later.

Gateway and network connectivity also need to be in place. You need either an on-premises data gateway or a virtual network data gateway. The gateway’s network must be able to connect to the SQL Server instance, either through a private endpoint *or* because the firewall rule allows it.

### What you need before adding Azure Arc to an existing server

**Before attempting to add Azure Arc to an existing server, you need PowerShell installed on your system and an appropriate Azure Arc user account. You’ll also need to have registered the required resource providers.**

These are the most important things to get right first time. Once everything’s in place, the GUI (graphical user interface) in the portal for setting up mirroring is simple.

### SQL Server managed identities

Over the years, I’ve been disappointed by the lack of support for managed identities across the SQL Server product line, including in Azure Analysis Services (AAS).

**For SQL Server 2025 specifically, the identity story relies on Azure Arc integration. This means that mirroring sits in a hybrid-management model rather than a simple direct-database pattern. Mirroring then makes use of a managed identity in conjunction with Azure Arc.**

Even so, source access still needs secure authentication and least privilege. Microsoft’s security guidance recommends creating a dedicated database user with limited permissions, following the principle of least privilege.

Supported source authentication methods are currently SQL authentication with username and password, and Microsoft Entra ID.

Once Arc is in place, you should be able to check the managed identities by querying:

| 1 2 3 | USE [master]; SELECT * FROM sys.dm_server_managed_identities; | 

The managed identity requires read/write permission on the workspace. If you use the portal to configure mirroring, it will automatically assign this for you.

Also note that, once Arc is in place, the user configured for mirroring doesn’t also need to be an admin.

Here’s an example for a SQL login. You can use this *or* an Entra ID:

| 1 2 3 4 5 6 7 | CREATE USER [Fabric User Name]  FOR LOGIN [Fabric User Name]; GRANT SELECT,        ALTER ANY EXTERNAL MIRROR,        VIEW DATABASE PERFORMANCE STATE,        VIEW DATABASE SECURITY STATE  TO [Fabric User Name]; | 

The user will also need permissions in the source database.

## How do Always On Availability groups work with Fabric mirroring for SQL Server?

**When using Always On Availability groups with Fabric mirroring for SQL Server, you  must ensure you connect to the listener during configuration – not directly to the primary node.**

Mirroring will then continue to work on failover as long as:

- The Fabric login is the *same user*
- Permissions are present in the database
- Permissions in the Fabric workspace are the same for the identity
- The System Assigned Managed Identity (SAMI) of every secondary node has been granted *contributor*on the workspace (at the very least). Ideally, though, it’ll be member or admin – useful if anything needs changing.

## What are some database-level limitations of Fabric mirroring for SQL Server?

**Similar to Azure SQL Database, there are some database-level limitations of Fabric mirroring for SQL Server data. Not  everything is supported; there are some features you may have enabled in the database that’ll prevent you from setting up mirroring.**

- An existing CDC setup is *not*a blocker until SQL Server 2025 and later.
- Azure synapse link for SQL *is*still a blocker.
- Each database can only be mirrored once (so if you have an existing mirror, that will also be a blocker.)
- Replication is also a blocker for mirroring.

Elsewhere, `.dacpac` deployments require the publish property `/p:DoNotAlterReplicatedObjects=False`. You can’t modify any mirrored tables without this.

Row-level security (RLS), object-level security (OLS), and Dynamic Data Masking (DDM) are not currently replicated. Additionally, cross-tenant setups aren’t supported with this form of mirroring, so you can’t have SQL Server in one tenant and your Fabric workspace in another.

### Enhance your data security with an automated masking approach

### Table-level limitations of Microsoft Fabric mirroring for SQL Server

There are also limitations at the table level:

- There can be no more than 1000 tables.
- There are restrictions to the primary key / clustered index data types.
- No clustered columnstore indexes.
- Other unsupported features include temporal and ledger history tables, Always Encrypted columns, in-memory or graph or external tables, switching partitions, and altering the primary key.

### Column-level limitations of Microsoft Fabric mirroring for SQL Server

And finally, there are limitations at the column level:

- No `JSON`or`VECTOR`data type.
- There are data type / precision issues.
- Large object (LOB) columns are potentially trimmed.
- Unsupported data types: computed columns, image/text/`ntext`,`XML`,`rowversion`/`timestamp`,`sql_variant`, user-defined data types (including SQL CLR types), geometry/geography.

## How to troubleshoot Microsoft Fabric mirroring for SQL Server 2025

**When you have mirroring problems, Dynamic Management Views (DMVs) can help with database-level checks. They can also be used to validate configuration.**

Firstly, the `sys.dm_change_feed_log_scan_sessions` DMV *should* show the progress and processing of incremental changes. If not, query `sys.dm_change_feed_errors` to see if any errors are being reported.

If there are no issues there, you can execute the `sp_help_change_feed` stored procedure to check if there are any configuration issues. Remember to check the `table_name` *and* the state. Any state value besides 4** **indicates a potential problem.

If the data replication is still not flowing, verify that the correct managed identity object has permissions. Do this by selecting the “…” ellipses option on the mirrored database item, choosing the *Manage Permissions* option, and seeing if the managed identity name shows with read/write permissions. 

At the same time, check that the `AppId` that shows up is identical to the managed identity ID for your SQL Server.

Finally, if you have a transaction log growing out of hand and not truncating, it could be that mirroring is preventing log truncation for the mirrored database.

To confirm this, check the `log_reuse_wait_desc` column in the `sys.databases` system catalog view. If it says `REPLICATION`, it *is* waiting for mirroring.

The impact of Fabric mirroring on the source database shouldn’t be high but, if you want to control it further, there are resource governor workload groups that you *can* control.

## Summary: Microsoft Fabric mirroring for SQL Server

In SQL Server 2016 – 2022, Microsoft Fabric mirroring was based on CDC; Azure SQL Database used a different mechanism. Now, with SQL Server 2025, mirroring has moved to a mechanism that’s *closer* to what’s in Azure SQL Database. 

This is a positive step but, since it’s designed for on-premises SQL Server systems, it comes with its own set of prerequisites. There are also a number of limitations, many of which I hope get addressed over time.

### Move fast. Govern at scale.

## FAQs: Microsoft Fabric mirroring for SQL Server

### 1. What is Microsoft Fabric mirroring for SQL Server?

A feature that replicates SQL Server data into OneLake in near real time, making it available for Fabric analytics and reporting without a full ETL pipeline.

### 2. Can I use it with SQL Server versions before 2025?

Yes. SQL Server 2016–2022 use CDC (Change Data Capture) for mirroring. The source database must support CDC.

### 3. What's different in SQL Server 2025?

It uses a new change feed feature instead of CDC, with different DMVs and stored procedures. It currently only supports on-premises instances — no Azure VMs or Linux.

### 4. What's needed to set up Fabric mirroring for SQL Server 2025?

Azure Arc connectivity, a data gateway, PowerShell, an Azure Arc user account, and registered resource providers.

### 5. Is Fabric mirroring the same as SQL Server database mirroring?

No. SQL Server’s older database mirroring (since 2005) is unrelated to Fabric mirroring.

### 6. Does it guarantee fully fresh data?

No. It reduces ETL complexity for near real-time analytics, but doesn’t match the freshness of synchronous availability group replicas.

### 7. What are the main limitations?

Blockers include Synapse Link, existing mirrors, and replication. Tables are capped at 1,000 with feature restrictions. Unsupported column types include `JSON`, `VECTOR`, `XML`, and geometry/geography.

### 8. How do I troubleshoot issues?

Check the `sys.dm_change_feed_log_scan_sessions` and `sys.dm_change_feed_errors` DMVs, run `sp_help_change_feed`, and verify managed identity permissions in the Fabric portal.

### 9. Does it work with Always On Availability Groups?

This document contains proprietary information and is protected by copyright law.

Copyright © 2026 Red Gate Software Limited. All rights reserved

Load comments
