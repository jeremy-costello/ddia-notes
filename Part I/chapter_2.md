# Chapter 2: Data Models and Query Languages

Applications layer data models on top of each other.

Key question: how is each layer represented in terms of the next-lower layer? \
(e.g. real world -> data model -> bytes -> hardware)

Each data model has benefits and drawbacks.

## Relational Model Versus Document Model
SQL is the best-known data model today. It is a relational database (relations/tables and tuples/rows).

### The Birth of NoSQL
NoSQL is a non-relational database. The benefits over relation databases are:
1. Greater scalability
2. FOSS
3. Specialized queries
4. More dynamic and expressive

### The Object-Relational Mismatch
OOP and relation databases don't mesh well together. This can be partially mitigated with ORM frameworks like ActiveRecord and Hibernate.

One-to-many relationships are harder to represent in relations databases. Some ways to do this are:
1. A separate table with a key reference
2. Structured datatypes, or queryable XML/JSON
    - Oracle, IBM DB2, MS SQL Server, PostgreSQL, and MySQL support one or both of these
3. Encoding XML/JSON as text

Document-oriented databases (e.g. MongoDB, RethinkDB, CouchDB, Espresso) support the JSON data model.

The JSON representation allows easy one-to-many relationships, but there are other problems with JSON (Chapter 4).

### Many-to-One and Many-to-Many Relationships
Sometimes, columns can use IDs instead of text. Each reusable ID refers to a different object, which can be chosen from a drop-down list or autocompleter. This removes duplication, which is the key idea behind database normalization. It also requires many-to-one relationships. Relational (SQL) databases can use joins for many-to-one relationships, while document (NoSQL) databases have weak support for joins, so joins must be emulated in applicaiton code.

### Are Document Databases Repeating History?
Issues with document databases were first discovered in the 1960s, and are being rediscovered today. Back then, IBM's Information Management System used the hierarchical model, which was similar to JSON. Hierarchical models and document models both make many-to-many relationships difficult, and neither support joins.

The network model was one solution which faded into obscurity, and the relational model (SQL) was the other solution. Network models were the most efficient given hardware in the 1970s, but made querying and updating the database very complicated (e.g. linked list access path). The relational model (SQL) was simple: a relation (table) is a collection of tuples (rows). Relational databases use query optimizers to automatically optimize queries.

### Relational Versus Document Databases Today
This only focuses on differences in the data model. Other differences include their fault-tolerance properties and handling of concurrency.

Advantages of document (NoSQL) models:
1. Schema flexibility
2. Better performance due to locality
3. Often closer to the data structure used by the application

Advantages of relational (SQL) models:
1. Better support for joins
2. Many-to-one and many-to-many relationships

If your data has a document-like structure, use a document model (NoSQL). One unmentioned limitation is that you cannot refer directly to a nested item within a document, and will instead require something like an access path in the hierarchical model.

The best data model depends on the kinds of relationships that exist between data items. Graph models are best for highly interconnected data.

Relational databases have schema, meaning data structure is enforced by the database. Document databases do not have explicit schemas, but data structure is usually assumed in the application code. Relational databases are schema-on-write (similar to static type checking), while document databases are schema-on-read (similar to dynamic type checking). For changing data formats, document databases can write new data in the new format and have application code handle old data, while relational databases can perform migrations.

Migration: ALTER TABLE (very slow in MySQL) and UPDATE (usually slow but can be done on read if necessary)

Schema-on-read (NoSQL) is advantageous if items in the collection don't have the same structure. In these situations, a schema (on write) can hurt more than it helps.

Documents are usually stored as a single string, so the whole document can be accessed at once. If large parts of the document are needed, this will be more efficient than accessing the document split across multiple tables in a relational database. Updates also require rewriting the whole document, so it is recommended to keep documents small. Locality is not limited to the document model, but also available in Google's Spanner database, Oracle (multie-table index cluster tables), and Cassandra/HBase (column-family concept).

Document and relational databases are converging in some aspects. Most relational database systems (other than MySQL) support XML, so applications using these systems can use data models similar to document models. PostgreSQL, MySQL, and IBM DB2 also support JSON. Some document databases support joins in their query languages (e.g. RethinkDB), and some have slightly less efficient workarounds for joins (e.g. MongoDB).

