****************
CSQL Interpreter
****************


To execute SQL statements in CUBRID, you need to use either a Graphical User Interface (GUI)-based CUBRID Manager or a console-based CSQL Interpreter.
CSQL is an application that allows users to use SQL statements through a command-driven interface. This section briefly explains how to use the CSQL Interpreter and associated commands.

Introduction to the CSQL Interpreter
====================================

**A Tool for SQL**

The CSQL Interpreter is an application installed with CUBRID that allows you to execute in an interactive or batch mode and viewing query results. The CSQL Interpreter has a command-line interface. With this, you can store SQL statements together with their results to a file for a later use.

The CSQL Interpreter provides the best and easiest way to use CUBRID. You can develop database applications with various APIs (e.g. JDBC, ODBC, PHP, CCI, etc.; you can use the CUBRID Manager, which is a management and query tool provided by CUBRID. With the CSQL Interpreter, users can create and retrieve data in a terminal-based environment.

The CSQL Interpreter directly connects to a CUBRID database and executes various tasks using SQL statements. Using the CSQL Interpreter, you can:

*   Retrieve, update and delete data in a database by using SQL statements
*   Execute external shell commands
*   Store or display query results
*   Create and execute SQL script files
*   Select table schema
*   Retrieve or modify parameters of the database server system
*   Retrieve database information (e.g. schema, triggers, queued triggers, workspaces, locks, and statistics)

**A Tool for DBA**

A database administrator (**DBA**) performs administrative tasks by using various administrative utilities provided by CUBRID; a terminal-based interface of CSQL Interpreter is an environment where **DBA** executes administrative tasks.

It is also possible to run the CSQL Interpreter in standalone mode. In this mode, the CSQL Interpreter directly accesses database files and executes commands including server process properties. That is, SQL statements can be executed to a database without running a separate database server process. The CSQL Interpreter is a powerful tool that allows you to use the database only with a **csql** utility, without any other applications such as the database server or the brokers.

Executing CSQL
==============

CSQL Execution Mode
-------------------

**Interactive Mode**

With CSQL Interpreter, you can enter and execute SQL statements to handle schema and data in the database. Enter statements in a prompt that appears when running the **csql** utility. After executing the statements, the results are listed in the next line. This is called the interactive mode.

**Batch Mode**

You can store SQL statements in a file and execute them later to have the **csql** utility read the file. This is called the batch mode. For more information on the batch mode, see `CSQL Startup Options <#csql_csql_exec_option_htm>`_ .

**Standalone Mode**

In the standalone mode, CSQL Interpreter directly accesses database files and executes commands including server process functions. That is, SQL statements can be sent and executed to a database without a separate database server process running for the task. Since the standalone mode allows only one user access at a given time, it is suitable for management tasks by Database Administrators (**DBAs**).

**Client/Server Mode**

CSQL Interpreter usually operates as a client process and accesses the server process.

Using CSQL (Syntax)
-------------------

**Connecting to Local Host**

Execute the CSQL Interpreter using the **csql** utility. You can set options as needed. To set the options, specify the name of the database to connect to as a parameter. The following is a **csql** utility statement to access the database on a local server: ::

	csql [options] database_name
	
**Connecting to Remote Host**

The following is a **csql** utility syntax to access the database on a remote host: ::

	csql [options] database_name@remote_host_name

Make sure that the following conditions are met before you run the CSQL Interpreter on a remote host.

*   The CUBRID installed on the remote host must be the same version as the one on the local host.
*   The port number used by the master process on the remote host must be identical to the one on the local host.
*   You must access the remote host in client/server mode using the **-C** option.

**Example**

The following example shows how to access the **demodb** database on the remote host with the IP address 192.168.1.3 and calls the **csql** utility. ::

	csql -C demodb@192.168.1.3

CSQL Options
------------

To display the option list in the prompt, execute the **csql** utilities without specifying the database name as follows: ::

	$ csql
	A database-name is missing.
	interactive SQL utility, version 9.0
	usage: csql [OPTION] database-name[@host]

	valid options:
	  -S, --SA-mode                standalone mode execution
	  -C, --CS-mode                client-server mode execution
	  -u, --user=ARG               alternate user name
	  -p, --password=ARG           password string, give "" for none
	  -e, --error-continue         don't exit on statement error
	  -i, --input-file=ARG         input-file-name
	  -o, --output-file=ARG        output-file-name
	  -s, --single-line            single line oriented execution
	  -c, --command=ARG            CSQL-commands
	  -l, --line-output            display each value in a line
	  -r, --read-only              read-only mode
		  --no-auto-commit         disable auto commit mode execution
		  --no-pager               do not use pager
		  --no-single-line         turn off single line oriented execution

	For additional information, see http://www.cubrid.com
	
**Options**

.. program:: csql

.. option:: -S

	The following example shows how to connect to a database in standalone mode and execute the **csql** utility. If you want to use the database exclusively, use the **-S** option. If both **-S** and **-C** options are omitted, the **-C** option will be specified. ::

		csql -S demodb

.. option:: -C

	The following example shows how to connect to a database in client/server mode and execute the **csql** utility. In an environment where multiple clients connect to the database, use the **-C** option. Even when you connect to a database on a remote host in client/server mode, the error log created during **csql** execution will be stored in the **cub.err** file on the local host. ::

		csql -C demodb

.. option:: -i

	The following example shows how to specify the name of the input file that will be used in a batch mode with the **-i** option. In the **infile** file, more than one SQL statement is stored. Without the **-i** option specified, the CSQL Interpreter will run in an interactive mode. ::

		csql -i infile demodb

.. option:: -o

	The following example shows how to store the execution results to the specified file instead of displaying on the screen. It is useful to retrieve the results of the query performed by the CSQL Interpreter afterwards. ::

		csql -o outfile demodb

.. option:: -u

	The following example shows how to specify the name of the user that will connect to the specified database with the **-u** option. If the **-u** option is not specified, **PUBLIC** that has the lowest level of authorization will be specified as a user. If the user name is not valid, an error message is displayed and the **csql** utility is terminated. If there is a password for the user name you specify, you will be prompted to enter the password. ::

		csql -u DBA demodb

.. option:: -p

	The following example shows how to enter the password of the user specified with the **-p** option. Especially since there is no prompt to enter a password for the user you specify in a batch mode, you must enter the password using the **-p** option. When you enter an incorrect password, an error message is displayed and the **csql** utility is terminated. ::

		csql -u DBA -p *** demodb

.. option:: -s

	As an option used with the **-i** option, it executes multiple SQL statement one by one in a file with the **-s** option. This option is useful to allocate less memory for query execution and each SQL statement is separated by semicolons (;). If it is not specified, multiple SQL statements are retrieved and executed at once. ::

		csql -s -i infile demodb

.. option:: -c

	The following example shows how to execute more than one SQL statement from the shell with the **-c** option. Multiple statements are separated by semicolons (;). ::

		csql -c "select * from olympic;select * from stadium" demodb

.. option:: -l

	The following example shows how to display the execution results of the SQL statement in a line format with the **-l** option. The execution results will be output in a column format if the **-l** option is not specified. ::

		csql -l demodb

.. option:: -e

	The following example shows how to ignore errors and keep execution even though semantic or runtime errors occur with the **-e** option. However, if any SQL statements have syntax errors, query execution stops after errors occur despite specifying the **-e** option. ::

		$ csql -e demodb

		csql> SELECT * FROM aaa;SELECT * FROM athlete WHERE code=10000;

		In line 1, column 1,

		ERROR: before ' ;SELECT * FROM athlete WHERE code=10000; '
		Unknown class "aaa".


		=== <Result of SELECT Command in Line 1> ===

				 code  name                  gender                nation_code           event               
		=====================================================================================================
				10000  'Aardewijn Pepijn'    'M'                   'NED'                 'Rowing'            


		1 row selected.

		Current transaction has been committed.

		1 command(s) successfully processed.

.. option:: -r

	You can connect to the read-only database with the **-r** option. Retrieving data is only allowed in the read-only database; creating databases and entering data are not allowed. ::

		$ csql -r demodb

.. option:: --no-auto-commit

	The following example shows how to stop the auto-commit mode with the **--no-auto-commit** option. If you don't configure **--no-auto-commit** option, the CSQL Interpreter runs in an auto-commit mode by default, and the SQL statement is committed automatically at every execution. Executing the **;AUtocommit** session command after starting the CSQL Interpreter will also have the same result. ::

		csql --no-auto-commit demodb

.. option:: --no-pager

	The following example shows how to display the execution results by the CSQL Interpreter at once instead of page-by-page with the **--no-pager** option. The results will be output page-by-page if **--no-pager** option is not specified. ::

		csql --no-pager demodb

.. option:: --no single-line

	The following example shows how to keep storing multiple SQL statements and execute them at once with the **;xr** or **;r** session command. If you do not specify this option, SQL statements are executed without **;xr** or **;r** session command. ::

		csql --no-single-line demodb

**Session Commands**

In addition to SQL statements, CSQL Interpreter provides special commands allowing you to control the Interpreter. These commands are called session commands. All the session commands must start with a semicolon (;).

Session Commands
================

Enter the **;help** command to display a list of the session commands available in the CSQL Interpreter. Note that only the uppercase letters of each session command are required to make the CSQL Interpreter to recognize it. Session commands are not case-sensitive. ::

	csql> ;help

	=== <Help: Session Command Summary> ===


	   All session commands should be prefixed by `;' and only blanks/tabs
	   can precede the prefix. Capitalized characters represent the minimum
	   abbreviation that you need to enter to execute the specified command.

	   ;REAd   [<file-name>]       - read a file into command buffer.
	   ;Write  [<file-name>]       - (over)write command buffer into a file.
	   ;APpend [<file-name>]       - append command buffer into a file.
	   ;PRINT                      - print command buffer.
	   ;SHELL                      - invoke shell.
	   ;CD                         - change current working directory.
	   ;EXit                       - exit program.

	   ;CLear                      - clear command buffer.
	   ;EDIT                       - invoke system editor with command buffer.
	   ;List                       - display the content of command buffer.

	   ;RUn                        - execute sql in command buffer.
	   ;Xrun                       - execute sql in command buffer,
									 and clear the command buffer.
	   ;COmmit                     - commit the current transaction.
	   ;ROllback                   - roll back the current transaction.
	   ;AUtocommit [ON|OFF]        - enable/disable auto commit mode.
	   ;REStart                    - restart database.

	   ;SHELL_Cmd  [shell-cmd]     - set default shell, editor, print and pager
	   ;EDITOR_Cmd [editor-cmd]      command to new one, or display the current
	   ;PRINT_Cmd  [print-cmd]       one, respectively.
	   ;PAger_cmd  [pager-cmd]

	   ;DATE                       - display the local time, date.
	   ;DATAbase                   - display the name of database being accessed.
	   ;SChema class-name          - display schema information of a class.
	   ;TRigger [`*'|trigger-name] - display trigger definition.
	   ;Get system_parameter       - get the value of a system parameter.
	   ;SEt system_parameter=value - set the value of a system parameter.
	   ;PLan [simple/detail/off]   - show query execution plan.
	   ;Info <command>             - display internal information.
	   ;TIme [ON/OFF]              - enable/disable to display the query
									 execution time.
	   ;HISTORYList                - display list of the executed queries.
	   ;HISTORYRead <history_num>  - read entry on the history number into command buffer.
	   ;HElp                       - display this help message.

**Reading SQL statements from a file (;REAd)**

The **;REAd** command reads the contents of a file into the buffer. This command is used to execute SQL commands stored in the specified file. To view the contents of the file loaded into the buffer, use the **;List** command. ::

	csql> ;rea nation.sql
	The file has been read into the command buffer.
	csql> ;list
	insert into "sport_event" ("event_code", "event_name", "gender_type", "num_player") values
	(20001, 'Archery Individual', 'M', 1);
	insert into "sport_event" ("event_code", "event_name", "gender_type", "num_player") values
	20002, 'Archery Individual', 'W', 1);
	....

