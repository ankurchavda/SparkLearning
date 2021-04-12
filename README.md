## Spark Learning Guide

### You can use this guide to learn about different components of spark and use this as reference material.
#### This material has been created using multiple sources from the internet and Spark Learning 2.0 book.

1. What is Spark?  
Apache Spark is a cluster computing platform designed to be fast and general-purpose. At its core, Spark is a “computational engine” that is responsible for scheduling, distributing, and monitoring applications consisting of many computational tasks across many worker machines or a computing cluster.
--------------------------
2. What is a Spark Core?  
Spark Core contains the basic functionality of Spark, including components for task scheduling, memory management, fault recovery, interacting with storage systems, and more. Spark Core is also home to the API that defines resilient distributed datasets (RDDs), which are Spark’s main programming abstraction. RDDs represent a collection of items distributed across many compute nodes that can be manipulated in parallel. Spark Core provides many APIs for building and manipulating these collections.
--------------------------
3. Key features of Spark -
- Spark can run over multiple file systems.
- Multiple software systems need not run to achieve a single task because spark provides a lot of capabilities under the hood. A single application can leverage streaming, ML and Spark SQL capabilities of spark.
- Spark has the philosophy of tight integration where each component is designed to interoperate closely. Hence any improvements at the lower level improve all the libraries running over it. 
- Spark offers in-memory computations
--------------------------
4. Major libraries that constitute the Spark Ecosystem -  
Spark MLib- Machine learning library in Spark for commonly used learning algorithms like clustering, regression, classification, etc.
Spark Streaming – This library is used to process real-time streaming data.
Spark GraphX – Spark API for graph parallel computations with basic operators like joinVertices, subgraph, aggregateMessages, etc.
Spark SQL – Helps execute SQL like queries on Spark data using standard visualization or BI tools.
--------------------------
5. What is an RDD?  
An RDD is, essentially, the Spark representation of a set of data, spread across multiple machines, with APIs to let you act on it. An RDD could come from any data source, e.g. text files, a database via JDBC, etc.
Def - "RDDs are fault-tolerant, parallel data structures that let users explicitly persist intermediate results in memory, control their partitioning to optimize data placement, and manipulate them using a rich set of operators."
--------------------------
6. How are RDDs created?  
Spark provides two ways to create RDDs: loading an external dataset and parallelizing a collection in your driver program.
--------------------------
7. What is a partition?  
A partition is a small chunk of a large distributed data set. Spark manages data using partitions that help parallelize data processing with minimal data shuffle across the executors.
--------------------------
8. How is RDD fault-tolerant?  
When a set of operations happen on an RDD the spark engine views these operations as a DAG. If a node processing the RDD crashes and was performing operations X->Y->Z on the RDD and failed at Z, then the resource manager assigns a new node for the operation and the processing begins from X again using the directed graph.
--------------------------
9. Why are RDDs immutable?  
Immutability rules out a big set of potential problems due to updates from multiple threads at once. Immutable data is safe to share across processes.
They're not just immutable but a deterministic function of their input. This plus immutability also means the RDD's parts can be recreated at any time. This makes caching, sharing and replication easy.
These are significant design wins, at the cost of having to copy data rather than mutate it in place. Generally, that's a decent tradeoff to make: gaining the fault tolerance and correctness with no developer effort is worth spending memory and CPU on since the latter are cheap.
A corollary: immutable data can as easily live in memory as on disk. This makes it reasonable to easily move operations that hit the disk to instead use data in memory, and again, adding memory is much easier than adding I/O bandwidth.
--------------------------
10. What are Transformations?  
Spark Transformations are functions that produce a new RDD from an existing RDD. An RDD Lineage is built when we apply Transformations on an RDD. Basic Transformations are - map and filter. After the transformation, the resultant RDD is always different from its parent RDD. It can be smaller (e.g. filter, count, distinct, sample), bigger (e.g. flatMap(), union(), Cartesian()) or the same size (e.g. map).
– Narrow dependency: RDD operations like map, union, filter can operate on a single partition and map the data of that partition to resulting single partition. These kinds of operations which maps data from one to one partition are referred to as Narrow operations. Narrow operations don’t require to distribute the data across the partitions.
Each partition of the parent RDD is used by at most one partition of the child RDD.
– Wide dependency: RDD operations like groupByKey, distinct, join may require to map the data across the partitions in new RDD. These kinds of operations which maps data from one to many partitions are referred to as Wide operations
Each partition of the parent RDD may be depended on by multiple child partitions.
--------------------------
11. What are Actions?  
Actions are RDD operations that produce non-RDD values. They materialize a value in a Spark program. In other words, an RDD operation that returns a value of any type but RDD[T] is an action. They trigger the execution of RDD transformations to return values. Simply put, an action evaluates the RDD lineage graph.
Actions are one of two ways to send data from executors to the driver (the other being accumulators).
Some examples of actions are - aggregate, collect, count, countApprox*, countByValue*, first, fold, foreach, foreachPartition, max, min, reduce, saveAs* actions, e.g. saveAsTextFile, saveAsHadoopFile, take, takeOrdered, takeSample, toLocalIterator, top, treeAggregate, treeReduce
--------------------------
Anatomy of Spark Application: https://luminousmen.com/post/spark-anatomy-of-spark-application

