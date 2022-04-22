## Communication Strategies  

### Big table-to-big table  
- Joining a big table with another big table leads to shuffle join.
- In shuffle join, every node talks with every other node and they share data according to which node has a certain key or a set of keys.
- If data is not partitioned well network can become congested with traffic.

#### Shuffle Sort Merge Join

As the name indicates, this join scheme has two phases: a sort phase followed by a merge phase. The sort phase sorts each data set by its desired join key; the merge phase iterates over each key in the row from each data set and merges the rows if the two keys match.

#### Optimizing the shuffle sort merge join

We can eliminate the `Exchange` step from this scheme if we create partitioned buckets for common sorted keys or columns on which we want to perform frequent equijoins. That is, we can create an explicit number of buckets to store specific sorted columns (one key per bucket). Presorting and reorganizing data in this way boosts performance, as it allows us to skip the expensive `Exchange` operation and go straight to `WholeStageCodegen`.

```scala
// Save as managed tables by bucketing them in Parquet format
usersDF.orderBy(asc("uid"))
    .write.format("parquet")
    .bucketBy(8, "uid")
    .mode(SaveMode.OverWrite)
    .saveAsTable("UsersTbl")

ordersDF.orderBy(asc("users_id"))
    .write.format("parquet")
    .bucketBy(8, "users_id")
    .mode(SaveMode.OverWrite)
    .saveAsTable("OrdersTbl")
```

Bucketing the data using user ids will help skip the expensive `Exchange` step as there is no need to perform sort now.

#### When to use a shuffle sort merge join

Use this type of join under the following conditions for maximum benefit:

- When each key within two large data sets can be sorted and hashed to the same partition by Spark
- When you want to perform only equi-joins to combine two data sets based on matching sorted keys
- When you want to prevent Exchange and Sort operations to save large shuffles across the network

### Big table-to-small table  
- Small enough table to fit into the memory of a worker node with some breathing room.
- Replicate our small dataframe onto every worker node
- Prevents all-to-all communication as earlier.
- Instead we perform it only once at the beginning and let each individual node perform the work.
- The driver first collects the data from executors and then broadcasts it back to the executor. Hence the driver size should be large enough to handle a collect operation.

#### When to use a broadcast hash join

​	Use this type of join under the following conditions for maximum benefit:

- When each key within the smaller and larger data sets is hashed to the same partition
  by Spark
- When one data set is much smaller than the other (and within the default config
  of 10 MB, or more if you have sufficient memory)
- When you only want to perform an equi-join, to combine two data sets based on
  matching unsorted keys
- When you are not worried by excessive network bandwidth usage or OOM
  errors, because the smaller data set will be broadcast to all Spark executors

|                    Broadcast Join                    |                 Shuffle Join                 |
|:----------------------------------------------------:|:--------------------------------------------:|
|           Avoids shuffling the larger side           |              Shuffles both sides             |
|              Naturally handles data skew             |           Can suffer from data skew          |
|               Cheap for selective joins              | Can produce unnecessary intermediate results |
|       Broadcasted data needs to fit into memory      |    Data can be spilled and read from disk    |
| Cannot be used for certain joins eg. Full outer join |           Can be used for all joins          |