**Storing SQL statements into a file (;Write)**

The **;Write** command stores the contents of the command buffer into a file. This command is used to store SQL commands that you entered or modified in the CSQL Interpreter. ::

	csql> ;w outfile
	Command buffer has been saved.

**Appending to a file (;APpend)**

This command appends the contents of the current command buffer to an **outfile** file. ::

	csql> ;ap outfile
	Command buffer has been saved.

**Executing a shell command (;SHELL)**

The **;SHELL** session command calls an external shell. Starts a new shell in the environment where the CSQL Interpreter is running. It returns to the CSQL Interpreter when the shell terminates. If the shell command to execute with the **;SHELL_Cmd** command has been specified, it starts the shell, executes the specified command, and returns to the CSQL Interpreter. ::

	csql> ;shell
	% ls -al
	total 2088
	drwxr-xr-x 16 DBA cubrid   4096 Jul 29 16:51 .
	drwxr-xr-x  6 DBA cubrid   4096 Jul 29 16:17 ..
	drwxr-xr-x  2 DBA cubrid   4096 Jul 29 02:49 audit
	drwxr-xr-x  2 DBA cubrid   4096 Jul 29 16:17 bin
	drwxr-xr-x  2 DBA cubrid   4096 Jul 29 16:17 conf
	drwxr-xr-x  4 DBA cubrid   4096 Jul 29 16:14 cubridmanager
	% exit
	csql>