12. What is a driver?  

https://stackoverflow.com/a/24638280

The driver process runs your main() function, sits on a node in the cluster, and is responsible for three things: maintaining information about the Spark Application; responding to a user’s program or input; and analyzing, distributing, and scheduling work across the executors (defined momentarily).
- Prepares Spark Context
- Declares operations on the RDD using Transformations and Actions. Submits serialized RDD graph to master.
--------------------------
13. What is a Task?  
A task is a unit of work that can be run on a partition of a distributed dataset and gets executed on a single executor. The unit of parallel execution is at the task level. All the tasks within a single stage can be executed in parallel.
--------------------------
14. What is Stage?  
A stage is a collection of tasks that can run in parallel. A new stage is created when there is data shuffling. 
--------------------------
15. What is Core?  
A core is a basic computation unit of CPU and a CPU may have one or more cores to perform tasks at a given time. The more cores we have, the more work we can do. In spark, this controls the number of parallel tasks an executor can run.
--------------------------
16. What is Hadoop, Hive, Hbase?  
Hadoop is basically 2 things: a Distributed FileSystem (HDFS) + a Computation or Processing framework (MapReduce). Like all other FS, HDFS also provides us with storage, but in a fault-tolerant manner with high throughput and lower risk of data loss (because of the replication). But, being an FS, HDFS lacks random read and write access. This is where HBase comes into the picture. It's a distributed, scalable, big data store, modelled after Google's BigTable. It stores data as key/value pairs.
Hive: It provides us with data warehousing facilities on top of an existing Hadoop cluster. Along with that, it provides an SQL like interface which makes your work easier, in case you are coming from an SQL background. You can create tables in Hive and store data there. Along with that you can even map your existing HBase tables to Hive and operate on them.
--------------------------
17. What is parquet?    
https://stackoverflow.com/a/36831549/8515731
--------------------------
18. What file systems does Spark support?  
- Hadoop Distributed File System (HDFS).
- Local File system.
- S3
--------------------------
19. What is a Cluster Manager?  
https://spark.apache.org/docs/latest/cluster-overview.html
An external service for acquiring resources on the cluster (e.g. standalone manager, Mesos, YARN). Spark is agnostic to a cluster manager as long as it can acquire executor processes and those can communicate with each other. We are primarily interested in Yarn as the cluster manager. A spark cluster can run in either yarn cluster or yarn-client mode:
yarn-client mode – A driver runs on client process, Application Master is only used for requesting resources from YARN.
yarn-cluster mode – A driver runs inside the application master process, the client goes away once the application is initialized
--------------------------
20. What is yarn?  
https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html
The fundamental idea of YARN is to split up the functionalities of resource management and job scheduling/monitoring into separate daemons. The idea is to have a global ResourceManager (RM) and per-application ApplicationMaster (AM). An application is either a single job or a DAG of jobs. 
The ResourceManager and the NodeManager form the data-computation framework. The ResourceManager is the ultimate authority that arbitrates resources among all the applications in the system. The NodeManager is the per-machine framework agent who is responsible for containers, monitoring their resource usage (CPU, memory, disk, network) and reporting the same to the ResourceManager/Scheduler.
The per-application ApplicationMaster is, in effect, a framework-specific library and is tasked with negotiating resources from the ResourceManager and working with the NodeManager(s) to execute and monitor the tasks.
The ResourceManager has two main components: Scheduler and ApplicationsManager.
The Scheduler is responsible for allocating resources to the various running applications subject to familiar constraints of capacities, queues etc. The Scheduler is pure scheduler in the sense that it performs no monitoring or tracking of status for the application. Also, it offers no guarantees about restarting failed tasks either due to application failure or hardware failures. The Scheduler performs its scheduling function based on the resource requirements of the applications; it does so based on the abstract notion of a Resource Container which incorporates elements such as memory, CPU, disk, network etc.
The Scheduler has a pluggable policy which is responsible for partitioning the cluster resources among the various queues, applications etc. The current schedulers such as the CapacityScheduler and the FairScheduler would be some examples of plug-ins.
The ApplicationsManager is responsible for accepting job-submissions, negotiating the first container for executing the application specific ApplicationMaster and provides the service for restarting the ApplicationMaster container on failure. The per-application ApplicationMaster has the responsibility of negotiating appropriate resource containers from the Scheduler, tracking their status and monitoring for progress.
A good guide to understand how Spark works with YARN - 
- https://youtu.be/N6pJhxCPe-Y
- https://stackoverflow.com/questions/24909958/spark-on-yarn-concept-understanding?  noredirect=1&lq=1
--------------------------
21. What is MapReduce?  
https://www.guru99.com/introduction-to-mapreduce.html
--------------------------
22. Spark vs MapReduce?  
https://medium.com/@bradanderson.contacts/spark-vs-hadoop-mapreduce-c3b998285578
--------------------------
23. What is an Executor?  
An executor is a single JVM process which is launched for an application on a worker node. Executor runs tasks and keeps data in memory or disk storage across them. Each application has its own executors. A single node can run multiple executors and executors for an application can span multiple worker nodes. An executor stays up for the duration of the Spark Application and runs the tasks in multiple threads. The number of executors for a spark application can be specified inside the SparkConf or via the flag –num-executors from command-line.
- Executor performs all the data processing.
- Reads from and writes data to external sources.
- Executor stores the computed data in-memory, cache or on hard disk drives.
- Interacts with the storage systems.
--------------------------
24. What are workers, executors, cores in Spark Standalone cluster?  
https://stackoverflow.com/questions/32621990/what-are-workers-executors-cores-in-spark-standalone-cluster
--------------------------
25. Name types of Cluster Managers in Spark.
The Spark framework supports three major types of Cluster Managers -
Standalone: a basic manager to set up a cluster.
Apache Mesos: generalized/commonly-used cluster manager, also runs Hadoop MapReduce and other applications.
Yarn: responsible for resource management in Hadoop
--------------------------
26. How can you minimize data transfers when working with Spark?  
Minimizing data transfers and avoiding shuffling helps write spark programs that run in a fast and reliable manner. The various ways in which data transfers can be minimized when working with Apache Spark are:
Using Broadcast Variable- Broadcast variable enhances the efficiency of joins between small and large RDDs.
Using Accumulators – Accumulators help update the values of variables in parallel while executing.
The most common way is to avoid operations ByKey, repartition or any other operations which trigger shuffles.
--------------------------
27. What are broadcast variables?  
Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks. They can be used, for example, to give every node a copy of a large input dataset in an efficient manner. Spark also attempts to distribute broadcast variables using efficient broadcast algorithms to reduce communication cost.
Spark actions are executed through a set of stages, separated by distributed “shuffle” operations. Spark automatically broadcasts the common data needed by tasks within each stage. The data broadcasted this way is cached in serialized form and deserialized before running each task. This means that explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in deserialized form is important.
Broadcast variables are created from a variable v by calling SparkContext.broadcast(v). The broadcast variable is a wrapper around v, and its value can be accessed by calling the value method. The code below shows this:

