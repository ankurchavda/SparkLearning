## Communication Strategies  

#### Big table-to-big table  
- Joining a big table with another big table leads to shuffle join.
- In shuffle join, every node talks every other node and they share data according to which node has a certain key or a set of keys.
- If data is not partitioned well network can become congested with traffic.

#### Big table-to-small table  
- Small enough table to fit into the memory of a worker node with some breathing room.
- Replicate our small dataframe onto every worker node
- Prevents all-to-all communication as earlier.
- Instead we perform it only once at the beginning and let each individual node perform the work.
- The driver first collects the data from executors and then broadcasts it back to the executor. Hence the driver size should be large enough to handle a collect operation.

    |                    Broadcast Join                    | Shuffle Join                                 |
    |:----------------------------------------------------:|----------------------------------------------|
    | Avoids shuffling the larger side                     | Shuffles both sides                          |
    | Naturally handles data skew                          | Can suffer from data skew                    |
    | Cheap for selective joins                            | Can produce unnecessary intermediate results |
    | Broadcasted data needs to fit into memory            | Data can be spilled and read from disk       |
    | Cannot be used for certain joins eg. Full outer join | Can be used for all joins                    |