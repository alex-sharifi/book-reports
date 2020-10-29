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
- Each row in a fact table corresponds to the most atomic measurement event<sup>pg 10</sup>.
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
    - Atomic data should be the foundation for every fact table design to withstand business users' ad hoc attacks.
- There are just three fundamental types of fact tables; transaction, periodic snapshot and accumulating snapshot<sup>pg 119</sup>. All three serve a useful purpose, and often need two to complement each other.
    - Transaction fact tables are the most fundamental view of operations, at the individual transaction line level<sup>pg 120</sup>. A row exists in the fact table only if a transaction event occurred. Naturally most atomic and enables analysis at extreme detail. However you cannot survive on transactions alone.
    - Snapshot fact tables
        - There is more to life than transactions alone. Some form of a snapshot table to give a more cumulative view of a process often complements a transaction fact table<sup>pg 117</sup>.
        - Periodic snapshot are needed to see the cumulative performance of an operation at regular and predictable intervals<sup>pg 120</sup>, and cover all facts at a given snapshot date<sup>pg 113</sup>. You take a 'picture' of the activity at the end of a given day, week, month. Stacked consecutively into the fact table<sup>pg 120</sup>. Represents an aggregation of the transactional activity that occurred during a time period. Can include non-events. Good for long-running scenarios<sup>pg 139</sup>.
        - Accumulating snapshot are used for processes that have a definite beginning, middle and end. Further changes to a single row are tracked on the same row. Suitable if rows are tracked by serial number, etc. Each fact table row is updated repeatedly until there are no futher updates to apply to a given row. Events that have not happened on a given row are populated with a '0' or null<sup>pg 118</sup>. Have multiple date foreign keys, representing the major events or process milestones (and perhaps an additional date columns that indicates when the snapshot row was last updated)<sup>pg 121</sup>. Easier to calculate lags/time elapsed between milestones than a transaction fact table because you would need to correlate rows to calculate time lapses.
    - Transactions and snapshots are the yin and yang of dimensional designs<sup>pg 122</sup>. Each provide different vantage points on the same story.
- Additive, semi-additive, non-additive
    - Additive can be summed across any of the dimensions associated with the fact table<sup>pg 42</sup>.
    - Semi-additive can be summed across some dimensions<sup>pg 42</sup>. All measures that record a static level (inventory, account balances, room temperatures, etc) are inherently non-additive across the Date dimension and possibly other dimensions<sup>pg 115</sup>. Deltas on the other hand, are fully additive<sup>pg 116</sup>.
    - Non-additive measures include ratios (it does not make sense to sum ratios). A good approach for non-additive facts is to store the fully additive components of the non-additive measure and sum these components in the BI application<sup>pg 42</sup>.
- Nulls in fact tables
    - Aggretations in SQL gracefully ignore `null`<sup>pg 42</sup>. In fact, substituting a zero in a null-valued metric instead would improperly skew aggregated metrics<sup>pg 92</sup>.
    - Avoid `null` in foreign keys, instead have an 'Unknown' or 'Not Applicable' row in the associated dimension table<sup>pg 42</sup>.
    - Referential integrity is violated if you put a null in a fact table column declared as a foreign key to a dimension table<sup>pg 92</sup>.