**Registering a shell command (;SHELL_Cmd)**

The **;SHELL_Cmd** command registers a shell command to execute with the **SHELL** session command. As shown in the example below, enter the **;shell** command to execute the registered command. ::

	csql> ;shell_c ls -la
	csql> ;shell
	total 2088
	drwxr-xr-x 16 DBA cubrid   4096 Jul 29 16:51 .
	drwxr-xr-x  6 DBA cubrid   4096 Jul 29 16:17 ..
	drwxr-xr-x  2 DBA cubrid   4096 Jul 29 02:49 audit
	drwxr-xr-x  2 DBA cubrid   4096 Jul 29 16:17 bin
	drwxr-xr-x  2 DBA cubrid   4096 Jul 29 16:17 conf
	drwxr-xr-x  4 DBA cubrid   4096 Jul 29 16:14 cubridmanager
	csql>

**Changing the current working directory (;CD)**

This command changes the current working directory where the CSQL Interpreter is running to the specified directory. If you don't specify the path, the directory will be changed to the home directory. ::

	csql> ;cd /home1/DBA/CUBRID
	Current directory changed to  /home1/DBA/CUBRID.

**Exiting the CSQL Interpreter (;EXit)**

This command exits the CSQL Interpreter. ::

	csql> ;ex

**Clearing the command buffer (;CLear)**

