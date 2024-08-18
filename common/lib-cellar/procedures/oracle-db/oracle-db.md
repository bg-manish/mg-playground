# Ways with Oracle Database

## Introduction

This lab explains how to install Oracle Database on a VM or a compute instance running Linux. It explains some database concepts, preinstallation setup, and postinstallation checks. It also describes the steps to update listener configuration and to deinstall Oracle Database along with some troubleshooting scenarios and tips.

## Database concepts

Error *`ORA-12505`*   
This error indicates that the listener is up and you can connect to it, but the listener cannot connect to the database. 

**Reason**   
The listener is aware of the database, but does not know whether the database is up because the listener has not received any notification from the database that the database is up.

It can be due to various reasons:
 - the database has not started, that is, the database instance is idle
 - the database is not registered with the listener because it started before the listener

When a database starts, it registers itself with the listener if the listener is already running. If the listener is not running, then the database does not register itself.   
When a listener starts, it does not go looking for databases that might register with it.

 > **Note**: If you try to connect to a wrong database using a wrong `ORACLE_SID`, then you get   
 `ORA-12154 error "TNS: could not resolve the connect identifier specified"`.

### Listener concepts

The default listener configuration is:

 - **name** - *LISTENER*
 - **port** - *1521*

**Scenario**   
You have different database versions (18c, 19c, 21c, 23ai,...) running on the same host. Each database has a listener associated with it. When you start a listener from any `$ORACLE_HOME/bin`, unless you specify the listener name Oracle Net Manager generally starts the default listener. All databases can use the same (default) listener.

However, if you want to use a specific listener for each database, then edit the listener configuration as explained in this lab.

## Preinstallation setup

Oracle Database shiphomes are available internally as gold images. 

