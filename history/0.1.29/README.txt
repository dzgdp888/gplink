README file for GPLink
########################################################################################
Site: http://www.PivotalGuru.com
Author: Jon Roberts
Email: jgronline@gmail.com
########################################################################################
GPLink links JDBC connections to Greenplum and Hawq External Tables.

Data is automatically cleansed for embedded carriage returns, newline, and/or null
characters.  Escape characters are retained by double escaping and embedded pipes
are retained by escaping. 

########################################################################################
#Installation:
########################################################################################
- gplink must be installed on a server that is accessible by all nodes of Greenplum
or Hawq.  A dedicated ETL server or the standby master are good candidates for 
hosting gplink.

1.  Download latest version from PivotalGuru.com
2.  Unzip <version>.zip
3.  source gplink_path.sh and add this to your .bashrc file
4.  Edit gplink.properties with correct Greenplum or Hawq connection information
5.  Download 3rd party JDBC drivers and place it in $GPLINK_HOME/jar
6.  Define source configurations in $GPLINK_HOME/connections/ 
7.  Define external table names and columns in $GPLINK_HOME/tables/
8.  Define SQL statements to execute in the source in $GPLINK_HOME/sql/
9.  Create the External Table with gpltable

########################################################################################
#Creating External Tables
########################################################################################
gpltable -s <source_config> -t <target_config> -f <sql> -a <source_table>
example:
gpltable -s sqlserver.properties -t $GPLINK_HOME/gplink.properties -f example.sql -a $GPLINK_HOME/tables/public.test.sql

########################################################################################
#Dropping External Tables
########################################################################################
gpldrop -t <target_config> -n <table_name>
example:
gpldrop -t $GPLINK_HOME/gplink.properties -n public.test

########################################################################################
#Start the gpfdist processes
########################################################################################
gplstart -t <target_config>
example:
gplstart -t $GPLINK_HOME/gplink.properties

Note: this is useful when the host is restarted and you need to start all of the gpfdist
processes needed by gplink External Tables.

########################################################################################
#Debugging
########################################################################################
export GPLINK_DEBUG=true

Turn off debugging:
export GPLINK_DEBUG=

Note: this will show all debug messages from gplstart, gpltable, and gpldrop.

########################################################################################
#Getting data
########################################################################################
The External Table references gpfdist which then executes ext_gpldata.  This does a 
basic parsing of the URL and then calls gpldata.  You never need to call ext_gpldata 
directly but you can call gpldata.  gpldata can be used to debug connections and SQL 
statements prior to creating a table with gpltable.

Usage is gpldata -s <source_config> -f <sql>
example:
gpldata -s connections/sqlserver.properties -f sql/sqlserver_example.sql

########################################################################################
#Known working JDBC connections
########################################################################################
Examples are in the connections directory.  Here are some notes on each connection type
that have been tested.

1.  SQL Server
connectionUrl=jdbc:sqlserver://jonnywin;CODEPAGE=65001;responseBuffering=adaptive;selectMethod=cursor;

You will want to use CODEPAGE 65001 which connects to SQL Server in UTF8 character set.
This allows the JDBC driver to translate the native character set to UTF8 which is used
by Greenplum and HAWQ.

responseBuffering=adaptive greatly improves the speed of exporting data.  Be sure to 
use this.

selectMethod=cursor is needed to tell SQL Server how the data will be fetched.

readCommitted=true performs a read consistent query in the database.  This can cause a 
problem if you aren't using Read Committed Snapshot Isolation and may instead prefer
to use a dirty read.

userName=sa
password=sa

All testing has been done with SQL Server authentication. 

2.  Oracle
connectionUrl=jdbc:oracle:thin:@//jonnywin:1521/XE is an example where the thin driver
is used to connect to the XE instance on port 1521.  

extraProps=defaultRowPrefetch=2000 
Be sure to use this!  By default, Oracle will fetch only 10 rows at a time which makes
exporting slow.  By fetching a larger number of rows, the speed will improve but you
may need to increase the memory settings.

3.  DB2
This is being used by customers but I don't have the details at this time.

4.  Teradata
This is being used by customers but I don't have the details at this time.

5.  Hive
Several jar files are needed to get Hive to work.  Download the following Jar files
from your Hadoop cluster and place it in the jar directory.
 
hadoop-common.jar
hive-jdbc.jar
log4j.jar
slf4j-api.jar
slf4j-log4j12.jar

Testing has been done with a cluster that isn't secure.  Refer to Hive JDBC 
documentation on how to configure your JDBC connection with a secure login.
