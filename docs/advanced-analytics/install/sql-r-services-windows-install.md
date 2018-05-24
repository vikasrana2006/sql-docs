---
title: Install SQL Server 2016 R Services (In-Database) | Microsoft Docs
ms.prod: sql
ms.technology: machine-learning
  
ms.date: 04/15/2018
ms.topic: conceptual
author: HeidiSteen
ms.author: heidist
manager: cgronlun
---
# Install SQL Server 2016 R Services (In-Database) 
[!INCLUDE[appliesto-ss-xxxx-xxxx-xxx-md-winonly](../../includes/appliesto-ss-xxxx-xxxx-xxx-md-winonly.md)]

This article explains how to install and configure **SQL Server 2016 R Services (In-Database)**. If you have SQL Server 2016, install this feature to enable execution of R code in SQL Server.

## <a name="bkmk_prereqs"> </a> Pre-install checklist

+ SQL Server 2016 setup is required if you want to install R Services. If instead you have SQL Server 2017 installation media, you should install [SQL Server 2017 Machine Learning Services (In-Database)](sql-machine-learning-services-windows-install.md) to get R integration for that version of SQL Server.

+ A database engine instance is required. You cannot install just R, athough you can add it incrementally to an existing instance.

+ Do not install R Services as failover cluster resource. The security mechanism used for isolating R processes is not compatible with a Windows Server failover cluster environment.

+ Do not install R Services on a domain controller. The R Services portion of setup will fail.

+ Do not install **Shared Features** > **R Server (Standalone)** on the same computer running an in-database instance. 

+ Side-by-side installation with other versions of R and Python are possible because the SQL Server instance uses its own copies of the open-source R and Anaconda distributions. However, running code that uses R and Python on the SQL Server computer outside SQL Server can lead to various problems:
    
  + You use a different library and different executable, and get different results, than you do when you are running in SQL Server.
  + R and Python scripts running in external libraries cannot be managed by SQL Server, leading to resource contention.
  
If you used any earlier versions of the Revolution Analytics development environment or the RevoScaleR packages, or if you installed any pre-release versions of SQL Server 2016, you must uninstall them. Running older and newer versions of RevoScaleR and other proprietary packages is not supported. For help removing previous versions, see [Upgrade and Installation FAQ for SQL Server Machine Learning Services](../r/upgrade-and-installation-faq-sql-server-r-services.md).

> [!IMPORTANT]
> After setup is complete, be sure to complete the additional post-configuration steps described in this article. These steps include enabling SQL Server to use external scripts, and adding accounts required for SQL Server to run R jobs on your behalf. Configuration changes generally require a restart of the instance, or a restart of the Launchpad service.

## Get the installation media

[!INCLUDE[GetInstallationMedia](../../includes/getssmedia.md)]

 ###  <a name="bkmk_ga_instalpatch"></a> Install patch requirement 