The **;CLear** session command clears the contents of the command buffer. ::

	csql> ;cl
	csql> ;list

**Displaying the contents of the command buffer (;List)**

The **;List** session command lists the contents of the command buffer that have been entered or modified. The command buffer can be modified by **;READ** or **;Edit** command. ::

	csql> ;l

**Executing SQL statements (;RUn)**

This command executes SQL statements in the command buffer. Unlike the **;Xrun** session command described below, the buffer will not be cleared even after the query execution. ::

	csql> ;ru

**Clearing the command buffer after executing the SQL statement (;Xrun)**

This command executes SQL statements in the command buffer. The buffer will be cleared after the query execution. ::

	csql> ;x

**Committing transaction (;COmmit)**

This command commits the current transaction. You must enter a commit command explicitly if it is not in auto-commit mode. In auto-commit mode, transactions are automatically committed whenever SQL is executed. ::

	csql> ;co
	Current transaction has been committed.

**Rolling back transaction (;ROllback)**

This command rolls back the current transaction. Like a commit command (**;COmmit**), it must enter a rollback command explicitly if it is not in auto-commit mode (**OFF**). ::

	csql> ;ro
	Current transaction has been rolled back.

**Setting the auto-commit mode (;AUtocommit)**

This command sets auto-commit mode to **ON** or **OFF**. If any value is not specified, current configured value is applied by default. The default value is **ON**. ::

	csql> ;au off
	AUTOCOMMIT IS OFF

**CHeckpoint Execution (;CHeckpoint)**

This command executes the checkpoint within the CSQL session. This command can only be executed when a DBA group member, who is specified for the custom option (**-u** *user_name*), connects to the CSQL Interpreter in system administrator mode (**--sysadm**).

**Checkpoint**

is an operation of flushing all dirty pages within the current data buffer to disks. You can also change the checkpoint interval using a command (**;set** *parameter_name* value) to set the parameter values in the CSQL session. You can see the examples of the parameter related to the checkpoint execution interval (**checkpoint_interval_in_mins** and **checkpoint_every_npages**). For more information, see `Logging-Related Parameters <#pm_pm_db_classify_logging_htm>`_ . ::

	csql> ;ch
	Checkpoint has been issued.

**Transaction Monitoring Or Termination (;Killtran)**

