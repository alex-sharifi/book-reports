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
  - Characteristically consist of fact tables linked to associated dimension tables via primary/foreign key relationships<sup>pg 40</sup>
- Normalized models are good for operational systems that involve minimal update/insert transactions, but too complicated for business users<sup>pg 8</sup>.
- Contains the same information as a normalized model, but provides data in a way that is understandable to users (not operational systems)<sup>pg 8</sup>.

### Fact Tables for Measurements

#### Fact Table Overview

- Stores the performance measurements resulting from an organisations business process events.
- Each row in a fact table corresponds to a measurement event<sup>pg 10</sup>.
- The data on each row is at a specific level of detail (_grain_)<sup>pg 10</sup>.
- All the measurement rows in a fact table must be at the same grain.
- Facts are often described as _continuously valued_ to help sort out what is a fact and what is a dimension attribute<sup>pg 11</sup>.
- Put textual data into dimensions<sup>pg 12</sup>.
- Fact tables tend to go deep in terms of number of rows, but narrow in terms of number of columns<sup>pg 12</sup>.
- All fact tables have two or more _foreign keys_ that relate to the dimension tables' _primary keys_.
- When all the keys in the fact table correctly match their respective primary keys in the corresponding dimension tables, the tables satisfy _referential integrity_.
- Fact table _primary keys_ are composed of a subset of the foreign keys, called a _composite key_.

#### Fact Table Techniques

- Fact table structure
  - Contains numeric measures produced by an operational measurement event in the real world<sup>pg 41</sup>.
  - Based on a _physical activity_ and is not influenced by the eventual reports that may be produced<sup>pg 41</sup>.
- Additive, semi-additive, non-additive
  - Additive can be summed across any of the dimensions associated with the fact table<sup>pg 42</sup>.
  - Semi-additive can be summed across some dimensions<sup>pg 42</sup>.
  - Non-additive measures include ratios (it does not make sense to sum ratios). A good approach for non-additive facts is to store the fully additive components of the non-additive measure and sum these components in the BI application<sup>pg 42</sup>.
- Nulls in fact tables
  - Aggretations in SQL gracefully ignore `null`<sup>pg 42</sup>.
  - Avoid `null` in foreign keys, instead have an 'Unknown' or 'Not Applicable' row in the associated dimension table<sup>pg 42</sup>.

### Dimension Tables for Descriptive Context

- Often have many columns (sometimes 50 to 100).
- The dimension tables _primary key_ which serves as basis for referential integrity with any given fact table to which it is joined.
- _Continuously valued_ numeric observations are almost always facts.
- _Discrete_ numeric observations drawn from a small list are almost always dimension attributes.
- Dimension tables often represent heirarchical relationships - heirarchical descriptive information is stored redundantly in the spirit of ease of use and query performance
  - Resist the urge to normalise data. This normalisation is called _snowflaking_<sup>pg 15</sup>.
  - Although snowflake represents heirarchical data accurately, you should avoid snowflakes because it is difficult for business users to understand and navigate<sup>pg 50</sup>.
  - A flattened denormalised dimension table contains exactly the same information as a snowflaked dimension<sup>pg 50</sup>.
  - Normalising values into separate tables defeats the primary goals of **simplicity and performance**<sup>pg 84</sup>.

#### Dimension Table Techniques

- Dimension table structure
  - Every dimension table has a single primary key column, which is also embedded in any associated fact tables<sup>pg 46</sup>.
  - Wide, flat, denormalised<sup>pg 46</sup>
  - Populated with verbose descriptions<sup>pg 46</sup>. Cryptic abbreviations, operational codes or true/false flags should be replaced with full text words<sup>pg 48</sup>.
  - Rather than decoding flags into understandable labels in the BI application, we prefer that decoded values be stored in the database so they are consistently available to all users regardless of their BI reporting environment<sup>pg 82</sup>.
  - Null-valued dimension attributes should be substituted with a descriptive string, such as 'Unknown' or 'Not Applicable'<sup>pg 48</sup>.
- Degenerate dimensions
  - Some fact tables have dimension keys for dimension tables that bear no context, i.e. invoice number. It is acknowledged there is no associated dimension table with this key<sup>pg 47</sup>.
- Separate heirarchies can gracefull coexist in the same dimension table<sup>pg 48</sup>.

#### Date Dimensions

- Special dimension because it is nearly guaranteed to be in every dimensional model<sup>pg 79</sup>.
- "Date dimension" means a daily grained dimension table<sup>pg 80</sup>.
- Even 20 years' worth of days is only approximately 7,300 rows<sup>pg 80</sup>.
- Explicit date dimensions are preferred (rather than a date key in the fact table as a date type column):
  1. Most database engines are efficient enough at resolving dimensional queries (it is not necessary to avoid joins like the plague)<sup>pg 81</sup>.
  2. Average business users are not versed in SQL date semantics, so provide a wide range of meaningful calendar groupings<sup>pg 81</sup>.
  3. Calendar logic belongs in a dimension table, not in the application code<sup>pg 82</sup>.
  4. There are many date attributes not supported by the SQL date function<sup>pg 82</sup>.
