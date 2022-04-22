## Optimizing and Tuning Spark Applications:  

### Static vs. Dynamic resource allocation - 
- One can use the `spark.dynamicAllocation.enabled` property to use dynamic resource allocation, which scales the number of executors registered with this application up and down based on the workload. Example use cases would be Streaming data, or on-demand analytics where more is asked of the application during peak hours. In a multi-tenant environment Spark may soak up resources from other applications.

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
  

### Configuring Spark executors' memory and shuffle service-

- The amount of memory available to each executor is controlled by `spark.executor.memory`.
  
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
    - Default size is 128 MB in HDFS and S3. A contiguous collection of these blocks is a partition.
    - The size of a partition in Spark is dictated by `spark.sql.files.maxPartitionBytes` which is 128 MB by default.
    - Decreasing the partition file size too much may cause the "small file problem" increasing disk I/O and performance degradation.
- Shuffle Partitions
    - For smaller workloads the shuffle partitions should be reduced from default 200, to number of cores or executors or less.
    - During shuffle operations, spark will spill results to executors' local disks at location specified in `spark.local.directory`. Having performant SSDs for this operation will boost performance.

- When writing, use `maxRecordsPerFile` option to control how many records go into each partition file. This would help mitigate the small or very large file problems.

### Caching and Persistence of Data -  

`Dataframe.cache()`
- cache() will store as many partitions as memory allows. 
- Dataframes can be fractionally cached, but a partition cannot.
- Note: A dataframe is not fully cached until you invoke an action that goes through all the records (eg. count). If you use take(1), only one partition will be cached because catalyst realizes that you do not need to compute all the partitions just to retrieve one record.  

``Dataframe.persist()  ``
- persist(StorageLevel.level) provides control over how your data is cached via StorageLevel. Data on disk is always serlialized using either Java or Kyro.
- MEMORY_ONLY, MEMORY_ONLY_SER, MEMORY_AND_DISK, DISK_ONLY, OFF_HEAP, MEMORY_AND_DISK_SER are different persist levels one can use.

#### When to Cache and Persist- 

- When you want to access large dataset repeatedly for queries and transformations.

#### When not to Cache and Persist-

- Dataframes are too big to cache in memory
- An inexpensive tranformation on a dataframe not requiring frequent use, regardless of size.

### Statistics Collection -  

- Cost-based query optimizer can make use of statistics for named tables and not on arbitrary dataframes or RDDs to make optimization decisions.  
- The statistics should be collected and maintained.

#### Table Level  

```SQL
ANALYZE TABLE table_name COMPUTE STATISTICS
```

#### Column Level  

```SQL
ANALYZE TABLE table_name COMPUTE STATISTICS FOR
COLUMNS column_name1, column_name2, ...
```

Column-level statistics are slower to collect, but provide more information for the cost-based optimizer to use about those data columns. Both types of statistics can help with joins, aggregations, filters, and a number of other potential things (e.g., automatically choosing when to do a broadcast join).

### Spark Joins -

#### Broadcast Hash Join-

- Also known as map-side only join
- By default spark uses broadcast join if the smaller data set is less than 10MB.
- When to use a broadcast hash join -
    - When each key within the smaller and larger data sets is hashed to the same partition by Spark.
    - When one data set is much smaller than the other.
    - When you are not worried by excessive network bandwidth usage or OOM errors because the smaller data set will be broadcast to all Spark executors

#### Shuffle Sort Merge Join-

- Over a common key that is sortable, unique and can be stored in the same partition.
- Sort phase sorts each data set by its desired join key. The merge phase iterates over each key in the row from each dataset and merges if two keys match
- Optimizing the shuffle sort merge join -
    - Create partitioned buckets for common sorted keys or columns on which we want to perform frequent equi-joins. 
    - In case of a column with high cardinality, use bucketing. Else use partitioning.
- When to use a shuffle sort merge join -
    - When each key within two large data sets can be sorted and hashed to the same partition by Spark.
    - When you want to perform only equi-joins to combine two data sets based on matching sorted keys.
    - When you want to prevent Exchange and Sort operations to save large shuffles across the network.