This command checks the transaction status information or terminates a specific transaction in the CSQL session. This command prints out the status information of all transactions on the screen if a parameter is omitted it terminates the transaction if a specific transaction ID is specified for the parameter. It can only be executed when a DBA group member, who is specified for the custom option (**-u** *user_name*), connects to the CSQL Interpreter in system administrator mode (**--sysadm**). ::

	csql> ;k
	Tran index      User name      Host name      Process id      Program name
	-------------------------------------------------------------------------------
		  1(+)            dba      myhost             664           cub_cas
		  2(+)            dba      myhost            6700              csql
		  3(+)            dba      myhost            2188           cub_cas
		  4(+)            dba      myhost             696              csql
		  5(+)         public      myhost            6944              csql
	 
	csql> ;k 3
	The specified transaction has been killed.

**Restarting database (;REStart)**

A command that tries to reconnect to the target database in a CSQL session. Note that when you execute the CSQL Interpreter in CS (client/server) mode, it will be disconnected from the server. When the connection to the server is lost due to a HA failure and failover to another server occurs, this command is particularly useful in connecting to the switched server while maintaining the current session. ::

	csql> ;res
	The database has been restarted.

**Displaying the current date (;DATE)**

The **;DATE** command displays the current date and time in the CSQL Interpreter. ::

	csql> ;date
	     Tue July 29 18:58:12 KST 2008

**Displaying the database informatio (;DATAbase)**

This command displays the database name and host name where the CSQL Interpreter is working. If the database is running, the HA mode (one of those followings: active, standby, or maintenance) will be displayed as well.  ::

	csql> ;data
	     demodb@localhost (active)

**Displaying schema information of a class (;SChema)**

The **;SChema** session command displays schema information of the specified table. The information includes the table name, its column name and constraints. ::

	csql> ;sc event
	=== <Help: Schema of a Class> ===
	 <Class Name>
		 event
	 <Attributes>
		 code           INTEGER NOT NULL
		 sports         CHARACTER VARYING(50)
		 name           CHARACTER VARYING(50)
		 gender         CHARACTER(1)
		 players        INTEGER
	 <Constraints>
		 PRIMARY KEY pk_event_event_code ON event (code)

**Displaying the trigger (;TRriger)**

This command searches and displays the trigger specified. If there is no trigger name specified, all the triggers defined will be displayed. ::

	csql> ;tr
	=== <Help: All Triggers> ===
		trig_delete_contents

**Checking the parameter value(;Get)**

You can check the parameter value currently set in the CSQL Interpreter using the **;Get** session command. An error occurs if the parameter name specified is incorrect. ::

	csql> ;g isolation_level
	=== Get Param Input ===
	isolation_level=4

**Setting the parameter value (;SEt)**

You can use the **;Set** session command to set a specific parameter value. Note that changeable parameter values are only can be changed. To change the server parameter values, you must have DBA authorization. For information on list of changeable parameters, see `cubrid_broker.conf Configuration File and Default Parameters <#pm_pm_broker_setting_htm>`_ . ::

	csql> ;se block_ddl_statement=1
	=== Set Param Input ===
	block_ddl_statement=1

	-- Dynamically change the log_max_archives value in the csql accessed by dba account
	csql>;se log_max_archives=5

**Setting the view level of executing query plan (;PLan)**

You can use the **;PLan** session command to set the view level of executing query plan the level is composed of **simple**, **detail**, and **off**. Each command refers to the following:

*   **off** : Not displaying the query execution plan
*   **simple** : Displaying the query execution plan in simple version (OPT LEVEL=257)
*   **detail** : Displaying the query execution plan in detailed version (OPT LEVEL=513)

**Displaying information (;Info)**

The **;Info** session command allows you to check information such as schema, triggers, the working environment, locks and statistics. ::

	csql> ;i lock
	*** Lock Table Dump ***
	 Lock Escalation at = 100000, Run Deadlock interval = 1
	Transaction (index  0, unknown, unknown@unknown|-1)
	Isolation REPEATABLE CLASSES AND READ UNCOMMITTED INSTANCES
	State TRAN_ACTIVE
	Timeout_period -1
	......

**Outputting statistics information of server processing (;.Hist)**

This command shows the statistics information of server processing. The information is collected after this command is entered. Therefore, the execution commands such as **;.dump_hist** or **;.x** must be entered to output the statistics information.

This command is executable while the **communication_histogram** parameter in the **cubrid.conf** file is set to **yes**. You can also view this information by using the **cubrid statdump** utility. Following options are provided for this session command.

