---
layout: post
title: Use Threading In Pyspark
---

### Background
In this post, I show you how to create python threading in Pyspark. The pros and cons won't be discussed. Instead, the implementation will be presented.

### Starting with a Pyspark application

A standalone Pyspark application may look like below.

main.py 
```{python}
from pyspark.sql import SparkSession

def main():
    spark = SparkSession.builder.appName('Test_threading').getOrCreate()
    # Create a dataframe
    df = spark.createDataFrame(
        [(1,'a',2.0), 
        (2,'b', 3.0)
        (3,'c', 4.0)],
        ['c1', 'c2', 'c3']
    )
    
    ## Save the dataframe
    df.write.mode('overwrite').save('./output')
    spark.stop()

if __name__ == '__main__':
    main()

```

This code creates a dataframe and save it to output folder.

### Adding Threads

#### Let's use two python threads to save to two different folders

In main.py, we define the thread function - thread_worker(), to do the saving.
In the main function, we creates two threads which will call the thread_worker
respectively.
```{python}
import threading

def thread_worker(df, id):
    df.write.mode('overwrite').save('./output_{0}'.format(id))
    
def main():
    ...
    
    threads = [threading.Thread(target=thread_worker, args=(df, id) for id in range(2)]
    for t in threads:
        t.start()
    
    for t in threads:
        t.join() 
    
```
This will write the dataframe to output_0 and output_1 folders.

#### Share SparkSession within threads

We can also pass the SparkSession object to multiple threads so that each thread can create their 
own spark jobs.

main.py
```{python}

def thread_worker(spark, df, id):
    df_extra = spark.createDataFrame(
        [(1,'aa',12.0 + id), 
        (2,'bb', 13.0 + id),
        (3,'cc', 14.0 + id)],
        ['c1', 'c4', 'c5']
    )
    
    df_out = df.join(df_extra, ['c1'])
    df_out.write.mode('overwrite').save('./output_{0}'.format(id))
    
def main():
    ...
    
    threads = [threading.Thread(target=thread_worker, args=(spark , df, id) for id in range(2)]
    
```

### Summary

Again, I only show the implementation. The pros and cons can be discussed in the future when I have time.

Thanks.
