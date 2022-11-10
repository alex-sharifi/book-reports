# The Art of SQL
###### Stephane Faroult, Peter Robson

## Chapter 1: Laying Plans

### The Relational View of Data
- A database is nothing but an (imperfect) model of a small part of a rich and complex real-life situation. There is rarely a single way to represent an activity, but there is usually one that best meets the business requirement.
- The relational model is named as a reference to the relationships between columns in a table, i.e. if several values belong to the same row in a table, they are _related_.
- Tables (i.e. relations) represent facts we accept as true. The queries we write are new truths that we prove.
- There is no one size fits all. A good design is a design that does not require crazy queries.

### The Importance of Being Normal
- Step 1: ensure atomicity. Deciding whether data can be considered atomic or not (i.e. can it be split) is chiefly a question of scale (i.e. a car may be atomic to a dealership; but not atomic to a car garage). If you need to refer to parts of an attribute inside the `where` clause (i.e. by searching for a substring), the attribute lacks the level of atomicity you need. Next identify what uniquely characterises a row - the primary key. This can be a compound key of multiple columns. Once all the attributes are atomic and keys are identified, our data is in 1NF.
- Step 2: check dependency on the whole key (2NF).   
