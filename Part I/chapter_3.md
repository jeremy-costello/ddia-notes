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
A simple change to the previous section: key-value pairs are sorted by key. Call this Sorted String Table, or SSTable for short.

Advantages of SSTables:
1. Merging segments is simple and efficient, even if files are bigger than available memory. Approach is similar to mergesort. Most recent key is kept in the case of duplicates.
2. Don't need an index of all keys, only some keys. Can seek from keys at certain intervals.
3. Can group and compress ranges of key-value pairs. Range can be between indexed keys.

How can we sort the data when incoming write can occur in any order? Tree data structures (e.g. red-black trees or AVL trees) can be used to sort the data in memory.

The storage would be constructed as follows:
1. Add new writes to an in-memory balanced tree structure (i.e. memtable)
2. When the memtable size is above some threshold, write it to disk as an SSTable file. This file becomes the newest database segment. Writes can continue to a new memtable while the SSTable is being written out to disk.
3. To serve read requests, try to find the key in the memtable, then in database segements by recency.
4. Merge and compact the database in the background at some frequency.

The only problem is that the memtable will be lost if the database crashes. This can be mitigated by appending to a log on disk from which the memtable can be restored. This log will be erased every time a memtable is written to the database.

This algorithm is used in the LevelDB and RocksDB storage engines. Similar storage engines are used in Cassandra and HBase (inspired by Google's Bigtable paper). This indexing structure was initially called Log-Structured Merge-Tree (or LSM-Tree). Lucene, an indexing engine for full-text search used by Elasticsearch and Solr, uses a similar method for storing its term dictionary.

Performance optimizations:
1. LSM-tree is slow when looking up keys that do not exist. To optimize this, storage engines often use Bloom filters (a memory-efficient data structure for approximating the contents of a set).
2. Two common strategies for ordering and timing compaction and merging: size-tiered and leveled. Size-tiered merges newer and smaller SSTables into older and larger SSTables. Leveled splits the key range into smaller SSTables and older data is moved into separate "levels".

### B-Trees
The B-Tree is the most common indexing structure. They keep key-value pairs sorted by key. The log-structured indexes we saw earlier break the database down into variable-size segments, while B-trees break the database down into fixed-size blocks or pages (similar to the arrangement of hardware disks). Pages can refer to each other, so pages refer to the range of sub-pages one level deeper in the tree until reaching a page containing individual keys and corresponding values. Branching factor is the number of sub-pages per page.

Adding a key-value pair or updating a value involves finding (and optionally inserting) the key, changing/adding the value, and writing the new page back to disk. If the page is too large, it it split into two half-full pages and the parent page is updated. This ensures the tree is balanced; n keys -> depth O(log n).

LSM-trees append to files, while B-trees modify files in place. Some operations (e.g. splitting) require several pages to be overwritten. This is dangerous if the database crashes in the middle of one of these operations. This is mitigated by having a write-ahead log (WAL), which is an append-only file of each B-tree modification that is used to restore the B-tree after a crash. Careful concurrency control is required if multiple threads are accessing the tree at once. This can be done using latches (lightweight locks).

Some B-tree optimizations include:
1. Use a copy-on-write scheme. Write modified pages to a different location and create a new version of the parent pages.
2. Store abbreviations of keys.
3. Lay out the tree so leaf pages appear in sequential order on disk.
4. Add additional pointers (e.g. to sibling pages on the left/right).
5. B-tree variants such as fractal trees.

### Comparing B-Trees and LSM-Trees
LSM-trees are faster for writes, while B-Trees are faster for reads. This will vary based on the workload, so should be tested for new systems.

Advantages of LSM-trees:
1. They have higher write thoroughput due to: \
    a. They only sequentially write compact SSTable files rather than having to overwrite several pages in the tree. \
    b. They usually have lower write amplification since they only write multiple times during compaction and merging.
2. They can be compressed better since they periodically rewrite SSTables to remove fragmentation.

1a is less important on SSDs, but 1b and 2 are still important.

Downsides of LSM-trees:
1. Compaction can interfere with the performance of ongoing reads and writes.
2. As the database gets bigger, more disk bandwidth is required for compaction.
3. Writes can eventually outpace compaction, leading to an explosion in used disk space.

Each key exists exactly once in B-trees. This is useful for databases that want strong transactional semantics since transaction isolation can be implemented using locks directly attached to the tree.

### Other Indexing Structures
Key-values indexes are like a primary key index in the relational model. A primary key identifies one row (relational model), document (document model), or vertex (graph model). It is common to also have secondary indexes. Relational databases allow this with CREATE INDEX. They are important for efficient joins. Secondary keys are not unique, which can be solved by (1) making each value in the index a list of matching row identifiers, or (2) making each key unique by appending a row identifier to it.

Queries search for keys, but values can be an actual row or a reference to the row stored elsewhere (commonly in a heap file). Using a heap file is efficient for updating a value without changing the key, assuming the new value is of equal or lesser size. If the new value is larger, it must be moved to a new location in the heap and the pointer must be updated to this new location (or a forwarding pointer can be used at the old location).

A clustered index stores the indexed row within the index, which is more performant for reads. In MySQL's InnoDB storage engine, the primary key of a table is always a clustered index. SQL Server allows one clustered index per table. The covering index (or index with included columns) is a compromise between clustered and nonclustered (heap) indexes. Clustered and covering indexes can speed up read, but require additional storage and can add overhead on writes.

If multiple columns/fields must be queried, a multi-column index is required. The most common multi-column index is the concatenated index, which combines fields into one key by appending columns. These can only be sorted in column order (i.e. first column first, etc.). Standard B-tree or LSM-tree indexes can't answer multi-column queries efficiently. One option to deal with this is to use a curve: transform n-dimensional float/integer columns into a single value, which can then be used as the index for a standard B-tree or LSM-tree. A more common alternative is to use a specialized spatial index such as a R-tree. This is done for latitude/longitude data in PostGIS using PostgreSQL's Generalized Search Tree indexing facility. Multi-dimensional indexes can filter by multiple columns simultaneously (e.g. 2D indexes in HyperDex).

Indexes discussed so far allow for exact key querying. Fuzzy key querying can search for similar keys. You can search for synonyms, ignore grammatical variations, nearby occurences of words, and other features based on linguistic analysis. For example, Lucene can search text for words within a certain edit distance (i.e. typos). Lucene uses a SSTable-like strucutre for its term dictionary. The in-memory index is a finite state automaton over the characters in the keys, and can be transformed into a Levenshtein automaton. Other fuzzy search techniques are classified as information retrieval (e.g. document classification and machine learning).

As RAM becomes cheaper, it becomes more viable to keep the entire dataset in memory. Some in-memory key-value stores (e.g. Memcached) are intended for caching and accept data loss, while others aim for durability through special hardware or backups. VoltDB, MemSQL, and Oracle TimesTen are in-memory databases with a relational model. RAMCloud is an open-source and durable in-memory key=value store. Redis and Couchbase provide weak durability. The main advantage of in-memory databases is that they avoid the overheads of encoding in-memory data structures in a form that can be written to disk. They can also provide data models that are difficult to implement with disk-based indexes (e.g. priority queues and set in Redis). Recent research is looking at extending these this architecture to support datasets larger than available memory by evicting the least recently used data to disk.

### Transaction Processing or Analytics?
