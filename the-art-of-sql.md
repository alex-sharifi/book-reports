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

### 