Microsoft has identified a problem with the specific version of Microsoft VC++ 2013 Runtime binaries that are installed as a prerequisite by SQL Server. If this update to the VC runtime binaries is not installed, SQL Server may experience stability issues in certain scenarios. Before you install SQL Server follow the instructions at [SQL Server Release Notes](../../sql-server/sql-server-2016-release-notes.md#bkmk_ga_instalpatch) to see if your computer requires a patch for the VC runtime binaries.  

## <a name="bkmk2016top"></a>Run Setup

For local installations, you must run Setup as an administrator. If you install [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] from a remote share, you must use a domain account that has read and execute permissions on the remote share.

1. Start the setup wizard for SQL Server 2016.

2. On the **Installation** tab, select **New SQL Server stand-alone installation or add features to an existing installation**.
    
   ![Install R Services (In-Database)](media/2016-setup-installation-rsvcs.png "Start installation of database engine with R Services")
   
3. On the **Feature Selection** page, select the following options:

   - Select **Database Engine Services**. The database engine is required in each instance that uses machine learning.
   - Select **R Services (In-Database)**. Installs support for in-database use of R.
    
     ![R Services feature selection](media/2016setup-rsvcs-features.png "Select these features for R Services In-Database")

    > [!IMPORTANT]
    > Do not install R Server and R Services at the same time. You would ordinarily install R Server (Standalone) to create an environment that a data scientist or developer uses to connect to SQL Server and deploy R solutions. Therefore, there is no need to install both on the same computer.

4.  On the **Consent to Install Microsoft R Open** page, click **Accept**.
  
    This license agreement is required to download Microsoft R Open, which includes a distribution of the open source R base packages and tools, together with enhanced R packages and connectivity providers from the Microsoft R development team.
  
5. After you have accepted the license agreement, there is a brief pause while the installer is prepared. Click **Next** when the button becomes available.

6. On the **Ready to Install** page, verify that the following items are included, and then select **Install**.

   + Database Engine Services
   + R Services (In-Database)

    Note of the location of the folder under the path `..\Setup Bootstrap\Log` where the configuration files are stored. When setup is complete, you can review the installed components in the Summary file.

7. When the installation is complete, restart your computer.

##  <a name="bkmk_enableFeature"></a>Enable external script execution

1. Open [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)]. 

    > [!TIP]
    > You can download and install the appropriate version from this page: [Download SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms).
    > 
    > You can also try out the preview release of [SQL Operations Studio](https://docs.microsoft.com/sql/sql-operations-studio/what-is), which supports administrative tasks and queries against SQL Server.
  
2. Connect to the instance where you installed Machine Learning Services, click **New Query** to open a query window, and run the following command:

   ```SQL
   sp_configure
   ```
    The value for the property, `external scripts enabled`, should be **0** at this point. That is because the feature is turned off by default. The feature must be explicitly enabled by an administrator before you can run R or Python scripts.
     
3. To enable the external scripting feature, run the following statement:
  
    ```SQL
    EXEC sp_configure  'external scripts enabled', 1
    RECONFIGURE WITH OVERRIDE
    ```
  
## Restart the service

When the installation is complete, restart the database engine before continuing to the next, enabling script execution.

Restarting the ervice also automatically restarts the related [!INCLUDE[rsql_launchpad](../../includes/rsql-launchpad-md.md)] service.

You can restart the service using the right-click **Restart** command for the instance in SSMS, or by using the **Services** panel in Control Panel, or by using [SQL Server Configuration Manager](../../relational-databases/sql-server-configuration-manager.md).

## Verify installation

Use the following steps to verify that all components used to launch external script are running.

1. In SQL Server Management Studio, open a new query window, and run the following command:
    
    ```SQL
    EXEC sp_configure  'external scripts enabled'
    ```

    The **run_value** should now be set to 1.

2. Open the **Services** panel or SQL Server Configuration Manager, and verify **SQL Server Launchpad service** is running. You should have one service for every database engine instance that has R or Python installed. Restart the service if is not running. For more information, see [Components to support Python integration](../python/new-components-in-sql-server-to-support-python-integration.md).

7. If Launchpad is running, you should be able to run simple R to verify that external scripting runtimes can communicate with SQL Server. 

    Open a new **Query** window in [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)], and then run a script such as the following:
    
    ```SQL
    EXEC sp_execute_external_script  @language =N'R',
    @script=N'
	OutputDataSet <- InputDataSet;
	',
    @input_data_1 =N'SELECT 1 AS hello'
    WITH RESULT SETS (([hello] int not null));
    GO
    ```

    The script can take a little while to run, the first time the external script runtime is loaded. The results should be something like this:

    | hello |
    |----|
    | 1|

## <a name="bkmk_FollowUp"></a> Additional configuration

If the external script verification step was successful, you can run Python commands from SQL Server Management Studio, Visual Studio Code, or any other client that can send T-SQL statements to the server.

If you got an error when running the command, review the additional configuration steps in this section. You might need to make additional appropriate configurations to the service or database.

Common scenarios that require additional changes include:

