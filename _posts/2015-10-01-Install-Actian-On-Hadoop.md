---
layout: post
title: Set up MapR, Actian dataflow and Squirrel SQL on Windows
---
# Set up MapR, Actian dataflow and Squirrel SQL on Windows

## Background
This document describes how to set up MapR Client, Actian dataflow in KNIME and Squirrel SQL on Windows to connect to Actian 
Vortex database on MapR cluster.

The environment is below.

* Cluster  
  MapR 4.0 on RedHad Linux 
* Actian Vortex  
  Actian Vector in Hadoop and Actian dataflow.
* Windows 7 Desktop

## Source
All source files are in folder C:\opt\dist .  
The folder structure is as below.  

```
	C:\opt
		dist
			extra-sources
				knime_2.11.3.zip
				mapr.zip
				squirrel-sql-3.7.zip
			KNIME2.11.3.MapRTest
			putty
			UnlimitedJCEPolicy
				local_policy.jar
				README.txt
				US_export_policy.jar
			
			actian-dataflow-knime-6.5.2-118-win.x86_64-p65202.zip
			com.actian.dataflow.site-6.5.2-118-site-p65202.zip
			datarush-hadoop-mapr4.jar
			iijdbc-v4.1.8.zip
			mapr-changes.xml
			mapr-client-4.0.1.31009GA-1.amd64.zip
			pvD121-2347414.slc
			squirrel-sql-3.7-standard.jar
			UnlimitedJCEPolicyJDK7.zip
		jdbc
		knime_2.11.3
		mapr
		squirrel-sql-3.7
```
## Install KNIME and Actian Dataflow  
1. Install KNIME with Actian dataflow  
    * Unzip C:\opt\dist\actian-dataflow-knime-6.5.2-118-win.x86_64-p65202.zip to C:\opt  
2. Copy License  
    * copy C:\opt\dist\pvD121-2347414.slc C:\opt\knime_2.11.3
3. Install Actian dataflow patch  
    * copy C:\opt\dist\datarush-hadoop-mapr4.jar C:\opt\knime_2.11.3\plugins\com.pervasive.datarush.hadoop.runtime_6.5.2-118\lib  
    This patch only works on KNIME 2.10 & 2.11. It is Actian dataflow version 6.5.2 build 118.
4. Remove public Server access in KNIME  
    * Run C:\opt\knime_2.11.3\knime.exe  
    * Go to Help -> About KNIME -> Installation Details -> Unintall KNIME Public Server Access in the Installed Software.  
    * Restart KNIME and Quit
5. Copy Java security file  
    * UnlimitedJCEPolicyJDK7.zip is for java 7. So check C:\opt\knime_2.11.3\jre is version 7  
    For java 6 and 8, google JCE for java 6/8.  
    * Unzip C:\opt\dist\UnlimitedJCEPolicyJDK7.zip to C:\opt\dist
    * copy C:\opt\dist\UnlimitedJCEPolicy\* C:\opt\knime_2.11.3\jre\lib\security  
	Three files to copy -
	  * local_policy.jar
	  * README.txt
	  * US_export_policy.jar
6. Now the bundle of KNIME is done.

## Install MapR Client  
1. Unzip mapr client  
    * unzip C:\opt\dist\mapr-client-4.0.1.31009GA-1.amd64.zip to c:\opt\mapr
2. Set up environment variables  
    * System Environment variables:
	
	    ```
		MAPR_HOME=c:\opt\mapr
		JAVA_HOME=C:\opt\knime_2.11.3\jre
		HADOOP_HOME=C:\opt\mapr\hadoop\hadoop-2.4.1
		```
    * Instead, create bat file run-knime.bat ( Optional. To be confirmed.)

		```
		set MAPR_HOME=c:\opt\mapr
		set JAVA_HOME=C:\opt\knime_2.11.3\jre
		set HADOOP_HOME=C:\opt\mapr\hadoop\hadoop-2.4.1
		knime.exe
		```
    * Modify C:\opt\mapr\conf\env.bat (Optional. To be confirmed)

		```
		@echo off
		rem Please set all environment variable you want to be used during MapR cluster
		rem runtime here.
		rem namely MAPR_HOME, JAVA_HOME, MAPR_SUBNETS
		rem The following JAVA_HOME is added by York
		set JAVA_HOME=C:\opt\knime_2.11.3\jre
		rem set MAPR_SUBNETS=
		rem set MAPR_HOME=
		```
3. Run mapr setup  
DON'T RUN IT TWICE!  
This will modify the three xml files. The packages already have the changes, so don't need to run again.

	```
		cd C:\opt\mapr\server
		configure.bat -N maprpoc.osr.nsw.gov.au -c -C bphdpd1:7222 -RM bphdpd1,bphdpd2 -HS bphdpd1

		-N cluster name
		-C CLDB nodes. For multiple CLDB nodes, use bphdpd1:7222,bphdpd2:7222 etc. (check MCE for running nodes)
		-RM Resource manager nodes
		-HS Job History Server
	```
4. Test  
Test hadoop comand line.

	```
	cd C:\opt\mapr\hadoop\hadoop-2.4.1\bin
	hdfs dfs -ls / 
	```
	
## Set up Hadoop configuration files
All changes are from C:\opt\dist\mapr-changes.xml.  
* Change C:\opt\mapr\hadoop\hadoop-2.4.1\etc\hadoop --- For yarn,
* Copy C:\opt\mapr\hadoop\hadoop-0.20.2\conf.new to C:\opt\mapr\hadoop\hadoop-0.20.2\conf -- For mapreduce