## Query Languages for Data
SQL is a declarative query language. It follows the structure of relational algebra fairly closely. You specify the pattern of the data you want; the query optimizer decides how to achieve that goal. The limited nature of declarative languages gives the database more room for automatic optimizations than imperative languages would. Declarative languages are often better for parallel execution as well.

### Declarative Queries on the Web
CSS is also a declarative language. Using CSS is better than using JavaScript for styling, just like how SQL is better than imperative query APIs.

### MapReduce Querying
MapReduce is a programming model for processing large amounts of data in bulk across many machines. A limited form of MapReduce is support by some NoSQL datastores (e.g. MongoDB, CouchDB). MapReduce is between declarative and imperative. It is based on the "map" and "reduce" functions that exist in many functional programming languages. The "map" and "reduce" functions must be pure functions; they cannot perform additional database queries. SQL can be implemented as a pipeline of MapReduce operations.

Some SQL databases can be extended with JavaScript functions. MapReduce requires two carefully coordinated JavaScript functions. MongoDB has a declarative query language called the aggregation pipeline that is similar in expressiveness to a subset of SQL.

## Graph-Like Data Models
If your application has mostly one-to-many relationships (tree-structured data) or no relationships between records, the document model is appropriate.

If many-to-many relationships are very common in your data, the graph model is more appropriate. Graphs consists of vertices (i.e. nodes/entities) and edges (i.e. relationships/arcs). Well-known algorithms can operate on these graphs (e.g. shortest path, PageRank) Vertices don't have to be the same data object; they can represent many different data objects. Same with edges.

Two popular graph models are the property graph model (Neo4j, Titan, InfiniteGraph) and the triple-store model (Datomic, AllegroGraph, others).

### Property Graphs
Vertices consist of: unique ID, outgoing edges, incoming edges, properties (key-value pairs)
Edges consist of: unique ID, tail vertex, head vertex, relationship label, properties (key-value pairs)

You can view a graph store as two relational tables: one for vertices and one for edges.

Some important aspects of property graphs:
1. Any vertex can have an edge connecting it with any other vertex.
2. Can efficiently find incoming and outgoing edges for each vertex.
3. Can store several different kinds of information in a single graph.

Graphs are very flexible. Examples are different kinds of regional structures in different countries, quirks of history such as a country within a country, and varying granularity of data. Graphs are also good for evolvability; they can be easily extended.

### The Cypher Query Language
Cypher is a declarative query language for property graphs, created for Neo4j. You can query based on incoming edges, outgoing edges, tail vertices, head vertices, relationship labels, properties, or some combination of all of the above. Cypher has a query optimizer like SQL.

### Graph Queries in SQL
Graph data can be represented in a relational database, but it will be much more difficult to query. Relational databases require a known number of joins, while graph databases can traverse a variable number of edges. SQL has added variable-length traversal paths (recursive common table expressions), however the syntax is clumsy in comparison to Cypher.

### Triple-Stores and SPARQL
The triple-store model is mostly equivalent to the property graph model, using different words to describe the same ideas. In a triple-store, all information is stored as a triple of (subject, predicate, object).

Equivalencies to the property graph model are:
1. Subject is a (head) vertex.
2. Predicate and object are the key and value of a property on the subject vertex.
2. Predicate and object are the relational label of the connecting edge and a (head) vertex.

Turtle is one format for representing triples.

The semantic web is the idea that websites should publish information as machine-readable data for computers. The Resource Description Framework (RDF) was an intended format for this machine-readable data. The semantic web is far from being realized in practice. RDF can be written in Turtle or XML, and tools like Apache Jena can convert between different RDF formats. The subject, predicate, and object in RDF triples are usually URIs. SPARQL is a query language for triple-stores using RDF. It is quite similar to Cypher.

### The Foundation: Datalog
Datalog is a language from the 1980s which provided the foundation for SPARQL and Cypher. It is the query language of Datomic, and Cascalog is a Datalog implementation for querying large datasets in Hadoop. Triples in Datalog are written as predicate(subject, object). Datalog allows the definition of reusable rules, which makes it better for complex data but less convenient for simple one-off queries.