1. Visit [DB shiphome annoucements](https://confluence.oraclecorp.com/confluence/x/jobgWQ) and look for the required version.

	For example,
	 - [23ai Linux64](https://confluence.oraclecorp.com/confluence/x/1oCca)
	 - [21c Linux64](https://confluence.oraclecorp.com/confluence/x/qVwAhg)
	 - [19c Linux64](https://confluence.oraclecorp.com/confluence/x/JbYi9g)

1. Under **Shiphome Location**, check <ins>Database Goldimage</ins>.

Production versions are available to download from [OTN](https://www.oracle.com/database/technologies/oracle-database-software-downloads.html). 

Use a production version or obtain a pre-release shiphome to proceed with the installation steps.

- 

	----
	## Extract the database installer

	1. Open a terminal window and create the folder structure for `$ORACLE_HOME`, if it does not exist. The standard Oracle home folder is `dbhome_n`.

		```
		$ <copy>mkdir -p /scratch/u01/app/oracle/product/23.4.0/dbhome_unzip01</copy>
		```

		> **Caution**: Do all installations only under */scratch*, never in `ade` or any other directory.

	1.  Go to the location where the image exists and extract the archive file (installer) into the Oracle home folder.

		**Syntax**

		```
		$ cd [location-of-db-installer-image]

		$ unzip -q [db_installer].zip -d [location-of-Oracle-home]
		```

		> where,   
		*`-q`* for quiet operation (to run in the background suppresses displaying the full inflating details)   
		*`-d`* to create the directory structure

		If you extract the installer without creating the directory, the following error may occur. 

		```
		checkdir:  cannot create extraction directory: /scratch/u01/app/oracle/product/23.4.0/dbhome_unzip01
           No such file or directory
		```

		**Example 1** - for 23ai

		```
		$ <copy>unzip -q db_home.zip -d /scratch/u01/app/oracle/product/23.4.0/dbhome_unzip01</copy>
		```

		Directory size of Oracle home 23ai - 

		 - just unzipped but not installed yet - 

			```
			$ <copy>du -sh dbhome_1</copy>

			4.8G	dbhome_1/
			```

		 - after database installation - 

			```
			$ <copy>du -sh dbhome_1</copy>

			5.4G	dbhome_1/
			```

		> **Tip**: Keep *extra copies of the installer* on your local system for future installations.

		**Example 2** - for 21c prod

		```
		$ <copy>unzip -q LINUX.X64_213000_db_home.zip -d /scratch/u01/app/oracle/product/21.0.0/dbhome_002</copy>
		```

		Optionally, you can rename the folder to the standard name.

		```
		$ <copy>mv dbhome_unzip01 dbhome_1</copy>
		```

	----
	## Run the installer

	1. Go to `$ORACLE_HOME` where you extracted the installer.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/21.0.0/dbhome_1</copy>
		```

	1. Run the Oracle Database Setup Wizard (installer).

		```
		$ <copy>./runInstaller</copy>
		```

## Install Oracle Database

Run the installer from the Oracle home location and specify the following. Most fields are autofilled.

> **Tip**: Run these steps after you extract the database installer (shiphome gold image or production version) into Oracle home. To get the installer, see [Preinstallation setup](?lab=oracle-db#Preinstallationsetup).

1. Create and configure a single instance database.

	![Select Configuration Option](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/common/images/db21c-common-001-createdb.png " ")   **Figure**: Select Configuration option

1. Select the type of installation.

	- Server Class for advanced database installation with detailed configuration 
	- Desktop class for quick installation with basic configuration

	![Select Server class](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-002-serverclass.png " ")   **Figure**: Select Server class

	This example uses *Server class*. 

1. Enterprise Edition.

	![Enterprise Edition](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-003-edition.png " ")   **Figure**: Select Enterprise Edition

1. Specify Oracle base location.

	![Specify Oracle base location](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-004-baseloc.png " ")   **Figure**: Specify Oracle base location

	Example -

	```
	$ <copy>/scratch/u01/app/[your-account]</copy>
	```

1. Central inventory location.

	![Specify inventory location](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-005-inventory.png " ")   **Figure**: Specify inventory location

	Example -

	```
	$ <copy>/scratch/u01/app/oraInventory</copy>
	```

	**oraInventory group name**: *dba* (or `wheel`, primary group of your account)

	> **Note**: If you have another version of Oracle Database already installed on your host, then the installer does not display the inventory window.

1. General Purpose / Transaction Processing.

	![Select Configuration type](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-006-configtemplate.png " ")   **Figure**: Select Configuration type

1. Set database identifiers.

	![Set database identifiers](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-007-id.png " ")   **Figure**: Set database identifiers

	 - Global database name -

		```
		<copy>orcl.us.oracle.com</copy>
		```

	 - Oracle System Identifier (SID) - *orcl*   
	 For typical installation, the option *Create as Container Database* is selected by default

	 - Pluggable database name -

		```
		<copy>orclpdb</copy>
		```

1. <i>Do not enable</i> **Automatic Memory Management**.   
Leave defaults for character set (and sample schemas).

	![Automatic Memory](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-008a-memory.png " ")   **Figure**: Leave default unchecked

1. Database storage options - *File system*.

	![Select File system](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-009-storagefilesys.png " ")   **Figure**: Select File system

	Database file location -

	```
	<copy>/scratch/u01/app/[your-account]/oradata</copy>
	```

1. <i>Do not register</i> with Oracle Enterprise Manager.

	![Register with EM](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-010-emcc.png " ")   **Figure**: Leave default unchecked

1. Select **Enable Recovery** &gt; **File system** and set recovery area location (leave default).

	![Enable recovery options](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-011-recovery.png " ")   **Figure**: Enable recovery options

	Example -

	```
	<copy>/scratch/u01/app/[your-account]/recovery_area</copy>
	```

1. Use the same password for all accounts.

	![Set the password](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-012-syspwd.png)   **Figure**: Set the password

1. For all OS groups, select *dba* (or `wheel`)

	![Select OS groups](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-013-osgroups.png " ")   **Figure**: Select OS groups

1. You may `Automatically run configuration scripts`.   
Do not select this checkbox and run the scripts manually later.

	![Manually run Config Scripts](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-014-rootscript.png " ")   **Figure**: Manually run Config Scripts

1. Prerequisites check - *Ignore All*, if required.

	![Prereq check](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-015-prereq-check.png " ")   **Figure**: Prereq check

1. Review the summary and **Install** to start the installation.

	![Review summary](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-016-summary.png " ")   **Figure**: Review summary

1. When prompted to run the script - 

	![Run Config scripts](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/common/images/db21c-common-002-rootscript.png " ")   **Figure**: Run Config scripts

	**Option 1**: Run the scripts as root

	- Open another terminal window and change to the root user.

		```
		$ <copy>sudo -i</copy>
		```

		Enter the password when prompted.

	- Run the `inventory` script.

		```
		$ <copy>source /scratch/u01/app/oraInventory/orainstRoot.sh</copy>
		```

		```
		Changing permissions of /scratch/u01/app/oraInventory.
		Adding read,write permissions for group.
		Removing read,write,execute permissions for world.

		Changing groupname of /scratch/u01/app/oraInventory to wheel.
		The execution of the script is complete.
		```

	- Run the `root` script.

		```
		$ <copy>source /scratch/u01/app/oracle/product/23.4.0/dbhome_1/root.sh</copy>
		```

		```
		Performing root user operation.

		The following environment variables are set as:
			ORACLE_OWNER= [your-account]
			ORACLE_HOME=  /scratch/u01/app/oracle/product/23.4.0/dbhome_1
		```

		Press **Enter** twice to complete the script.

		```
		Enter the full pathname of the local bin directory: [/usr/local/bin]:

		/usr/local/bin is read only.  Continue without copy (y/n) or retry (r)? [y]:

		Warning: /usr/local/bin is read only. No files will be copied.


		Creating /etc/oratab file...
		Entries will be added to the /etc/oratab file as needed by
		Database Configuration Assistant when a database is created
		Finished running generic part of root script.
		Now product-specific root actions will be performed.
		```

	**Option 2**: Run the scripts as a regular user.

	```
	<copy>
	sudo /scratch/u01/app/oraInventory/orainstRoot.sh

	sudo /scratch/u01/app/oracle/product/23.4.0/dbhome_1/root.sh
	</copy>
	```

1. Go back to the installer window and click **OK**.

	![Database installation complete](https://oracle-livelabs.github.io/database/odb-21c/dba-essentials/install-db/install-desktop-server/images/db21c-srv-018-install-success.png " ")   **Figure**: Database installation complete

On completion, the installer displays the finish window.

## Database postinstallation checks

1. View Oracle SIDs and Oracle homes
1. View Oracle base location
1. View system/config files
1. Set environment variables
1. Connect to SQL command line

	----
	## 1. View Oracle SIDs and Oracle homes

	- `/etc/oratab`
	- `../oraInventory/ContentsXML`
	- `/opt/ORCLfmap/prot1_64/etc/filemap.ora`

	<ins>Using *`/etc/oratab`*</ins>

	```
	$ <copy>cat /etc/oratab</copy>
	```

	Example -

	```
	# This file is used by ORACLE utilities.  It is created by root.sh
	# and updated by either Database Configuration Assistant while creating
	# a database or ASM Configuration Assistant while creating ASM instance.

	# A colon, ':', is used as the field terminator.  A new line terminates
	# the entry.  Lines beginning with a pound sign, '#', are comments.
	#
	# Entries are of the form:
	#   $ORACLE_SID:$ORACLE_HOME:<N|Y>:
	#
	# The first and second fields are the system identifier and home
	# directory of the database respectively.  The third field indicates
	# to the dbstart utility that the database should , "Y", or should not,
	# "N", be brought up at system boot time.
	#
	# Multiple entries with the same $ORACLE_SID are not allowed.
	#
	#
	orcl:/scratch/u01/app/[your-account]/product/23.4.0/dbhome_1:N
	orcl21:/scratch/u01/app/oracle/product/21.0.0/dbhome_002:N
	orcl19:/scratch/u01/app/oracle/product/19.0.0/dbhome_02:N
	orclmm:/scratch/u01/app/oracle/product/21.0.0/dbhome_mm01:N

	```

	 > **Did you know..?**   
	 The file *`oratab`* is informational only.   
	 You can connect to the SQL prompt even if `/etc/oratab` is moved or deleted or does not contain entries of `$ORACLE_SID` and `$ORACLE_HOME`.

	<ins>Using *`oraInventory`*</ins>

	1. Go to this location.

		```
		$ <copy>cd /scratch/u01/app/oraInventory/ContentsXML</copy>
		```

	1. View the contents of `inventory.xml`.

		```
		$ <copy>cat inventory.xml</copy>
		```

	<ins>Using *`/opt`*</ins>

	1. Go to this location.

		```
		$ <copy>cd /opt/ORCLfmap/prot1_64/etc</copy>
		```

	1. View the contents of `filemap.ora`.

		```
		$ <copy>cat filemap.ora</copy>
		```

		```
		###############################################################################
		#                                                                             #
		# This is the configuration file that describes all the mapping libraries     #
		# available.                                                                  #
		#                                                                             #
		# The following row needs to be created for each library:                     #
		# lib={vendor name}: {mapping library path}                                   #
		#                                                                             #
		# Note that the ordering of the libraries in this file is extremely           #
		# important. The libraries are queried based on their order in the            #
		# configuration file.                                                         #
		#                                                                             #
		# Oracle provides a mapping library for EMC Symmetrix arrays. This library    #
		# (LIBMAPSYM.SO) uses the SYMAPI and SYMLVM EMC libraries.                    #
		#                                                                             #
		# UNCOMMENT THE ROW CORRESPONDING TO THAT LIBRARY ONLY IF THE SYM LIBRARIES   #
		# ARE AVAILABLE ON SOLARIS.                                                   #
		###############################################################################

		## The next row needs to be commented out only for Solaris on EMC storage
		#lib=Oracle: /scratch/u01/app/oracle/product/23.4.0/dbhome_1/lib/%s_mapsym%

		## The next row can be comented out on ALL UNIX PLATFORMS if no other
		## libraries exist
		#lib=Oracle: /scratch/u01/app/oracle/product/23.4.0/dbhome_1/lib/%s_mapdummy%
		```

	----
	## 2. View Oracle base location

	1. Go to `$ORACLE_HOME/install`.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/23.4.0/dbhome_1/install</copy>
		```

	1. View the contents of the `orabasetab` file.

		```
		$ <copy>cat orabasetab</copy>
		```

		```
		#orabasetab file is used to track Oracle Home associated with Oracle Base
		/scratch/u01/app/oracle/product/23.4.0/dbhome_1:/scratch/u01/app/[your-account]:OraDB23Home1:Y:
		```

		Base location for Oracle Database

		```
		$ /scratch/u01/app/[your-account]
		```

		Contents of Oracle base - 23ai

		```
		$ ls

		$ admin  audit  diag  oradata  recovery_area
		```

		Contents of Oracle base - 21c

		```
		$ ls

		$ admin  audit  cfgtoollogs  dbs  diag  fast_recovery_area  homes  oradata  recovery_area
		```
		Deinstall removes all these folders under the base location.

	----
	## 3. View system/config files

	Depending on the database version, the location of critical files may vary. For example, the location of central inventory, system config, network config, etc.

	 - To know the inventory location, check *`oraInst.loc`*

		```
		$ <copy>cat /etc/oraInst.loc</copy>

		inventory_loc=/scratch/u01/app/oraInventory
		```

	 - Network files *`listener.ora`*, *`tnsnames.ora`*, *`sqlnet.ora`*

		| db <br>version | location                      | full path        |
		|:--------------:|-------------------------------|------------------|
		| 23ai           | `$ORACLE_HOME/network/admin/`                    |  `/scratch/u01/app/oracle/product/23.4.0/dbhome_1/network/admin/`    |
		| 21c            | `$ORACLE_BASE/homes/OraDB21Home1/network/admin/` | `/scratch/u01/app/[your-account]/homes/OraDB21Home1/network/admin/` |
		| 19c            | `$ORACLE_HOME/network/admin/`                    | `/scratch/u01/app/oracle/product/19.0.0/dbhome_02/network/admin/`   |
		{: title="Network files location"}

	 - **pfile** - *`init.ora.xxxxxxxxxxxxx`* resides at `$ORACLE_BASE/admin/[ORACLE_SID]/pfile/`

		Full path examples

		```
		$ /scratch/u01/app/[oracle-base-1]/admin/orcl/pfile/
		$ /scratch/u01/app/[oracle-base-2]/admin/orcl21/pfile/
		$ /scratch/u01/app/[oracle-base-3]/admin/orcl19/pfile/
		```

	 - **spfile** - *`spfile[ORACLE_SID].ora`*

		| db <br>version          | location                | full path                         |
		|-------------------------|-------------------------|-----------------------------------|
		| 23ai | `$ORACLE_HOME/dbs/`  | `/scratch/u01/app/oracle/product/23.4.0/dbhome_1/dbs/`  |
		| 21c  | `$ORACLE_BASE/dbs/`  | `/scratch/u01/app/[oracle-base]/dbs/`                   |
		| 19c  | `$ORACLE_HOME/dbs/`  | `/scratch/u01/app/oracle/product/19.0.0/dbhome_02/dbs/` |

	 - *`init.ora`* - `$ORACLE_HOME/dbs/`

		Full path examples

		```
		$ /scratch/u01/app/oracle/product/23.4.0/dbhome_1/dbs/
		$ /scratch/u01/app/oracle/product/21.0.0/dbhome_002/dbs/
		$ /scratch/u01/app/oracle/product/19.0.0/dbhome_02/dbs/
		```

	 - **oradata** - control files, redo log files, pdbs, seed, tablespaces are at *`$ORACLE_BASE/oradata/`*

		Full path

		```
		$ <copy>cd /scratch/u01/app/[oracle-base]/oradata/</copy>
		```

		Default tablespaces -
		 - `sysaux01.dbf`
		 - `system01.dbf`
		 - `temp01.dbf`
		 - `undotbs01.dbf`
		 - `users01.dbf`

	----
	## 4. Set environment variables

	Set environment variables, `$ORACLE_SID` and `$ORACLE_HOME`, to connect to the database and to run the listener.

	- Set env manually
	- Set env automatically

		----
		## Set env manually

		Use any of the following options.

		**For 23ai**

		<ins>Shell - csh</ins>

		----------- ------- --------------- -------------------------
			setenv ORACLE_SID orcl
			setenv ORACLE_HOME /scratch/u01/app/oracle/product/23.4.0/dbhome_1
		----------- ------- --------------- -------------------------

		<ins>Shell - Bash</ins>

		```
		<copy>
		export ORACLE_SID=orcl
		export ORACLE_HOME=/scratch/u01/app/oracle/product/23.4.0/dbhome_1
		</copy>
		```

		**For 21c**

		<ins>Shell - csh</ins>

		----------- ------- --------------- -------------------------
			setenv ORACLE_SID orcl21
			setenv ORACLE_HOME /scratch/u01/app/oracle/product/21.0.0/dbhome_002
		----------- ------- --------------- -------------------------

		<ins>Shell - Bash</ins>

		```
		<copy>
		export ORACLE_SID=orcl21
		export ORACLE_HOME=/scratch/u01/app/oracle/product/21.0.0/dbhome_002
		</copy>
		```

		**For 19c**

		<ins>Shell - csh</ins>

		----------- ------- --------------- -------------------------
			setenv ORACLE_SID orcl19
			setenv ORACLE_HOME /scratch/u01/app/oracle/product/19.0.0/dbhome_02
		----------- ------- --------------- -------------------------

		<ins>Shell - Bash</ins>

		```
		<copy>
		export ORACLE_SID=orcl19
		export ORACLE_HOME=/scratch/u01/app/oracle/product/19.0.0/dbhome_02
		</copy>
		```

		----
		## Set env automatically

		**Option 1**   
		Works from any directory

		```
		$ <copy>. oraenv</copy>
		```

		**Option 2**   
		Works only from `$ORACLE_HOME/bin`

		```
		$ <copy>./oraenv</copy>
		```

	----
	## 5. Connect to SQL command line

	 - Using <i>SQL Plus</i>
	 - Using <i>SQL cl</i>

	**SQL Plus**   
	<ins>Option 1</ins> - Connect as `system` and enter the user name and password.

	```
	$ <copy>./sqlplus</copy>
	```

	 - Enter user name: *system*
	 - Enter password: Enter the password

	<ins>Option 2</ins> - Connect as `sysdba` (requires no password).

	```
	$ <copy>./sqlplus / as sysdba</copy>
	```
	```
	SQL*Plus: Release 23.0.0.0.0 - Production on Wed Aug 7 12:42:33 20XX
	Version 23.4.0.24.05

	Copyright (c) 1982, 2024, Oracle.  All rights reserved.


	Connected to:
	Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - Production
	Version 23.4.0.24.05
	```

	**SQL cl**   
	Connect as `sysdba` (requires no password).

	```
	$ <copy>./sql / as sysdba</copy>
	```
	```
	SQLcl: Release 21.4 Production on Thu Aug 15 11:31:55 20XX

	Copyright (c) 1982, 2024, Oracle.  All rights reserved.

	Connected to:
	Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - Production
	Version 23.4.0.24.05
	```

	### Exit from SQL prompt

	```
	$ SQL> <copy>exit</copy>
	```
	```
	Disconnected from Oracle Database 23ai Enterprise Edition Release 23.0.0.0.0 - Production
	Version 23.4.0.24.05
	```

## Edit listener config

To modify the Listener configuration, for example the listener name, the high-level sequence is -

![Edit listener configuration](./images/edit-listener-config.png " ") **Figure:** Edit Listener config

After stopping the listener -

 1. Edit the files - *`listener.ora`*, *`tnsnames.ora`*
 1. Start the listener with the new name - *`./lsnrctl start [new-name]`*

### Register database with the modified listener

![Register Database with the modified Listener](./images/register-db-with-listener.png " ") **Figure:** Register Database with the modified Listener

- The following sections explain how to modify listener configuration.

	## High-level steps to register database with listener

	 1. Log in to the database
		 - `./sqlplus / as sysdba`

	 1. Check the existing listener
		 - `show parameter local_listener`

	 1. Register the new listener -    
		 - `alter system set local_listener='(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1519))';`   
		 - `alter system register;`

	 1. Restart the database instance
		 - `shutdown immediate` followed by `startup`

	 1. Open the PDBs
		 - `alter pluggable database all open;`

	 1. Verify the new listener
		 - `show parameter local_listener`

	Thus, you can set the listener according to database versions on your host.

	For example,
	 - for **23ai** - *LISTENER23AI* port *1523*
	 - for **21c** - *LISTENER21C* port *1521*
	 - for **19c** - *LISTENER19C* port *1519*

	> For detailed steps, check the corresponding database section.

	----
	## Configure the Listener for 23ai

	You can modify the listener name, say *LISTENER23AI*, and the port to, say *1523*. Register 23ai with the modified listener.

	 1. Go the listener config location under `$ORACLE_HOME/network/admin`.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/23.4.0/dbhome_1/network/admin</copy>
		```

	 1. Open the file *`listener.ora`* and edit listener details at all occurrences. For example,	 
		 - name - Change to *LISTENER23AI* or leave the default
		 - port - *1523*

		```
		$ <copy>vi listener.ora</copy>
		```

		> Press ***i*** to go to the edit (insert) mode.

		```
		...
		LISTENER23AI =
		  (DESCRIPTION_LIST =
			(DESCRIPTION =
			  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1523))
			  (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1523))
			)
		  )
		...
		```

		Save the file.
		 - **Esc** + **:wq** or **Esc** + **Shift** + **zz**

	 1. At the same location, open the file *`tnsnames.ora`* and change the port to *1523* at all occurrences.

		```
		$ <copy>vi tnsnames.ora</copy>
		```

		> Press ***i*** to go to the edit (insert) mode.

		```
		...
		LISTENER_ORCL =
		  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1523))


		ORCL =
		  (DESCRIPTION =
			(ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1523))
			(CONNECT_DATA =
			  (SERVER = DEDICATED)
			  (SERVICE_NAME = orcl.us.oracle.com)
			)
		  )
		...
		```

		Save the file.
		 - **Esc** + **:wq** or **Esc** + **Shift** + **zz**

	 1. Go to `$ORACLE_HOME/bin` and start the new listener.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/23.4.0/dbhome_1/bin</copy>
		```

		```
		$ <copy>./lsnrctl start listener23ai</copy>
		```

		> **Note:** If you do not specify the listener name, then this command starts the default, *LISTENER*.

	 1. Log in to the database as *sysdba*.

		```
		$ <copy>./sqlplus / as sysdba</copy>
		```

	 1. Check the existing listener.

		```
		SQL> <copy>show parameter local_listener</copy>

		NAME            TYPE        VALUE
		-----           --------    ------
		local_listener  string      LISTENER_ORCL
		```

	 1. Set the listener details and register database with the new listener.

		```
		SQL> <copy>alter system set local_listener='(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1523))';</copy>
		```

		```
		SQL> <copy>alter system register;</copy>
		```

	 1. Restart the database and open all PDBs.

		```
		SQL> <copy>shutdown immediate</copy>

		Database closed.
		Database dismounted.
		ORACLE instance shut down.
		```

		```
		SQL> <copy>startup</copy>

		ORACLE instance started.

		Total System Global Area 9950893832 bytes
		Fixed Size		    	9926408 bytes
		Variable Size		 1543503872 bytes
		Database Buffers	 8388608000 bytes
		Redo Buffers		    8855552 bytes
		Database mounted.
		Database opened.
		```

		```
		SQL> <copy>alter pluggable database all open;</copy>

		Pluggable database altered.
		```

	 1. Verify the new listener.

		```
		SQL> <copy>show parameter local_listener</copy>

		NAME			TYPE	 	VALUE
		--------------- ---------- 	-----------------------------
		local_listener	string	 	(ADDRESS=(PROTOCOL=tcp)(HOST=
									localhost.example.com)(PORT
									=1523))
		```

		You can now exit from SQL prompt.

		```
		SQL> <copy>exit</copy>
		```

		You have modified the listener name and port and registered 23ai with the new listener. 

	----
	## Configure the Listener for 21c

	Modify listener name, for example *LISTENER21C*, and port, for example, *1521*. Register 21c with the modified listener.

	 1. Go the listener config location under `$ORACLE_BASE/homes/OraDB21Home1/network/admin/`.

		```
		$ <copy>cd /scratch/u01/app/[oracle-base]/homes/OraDB21Home1/network/admin/</copy>
		```

	 1. Open the file *`listener.ora`* and edit listener details at all occurrences. For example,	 
		 - name - *LISTENER21C*
		 - port - *1521*

		```
		$ <copy>vi listener.ora</copy>
		```

		> Press ***i*** to go to the edit (insert) mode.

		```
		...
		LISTENER21C =
		  (DESCRIPTION_LIST =
			(DESCRIPTION =
			  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1521))
			  (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
			)
		  )
		...
		```

		Save the file.
		 - **Esc** + **:wq** or **Esc** + **Shift** + **zz**

	 1. At the same location, open the file *`tnsnames.ora`* and change the port to *1521* at all occurrences.

		```
		$ <copy>vi tnsnames.ora</copy>
		```

		> Press ***i*** to go to the edit (insert) mode.

		```
		...
		ORCL21 =
		  (DESCRIPTION =
			(ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1521))
			(CONNECT_DATA =
			  (SERVER = DEDICATED)
			  (SERVICE_NAME = orcl21.us.oracle.com)
			)
		  )

		LISTENER_ORCL21 =
		  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1521))
		...
		```

		Save the file.
		 - **Esc** + **:wq** or **Esc** + **Shift** + **zz**

	 1. Go to `$ORACLE_HOME/bin` and start the new listener.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/21.0.0/dbhome_002/bin</copy>
		```

		```
		$ <copy>./lsnrctl start listener21c</copy>
		```

	 1. Log in to the database as *sysdba*.

		```
		$ <copy>./sqlplus / as sysdba</copy>
		```

	 1. Check the existing listener.

		```
		SQL> <copy>show parameter local_listener</copy>

		NAME            TYPE        VALUE
		-----           --------    ------
		local_listener  string      LISTENER_ORCL21
		```

	 1. Set the listener details and register database with the new listener.

		```
		SQL> <copy>alter system set local_listener='(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1521))';</copy>
		```

		```
		SQL> <copy>alter system register;</copy>
		```

	 1. Restart the database and open all PDBs.

		```
		SQL> <copy>shutdown immediate</copy>

		Database closed.
		Database dismounted.
		ORACLE instance shut down.
		```

		```
		SQL> <copy>startup</copy>

		ORACLE instance started.

		Total System Global Area 9965663072 bytes
		Fixed Size		    	9696096 bytes
		Variable Size		 1509949440 bytes
		Database Buffers	 8422162432 bytes
		Redo Buffers		   23855104 bytes
		Database mounted.
		Database opened.
		```

		```
		SQL> <copy>alter pluggable database all open;</copy>

		Pluggable database altered.
		```

	 1. Verify the new listener.

		```
		SQL> <copy>show parameter local_listener</copy>

		NAME			TYPE	 	VALUE
		--------------- ---------- 	-----------------------------
		local_listener	string	 	(ADDRESS=(PROTOCOL=tcp)(HOST=
									localhost.example.com)(PORT
									=1521))
		```

		You can now exit from SQL prompt.

		```
		SQL> <copy>exit</copy>
		```

		You have modified the listener name and port and registered 21c with the new listener.

	----
	## Configure the Listener for 19c

	Modify listener name, say *LISTENER19C*, and port, say *1519*. Register 19c with the modified listener.

	 1. Go the listener config location under `$ORACLE_HOME/network/admin`.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/19.0.0/dbhome_02/network/admin</copy>
		```

	 1. Open the file *`listener.ora`* and edit listener details at all occurrences. For example,	 
		 - name - *LISTENER19C*
		 - port - *1519*

		```
		$ <copy>vi listener.ora</copy>
		```

		> Press ***i*** to go to the edit (insert) mode.

		```
		...
		LISTENER19C =
		  (DESCRIPTION_LIST =
			(DESCRIPTION =
			  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1519))
			  (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1519))
			)
		  )
		...
		```

		Save the file.
		 - **Esc** + **:wq** or **Esc** + **Shift** + **zz**

	 1. At the same location, open the file *`tnsnames.ora`* and change the port to *1519* at all occurrences.

		```
		$ <copy>vi tnsnames.ora</copy>
		```

		> Press ***i*** to go to the edit (insert) mode.

		```
		...
		ORCL19 =
		  (DESCRIPTION =
			(ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1519))
			(CONNECT_DATA =
			  (SERVER = DEDICATED)
			  (SERVICE_NAME = orcl19.us.oracle.com)
			)
		  )

		LISTENER_ORCL19 =
		  (ADDRESS = (PROTOCOL = TCP)(HOST = localhost.example.com)(PORT = 1519))
		...
		```

		Save the file.
		 - **Esc** + **:wq** or **Esc** + **Shift** + **zz**

	 1. Go to `$ORACLE_HOME/bin` and start the new listener.

		```
		$ <copy>cd /scratch/u01/app/oracle/product/19.0.0/dbhome_02/bin</copy>
		```

		```
		$ <copy>./lsnrctl start listener19c</copy>
		```

	 1. Log in to the database as *sysdba*.

		```
		$ <copy>./sqlplus / as sysdba</copy>
		```

	 1. Check the existing listener.

		```
		SQL> <copy>show parameter local_listener</copy>

		NAME            TYPE        VALUE
		-----           --------    ------
		local_listener  string      LISTENER_ORCL
		```

	 1. Set the listener details and register database with the new listener.

		```
		SQL> <copy>alter system set local_listener='(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1519))';</copy>
		```

		```
		SQL> <copy>alter system register;</copy>
		```

	 1. Restart the database and open all PDBs.

		```
		SQL> <copy>shutdown immediate</copy>

		Database closed.
		Database dismounted.
		ORACLE instance shut down.
		```

		```
		SQL> <copy>startup</copy>

		ORACLE instance started.

		Total System Global Area 9965664976 bytes
		Fixed Size		    	9145040 bytes
		Variable Size		 1543503872 bytes
		Database Buffers	 8388608000 bytes
		Redo Buffers		   24408064 bytes
		Database mounted.
		Database opened.
		```

		```
		SQL> <copy>alter pluggable database all open;</copy>

		Pluggable database altered.
		```

	 1. Verify the new listener.

		```
		SQL> <copy>show parameter local_listener</copy>

		NAME			TYPE	 	VALUE
		--------------- ---------- 	-----------------------------
		local_listener	string	 	(ADDRESS=(PROTOCOL=tcp)(HOST=
									localhost.example.com)(PORT
									=1519))
		```

		You can now exit from SQL prompt.

		```
		SQL> <copy>exit</copy>
		```

		You have modified the listener name and port and registered 19c with the new listener.

	----
	## Verify modified listeners

	**Option 1** - Check Listener status from `ORACLE_HOME/bin`

	-	For 23ai

		```
		$ <copy>./lsnrctl stat listener23ai</copy>
		```

		## Output

		```
		LSNRCTL for Linux: Version 23.0.0.0.0 - Production on 16-AUG-20XX 08:18:08

		Copyright (c) 1991, 2024, Oracle.  All rights reserved.

		Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost.example.com)(PORT=1523)))
		STATUS of the LISTENER
		------------------------
		Alias                     LISTENER
		Version                   TNSLSNR for Linux: Version 23.0.0.0.0 - Production
		Start Date                07-AUG-20XX 13:33:37
		Uptime                    8 days 18 hr. 44 min. 31 sec
		Trace Level               off
		Security                  ON: Local OS Authentication
		SNMP                      OFF
		Listener Parameter File   /scratch/u01/app/oracle/product/23.4.0/dbhome_1/network/admin/listener.ora
		Listener Log File         /scratch/u01/app/oracle/diag/tnslsnr/localhost/listener/alert/log.xml
		Listening Endpoints Summary...
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1523)))
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1523)))
		Services Summary...
		Service "16dbed12d9b5bd08e0631fc45e640b06.us.oracle.com" has 1 instance(s).
		  Instance "orcl", status READY, has 1 handler(s) for this service...
		Service "1ca6f267eae900bfe0638e084664e079.us.oracle.com" has 1 instance(s).
		  Instance "orcl", status READY, has 1 handler(s) for this service...
		Service "orcl.us.oracle.com" has 1 instance(s).
		  Instance "orcl", status READY, has 1 handler(s) for this service...
		Service "orclXDB.us.oracle.com" has 1 instance(s).
		  Instance "orcl", status READY, has 1 handler(s) for this service...
		Service "orclpdb.us.oracle.com" has 1 instance(s).
		  Instance "orcl", status READY, has 1 handler(s) for this service...
		The command completed successfully
		```

	-	For 21c

		```
		$ <copy>./lsnrctl stat listener21c</copy>
		```

		## Output

		```
		LSNRCTL for Linux: Version 21.0.0.0.0 - Production on 31-JAN-20XX 11:38:28

		Copyright (c) 1991, 2021, Oracle.  All rights reserved.

		Connecting to (ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1521))
		STATUS of the LISTENER
		------------------------
		Alias                     listener21c
		Version                   TNSLSNR for Linux: Version 21.0.0.0.0 - Production
		Start Date                26-JAN-20XX 20:16:49
		Uptime                    4 days 15 hr. 21 min. 38 sec
		Trace Level               off
		Security                  ON: Local OS Authentication
		SNMP                      OFF
		Listener Parameter File   /scratch/u01/app/[oracle-base]/homes/OraDB21Home1/network/admin/listener.ora
		Listener Log File         /scratch/u01/app/[oracle-base]/diag/tnslsnr/localhost/listener21c/alert/log.xml
		Listening Endpoints Summary...
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1521)))
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcps)(HOST=localhost.example.com)(PORT=5500))(Security=(my_wallet_directory=/scratch/u01/app/[oracle-base]/homes/OraDB21Home1/admin/orcl21/xdb_wallet))(Presentation=HTTP)(Session=RAW))
		Services Summary...
		Service "c8209f27c6b16005e053362ee80ae60e.us.oracle.com" has 1 instance(s).
		  Instance "orcl21", status READY, has 1 handler(s) for this service...
		Service "f31b33958c07d8f8e0538e0846649415.us.oracle.com" has 1 instance(s).
		  Instance "orcl21", status READY, has 1 handler(s) for this service...
		Service "orcl21.us.oracle.com" has 1 instance(s).
		  Instance "orcl21", status READY, has 1 handler(s) for this service...
		Service "orcl21XDB.us.oracle.com" has 1 instance(s).
		  Instance "orcl21", status READY, has 1 handler(s) for this service...
		Service "orcl21pdb.us.oracle.com" has 1 instance(s).
		  Instance "orcl21", status READY, has 1 handler(s) for this service...
		The command completed successfully
		```

	-	For 19c

		```
		$ <copy>./lsnrctl stat listener19c</copy>
		```

		## Output

		```
		LSNRCTL for Linux: Version 19.0.0.0.0 - Production on 25-JAN-20XX 20:57:03

		Copyright (c) 1991, 2019, Oracle.  All rights reserved.

		Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=localhost.example.com)(PORT=1519)))
		STATUS of the LISTENER
		------------------------
		Alias                     listener19c
		Version                   TNSLSNR for Linux: Version 19.0.0.0.0 - Production
		Start Date                25-JAN-20XX 05:20:40
		Uptime                    0 days 15 hr. 36 min. 22 sec
		Trace Level               off
		Security                  ON: Local OS Authentication
		SNMP                      OFF
		Listener Parameter File   /scratch/u01/app/oracle/product/19.0.0/dbhome_02/network/admin/listener.ora
		Listener Log File         /scratch/u01/app/[oracle-base]/diag/tnslsnr/localhost/listener19c/alert/log.xml
		Listening Endpoints Summary...
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=localhost.example.com)(PORT=1519)))
		  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1519)))
		Services Summary...
		Service "86b637b62fdf7a65e053f706e80a27ca.us.oracle.com" has 1 instance(s).
		  Instance "orcl19", status READY, has 1 handler(s) for this service...
		Service "f30feb5c998c60ebe0538e0846641b62.us.oracle.com" has 1 instance(s).
		  Instance "orcl19", status READY, has 1 handler(s) for this service...
		Service "orcl19.us.oracle.com" has 1 instance(s).
		  Instance "orcl19", status READY, has 1 handler(s) for this service...
		Service "orcl19XDB.us.oracle.com" has 1 instance(s).
		  Instance "orcl19", status READY, has 1 handler(s) for this service...
		Service "orcl19pdb.us.oracle.com" has 1 instance(s).
		  Instance "orcl19", status READY, has 1 handler(s) for this service...
		The command completed successfully
		```

	**Option 2** - Check the listener service for each database

	-	For 23ai

		> Note the listener name in uppercase.

		```
		$ <copy>ps -ef | grep LISTENER</copy>
		```
		```
		[your-account]  694002       1  0 Jan26 ?        00:00:09 /scratch/u01/app/oracle/product/23.4.0/dbhome_1/bin/tnslsnr LISTENER -inherit
		[your-account] 2539150  680000  0 12:01 pts/4    00:00:00 grep --color=auto LISTENER
		```

	-	For 21c

		```
		$ <copy>ps -ef | grep listener21c</copy>
		```
		```
		[your-account]  675596       1  0 Jan26 ?        00:00:06 /scratch/u01/app/oracle/product/21.0.0/dbhome_002/bin/tnslsnr listener21c -inherit
		[your-account] 2538381  672005  0 11:58 pts/3    00:00:00 grep --color=auto listener21c
		```

	-	For 19c

		```
		$ <copy>ps -ef | grep listener19c</copy>
		```
		```
		[your-account]   27785       1  0 05:20 ?        00:00:02 /scratch/u01/app/oracle/product/19.0.0/dbhome_02/bin/tnslsnr listener19c -inherit
		[your-account]  302476   35076  0 20:56 pts/0    00:00:00 grep --color=auto listener19c
		```

	**Option 3** - Check all listener services

	- For all databases

		```
		$ <copy>ps -ef|grep lsn</copy>
		```
		```
		[your-account]   27785       1  0 Jan25 ?        00:00:07 /scratch/u01/app/oracle/product/19.0.0/dbhome_02/bin/tnslsnr listener19c -inherit
		[your-account]  675596       1  0 20:16 ?        00:00:00 /scratch/u01/app/oracle/product/21.0.0/dbhome_002/bin/tnslsnr listener21c -inherit
		[your-account]  694002       1  0 20:44 ?        00:00:00 /scratch/u01/app/oracle/product/23.4.0/dbhome_1/bin/tnslsnr LISTENER -inherit
		[your-account]  696062  680000  0 20:51 pts/4    00:00:00 grep --color=auto lsn
		```

