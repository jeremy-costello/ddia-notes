# Chapter 3: Storage and Retrieval
Databases do two things: store data and give that data back. Chapter 2 discussed how the developer can use queries to store and retrieve data, while this chapter will discuss how the database itself uses a storage engine to store and retrieve data. Two storage systems will be discussed: log-structured and page-oriented.

## Data Structures That Power Your Database
Consider the simplest database: two bash functions of "db_set" (appends a key-value pair to a text file [e.g. CSV]) and "db_get" (retrieves the most recent value for a key).

The "db_set" function has pretty good performance. Many databases use an append-only data file (i.e. log). Real databases face more issues: concurrency control, reclaiming disk space, and handling errors/partially written records.

The "db_get" function has terrible performance. The cost of a lookup is O(n). A more efficient data structure for this is the index. Well-chosen indexes speed up read queries, but every index slows down writes. Indexes are usually chosen manually by the application developer or database administrator.

### Hash Indexes
Key-value stores are similar to dictionaries, which are usually implemented as hash maps. The simplest hash map is from each key to the byte offset location at which the value can be found. This is essentially what Bitcask (the default storage engine in Riak) does. This is well suited to situations where the value for each key is updated frequently. One downside is that all keys must fit in the available RAM. To avoid running out of disk space, the log can be broken into segments of a certain size and these segments can be compacted (i.e. removing duplicate keys). These compacted segments can then be merged and re-compacted. During merging, old segment files can be used to serve read and write requests. Segments are checked for keys in reverse order.

Some implementation details:
1. Binary file formats are better than CSV
2. Deleting key-value pairs requires a tombstone that will be activated during merging
3. In-memory hash maps are lost upon restarting the database
4. Files include checksums so corrupted (partially written) records can be detected and ignored
5. One writer thread. Data file segments are append-only and otherwise immutable, so they can be read concurrently by multiple threads

Benefits of an append-only design are:
1. It uses sequential writes (appending and segment merging), which allows much faster writes
2. It makes concurrency and crash recovery much simpler
3. Merging avoids data file fragmentation

Limitations of an append-only design are:
1. The hash table must fit in memory
2. Range queries are not efficient

### SSTable and LSM-Trees