```python
broadcastVar = sc.broadcast([1, 2, 3])
<pyspark.broadcast.Broadcast object at 0x102789f10>
>>> broadcastVar.value
[1, 2, 3]
#After the broadcast variable is created, it should be used instead of the value v in any functions run on the cluster so that v is not shipped to the nodes more than once. In addition, the object v should not be modified after it is broadcast in order to ensure that all nodes get the same value of the broadcast variable (e.g. if the variable is shipped to a new node later).
```
.
--------------------------
28. Why is there a need for broadcast variables when working with Apache Spark?  
These are read-only variables, present in-memory cache on every machine. When working with Spark, usage of broadcast variables eliminates the necessity to ship copies of a variable for every task, so data can be processed faster. Broadcast variables help in storing a lookup table inside the memory which enhances the retrieval efficiency when compared to an RDD lookup ().  
https://stackoverflow.com/questions/26884871/what-are-broadcast-variables-what-problems-do-they-solve
--------------------------
29. What is a Closure?  
The closure is those variables and methods which must be visible for the executor to perform its computations on the RDD (in this case foreach()). Closure is serialized and sent to each executor. The variables within the closure sent to each executor are copies.  
https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka
--------------------------
30. What are Accumulators?  
Accumulators are variables that are "added" to through an associative and commutative "add" operation. They act as a container for accumulating partial values across multiple tasks (running on executors). They are designed to be used safely and efficiently in parallel and distributed Spark computations and are meant for distributed counters and sums.  
https://spark.apache.org/docs/2.2.0/rdd-programming-guide.html#accumulators
--------------------------
31. How can you trigger automatic clean-ups in Spark to handle accumulated metadata?  
You can trigger the clean-ups by setting the parameter ‘spark.cleaner.ttl’ or by dividing the long-running jobs into different batches and writing the intermediary results to the disk.
--------------------------
32. What is the significance of Sliding Window operation?  
Sliding Window controls the transmission of data packets between various computer networks. Spark Streaming library provides windowed computations where the transformations on RDDs are applied over a sliding window of data. Whenever the window slides, the RDDs that fall within the particular window are combined and operated upon to produce new RDDs of the windowed DStream.
--------------------------
33. What is a DStream?  
Discretized Stream is a sequence of Resilient Distributed Databases that represent a stream of data. DStreams can be created from various sources like Apache Kafka, HDFS and Apache Flume. DStreams have two operations –
- Transformations that produce a new DStream.
- Output operations that write data to an external system.
--------------------------
34. When running Spark applications, is it necessary to install Spark on all the nodes of YARN cluster?  
Spark need not be installed when running a job under YARN or Mesos because Spark can execute on top of YARN or Mesos clusters without affecting any change to the cluster.
--------------------------
35. What is the Catalyst framework?  
Catalyst framework is a new optimization framework present in Spark SQL. It allows Spark to automatically transform SQL queries by adding new optimizations to build a faster processing system.