## Deinstall Oracle Database

1. Go to the `deinstall` folder.

	```
	$ <copy>cd /u01/app/oracle/product/23.4.0/dbhome_1/deinstall</copy>
	```

1. Run the `deinstall` command to start the deinstallation.

	```
	$ <copy>./deinstall</copy>
	```

	Press **Enter** at most prompts to select the default. At the final step, press **y** to continue. The option **n** exists the deinstallation without removing the database. 

	> **Note**: Only the user who installed the database on a host can run `deinstall` to remove the database.

	So, if you install the database as *oracle*, then you cannot deinstall as `[your-account]` or any other user or as `root`. In such case, it returns the following error.

	```
	ERROR: Oracle home not owned by user.
	The user running the deinstall tool is not the same as owner of Oracle home
	based on inventory ownership.
	You must run this tool as the same user who installed the software.
	```

## Troubleshooting

- 

	----
	## Create directory under root

	**Problem:** In a VM, you can create the folder `/u01` in `/scratch` but not in the root directory `/`.   
	**Workaround:** Create `/u01` as root, change the ownership to `oracle`, and the mode to executable.

	1. Change the user to `root`.

		```
		$ <copy>/usr/local/packages/aime/install/run_as_root "bash"</copy>
		```

		Other options:

		- `sudo su -`
		- `sudo -i`   
			Enter the password when prompted

	1. Create the folder under the root directory.

		```
		$ <copy>mkdir -p /u01/app/oracle/product/23.4.0/dbhome_1</copy>
		```
	1. Change ownership to *oracle* (user) for the folder `/u01`.

		```
		$ <copy>chown -R oracle:dba /u01</copy>
		```

	1. Provide full control to `/u01`.

		```
		$ <copy>chmod 777 /u01</copy>
		```

	----
	## Graphical display error

	**Problem statement**   
	You set the display and run the database installer. 

	```
	$ export DISPLAY=:0.0
	$ ./runInstaller
	```

	It returns the following error

	```
	ERROR: Unable to verify the graphical display setup. This application requires X display. Make sure that xdpyinfo exist under PATH variable.

	Can't connect to X11 window server using ':0.0' as the value of the DISPLAY variable.
	OP's X server was Hummingbird Exceed. Xming will work as well.
	```

	**Solution**   
	Copy `~/.Xauthority` to `/home/oracle` location.

	```
	$ <copy>cp ~/.Xauthority /home/oracle</copy>
	```

	The actual solution depends on:
	- whether you changed to oracle user from a different user.
	- whether you are using X11 forwarding.
	- the actual port used by X11 forwarding, which may be 6010 or a different port.
	- If you logged in and then su'd to oracle, then copy ~/.Xauthority to the oracle account.
	- If you are using X11 forwarding and the forwarding port is 6010, then DISPLAY=localhost:10.0 is correct.
	- If you are not using X11 forwarding, and if port 6000 on your PC is reachable from the oracle server, then `export DISPLAY=your.pc.ip.address:0.0` is correct.

	1. Install `xterm` and use it to test X Window System.
	1. Install `xdpyinfo.

	----
	## xdpyinfo not installed

	**Problem statement**   
	You run the database installer and get a message - The `xdpyinfo` program must be installed.

	**Solution**   
	Install `xdpyinfo` -

	```
	$ <copy>yum install xorg-x11-utils-[version-number]</copy>
	```

	- If already installed, check whether the `oracle` user has execute privileges.

		```
		<copy>
		cd /usr/bin
		ls -al | grep xdpyinfo
		</copy>
		```

		If the `oracle` user has execute privileges, it displays the following information.

		```
		-rwxr-xr-x   1 root root      38112 Feb 23  2015 xdpyinfo
		```

	- If the `oracle` user does not have executable privileges, then log in as `root` and run this.

		```
		$ <copy>xhost +SI:localuser:oracle</copy>
		```

		It returns the following.

		```
		localuser:oracle being added to access control list
		```

	Refer [Installation Guide - xdpyinfo Errors](https://docs.oracle.com/cd/E89497_01/html/rpm_80_installation/GUID-842C3883-9BC1-4D37-82C1-9E7F24628AA7.htm).

	----
	## 19c installer does not start - [INS-08101] Unexpected error: 'supportedOSCheck'

	**Problem statement**  
	You run the database installer but the wizard refuses to start. The installer returns the following message.

	```
	[INS-08101] Unexpected error while executing the action at state: 'supportedOSCheck'. Are you sure you want to continue?
	```

	![DB 19c installation - supportedOSCheck error](./images/install-db19c-err-supportedoscheck.png " ")

	**What happened**   
	Oracle Database 19c supports up to Oracle Enterprise Linux (OEL) v8.1. Apparently, the operating system on your host is OEL v8.2 or later.

	The code `[INS-08101]` indicates that `CV_ASSUME_DISTID`, an environmental variable set by the verification program, is not set.

	**What to do**  

	To fix this error, follow one of the workarounds - <i>temporary fix</i> or <i>permanent fix</i>.

	 - Temporary fix

		Export the variable or `setenv` at runtime.

		 - **Bash**

			```
			$ <copy>export CV_ASSUME_DISTID=OEL8.1</copy>
			```

		 - **csh**

			```
			$ <copy>setenv CV_ASSUME_DISTID OEL8.1</copy>
			```

		Interestingly, any distribution id is acceptable as long as it's valid.

	 - Permanent fix

		1. Open the CVU configuration file, `$ORACLE_HOME/cv/admin/cvu_config`, in an editor.

			```
			$ <copy>vi $ORACLE_HOME/cv/admin/cvu_config</copy>
			```

		1. Uncomment the line (around line `#20`) containing the variable. That is, remove the leading pound sign (`#`). Otherwise, add a new line if the variable is missing. The fallback value can also work.

			```
			CV_ASSUME_DISTID=OEL5
			```

		Save the file.

		> **Esc** + **:wq** or **Esc** + **Shift** + **zz**

		You can now start the database installer for 19c without any issues `./runInstaller`.

		----
		## Cite

		 - [Resolve Ins-08101 Unexpected Error - Ed Chen Logic](https://logic.edchen.org/how-to-resolve-ins-08101-unexpected-error/)
		 - [Unexpected Error Supportedoscheck - dbsguru](https://dbsguru.com/solution-for-ins-08101-unexpected-error-supportedoscheck-while-oracle-19c-installation/)
		 - [martinberger.com](https://www.martinberger.com/2020/05/install-oracle-19c-rdbms-on-oracle-linux-8-avoid-warning-ins-08101-unexpected-error-while-executing-the-action-at-state-supportedoscheck/)

	----
	## Listener does not know of `$ORACLE_SID`

	You run the listener status and find that the database service is missing.

	```
	$ ./lsnrctl status
	```

	**Solution**

	Option 1

	```
	sqlplus /nolog
	conn system
	alter system register;  
	exit
	lsnrctl status
	```

	Restart the listener. You can now see the service in the listener status information. If the service is still not displayed, then manually register the listener with the database.

	```
	./sqlplus /nolog

	SQL> conn system

	SQL> alter system set local_listener = '(ADDRESS=(PROTOCOL=TCP)(HOST=localhost.exampple.com)(PORT=1521))' scope = both;

	SQL> alter system register;

	SQL> exit

	$ ./lsnrctl status
	```

	Restart the listener and check the status again. Verify that the database service is registered with the listener.

	----
	## Check dependent libraries/packages for database installation

	**Problem statement**   
	You run the database installer and it returns this error.

	```
	/u01/app/oracle/product/23.4.0/dbhome_2/perl/bin/perl: error while loading shared libraries: libnsl.so.2: cannot open shared object file: No such file or directory
	```

	**What happened**   
	Some libraries or packages are missing or not installed on the host.

	**What to do**

	1. Run ldd (List Dynamic Dependencies).

		```
		$ <copy>ldd /scratch/u01/app/oracle/product/23.4.0/dbhome_1/perl/bin/perl</copy>
		```

		Verify that the shared libraries and packages required for the installer are available on the host.

		```
		linux-vdso.so.1 (0x00007ffd99cd1000)
		libpthread.so.0 => /lib64/libpthread.so.0 (0x00007ff236322000)
		libnsl.so.2 => /lib64/libnsl.so.2 (0x00007ff236108000)
		libdl.so.2 => /lib64/libdl.so.2 (0x00007ff235f04000)
		libm.so.6 => /lib64/libm.so.6 (0x00007ff235b82000)
		libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00007ff235959000)
		libutil.so.1 => /lib64/libutil.so.1 (0x00007ff235755000)
		libc.so.6 => /lib64/libc.so.6 (0x00007ff235390000)
		/lib64/ld-linux-x86-64.so.2 (0x00007ff236542000)
		libtirpc.so.3 => /lib64/libtirpc.so.3 (0x00007ff23515d000)
		libgssapi_krb5.so.2 => /lib64/libgssapi_krb5.so.2 (0x00007ff234f08000)
		libkrb5.so.3 => /lib64/libkrb5.so.3 (0x00007ff234c1e000)
		libk5crypto.so.3 => /lib64/libk5crypto.so.3 (0x00007ff234a07000)
		libcom_err.so.2 => /lib64/libcom_err.so.2 (0x00007ff234803000)
		libkrb5support.so.0 => /lib64/libkrb5support.so.0 (0x00007ff2345f2000)
		libkeyutils.so.1 => /lib64/libkeyutils.so.1 (0x00007ff2343ee000)
		libcrypto.so.1.1 => /lib64/libcrypto.so.1.1 (0x00007ff233f05000)
		libresolv.so.2 => /lib64/libresolv.so.2 (0x00007ff233cee000)
		libselinux.so.1 => /lib64/libselinux.so.1 (0x00007ff233ac4000)
		libz.so.1 => /lib64/libz.so.1 (0x00007ff2338ac000)
		libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007ff233628000)
		```

		If any required libraries are missing, then the installer refuses to run.   
		In this example, the *`libnsl`* 32-bit package is missing.

		```
		/scratch/u01/app/oracle/product/23.4.0/dbhome_3/perl/bin/perl: /lib64/libcrypt.so.1: version `XCRYPT_2.0' not found (required by /scratch/u01/app/oracle/product/23.4.0/dbhome_3/perl/bin/perl)
		/scratch/u01/app/oracle/product/23.4.0/dbhome_3/perl/bin/perl: /lib64/libc.so.6: version `GLIBC_2.28' not found (required by /scratch/u01/app/oracle/product/23.4.0/dbhome_3/perl/bin/perl)
			linux-vdso.so.1 =>  (0x00007ffc023aa000)
			libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f4e483d8000)
			libnsl.so.2 => not found
			libdl.so.2 => /lib64/libdl.so.2 (0x00007f4e47fba000)
			libm.so.6 => /lib64/libm.so.6 (0x00007f4e47cb8000)
			libcrypt.so.1 => /lib64/libcrypt.so.1 (0x00007f4e47a81000)
			libutil.so.1 => /lib64/libutil.so.1 (0x00007f4e4787e000)
			libc.so.6 => /lib64/libc.so.6 (0x00007f4e474b0000)
			/lib64/ld-linux-x86-64.so.2 (0x00007f4e485f4000)
			libfreebl3.so => /lib64/libfreebl3.so (0x00007f4e472ad000)
		```
		 - 

			----
			## screenshot

			![Database libraries missing](./images/lld-db-lib-missing.png " ")

	1. Install the missing packages, in this example, `libnsl`.

		```
		$ <copy>dnf install -y libnsl</copy>
		```
		```
		$ <copy>dnf install -y libnsl.i686</copy>
		```
		```
		$ <copy>dnf install -y libnsl2</copy>
		```
		```
		$ <copy>dnf install -y libnsl2.i686</copy>
		```

## References

- [Database all-at-once](https://oracle-base.com)

## Acknowledgments

 - **Author** - [](include:author)
 - **Created on** - January 5, (Wed) 2022
 - **Last Updated on** - August 16, (Fri) 2024
 - **Questions/Feedback?** - Blame [](include:profile)