1. mapred-site.xml  
Add to both yarn and mapreduce

	```
	  <property>
		<name>mapreduce.app-submission.cross-platform</name>
		<value>true</value>
	  </property>
	```
2. core-site.xml  
Add to yarn and mapreduce
    * Logon all three nodes as root/password
	
	```
	$groupadd -g 10003 operyhg
	$useradd -g 10003 -u 10003 operyhg
	$usermod -s /bin/false operyhg   --- do this for all users 
	$cat /etc/group   -- list 
	$cat /etc/passwd
	Also this can be useful - $getent passwd | grep operyhg
	```
	* Add the following to core-site.xml
	
	```
	  <property>
		<name>hadoop.spoofed.user.uid</name>
		<value>10003</value>
	  </property>
	  <property>
		<name>hadoop.spoofed.user.gid</name>
		<value>10003</value>
	  </property>
	  <property>
		<name>hadoop.spoofed.user.username</name>
		<value>operyhg</value>
	  </property>
	```
3. yarn-site.xml  
	Add to yarn only.
	
	```
	CD C:\opt\mapr\hadoop\hadoop-2.4.1\bin
	yarn classpath  -- show all the classpath
	```
	Copy classpath in ONE LINE! to replace the value below.
	
	```
	<property>
		<name>yarn.application.classpath</name>
		<value>C:\opt\mapr\hadoop\hadoop-2.4.1\etc\hadoop;C:\opt\mapr\hadoop\hadoop-2.4.1\etc\hadoop;C:\opt\mapr\hadoop\hadoop-2.4.1\etc\hadoop;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\common\lib\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\common\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\hdfs;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\hdfs\lib\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\hdfs\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\yarn\lib\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\yarn\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\mapreduce\lib\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\mapreduce\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\yarn\*;C:\opt\mapr\hadoop\hadoop-2.4.1\share\hadoop\yarn\lib\*</value>
	</property>
	```
	
## Package

Now, we can zip c:\opt\mapr and c:\opt\knime-2.11.3 to use on other machines.

## Add Patch to Actian dataflow
1. Install new software

	```
	Open knime,
	HELP-> Install New Software -> Add ->Archive 
	Select C:\opt\dist\com.actian.dataflow.site-6.5.2-118-site-p65202.zip
	(273,194KB)
	Follow the instructions to finish.
	```
2. Copy datarush-hadoop-mapr4.jar
	* Copy C:\opt\dist\datarush-hadoop-mapr4.jar C:\opt\knime_2.11.3\plugins\com.pervasive.datarush.hadoop.runtime_6.5.2-118\lib\

## Test KNIME dataflow project
1. Open Knime from C:\opt\knime-2.11.3  

	```
	File->Preferences->Actian->Module Preferences
	Pick MapR4 for hadoop version.
	```
2. Add remote file system

	```
	File->Preferences->Actian->Remote File System
	Add maprfs:///
	```
3. Set up preferences

	```
	File->Preferences->Actian
	Add profile mapr with the following:
	Parallelism =4
	Minimum Parallelism =2
	Execute in cluster = true
	cluster URL = yarn://bphdpd1.internal.osr.nsw.gov.au:47000
	Save
	Right Click on Job to select job manager
	Configure->Job Manager Selection->Select the job manager for this node: DataFlow Executor
	->Settings for selected job manager: profile -> select mapr->OK
	```
4. Set up connection
	
	```
	Temporary directory maprfs:///tmp
	User/password = actian/password
	```
5. Error
	Delete from the cluster.
	
	```
	$rm -rf /mapr/maprpoc.osr.nsw.gov.au/apps/actian/dataflow/cache/operyhg
	```
	
## Website to monitor
1. Use this site to monitor Actian Vortex  

	```
	http://bphdpd1.internal.osr.nsw.gov.au:47100
	root/password
	```
	
## Install Squirrel SQL
Squirrel SQL is used to query Actian Vortex  
1.  Copy JDBC driver  
extract C:\opt\dist\iijdbc-v4.1.8.zip to C:\opt\jdbc\
2. Squirrel  
extract C:\opt\dist\extra-sources\squirrel-sql-3.7.zip to c:\opt
3. Run  
C:\opt\squirrel-sql-3.7\squirrel-sql.bat

	```
	Drivers tag-> + -> Extra Class Path -> Add -> FileName: C:\opt\jdbc\iijdbc.jar

	Example URL: 
	jdbc:ingres://<hostname>:<VW7>/<database>

	VW7 -- Vector ( for connection in KNIME, use VW)
	VH7 -- Vortex ( for connection in KNIME, use VH)

	Name: Action Vortex

	->List Drivers -> Class Name: com.ingres.jdbc.IngresDriver

	Alias tag-> + -> Add Alias
	Name: Actian Vortex
	Driver: select Actian Vortex
	URL: jdbc:ingres://bphdpd1.internal.osr.nsw.gov.au:VH7/osr_poc
	```

## Works on Server
1. Refer to the following documents.
	* MapR Document
	
	```
	MapR4.0 set up the client
	doc.mapr.com -> mapr4.0-> advanced Installation Topics-> setting up the client
	http://doc.mapr.com/display/MapR40x/Setting+Up+the+Client
	```

	* Actian help doc
	
	```
	http://help.pervasive.com/ -> Installation and Configuration-> Installing and Configuring DataFlow on a YARN-enabled Hadoop Cluster
	http://help.pervasive.com/display/DF652/Installing+and+Configuring+DataFlow+on+a+YARN-enabled+Hadoop+Cluster
	```
	
	* Integrating Dataflow with MapR
	
	```
	http://help.pervasive.com/display/DF652/Integrating+DataFlow+with+MapR
	```