- Time-of-day should be handled as a simple date/time fact in the fact table<sup>pg 83</sup>.

### Facts and Dimensions Joined in a Star Schema

- The most granular or atomic data has the most dimensionality.
- Atomic data should be the foundation for every fact table design to withstand business users' ad hoc attacks.
- You can add completely new dimensions to the schema as long as a single value of that dimension is defined for each existing fact row.
- You can add new facts to the fact table, assuming that the level of detail is consistent with the existing fact table.

### Kimball's DW/BI Architecture

- There are four separate and distinct components to consider in the DW/BI environment
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

### Dimensional Modelling Myths

1. Dimensional Models are for Summary Data Only
  - Should provide queryable access to the most detailed data<sup>pg 30</sup>.
  - Should store substantial history<sup>pg 31</sup>.

2. Dimensional Models are Departmental, Not Enterprise
  - Multiple extracts of the same source data that create multiple, inconsistent analytic databases should be avoided<sup>pg 31</sup>.

3. Dimensional Models are Not Scalable
  - Both normalised and dimensional models can answer exactly the same questions, albeit with varying difficulty (normalised is less user-friendly)<sup>pg 31</sup>.

4. Dimensional Models are for Predictable Usage Only
  - Focus on the organisation's measurement events that are typically stable, unlike analyses that are constantly evolving<sup>pg 31</sup>.
  - The correct starting point for your dimensional models is to express data at the lowest detail possible for maximum flexibility and extensibility<sup>pg 31</sup>.

### More Reasons to Think Dimensionally

- Do not focus on a set of required reports or dashboard gauges. Instead constantly ask about the business process measurement events producing that report or dashboard<sup>pg 32</sup>

## Kimball Modelling Techniques

### Gather Business Requirements and Data Realities

- Understand the source data and business requirements.
- Objectives based on KPIs, decision making processes and analytic needs.


### Four-Step Dimensional Design Process

1. Select the business process
  - i.e. taking an order, processing an insurance claim, etc.
  - Business process events generate or capture performance metrics hat translate into facts in a fact table. Most fact tables focus on the results of a single business process<sup>pg 39</sup>.
  - Frequently expressed as action verbs<sup>pg 70</sup>.
  - Typically supported by an operational system<sup>pg 70</sup>.
  - A series of processes results in a series of fact tables<sup>pg 70</sup>.
  - The performance measurements users want to analyse in the DW/BI system result from business process events<sup>pg 70</sup>.
2. Declare the grain
  - Specifies exactly what a single fact table row represents<sup>pg 39</sup>.
  - Atomic grain refers to the lowest level at which data is captured by a given business process<sup>pg 39</sup>.
  - Each proposed fact table grain results in a separate physical table; different grains must not be mixed in the same fact table<sup>pg 39</sup>.
  - Fact table grain can be made more atomic by adding attributes to an existing dimension table, and then restating the fact table at the lower grain, being careful to preserve the existing column names in the fact and dimension tables<sup>pg 41</sup>.
  - "How do you describe a single row in the fact table?"<sup>pg 71</sup>.
  - Determined by the physical realities of the operational system<sup>pg 71</sup>.
  - Most frequent error is not declaring grain at the beginning of the design process<sup>pg 71</sup>.
  - Although aggregated data plays an important role for performance tuning, it is not a substitute for giving users access to the lowest level details<sup>pg 75</sup>.
  - DW/BI systems almost always demand data expressed at the lowest possible grain, not because queries want to see individual rows but because queries need to cut through the details in very precise ways<sup>pg 75</sup>.
3. Identify the dimensions
  - Descriptive attributes used by BI applications for filtering and grouping facts<sup>pg 40</sup>.
  - New dimensions can be added to an existing fact table by creating new foreign key columns, presuming they don't alter the table's grain<sup>pg 41</sup>.
  - Attributes can be added to an existing dimension table by creating new columns<sup>pg 41</sup>.
  - "How do business people describe the data resulting from the business process measurement events?"<sup>pg 72</sup>.
  - Common dimensions include date, product, customer, employee, and facility<sup>pg 72</sup>.
4. Identify the facts
  - A single fact row has a one-one relationship to a measurement event<sup>pg 40</sup>.
  - Result from a single business process event and are almost always numeric<sup>pg 40</sup>.
  - New facts (consistent with the grain of an existing fact table) can be added by creating new columns<sup>pg 41</sup>.
  - "What is the process measuring?"<sup>pg 72</sup>.
  - Facts that clearly belong to a different grain must be in a separate fact table<sup>pg 72</sup>.
  - Consider both business users' requirements and the realities of the operational source data in tandem<sup>pg 72</sup>.
  - Derived facts
    - New facts calculated by performing calculation on other facts
    - It is recommended derived facts are stored physically, computing it consistently at ETL stage, eliminating possibility of user calculation errors<sup>pg 77</sup>.
    - Avoid performing calculation in BI application because users may access data using different tools<sup>pg 78</sup>.
    - However non-additive metrics need to be calculated in BI application, because calculation cannot be pre-calculated across every dimension slice<sup>pg 78</sup>.
