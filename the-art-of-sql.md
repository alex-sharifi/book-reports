# The Art of SQL
###### Stephane Faroult, Peter Robson

## Chapter 1: Designing Databases for Performance

### The Relational View of Data
- A database is nothing but an (imperfect) model of a small part of a rich and complex real-life situation. There is rarely a single way to represent an activity, but there is usually one that best meets the business requirement.
- The relational model is named as a reference to the relationships between columns in a table, i.e. if several values belong to the same row in a table, they are _related_.
- Tables (i.e. relations) represent facts we accept as true. The queries we write are new truths that we prove.
- There is no one size fits all. A good design is a design that does not require crazy queries.

### The Importance of Being Normal
- The normalization process is fundamentally based on the application of atomicity to the world you are modelling.
- Step 1: ensure atomicity (1NF). Deciding whether data can be considered atomic or not (i.e. can it be split) is chiefly a question of scale (i.e. a car may be atomic to a dealership; but not atomic to a car garage). If you need to refer to parts of an attribute inside the `where` clause (i.e. by searching for a substring), the attribute lacks the level of atomicity you need. Next identify what uniquely characterises a row - the primary key. This can be a compound key of multiple columns. Once all the attributes are atomic and keys are identified, our data is in 1NF.
- Step 2: check dependency on the whole key (2NF). We may have attributes that depend on only part of the key. This can lead to data redundancy due to attributes not specific to one entry being stored many times. Data redundancy can cause inconsistent data, wasted storage (which gets multiplied by mirrors and back-ups), and scanning duplicate data unneccessarily. Basically remove any attributes that do not contribute to the uniqueness of a row.
- Step 3: check attribute indepdence (3NF). Every pair of attributes in our 2NF data set should be examined in turn to check whether one depends on the other.