## Notes from the video

[Fine Tuning and Enhancing Performance of Apache Spark Jobs](https://youtu.be/WSplTjBKijU)
Some general points to note-

- Memory increase for executors will increase garbage collection time
- Adding more CPUs can lead to scheduling issues as well as additional shuffles

### Skew

#### How to check?

- Spark UI shows job waiting only for some of the tasks
- Executor missing a heartbeat
- Check partition size of RDDs (row count) while debugging to confirm
- In spark logs check the partition sizes

#### How to handle?

- A fix a ingestion time goes way far
- JDBC
  - When reading from an RDBMS source for using JDBC connectors use option to do partitioned reads. Default fetch size depends on the database that you are reading from
  - Partition column should be numeric that is relatively evenly distributed
  - If no such column is present, you should create on using mod, hash functions
- Already partitioned data (S3 etc.)
  - Read the data and repartition if needed

### Cache/Persist

- Unpersist when done to free up memory for Garbage collection
- For self joins, cache the table to avoid reading the same data and deserialization of data twice
- Don't over persist, it can lead to -
  - Increased spill to disk
  - Slow garbage collection

### Avoid UDFs

- UDFs have to deserialize every row to object
- Then apply the lambda function
- And then reserialize it
- This leads to increased garbage collection

### Join Optimizations

- Filter trick
  - Get keys from the medium table and filter the records from the large table before performing a join
  - Dynamic partition pruning
- Salting
  - Use salting technique to reduce/eliminate skew on the joining keys
    - Salting will use up more memory

### Things to remember

- Keep the largest dataframe at left because Spark tries to shuffle the right dataframe first. Smaller dataframe will lead to lesser shuffle
- Follow good partitioning strategies
- Filter as early as possible
- Try to use the same partitioner between DFs for joins

### Task Scheduling

- Default scheduling is FIFO
- Fair Scheduling
  - Allows scheduling longer tasks with smaller tasks
  - Better resource utilization
  - Harder to debug. Turn off in local when debugging

### Serialization

| Java                     | Kryo                                                 |
| ------------------------ | ---------------------------------------------------- |
| Default for most types   | Default for shuffling RDDs and simple types like int |
| Can work for any class   | For serializable types                               |
| More flexible but slower | Significantly faster and more compact                |
|                          | Set you sparkconf to use kryo serialization          |

### Garbage Collection

#### How to check?

- Check time spent on tasks vs GC on Spark UI
- Check the memory used in server

## Databricks Delta

[Optimize performance with file management](https://docs.databricks.com/delta/optimizations/file-mgmt.html)

### Compaction (Bin packing)

- Improve speed of the queries by coalescing smaller files into larger files

- You trigger compaction by running the following command

  ```bash
  OPTIMIZE events
  ```

- Bin packing optimization is idempotent

- Evenly balanced in size but not necessarily in terms of records. The two measures are more often correlated

- Returns min, max, total and so on for the files removed and added

- Also returns Z-Ordering statistic

### Data Skipping

- Works for any comparison of nature `column 'op' literal` where op could be `>`,`<`,`=`,`like`, `and`, `or` etc.
- By default, generates stats for only the first 32 columns in the data. For more columns, reordering is required. Or the threshold limit can be changed
- Collecting stats on long string is expensive. One can skip long string like columns using `delta.dataSkippingNumIndexedCols`

### Z-Ordering (multi-dimensional clustering)

- Colocates related information in same set of files
- Use for high cardinality columns
- Effectiveness drops with each additional column
- Columns that do not have statistics collected on them, would be ineffective as it requires stats like min, max, count etc. for data skipping
- Z-ordering is not idempotent
- Evenly balanced files in terms of records but not necessarily size. The two measures are more often correlated

For running`OPTIMIZE` (bin-packing or z-ordering) compute intensive machines like c5d series is recommended as both operations will be doing large amounts of Parquet decoding and encoding