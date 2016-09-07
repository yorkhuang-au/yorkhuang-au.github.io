---
layout: post
title: Setup Pyspark
---

### Background
In this post, I will show how to setup pyspark with other packages.

### Hadoop
I use MapR 5.1 sandbox.

ssh into it as root.

Run the following command to check that pyspark is using python2.6.6 by default.

```
/opt/mapr/spark/spark-1.5.2/bin/pyspark --marter yarn-client
```

I am going to install Anaconda python 2.7 as in the next section.

### Python
I use Anaconda2 python 2.7 for pyspark.

* Download link is https://www.continuum.io/downloads.
  Download the Python2.7 64 bit installer Anaconda2-4.1.1-Linux-x86_64.sh.
* Copy it to sanbox.
* Run the following commands.
```
[root@maprdemo ~]# bash Anaconda2-4.1.1-Linux-x86_64.sh
```
Use /opt/anaconda2 for the folder.

Set up update-alternatives for python.
```
rm /usr/bin/python
update-alternatives --install /usr/bin/python python /opt/anaconda2/bin/python 1
update-alternatives --install /usr/bin/python python /usr/bin/python2.6 1
```
Double check that anaconda2 python is active.

* Run pyspark to check that it is using anaconda python2.7 now.

### Spark-avro
I will install spark-avro for avro access.

Download link is http://repo1.maven.org/maven2/com/databricks/spark-avro_2.10/2.0.1/spark-avro_2.10-2.0.1.jar.
Copy to sandbox. /opt/spark-jars/

vi /opt/mapr/spark/spark-1.5.2/conf/spark-defaults.conf
and add the following two lines at the end.
```
spark.driver.extraClassPath /opt/spark-jars/spark-avro_2.10-2.0.1.jar
spark.executor.extraClassPath /opt/spark-jars/spark-avro_2.10-2.0.1.jar
``` 
Otherwise, I have to run pyspark as
```
/opt/mapr/spark/spark-1.5.2/bin/pyspark --marter yarn-client --jars /opt/spark-jars/spark-avro_2.10-2.0.1.jar
```
See the link http://stackoverflow.com/questions/29099115/spark-submit-add-multiple-jars-in-classpath

Test reading avro.
Prepare t.avro.
In pyspark, do the following. See the link https://www.cloudera.com/documentation/enterprise/5-6-x/topics/spark_avro.html#concept_hsz_nvn_45__section_pgn_r45_st.
```
df = sqlContext.sqlContext.read.format("com.databricks.spark.avro").load("/user/mapr/avro-test/t.avro")
a = df.collect()
a[1]
```

### Another reference:
https://github.com/databricks/spark-avro