*   **on** : Starts collecting statistics information for the current connection.
*   **off** : Stops collecting statistics information of server.

This example shows the server statistics information for current connection. For information on specific items, see `Outputting Statistics Information of Server <#admin_admin_db_statdump_htm>`_ . ::

	csql> ;.hist on
	csql> ;.x
	Histogram of client requests:
	Name                            Rcount   Sent size  Recv size , Server time
	 No server requests made
	 
	 *** CLIENT EXECUTION STATISTICS ***
	System CPU (sec)              =          0
	User CPU (sec)                =          0
	Elapsed (sec)                 =         20
	 
	 *** SERVER EXECUTION STATISTICS ***
	Num_file_creates              =          0
	Num_file_removes              =          0
	Num_file_ioreads              =          0
	Num_file_iowrites             =          0
	Num_file_iosynches            =          0
	Num_data_page_fetches         =         56
	Num_data_page_dirties         =         14
	Num_data_page_ioreads         =          0
	Num_data_page_iowrites        =          0
	Num_data_page_victims         =          0
	Num_data_page_iowrites_for_replacement =          0
	Num_log_page_ioreads          =          0
	Num_log_page_iowrites         =          0
	Num_log_append_records        =          0
	Num_log_archives              =          0
	Num_log_checkpoints           =          0
	Num_log_wals                  =          0
	Num_page_locks_acquired       =          2
	Num_object_locks_acquired     =          2
	Num_page_locks_converted      =          0
	Num_object_locks_converted    =          0
	Num_page_locks_re-requested   =          0
	Num_object_locks_re-requested =          1
	Num_page_locks_waits          =          0
	Num_object_locks_waits        =          0
	Num_tran_commits              =          1
	Num_tran_rollbacks            =          0
	Num_tran_savepoints           =          0
	Num_tran_start_topops         =          3
	Num_tran_end_topops           =          3
	Num_tran_interrupts           =          0
	Num_btree_inserts             =          0
	Num_btree_deletes             =          0
	Num_btree_updates             =          0
	Num_btree_covered             =          0
	Num_btree_noncovered          =          0
	Num_btree_resumes             =          0
	Num_query_selects             =          1
	Num_query_inserts             =          0
	Num_query_deletes             =          0
	Num_query_updates             =          0
	Num_query_sscans              =          1
	Num_query_iscans              =          0
	Num_query_lscans              =          0
	Num_query_setscans            =          0
	Num_query_methscans           =          0
	Num_query_nljoins             =          0
	Num_query_mjoins              =          0
	Num_query_objfetches          =          0
	Num_network_requests          =          8
	Num_adaptive_flush_pages      =          0
	Num_adaptive_flush_log_pages  =          0
	Num_adaptive_flush_max_pages  =          0
	 
	 *** OTHER STATISTICS ***
	Data_page_buffer_hit_ratio    =     100.00
	csql> ;.h off

**Displaying query execution time (;TIme)**

The **;TIme** session command can be set to display the elapsed time to execute the query. It can be set to **ON** or **OFF**. The current setting is displayed if there is no value specified.

The **SELECT** query includes the time of outputting the fetched records. Therefore, to check the execution time of complete output of all records in the **SELECT** query, use the **--no-pager** option while executing the CSQC interpreter. ::

	$ csql -u dba --no-pager demodb
	csql> ;ti ON
	csql> ;ti
	TIME IS ON

**Displaying query history (;HISTORYList)**

This command displays the list that contains previously executed commands (input) and their history numbers. ::

	csql> ;historyl
	----< 1 >----
	select * from nation;
	----< 2 >----
	select * from athlete;

**Reading input with the specified history number into the buffer (;HISTORYRead)**

You can use **;HISTORYRead** session command to read input with history number in the **;HISTORYList** list into the command buffer. You can enter **;ru** or **;x** directly because it has the same effect as when you enter SQL statements directly. ::

	csql> ;historyr 1

**Calling the default editor (;EDIT)**

This command calls the specified editor. The default editor is **vi** on Linux **Notepad** on Windows environment. Use **;EDITOR_Cmd** command to specify a different editor. ::

	csql> ;edit

**Specifying the editor (;EDITOR_Cmd)**

This command specifies the editor to be used with **;EDIT** session command. As shown in the example below, you can specify other editor (ex: emacs) which is installed in the system. ::

	csql> ;editor_c emacs
	csql> ;edit