* [Configure Windows firewall for in-bound connections](../../database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access.md)
* [Enable additional network protocols](../../database-engine/configure-windows/enable-or-disable-a-server-network-protocol.md)
* [Enable remote connections](../../database-engine/configure-windows/configure-the-remote-access-server-configuration-option.md)
* [Extend built-in permissions to remote users](#bkmk_configureAccounts)
* [Grant permission to run external scripts](#bkmk_AllowLogon)
* [Grant access to individual databases](#permissions-db)

> [!NOTE]
> Not all the listed changes are required, and none might be required. Requirements depend on your security schema, where you installed SQL Server, and how you expect users to connect to the database and run external scripts. Additional troubleshooting tips can be found here: [Upgrade and installation FAQ](../r/upgrade-and-installation-faq-sql-server-r-services.md)

### <a name="bkmk_configureAccounts"></a>Enable implied authentication for the Launchpad account group

During setup, some new Windows user accounts are created for running tasks under the security token of the [!INCLUDE[rsql_launchpad_md](../../includes/rsql-launchpad-md.md)] service. When a user sends an R script from an external client, [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] activates an available worker account, maps it to the identity of the calling user, and runs the R script on behalf of the user. This new service of the database engine supports the secure execution of external scripts, called *implied authentication*.

You can view these accounts in the Windows user group **SQLRUserGroup**. By default, 20 worker accounts are created, which is usually more than enough for running R jobs.

However, if you need to run R scripts from a remote data science client and are using Windows authentication, you must grant these worker accounts permission to sign in to the [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] instance on your behalf.

1. In [!INCLUDE[ssManStudioFull](../../includes/ssmanstudiofull-md.md)], in Object Explorer, expand **Security**, right-click **Logins**, and select **New Login**.
2. In the **Login - New** dialog box, select **Search**.
3. Select the **Object Types** and **Groups** check boxes, and clear all other check boxes.
4. Click **Advanced**, verify that the location to search is the current computer, and then click **Find Now**.
5. Scroll through the list of group accounts on the server until you find one beginning with `SQLRUserGroup`.
    
    + The name of the group that's associated with the Launchpad service for the _default instance_ is always just **SQLRUserGroup**. Select this account only for the default instance.
    + If you are using a _named instance_, the instance name is appended to the default name, `SQLRUserGroup`. Hence, if your instance is named "MLTEST", the default user group name for this instance would be **SQLRUserGroupMLTest**.
5. Click **OK** to close the advanced search dialog box, and verify that you've selected the correct account for the instance. Each instance can use only its own Launchpad service and the group created for that service.
6. Click **OK** once more to close the **Select User or Group** dialog box.
7. In the **Login - New** dialog box, click **OK**. By default, the login is assigned to the **public** role and has permission to connect to the database engine.

### <a name="bkmk_AllowLogon"></a>Give users permission to run external scripts

> [!NOTE]
> If you use a SQL login for running R scripts in a SQL Server compute context, this step is not required.

If you installed [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] in your own instance, you are usually executing scripts as an administrator, or at least as a database owner, and thus have implicit permissions for various operations, all data in the database, and the ability to install new packages as needed.

However, in an enterprise scenario, most users, including users who access the database by using SQL logins, do not have such elevated permissions. Therefore, for each user who will be running R scripts, you must grant the user permissions to run scripts in each database where external scripts will be used.

```SQL
USE <database_name>
GO
GRANT EXECUTE ANY EXTERNAL SCRIPT  TO [UserName]
```

> [!TIP]
> Need help with setup? Not sure you have run all the steps? Use these custom reports to check installation status and run additional steps. 
> 
> [Monitor Machine Learning Services using Custom Reports](../r/monitor-r-services-using-custom-reports-in-management-studio.md).

### <a name="permissions-db"></a> Give your users read, write, or DDL permissions to the database

The user account that is used to run R might need to read data from other databases, create new tables to store results, and write data into tables. Therefore, for each user who will be executing R scripts, ensure that the user has the appropriate permissions on the database: *db_datareader*, *db_datawriter*, or *db_ddladmin*.

For example, the following [!INCLUDE[tsql](../../includes/tsql-md.md)] statement gives the SQL login *MySQLLogin* the rights to run T-SQL queries in the *RSamples* database. To run this statement, the SQL login must already exist in the security context of the server.

```SQL
USE RSamples
GO
EXEC sp_addrolemember 'db_datareader', 'MySQLLogin'
```

For more information about the permissions included in each role, see [Database-level roles](../../relational-databases/security/authentication-access/database-level-roles.md).


### Create an ODBC data source for the instance on your data science client

If you create an R solution on a data science client computer and need to run code by using the SQL Server computer as the compute context, you can use either a SQL login or integrated Windows authentication.

* For SQL logins: Ensure that the login has appropriate permissions on the database where you will be reading data. You can do so by adding *Connect to* and *SELECT* permissions, or by adding the login to the *db_datareader* role. For logins that need to create objects, add *DDL_admin* rights. For logins that must save data to tables, add the login to the *db_datawriter* role.

* For Windows authentication: You might need to configure an ODBC data source on the data science client that specifies the instance name and other connection information. For more information, see [ODBC data source administrator](https://docs.microsoft.com/sql/odbc/admin/odbc-data-source-administrator).

## Suggested optimizations

Now that you have everything working, you might also want to optimize the server to support machine learning, or install pretrained models.

### Add more worker accounts

If you think you might use R heavily, or if you expect many users to be running scripts concurrently, you can increase the number of worker accounts that are assigned to the Launchpad service. For more information, see [Modify the user account pool for SQL Server Machine Learning Services](../r/modify-the-user-account-pool-for-sql-server-r-services.md).

### <a name="bkmk_optimize"></a>Optimize the server for external script execution

The default settings for [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] setup are intended to optimize the balance of the server for a variety of services that are supported by the database engine, which might include extract, transform, and load (ETL) processes, reporting, auditing, and applications that use [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] data. Therefore, under the default settings, you might find that resources for machine learning are sometimes restricted or throttled, particularly in memory-intensive operations.

To ensure that machine learning jobs are prioritized and resourced appropriately, we recommend that you use SQL Server Resource Governor to configure an external resource pool. You might also want to change the amount of memory that's allocated to the [!INCLUDE[ssNoVersion](../../includes/ssnoversion-md.md)] database engine, or increase the number of accounts that run under the [!INCLUDE[rsql_launchpad](../../includes/rsql-launchpad-md.md)] service.

- To configure a resource pool for managing external resources, see [Create an external resource pool](../../t-sql/statements/create-external-resource-pool-transact-sql.md).
  
- To change the amount of memory reserved for the database, see [Server memory configuration options](../../database-engine/configure-windows/server-memory-server-configuration-options.md).
  
- To change the number of R accounts that can be started by [!INCLUDE[rsql_launchpad](../../includes/rsql-launchpad-md.md)], see [Modify the user account pool for machine learning](../r/modify-the-user-account-pool-for-sql-server-r-services.md).

If you are using Standard Edition and do not have Resource Governor, you can use Dynamic Management Views (DMVs) and Extended Events, as well as Windows event monitoring, to help manage the server resources that are used by R. For more information, see [Monitoring and managing R Services](../r/managing-and-monitoring-r-solutions.md).

### Install additional R packages

The R solutions you create for SQL Server can call basic R functions, functions from the properietary packes installed with SQL Server, and third-party R packages compatible with the version of open-source R installed by SQL Server.

Packages that you want to use from SQL Server must be installed in the default library that is used by the instance. If you have a separate installation of R on the computer, or if you installed packages to user libraries, you won't be able to use those packages from T-SQL.

The process for installing and managing R packages is different in SQL Server 2016 and SQL Server 2017. In SQL Server 2016, a database administrator must install R packages that users need. In SQL Server 2017, you can set up user groups to share packages on a per-database level, or configure database roles to enable users to install their own packages. For more information, see [Package management](../r/r-package-management-for-sql-server-r-services.md).


## Get help

Need help with installation or upgrade? For answers to common questions and known issues, see the following article:

* [Upgrade and installation FAQ - Machine Learning Services](../r/upgrade-and-installation-faq-sql-server-r-services.md)

To check the installation status of the instance and fix common issues, try these custom reports.

* [Custom reports for SQL Server R Services](../r/monitor-r-services-using-custom-reports-in-management-studio.md)

## Next steps

R developers can get started with some simple examples, and learn the basics of how R works with SQL Server. For your next step, see the following links:

+ [Tutorial: Run R in T-SQL](../tutorials/rtsql-using-r-code-in-transact-sql-quickstart.md)
+ [Tutorial: In-database analytics for R developers](../tutorials/sqldev-in-database-r-for-sql-developers.md)

To view examples of machine learning that are based on real-world scenarios, see [Machine learning tutorials](../tutorials/machine-learning-services-tutorials.md).


