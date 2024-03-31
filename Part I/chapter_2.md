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

Document and relational databases are converging in some aspects. Most relational database systems (other than MySQL) support XML, so applications using these systems can use data models similar to document models. PostgreSQL, MySQL, and IBM DB2 also suport JSON. Some document databases support joins in their query languages (e.g. RethinkDB), and some have slightly less efficient workarounds for joins (e.g. MongoDB).

## Query Languages for Data
