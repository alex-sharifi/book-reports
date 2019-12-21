# The Data Warehouse Toolkit
###### Ralph Kimball, Margy Ross

## Overview

### Different Worlds of Data Capture and Data Analysis

- The *operational systems* are optimised to process transactions quickly. These systems almost always deal with one transaction record at a time. They predicatably perform the same operational tasks over and over<sup>pg 2</sup>.
- DW/BI users almost never deal with one transaction at a time<sup>pg 2</sup>.

### Goals of DW/BI<sup>pg 3</sup>

- Must make information easily accessible (intuitive to everyone, not only developers, so structures and labels should mimic business users' thought processes). Simple and fast.
- Present information consistently (carefully assembled, common labels and definitions).
- Adaptable to change.
- Present information in a timely way (realistic expectations for what it means to deliver data when there is less time to clean and validate).
- Secure bastion that protects the information assets (effective access controls).
- Authoritative and trustworthy foundation for decision making.
- Accepted by business community.
- Ultimately: *understandable to business users* and fast query *performance*.

### Dimensional Modelling

- Often instantiated in relational database management systems<sup>pg 7</sup>, called _Star Schemas_<sup>pg 8</sup>, which is recommended by the book<sup>pg 9</sup>.
- Normalized models are good for operational systems that involve minimal update/insert transactions, but too complicated for business users<sup>pg 8</sup>.
- Contains the same information as a normalized model, but provides data in a way that is understandable to users (not operational systems)<sup>pg 8</sup>.

### Fact Tables for Measurements

- Stores the performance measurements resulting from an organisations business process events.
- Each row in a fact table corresponds to a measurement event<sup>pg 10</sup>.
- The data on each row is at a specific level of detail (_grain_)<sup>pg 10</sup>.
- All the measurement rows in a fact table must be at the same grain.
- Facts are often described as _continuously valued_ to help sort out what is a fact and what is a dimension attribute<sup>pg 11</sup>.
- Put textual data into dimensions<sup>pg 12</sup>.
- Fact tables tend to go deep in terms of number of rows, but narrow in terms of number of columns<sup>pg 12</sup>.
- All fact tables have two or more _foreign keys_ that relate to the dimension tables' _primary keys_.
- When all the keys in the fact table correctly match their respective primary keys in the corresponding dimension tables, the tables satisfy _referential integrity_.
- Fact table _primary keys_ are composed of a subset of the foreign keys, called a _composite key_

### Dimension Tables for Descriptive Context

- Often have many columns (sometimes 50 to 100).
- The dimension tables _primary key_ which serves as basis for referential integrity with any given fact table to which it is joined.
- _Continuously valued_ numeric observations are almost always facts.
- _Discrete_ numeric observations drawn from a small list are almost always dimension attributes.
- Dimension tables often represent heirarchical relationships - heirarchical descriptive information is stored redundantly *in the spirit of ease of use and query performance*.
  - Resist the urge to normalise data. This normalisation is called _snowflaking_.

### Facts and Dimensions Joined in a Star Schema

- The most granular or atomic data has the most dimensionality.
- Atomic data should be the foundation for every fact table design to withstand business users' ad hoc attacks.
- You can add completely new dimensions to the schema as long as a single value of that dimension is defined for each existing fact row.
- You can add new facts to the fact table, assuming that the level of detail is consistent with the existing fact table.

### Kimball's DW/BI Architecture

- There are four separate and distinct components to consider in the DW/BI environment:
  - Operational source systems
    - Special purpose applications
    - Intended for one-record-at-a-time queries
    - Without any commitment to sharing common data such as product, customer or calendar with other operational systems
  - ETL systems
    - Everything between the operational source systems and the DW/BI presentation area
    - Extracting means reading and understanding the source data
    - Cleansing the data
    - Combining the data from multiple sources, and de-duplication
    - Primary mission is to hand off the dimension and fact tables
    - Splitting/combining columns or joining underlying 3NF table structures into flattened denormalised dimensions
  - Data presentation layer
    - Data are organised, stored and made available to users, report writers and other analytical BI applications
    - To be presented, stored and accessed in dimensional schemas
    - Most finely-grained data, at the most exquisite details, must be available so that users can ask the most precise questions possible
    - Single fact table for atomic sales metrics, rather than separate similar, but slightly different, databases containing metrics for sales, marketing, logistics, etc
  - BI applications

### Alternative DW/BI Architectures

- Independent Data Mart
  - Working in isolation, departmental data addresses departments's analytic requirements.
  - Other departments interested in same source data progresses down same path but ends up with different data and definitions.
  - Numbers rarely match.
  - Never advocated, but approach is prevalent.