- Surrogate keys
    - Compulsory for dimension tables, less important for fact tables<sup>pg 102</sup>.
    - Immediate unique identification. Identify single fact table rows without navigating multiple dimensions<sup>pg 103</sup>.
    - Revert bulk loads. Identify exactly where load processes halted, and specify range of keys to revert, or resume from<sup>pg 103</sup>.
    - Using surrogate key as a parent in a parent/child schema. In cases where one fact table contains rows that are _parents_ of those in a lower grain fact table (_child_), the fact table surrogate key in the parent table is also exposed in the child table (the child fact table should also include parent's dimension foreign keys)<sup>pg 103</sup>.
- Extending dimensional design
    - Add a foreign key to the new dimension table in the fact table<sup>pg 96</sup>.
    - Since we should avoud `null` in foreign keys, substitute historical data with a default dimension surrogate key corresponding to 'Prior to New Change' and new data that does not include the attribute with 'Not Applicable'<sup>pg 96</sup>.
    - New measured facts can be added by a new column to the fact table, and values populated (assuming same grain as the existing facts)<sup>pg 97</sup>.
- Factless fact tables
    - To do
- 'Centipede' fact tables
    - A very large number of dimensions (more than 25) typically are a sign that several dimensions are not completely independent and should be combined into a single dimension. For example, it is a mistake to model a fact table with separate keys for `Date`, `Week`, `Month`, `Quarter`, `Year` dimensions or `Product`, `Product Category`, `Package Type` dimensions - these are clearly related attributes so should be included in the same dimension tables<sup>pg 108</sup>.
- Conformed facts
    - If facts live in more than one dimensional model, the underlying definitions and equations for these facts must be the same if they are to be called the same thing<sup>pg 139</sup>.

### Dimension Tables for Descriptive Context

#### Dimension Table Overview

- Provide the "who, what, where, when, why and how: context surrounding a business process event<sup>pg 40</sup>.
- Contain the entry points and descriptive labels that enable the DW/BI system to be leveraged for analysis<sup>pg 40</sup>.
- Often have many columns (sometimes 50 to 100).
- The dimension tables _primary key_ which serves as basis for referential integrity with any given fact table to which it is joined.
- _Continuously valued_ numeric observations are almost always facts.
- _Discrete_ numeric observations drawn from a small list are almost always dimension attributes.
- Dimension tables often represent heirarchical relationships - heirarchical descriptive information is stored redundantly in the spirit of ease of use and query performance.
    - Resist the urge to normalise data. This normalisation is called _snowflaking_<sup>pg 15</sup>.
    - Although snowflake represents heirarchical data accurately, you should avoid snowflakes because it is difficult for business users to understand and navigate<sup>pg 50</sup>.
    - A flattened denormalised dimension table contains exactly the same information as a snowflaked dimension<sup>pg 50</sup>.
    - Normalising values into separate tables defeats the primary goals of **simplicity and performance**<sup>pg 84</sup>.
    - Dimensional model consciously breaks traditional data modelling rules to focus on simplicity and performance, not on transaction processing efficiencies<sup>pg 104</sup>.

#### Dimension Table Techniques

- Dimension table structure
    - Every dimension table has a single primary key column, which is also embedded in any associated fact tables<sup>pg 46</sup>.
    - Wide, flat, denormalised<sup>pg 46</sup>
    - Populated with verbose descriptions<sup>pg 46</sup>. Cryptic abbreviations, operational codes or true/false flags should be replaced with full text words<sup>pg 48</sup>.
    - Rather than decoding flags into understandable labels in the BI application, we prefer that decoded values be stored in the database so they are consistently available to all users regardless of their BI reporting environment<sup>pg 82</sup>.
    - Separate heirarchies can gracefully coexist in the same dimension table<sup>pg 48</sup>.
- Degenerate dimensions
    - Some fact tables have dimension keys for dimension tables that bear no context, i.e. invoice number. It is acknowledged there is no associated dimension table with this key<sup>pg 47</sup>.
    - These can be useful for grouping purposes (i.e. grouping across basket transactions), and also linking back to operational system<sup>pg 93</sup>.
    - Typically sits in a fact table but does not have a corresponding dimension table<sup>pg 94</sup>.
    - Plays an important role in a fact table's primary key<sup>pg 94</sup>.
- Nulls in dimension attribute values
    - When a dimension row has not been fully populated, or when there are attributes that are not applicable to all the dimension's rows<sup>pg 92</sup>.
    - Null-valued dimension attributes should be substituted with a descriptive string, such as 'Unknown' or 'Not Applicable'<sup>pg 48</sup>.
    - Null values disappear in pull-down menus, so it should be made clear that users are excluding these null values<sup>pg 92</sup>.
- Surrogate keys
    - The unique primary key of a dimension table should be a _surrogate key_, rather than relying on the operational source identifier, known as the _natural key_<sup>pg 98</sup>.
    - The DW/BI system needs to claim control of the primary keys of all dimensions, rather than using explicit natural keys<sup>pg 46</sup>.
    - Simply integers that are assigned sequentially as needed to populate a dimension, i.e. first row is assigned a surrogate key of 1, next is assigned key of 2, and so on<sup>pg 98</sup>.
    - The value of the surrogate key has no business significance. Merely serve to join dimension tables to fact tables<sup>pg 98</sup>.
    - Every join between dimension and fact tables should be based on meaningless integer surrogate keys. Avoid using a natural key as the dimension table's primary key<sup>pg 99</sup>.
    - Protects data warehouse from operational system changes. Maintain control over DW/BI environment, rather than being dictated by an operational systems' rule changes for generating, updating, recycling, reusing codes for natural keys. Many operational systems reuse IDs after a period of dormancy, surrogate keys provide a mechanism to differentiate separate instances of the same natural key<sup>pg 99</sup>.
    - Handle unknown conditions. Assign surrogate keys to identify cases not present in operational systems<sup>pg 100</sup>.
- Extending dimensional design
    - You can add completely new dimensions to the schema as long as a single value of that dimension is defined for each existing fact row.
    - Create new dimension table and add another foreign key in the fact table<sup>pg 96</sup>.
    - Likewise, include a 'Not Applicable' row in the new dimension<sup>pg 96</sup>.
    - New dimension attributes can be added to dimension tables as new columns<sup>pg 96</sup>.
    - New dimensions can be gracefully added to existing fact tables by adding a new foreign key column and populating it with values of the primary key from the new dimension<sup>pg 96</sup>.
- Snowflaking
    - Dimension table normalisation is referred to as _snowflaking_<sup>pg 104</sup>.
    - Data modelers from the operational world often feel unconfomfortable working with flattened, denormalised tables<sup>pg 104</sup>.
    - It is true that normalised dimensions are easier to maintain (if an attribute value changes, we only need to change it once in the lookup table, rather than thousands of times in a flattened table). In a DW/BI environment, this repetitive task is taken care of in ETL system<sup>pg 104</sup>.
    - Snowflaked tables make for more complex presentation<sup>pg 105</sup>.
    - Snowflaked tables require more joins so are usually slower to query<sup>pg 105</sup>.
    - Efforts to denormalise dimension tables (snowflaking) to save disk space is usually a waste of time. It is true that repeated, verbose textual descriptors take up more storage than code lookups, but total storage savings are usually in the MB. Meanwhile, fact tables can easily be in the GB size<sup>pg 105</sup>.
    - Impacts usability. Having all dimension attribute values within easy reach makes browsing within a dimension simpler<sup>pg 105</sup>.
    - Snowflaked tables respond fine if there is just one lookup table, but become unwiedly when the user is required to traverse multiple dimension tables<sup>pg 106</sup>.
    - Even if a database vendor claims the horsepower to query a fully normalised dimensional model without query penalties, a logical and easy to use dimensional model is still required<sup>pg 106</sup>.
    - _Outriggers_ (special purpose tables specific to a particular dimension table) can be used sparingly. For example, when an analyst wants to filter and group a dimension table by non-standard calendar attributes (i.e. product introduction date attribute)<sup>pg 107</sup>.
- Date dimensions
    - A special dimension because it is nearly guaranteed to be in every dimensional model<sup>pg 79</sup>.
    - "Date dimension" means a _daily_ grained dimension table<sup>pg 80</sup>.
    - Even 20 years' worth of days is only approximately 7,300 rows<sup>pg 80</sup>.
    - Explicit date dimensions are preferred (rather than a date key in the fact table as a date type column):
        1. Most database engines are efficient enough at resolving dimensional queries (it is not necessary to avoid joins like the plague)<sup>pg 81</sup>.
        2. Average business users are not versed in SQL date semantics, so provide a wide range of meaningful calendar groupings<sup>pg 81</sup>.
        3. Calendar logic belongs in a dimension table, not in the application code<sup>pg 82</sup>.
        4. There are many date attributes not supported by the SQL date function<sup>pg 82</sup>.
    - Time-of-day should be handled as a simple date/time fact in the fact table<sup>pg 83</sup>.
    - Date dimension smart keys: it is common to assign the primary key of a date dimension, and foreign key of a fact table, as a meaningful integer such as yyyymmdd. Although not intended to provide BI applications with a mechanism to bypass a join between fact table and date dimension (and directly query the fact table), it is useful for partitioning fact tables. It allows old data to be removed gracefully<sup>pg 102</sup>.
- Conformed dimensions
    - Shared across business process fact tables<sup>pg 130</sup>.
    - Built once in ETL system and replicated either logically or physically throughout the enterprise DW/BI environment, a policy mandated by CIO<sup>pg 130</sup>.
    - Enable you to combine performance measurements from different business processes in a single report ("drilling across fact tables")<sup>pg 130</sup>. Query each dimensional model separately and then outer-join the query results based on a common dimension attribute. Full outer-join ensures all rows are included in the report, even if they only appear in one set of query results. Also known as multipass, multi-select, multi-fact, or stitch queries, because metrics from different facts tables are brought together.
    - Identical conformed dimensions mean the same thing with every possible fact table to which they are joined. Dimension built once in ETL system and duplicated outward to each dimensional model<sup>pg 131</sup>.
    - Shrunken rollup conformed dimensions conform to the base atomic dimension if the attributes are a strict subset of the atomic dimension's attributes<sup>pg 132</sup>.

### Kimball's DW/BI Architecture

- There are four separate and distinct components to consider in the DW/BI environment<sup>pg 18</sup>:
    - Operational source systems
        - Special purpose applications.
        - Intended for one-record-at-a-time queries.
        - Without any commitment to sharing common data such as product, customer or calendar with other operational systems.
    - ETL systems
        - Everything between the operational source systems and the DW/BI presentation area<sup>pg 19</sup>.
        - Extracting means reading and understanding the source data.
        - Cleansing the data.
        - Combining the data from multiple sources, and de-duplication.
        - Primary mission is to hand off the dimension and fact tables<sup>pg 20</sup>.
        - Splitting/combining columns or joining underlying 3NF table structures into flattened denormalised dimensions.
    - Data presentation layer
        - Data are organised, stored and made available to users, report writers and other analytical BI applications.
        - To be presented, stored and accessed in dimensional schemas.
        - Most finely-grained data, at the most exquisite details, must be available so that users can ask the most precise questions possible.
        - Single fact table for atomic sales metrics, rather than separate similar, but slightly different, databases containing metrics for sales, marketing, logistics, etc.
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

---

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
    - Data involved in calculations should be in fact tables and data involved in constraints, groups and labels should be in dimension tables<sup>pg 86</sup>

### Enterprise Data Warehouse Bus Architecture<sup>pg 123</sup>

- For long-term DW/BI success, you need to use an architected, incremental approach to build the enterprise's warehouse.
- A 'bus' is a common structure to which everything connects and from which everything derives power<sup>pg 124</sup>.
- Ultimately, all the processes of an organisation's value chain create a family of dimensional models that share a comprehensive set of common, conformed dimensions<sup>pg 124</sup>.
- Enterprise data warehouse bus matrix is used to document and communicate the bus architecture.
    - Matrix rows translate into dimensional models, often recognisable by their operational source<sup>pg 126</sup>.
    - Matrix rows should refer to business processes, not derivative reports or analytics<sup>pg 128</sup>.
    - Matrix columns translate the common dimensions used across the enterprise.
    - Matrix columns should not be overly generalised, i.e. use "Customer" or "Employee" or "Supplier" dimensions instead of "Person".
    - Delivers the big picture perspective, regardless of database or technology<sup>pg 127</sup>.
    - Shared dimensions supply potent integration glue, allowing users to drill across processes<sup>pg 127</sup>.
    - Illustrates the importance of identifying experts to serve as data governance leaders for the common dimensions<sup>pg 127</sup>.
- Data governance objectives<sup>pg 137</sup>:
    - Reach agreement on data definitions, labels and domain values so that everyone is speaking the same language.
    - Establishes policies and responsibilities for data quality and accuracy, as well as data security and access controls.