### To Be or Not to Be, or to Be Null
- Nulls can be hazardous to your logic; if you must use them, be very sure you understand the consequences of doing so in your particular situation.
- Tables in which specific columns appear as null indicate the need for [subtypes](#understanding-subtypes).
- A sure sign that a database design is flawed is when columns of some prominent tables mostly contain null values, and especially when two columns cannot possibly contain a value at the same time.
- All columns in a row should ultimately contain a value.
- The completeness of a relational model is founded on the application of _two-valued logic_, in which things _are_ or _are not_. Null values are indeterminate, and conditions in a `where` clause cannot be indeterminate. This can cause problems when a subquery potentially returns a null value.

### Qualifying Boolean Columns
- You should aim for increasing the density of your data - a boolean `order_completed` column may be useful information to know, but then it might be nice to store other information too, such as `completion_date` and `completed_by`, both of which tell us more than a single T/F column.

### Understanding Subtypes
- Unncessarily wide tables is a lack of understanding of the true relationship between the data items.
- The manner in which the common attributes can be shared, while ensuring that the distinctive features are kept separate, introduces the topic of subtypes.
- Entities may have some common attributes, but subtypes may have properties unique to each type. For example, employees can be permanent or contract, and each subtype has different properties.
- First create the table that contains information common to every entity (i.e. employee). Second create a table for each subtype (i.e. contract, permanent).
- The unique key for all tables is the unique identifier for each entity (i.e. employee number). The primary keys of the common table is the union of the primary keys of the subtype tables. The primary keys of the subtype tables are also foreign keys referencing the primary key of the common table.

### Stating the Obvious
- Data semantics belong in the DBMS, not in the application programs, i.e. `if column is null, then apply this value`, opening up inconsistent interpretations of data.

### The Dangers of Excess Flexibility
- Entity-attribute-value designs store everything as a character string in `attribute_value`.
- This model avoids the risk of [null values](#to-be-or-not-to-be-or-to-be-null) but your database integrity is totally sacrificed, because you can hardly have a weaker way of typing your data. The simplest query becomes a monstrous join.

### The Difficulties of Historical Data
- Handling data that both accumulates and changes requires very careful design and tactics that vary according to the rate of change. Be careful when designing tables that contain historical/future data as this can involve multiple passes over the same data.

### Design and Performance
- Adding indexes do not belong to the tuning of production databases. Most indexes can and must be defined from the outset as part of the design process.

### Processing Flow
- A data model is not complete until consideration has been taken of data flow.
- When you have massive batch programs, you are mostly interested in throughput - raw effiency, using as much of the hardware resources as possible.
- Processing streams of data may simply be the most efficient way to manage large quantities.

### Centralizing Your Data
- If you need everyone to have the global picture, replication solutions and products (as opposed to remote access) should be considered.

## Chapter 2: Accessing Databases Efficiently

### Query Identification
- Adopt the habit of identifying your programs and critical modules whenever possible by inserting comments into your SQL to help identify where in the program a given query is used and track down any erroneous code.

### Stable Database Connections 
- Database connections and round-trips are like Chinese Walls - the more you have, the longer it takes to recieve the correct message.

### Stable Database Schema
- The use of data definition language (DDL) to create, alter or drop database objects inside an application is bad practice. There is no reason to dynamically create, alter or drop objects.

### Set Processing in SQL
- Any attempt to process a large amount of data in small chunks is usually a bad idea and can be inefficient.

### Action-Packed SQL Statements
- Many relatively complex operations can be accomplished in a single SQL statement. Leave as much as you can to the database optimiser to sort out. PL/SQL is not "closer to the kernel" than an SQL function. The code of user-written functions is beyond the examination of the optimiser.

### Profitable Database Access
- It is inefficient to retrieve data in several separate visits to the database. Do as much work in one operation. Maximise each visit to the database to complete as much work as can reasonably be achieved for every visit.

### Do Only What Is Required
- If an action is required to operate on a number of rows, just do it. i.e. there is no need to check the existence of a value if you are going to update anyway. Just do the update.

### Program Logic Into Queries
- It is preferable to embed as much possible procedural logic as possible within an SQL statement, rather than the host language. Use `CASE` logic in the SQL statement for logic instead of in host application.

### Do Multiple Updates At Once
- In most cases, one update is a lot faster than several separate ones. Apply updates in one fell swoop and try to minimise repeated visits to the same table, i.e. `UPDATE table SET column1=(query1),colum2=(query2) WHERE...`

## Chapter 3: Indexing

### Identify The Entry Points
- Indexes are a technique for achieving the fastest possible access to specific data.
- Indexes come with heavy costs, both in terms of disk space used and processing costs. Maintenance costs for one index may exceed those for one table.
- Primary key columns are indexed automatically simply by virtue of declaration as a primary key.
- In a transactional database, too many indexes is often the sign of an uncertain design.
- When an indexing strategy is used to pull in large quantities of data, the role of the indexes has been misunderstood. Be sure what you are indexing and why you are indexing it.

### Making Indexes Work
- The applicability of an index has long been judged on the percentage of the total data retrieved by a query that uses a key value as the only search criteria. Conventionally, 10% of rows should be matched by the index query.
- In a transactional database, it is useless to index columns with a low number of distinct values.
- _Index-organised tables_ push as much data into the indexes as possible, so data does not need to be retrieved from the table. Some queries can be answered by retrieving only the index data.

### Indexes With Functions And Conversions
- Indexes are usually implemented as tree structures.

## Chapter 4: Thinking SQL Statements

### The Nature of SQL
- We can be confident that relational expressions can be written in different ways and yet return the same result.
- The relational phase consists of correctly identifying the rows that will belong to the result set - not necessarily in order.
- Relational theory operates on sets, and knows nothing of the ordering of these sets.
- We start with a relational core that identifies the set of data we are going to operate on, then move to a non-relational layer which works on this now finite set to produce the final result. Once we have left the relational core in the execution (i.e. imposed ordering,) we no longer have a relation, and we can no longer return to the relational core. It is safest to do as much of the job as possible in the relational layer.
- One should use SQL to express what is required, rather than how that requirement is to be met.
- Try to maximise the amount of true relational processing.

### Views
- A view can be considered shorthand for a query, and this is probably one of the most common usages of views.

### Filtering
- _Correlated subqueries_ have an inner query referring to the current row of the outer query.
- _Uncorrelated subqueries_ are different and better. Just do these.

## Understanding Physical Implementation

### Structural Types
- The base units of data that a DBMS handles are known as pages or blocks.
- We must try to keep the number of data pages that have to be accessed by the database engine as low as possible.
- Reads and writes do not live in harmony: readers want data clustered; concurrent writers want data scattered.

### Forcing Row Ordering
- Even though order is alien to relational theory, it helps to find related rows together rather than scattered over a table, i.e. range searching on time series data.
- Range scanning on clustered data can give impressive performance, but other queries will suffer as a consequence.

### Automatically Grouping Data
- _Partitions_ on tables and indexes splits a large table into manageable chunks.
- _Round-robin partitioning_ involves arbitrarily defining a number of partitions, and data assigned to each partition in a round-robin fashion.
- _Data-driven partitioning_ involves partitioning rows based on values in columns.
- Partitioned tables give us a single table at a logical level with a true primary key, and several columns that are defined as the _partition key_ - their values are used to determine into which partition a row is inserted.
- Partitioning approaches are: _hash partitioning_ (fast access to rows for any specific value; but takes no consideration of distribution of data values), _range partitioning_ (gathers values falling within a certain range), _list partitioning_ (explicitly specify that rows containing a list of possible values for the partition key).
- Partitioning can lead to hot spots (i.e most current data)
- Can be used to cluster data to achieve faster data retrieval; spread data during concurrent writes to avoid hot spots.
- The biggest benefits to queries of table partitioning are obtained when data is uniformly spread in respect to the partitioning key.