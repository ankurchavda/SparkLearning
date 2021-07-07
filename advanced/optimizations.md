## Optimizing and Tuning Spark Applications:  

### Static vs. Dynamic resource allocation - 
- One can use the `spark.dynamicAllocation.enabled` property to use dynamic resource allocation, which scales the number of executors registered with this application up and down based on the workload. Example use cases would be Streaming data, or on-demand analytics where more is asked of the application during peak hours. In a multi-tenant environment Spark may soak up recoures from other applications.

    ```python
    spark.dynamicAllocation.enabled true
    spark.dynamicAllocation.minExecutors 2
    spark.dynamicAllocation.schedulerBacklogTimeout 1m
    spark.dynamicAllocation.maxExecutors 20
    spark.dynamicAllocation.executorIdleTimeout 2min
    ```
- Request two executors to be created at start.
- Whenever there are pending tasks that have not been scheduled for over 1 minute, the driver will request a new executor. Max upto 20.
- If an executor is idle for 2 minutes, driver will terminate it.  
  
Configuring Spark executors' memory and shuffle service-
- Executor memory is divided into three sections
    - Execution Memory
    - Storage Memory
    - Reserved Memory
- The default division is 60% for execution, 40% for storage after allowing for 300mb of reserved memory, to safegaurd against OOM errors.
- Execution memory is used for shuffles, joins, sorts and aggregations.
- Storage memory is primarily used for caching user data structures and partitions derived from DataFrames.
- During map and shuffle operations, spark writes to and reads from the local disk's shuffle file, so there's heaving I/O activity. 
- Following configurations can be used during heavy work loads to reduce I/O bottlenecks.
    ```python
    spark.driver.memory
    spark.shuffle.file.buffer
    spark.file.transferTo
    spark.shuffle.unsafe.file.output.buffer
    spark.io.compression.lz4.blockSize
    spark.shuffle.service.index.cache.size
    spark.shuffle.registration.timeout
    spark.shuffle.registration.maxAttempts
    ```

### Spark Parallelism -  
- To optimize resource utilization and maximize parallelism, the ideal is atleast as many partitions as cores on the executor.
- How partitions are created - 
    - Data on disk is laid out in chunks or contiguous file blocks
    - Default size is 128 in HDFS and S3. A contiguous collection of these blocks is a partition.
    - Decreasing the partition file size too much may cause the "small file problem" increasing disk I/O and performance degradation.
    - For smaller workloads the shuffle partitions should be reduced from default 200, to number of cores or executors.
    - Use maxRecordsPerFile option to control how many records go into each partition file. This would help reduce skewed partition.

### Caching and Persistence of Data -  

`Dataframe.cache()`
- cache() will store as many partitions as memory allows. 
- Dataframes can be fractionally cached, but a partition cannot.
- Note: A dataframe is not fully cached until you invoke an action that goes through all the records (eg. count). If you use take(1), only one partition will be cached because catalyst realizes that you do not need to compute all the partitions just to retrieve one record.  

``Dataframe.persist()  ``
- persist(StorageLevel.level) provides control over how your data is cached via StorageLevel. Data on disk is always serlialized using either Java or Kyro.
- MEMORY_ONLY, MEMORY_ONLY_SER, MEMORY_AND_DISK, DISK_ONLY, OFF_HEAP, MEMORY_AND_DISK_SER are different persist levels one can use.

When to Cache and Persist- 
- When you want to access large dataset repeatedly for queries and transformations.
When not to Cache and Persist-
- Dataframes are too big to cache in memory
- An inexpensive tranformation on a dataframe not requiring frequent use, regardless of size.

Statistics Collection -  
- Cost-based query optimizer can make use of statistics for named tables and not on arbitrary dataframes or RDDs to make optimization decisions.  
- The statistics should be collected and maintained.

Table Level  
```SQL
ANALYZE TABLE table_name COMPUTE STATISTICS
```

Column Level  
```SQL
ANALYZE TABLE table_name COMPUTE STATISTICS FOR
COLUMNS column_name1, column_name2, ...
```

### Spark Joins -

Broadcast Hash Join-
- Also known as map-side only join
- By default spark uses broadcast join if the smaller data set is less than 10MB.
- When to use a broadcast hash join -
    - When each key within the smaller and larger data sets is hashed to the same partition by Spark.
    - When one data set is much smaller than the other.
    - When you are not worried by excessive network bandwidth usage or OOM errors because the smaller data set will be broadcast to all Spark executors

Shuffle Sort Merge Join-
- Over a common key that is sortable, unique and can be stored in the same partition.
- Sort phase sorts each data set by its desired join key. The merge phase iterates over each key in the row from each dataset and merges if two keys match
- Optimizing the shuffle sort merge join -
    - Create partitioned buckets for common sorted keys or columns on which we want to perform frequent equi-joins. 
    - In case of a column with high cardinality, use bucketing. Else use partitioning.
- When to use a shuffle sort merge join -
    - When each key within two large data sets can be sorted and hashed to the same partition by Spark.
    - When you want to perform only equi-joins to combine two data sets based on matching sorted keys.
    - When you want to prevent Exchange and Sort operations to save large shuffles across the network.