It goes through 4 transformational phases - 
- Analysis
- Logical optimization
- Physical planning
- Code generation

Phase 1: Analysis
Abstract Syntax Tree is generated from dataframe or query
List of column names, datatypes, functions, databases etc. are resolved by consulting internal 'Catalog'.

Phase 2 : Logical Optimization
Comprises of two internal stages.
In Stage 1, Catalyst optimizer will construct a set of multiple plans
In Stage 2, Cost-based optimizer assigns cost to each plan. This may include for example, the process of constant folding, predicate pushdown, projection pruning, boolean expression simplification etc.

Phase 3 : Physical Planning
In this phase, Spark SQL generates an optimal physical plan for the selected logical plan, using physical operators that match those available in the Spark execution engine.

Phase 4 : Code generation
Generation of efficient java byte code to run on each machine. 
Spark acts a compiler facilitated by Project Tungsten for whole stage code generation.
Whole stage code generation is a physical query optimization, getting rid of virtual function calls and employing CPU registers for intermediate data. This generates a compact RDD for final execution.

--------------------------
36. Which spark library allows reliable file sharing at memory speed across different cluster frameworks?  
Tachyon
--------------------------
37. Explain about the different types of transformations on DStreams?  
Stateless Transformations- Processing of the batch does not depend on the output of the previous batch. Examples- map (), reduceByKey (), filter ().
Stateful Transformations- Processing of the batch depends on the intermediary results of the previous batch. Examples- Transformations that depend on sliding windows
--------------------------
37. Explain about the popular use cases of Apache Spark
Apache Spark is mainly used for - 
Iterative machine learning.
Interactive data analytics and processing.
Stream processing
Sensor data processing
--------------------------
38. How can you remove the elements with a key present in any other RDD?  
Use the subtractByKey() function
--------------------------
39. What is the difference between persist() and cache()
persist() allows the user to specify the storage level whereas cache() uses the default storage level.
--------------------------
40. What are the various levels of persistence in Apache Spark?  
Apache Spark automatically persists the intermediary data from various shuffle operations, however, it is often suggested that users call persist () method on the RDD in case they plan to reuse it. Spark has various persistence levels to store the RDDs on disk or in memory or as a combination of both with different replication levels.
The various storage/persistence levels in Spark are -
MEMORY_ONLY
MEMORY_ONLY_SER
MEMORY_AND_DISK
MEMORY_AND_DISK_SER, DISK_ONLY
OFF_HEAP
--------------------------
41. Does Apache Spark provide checkpointing?  
Lineage graphs are always useful to recover RDDs from a failure but this is generally time-consuming if the RDDs have long lineage chains. Spark has an API for checkpointing i.e. a REPLICATE flag to persist. However, the decision on which data to the checkpoint - is decided by the user. Checkpoints are useful when the lineage graphs are long and have wide dependencies.
--------------------------
42. Hadoop uses replication to achieve fault tolerance. How is this achieved in Apache Spark?  
Data storage model in Apache Spark is based on RDDs. RDDs help achieve fault tolerance through lineage. RDD always has information on how to build from other datasets. If any partition of an RDD is lost due to failure, lineage helps build only that particular lost partition.
--------------------------
43. Explain about the core components of a distributed Spark application.
Driver- The process that runs the main() method of the program to create RDDs and perform transformations and actions on them.
Executor –The worker processes that run the individual tasks of a Spark job.
Cluster Manager-A pluggable component in Spark, to launch Executors and Drivers. The cluster manager allows Spark to run on top of other external managers like Apache Mesos or YARN.
--------------------------
44. What do you understand by Lazy Evaluation?  
Spark is intellectual in the manner in which it operates on data. When you tell Spark to operate on a given dataset, it heeds the instructions and makes a note of it, so that it does not forget - but it does nothing, unless asked for the final result. When a transformation like map() is called on an RDD- the operation is not performed immediately. Transformations in Spark are not evaluated until you perform an action. This helps optimize the overall data processing workflow.
--------------------------
45. Define a worker node-
A node that can run the Spark application code in a cluster can be called as a worker node. A worker node can have more than one worker which is configured by setting the SPARK_ WORKER_INSTANCES property in the spark-env.sh file. Only one worker is started if the SPARK_ WORKER_INSTANCES property is not defined.
--------------------------
46. What do you understand by SchemaRDD?  
An RDD that consists of row objects (wrappers around the basic string or integer arrays) with schema information about the type of data in each column.
--------------------------
47. What are the disadvantages of using Apache Spark over Hadoop MapReduce?  
Apache spark does not scale well for compute-intensive jobs and consumes a large number of system resources. Apache Spark’s in-memory capability at times comes a major roadblock for cost-efficient processing of big data. Also, Spark does not have its own file management system and hence needs to be integrated with other cloud-based data platforms or apache Hadoop.
--------------------------
48. What do you understand by Executor Memory in a Spark application?  
Every spark application has same fixed heap size and a fixed number of cores for a spark executor. The heap size is what referred to as the Spark executor memory which is controlled with the spark.executor.memory property of the –executor-memory flag. Every spark application will have one executor on each worker node. The executor memory is a measure of how much memory of the worker node will the application utilize.
--------------------------
49. What according to you is a common mistake apache-spark developers make when using spark?  
Maintaining the required size of shuffle blocks.
Spark developer often makes mistakes with managing directed acyclic graphs (DAG's.)
--------------------------
50. Suppose that there is an RDD named Samplerdd that contains a huge list of numbers. The following spark code is written to calculate the average -

```python
def SampleAvg(x, y):
    return (x+y)/2.0;
    avg = Samplerdd.reduce(SampleAvg);
```
.
----------------------------
50. (A) What is wrong with the above code and how will you correct it?  
Average function is neither commutative nor associative. The best way to compute average is to first sum it and then divide it by count as shown below -
def sum(x, y):
return x+y;
total =Samplerdd.reduce(sum);
avg = total / Samplerdd.count();
However, the above code could lead to an overflow if the total becomes big. So, the best way to compute average is to divide each number by count and then add up as shown below -
cnt = Samplerdd.count();
def divideByCnt(x):
return x/cnt;
myrdd1 = Samplerdd.map(divideByCnt);
avg = Samplerdd.reduce(sum);
--------------------------
51. Explain the difference between Spark SQL and Hive.
Spark SQL is faster than Hive.
Any Hive query can easily be executed in Spark SQL but vice-versa is not true.
Spark SQL is a library whereas Hive is a framework.
It is not mandatory to create a metastore in Spark SQL but it is mandatory to create a Hive metastore.
Spark SQL automatically infers the schema whereas in Hive schema needs to be explicitly declared.
--------------------------
52. What is a Spark Session?
The first step of any Spark Application is creating a SparkSession, which enables you to run Spark code. 
The SparkSession class provides the single entry point to all functionality in Spark using the DataFrame API. 
This is automatically created for you in a Databricks notebook as the variable, spark.
--------------------------
53. Why should one not use a UDF?
UDFs can not be optimized by Catalyst Optimizer. To use UDFs, functions must be serialized and sent to executors. And for Python, there is additional overhead from spinning up a Python interpreter on an executor to run a UDF.
--------------------------
54. What is an UnsafeRow?
The data that is "shuffled" is in a format known as UnsafeRow, or more commonly, the Tungsten Binary Format. UnsafeRow is the in-memory storage format for Spark SQL and DataFrames. Advantages include:
Compactness:
Column values are encoded using custom encoders, not as JVM objects (as with RDDs).
The benefit of using Spark custom encoders is that you get almost the same compactness as Java serialization, but significantly faster encoding/decoding speeds.
For custom data types, it is possible to write custom encoders from scratch.
Efficiency: 
Spark can operate directly out of Tungsten, without first deserializing Tungsten data into JVM objects.
--------------------------
55. What are some best Caching Practices?
Don't cache unless you're sure the dataframe is going to be used multiple times.
Omit unneeded columns to reduce storage footprint
--------------------------
56. Understanding the Spark UI
Use setJobDescription for better tracking in the Spark UI.
Use event timeline to analyze jobs that are taking long time to execute
Event timeline for a stage has various tasks including Executor computing time, which btw should be the most occuring in the timeline. Any other coloured tasks are overhead and should be considered if we want to optimize the process. If there's a lot of overhead time then one should consider creater larger partitions of data.
In Summary Metrics tab we can see the statistics by quartile for the green tasks in the event timeline. Here we should anaylize Duration to see if the partitions are skewed. If the min-max fields show a greater difference, that indicates skewed partitions.
Input Size/Records can also be used in the similar way to analyze where there is a larger difference between the min,median and max sizes of partitions.
Inside the SQL tab, we can click on the hob descriptions that we set. This will lead us to a more explanatory visualization mapped to the actual code that we wrote.
--------------------------
57. Shared resources 
Executors share machine level resources. That is if node has 4 executors, all the resources would be shared between those.
Tasks share executor level resources.
Resources are shared by the cores in a single node. Meaning they share the memory, disk, network. If any of the cores under an executor fail because of OOM or any other reason, the whole executor will be affected and the process on that executor will have to be stopped.
--------------------------
58. Local and Global Results -  
When certain actions and tranformations are performed there are scenarios when the tasks operate on a partition individually, and then the same operation needs to be performed again globally to get the accurate results. For example if 5 executors give record count of each partition to be 4,5,5,6,4 then a final global count operation is needed to say that the dataset has 24 records. More such operations are -

|Stage 1         |    Stage 2 |
|----------------|--------------------|
|Local Filter    |    No Global Filter|
|Local count     |    Global Count|
|Local distinct  |    Global distinct|
|Local sort      |    Global sort|
|Local aggregate |    Global aggregate|
--------------------------
59. What is shuffling?
Shuffling is the process of rearranging data within a cluster between stages.
Triggered by wide transformations like -  
Repartition  
ByKey opertaions (except counting)  
Joins, the worse being cross joins  
Sorting  
Distinct  
GroupBy  
--------------------------
60. What is a dataframe?
A dataframe is a distributed collection of data grouped into named columnns.
--------------------------
61. Why not Dataframes and not RDDs?
The computations are not knows to Spark when it happens under an RDD. Whether you are performing a join, filter, select or aggregation, Spark only sees it as a lamba expression. Even the Iterator[T] dataype is not visible to spark. That leaves no room for Spark to perform optimizations
--------------------------
62. Why should you always define your schema upfront when reading a file?  
- You relieve Spark from the onus of inferring data types.
- You prevent Spark from creating a separate job just to read a large portion of
your file to ascertain the schema, which for a large data file can be expensive and
time-consuming.
- You can detect errors early if data doesn’t match the schema.

--------------------------
63. Managed vs Unmanaged tables?  
For a managed table, spark manages both the data and the metadata for the table. While for unmanaged data, spark only manages the metadata. So for a command like `DROP TABLE` spark will only delete the metadata for the managed table.

Unmanaged tables in Spark can be created like this -

```python
(flights_df
.write
.option("path", "/tmp/data/us_flights_delay")
.saveAsTable("us_delay_flights_tbl"))
```

--------------------------
64. How can you speed up Pyspark UDFs?  
One can create Pandas UDF using the pandas_udf decorator.

Before introduction of Pandas UDF - 
- Collect all rows to Spark driver
- Each row serialized into python's pickle format and sent to python worker process.
- Child process unpickles each row into huge list of tuples.
- Pandas dataframe created using `pandas.DataFrame.from_records()`
This causes issues like - 
- Even using Cpickle, Python serialization is a slow process
- `from_records` iterates over the list of pure python data and convert each value to pandas format.

Introduction of Arrow - 
- Once data is in Arrow format there is no need for pickle/serialization as Arrow data can be directly sent to Python.
- PyArrow in python utilizes zero-copy methods to create a `pandas.DataFrame` from entire chunks of data instead of processing individual scalar values. 
- Additionally, the conversion to Arrow data can be done on the JVM and pushed back for the Spark executors to perform in parallel, drastically reducing the load on the driver.

The use of Arrow when calling `toPandas()` needs to be enabled by setting `spark.sql.execution.arrow.enabled` to `true`.  

--------------------------  

#### Optimizing and Tuning Spark Applications:  

Static vs. Dynamic resource allocation - 
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

Spark Parallelism -  
- To optimize resource utilization and maximize parallelism, the ideal is atleast as many partitions as cores on the executor.
- How partitions are created - 
    - Data on disk is laid out in chunks or contiguous file blocks
    - Default size is 128 in HDFS and S3. A contiguous collection of these blocks is a partition.
    - Decreasing the partition file size too much may cause the "small file problem" increasing disk I/O and performance degradation.
    - For smaller workloads the shuffle partitions should be reduced from default 200, to number of cores or executors.

Caching and Persistence of Data -  

Dataframe.cache()
- cache() will store as many partitions as memory allows. 
- Dataframes can be fractionally cached, but a partition cannot.
- Note: A dataframe is not fully cached until you invoke an action that goes through all the records (eg. count). If you use take(1), only one partition will be cached because catalyst realizes that you do not need to compute all the partitions just to retrieve one record.  

Dataframe.persist()  
- persist(StorageLevel.level) provides control over how your data is cached via StorageLevel. Data on disk is always serlialized using either Java or Kyro.
- MEMORY_ONLY, MEMORY_ONLY_SER, MEMORY_AND_DISK, DISK_ONLY, OFF_HEAP, MEMORY_AND_DISK_SER are different persist levels one can use.

When to Cache and Persist- 
- When you want to access large dataset repeatedly for queries and transformations.
When not to Cache and Persist-
- Dataframes are too big to cache in memory
- An inexpensive tranformation on a dataframe not requiring frequent use, regardless of size.

Spark Joins -

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
--------------------------
*General Perfomance guidelines - *  
One Executor per node is considered to be more stable than two or three executors per node as is used in systems like YARN.
Try to group wide tranformations together for best automatic optimization 

