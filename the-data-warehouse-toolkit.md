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
    - _Continuously valued_ numeric observations are almost always facts.
    - If the numeric attributes are summarized rather than simply constrained upon, they belong in a fact table<sup>pg 265</sup>.
    - Fact tables are typically limited to foreign keys, degenerate dimensions and numeric facts<sup>pg 278</sup>.
- There are just three fundamental types of fact tables; transaction, periodic snapshot and accumulating snapshot<sup>pg 119</sup>. All three serve a useful purpose, and often need two to complement each other.
    - Transaction fact tables are the most fundamental view of operations, at the individual transaction line level<sup>pg 120</sup>. A row exists in the fact table only if a transaction event occurred. Naturally most atomic and enables analysis at extreme detail. However you cannot survive on transactions alone.
    - Snapshot fact tables
        - There is more to life than transactions alone. Some form of a snapshot table to give a more cumulative view of a process often complements a transaction fact table<sup>pg 117</sup>.
        - Periodic snapshot are needed to see the cumulative performance of an operation at regular and predictable intervals<sup>pg 120</sup>, and cover all facts at a given snapshot date<sup>pg 113</sup>. You take a 'picture' of the activity at the end of a given day, week, month. Stacked consecutively into the fact table<sup>pg 120</sup>. Represents an aggregation of the transactional activity that occurred during a time period. Can include non-events. Good for long-running scenarios<sup>pg 139</sup>.
        - Accumulating snapshot are used for processes that have a definite beginning, middle and end.
            - Preferably short-lived processes<sup>196</sup>.
            - Further changes to a single row are tracked on the same row. Suitable if rows are tracked by serial number, etc. Fundamental difference between accumulating snapshot and other fact tables is that existing fact table rows are revisited and updated as more information becomes available<sup>pg 195</sup>.
            - Each fact table row is updated repeatedly until there are no futher updates to apply to a given row.
            - Events that have not happened on a given row are populated with a '0' or null<sup>pg 118</sup>, with '0' mapping to 'Unknown' or 'To Be Determined' in the date dimension table<sup>pg 195</sup>.
            - Have multiple date foreign keys, representing the major events or process milestones (and perhaps an additional date columns that indicates when the snapshot row was last updated)<sup>pg 121</sup>.
            - Easier to calculate lags/time elapsed between milestones than a transaction fact table because you would need to correlate rows to calculate time lapses.
            - Used for analysing the entire transaction pipeline, because it would be much harder to calculate average days between milestones in a transaction event fact table</sup>pg 194</sup>.
            - Reuse of conformed dimensions is to be expected<sup>pg 194</sup>.
            - Lag calulcations between events should be calculated in ETL system rather than views because they are better at incorporating intelligence like workday lags, accounting for weekends and holidays<sup>pg 196</sup>.
            - This kind of design is successful when 90 percent or more of the events progress through the same steps without any unusual exceptions, but there is no good way to reveal what happened when an event deviates from the standard scenario<sup>pg 255</sup>. Unusual departures from the standard scenario are described by a Status dimension on the accumulating snapshot fact table: tag the fact row with the Status 'Weird', and join back onto fact table using companion Keys<sup>pg 256</sup>. 
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
- Fact normalisation
    - Some designers want to normalise a fact table so there is a single, generic fact amount along with a dimension that identifies the type of measurement. In this scenario, the fact table granularity is one row per measurement per transaction (or whatever the declared grain is), instead of a more natural one row per transaction<sup>pg 169</sup>
    - This would only make sense when the set of facts is extremely lengthy but sparsely populated, but facts are usually not sparsely populated within a row.
    - Arithmetic performed between facts is far easier if the facts are in the same row in a relational star schema.
- Dates in fact tables
    - Sometimes you discover other dates associated with each transaction, such as requested ship date. Each of the dates should be a foreign key in the fact table<sup>pg 170</sup>.
    - Usually these dates are known when transaction occurs</sup>pg 188</sup>.
    - Multiple dates in fact tables allow users to filter, group and trend on any of the dates provided<sup>pg 188</sup>.
    - Just because a fact table has several dates does not mean that it is an accumulating snapshot. The primary differentiator of an accumulating snapshot is that fact rows are revisited as activity occurs.
    - In general, "to-date" totals should be calculated in BI software, not stored in the fact table<sup>pg 206</sup>.
- Transaction facts at different granularity
    - Common in header/line operational data to encounter facts of different granularity. For example, shipping charges on an order line fact table<sup>pg 185</sup> or multiple allowances applied to a single invoice line<sup>pg 190</sup>. First response should be to try to force all facts down to lowest level, broadly referred to as _allocating_<sup>pg 184</sup>.
    - Do not mix fact granularities. Either allocate the higher-level facts to a more detailed level, or create two separate fact tables to handle the differently grained facts<sup>pg 185</sup>.
    - There are several rather unappealing options to handling facts at different granularity:
        - Repeat the unallocated header fact on every line, but runs risk of overstating header amount when summed on every line<sup>pg 186</sup>.
        - Store the unallocated amount on the transaction's first or last line. Reduces risk of overcounting but requires query constraints to filter to first or last line<sup>pg 186</sup>.
        - Set up a special product key for the header fact, but makes dimensional model harder to interpret<sup>pg 186</sup>.
    - Transactions at progressively lower levels of detail should be captured in separate fact tables with additional dimensionality<sup>pg 207</sup>.
- Consolidated fact tables
    - If drill-across analysis of multiple fact tables is extremely common in the user community, it likely makes sense to create a single fact table that combines the metrics once rather than relying on business users or their BI reporting applications to stitch together result sets<sup>pg 224</sup>.
    - Fact tables that combine metrics from multiple business processes at a common granularity are referred to as _consolidated fact tables_<sup>pg 225</sup>.
    - They often represent a dimensionality compromise as they consolidate facts from at the 'least common denominator' of dimensionality. Consolidated fact table facts must live at the same level of granularity and dimensionality. You are often forced to eliminate or aggregate some dimensions<sup>pg 225</sup>.
    - If business users are frequently combining data from multiple business processes, a final approach is to define an additional fact table that combines the data once into a consolidated fact table rather than relying on users to consistently and accurately combine the data on their own<sup>pg 260</sup>.
   - Avoid fact-to-fact table joins. Be very careful when simultaneously joining a single dimension table to two fact tables of different cardinality. In many cases, relational engines return the "wrong" answer<sup>pg 260</sup>.

### Dimension Tables for Descriptive Context

#### Dimension Table Overview

- Provide the "who, what, where, when, why and how: context surrounding a business process event<sup>pg 40</sup>.
- Contain the entry points and descriptive labels that enable the DW/BI system to be leveraged for analysis<sup>pg 40</sup>.
- Often have many columns (sometimes 50 to 100).
- The dimension tables _primary key_ (surrogate key) which serves as basis for referential integrity with any given fact table to which it is joined.
- _Discrete_ numeric observations drawn from a small list are almost always dimension attributes.
- Dimension tables often represent heirarchical relationships - heirarchical descriptive information is stored redundantly in the spirit of ease of use and query performance.
    - Resist the urge to normalise data. This normalisation is called _snowflaking_<sup>pg 15</sup>.
    - Although snowflake represents heirarchical data accurately, you should avoid snowflakes because it is difficult for business users to understand and navigate<sup>pg 50</sup>.
    - A flattened denormalised dimension table contains exactly the same information as a snowflaked dimension<sup>pg 50</sup>.
    - Normalising values into separate tables defeats the primary goals of **simplicity and performance**<sup>pg 84</sup>.
    - Dimensional model consciously breaks traditional data modelling rules to focus on simplicity and performance, not on transaction processing efficiencies<sup>pg 104</sup>.
- Document the attribute definitions, interpretations and origins in the metadata (metadata is analogous to the DW/BI encyclopedia). Be vigilant about populating<sup>pg 173</sup>.
- The customer dimension is typically the most challenging dimension for any DW/BI system:
    - Extremely deep (many millions of rows), extremely wide (hundreds of attributes), subject to rapid change, and often represents an amalgamation of data from multiple internal and external source systems<sup>pg 233</sup>.
    - The data warehouse is the foundation that supports the panoramic 360-degree view of your customers<sup>pg 232</sup>.
    - You can build a single customer dimension that us the "best of breed" choice among a number of available customer data sources, and is likely a distillation of data from several operational systems within the organisation<sup>pg 256</sup>. The attributes in the customer dimension should represent the "best" source available in the enterprise<sup>pg 257</sup>.
    - Effective consolidation of customer data depends on a balance of capturing the data as accurately as possible in the source systems, coupled with powerful data cleaning/merging tools in the ETL process<sup>pg 258</sup>.

#### Dimension Table Techniques

- Dimension table structure
    - Every dimension table has a single primary key column, which is also embedded in any associated fact tables<sup>pg 46</sup>.
    - Wide, flat, denormalised<sup>pg 46</sup>.
    - Populated with verbose, descriptive columns<sup>pg 46,172</sup>. Cryptic abbreviations, operational codes or true/false flags should be replaced with full text words<sup>pg 48</sup>.
    - Rather than decoding flags into understandable labels in the BI application, we prefer that decoded values be stored in the database so they are consistently available to all users regardless of their BI reporting environment<sup>pg 82</sup>. Columns in a dimension are the sole source of query constraints and report labels so should be legible<sup>pg 173</sup>. Quality check the attribute values to ensure no misspellings, impossible values or multiple variations, and are completely populated (SQL will produce another line in a report if the attribute values varies in any way based on trivial punctuation or spelling differences)<sup>pg 173</sup>.
    - Separate heirarchies can gracefully coexist in the same dimension table<sup>pg 48</sup>. Hierarchical data should be presented in a single, flattened denormalized table<sup>pg 172</sup>.
    - Millions of rows in a dimension table is considered large<sup>pg 176</sup>.
    - 25 dimensions in a single dimensional model is considered large, and model should consider combining dimensions<sup>pg 176</sup>.
    - You certainly do not want to have as many rows in a fact table as you do in a related dimension table<sup>pg 264</sup>.
    - If the cardinality of a dimension table is less than the number of transactions in the fact table, attribute values should be captured in a separate dimension table. If there is a unique attribute value for every event, it is treated as a transaction-grained dimension attribute.
- Degenerate dimensions
    - Some fact tables have dimension keys for dimension tables that bear no context, i.e. invoice number. It is acknowledged there is no associated dimension table with this key<sup>pg 47</sup>.
    - These can be useful for grouping purposes (i.e. grouping across basket transactions), and also linking back to operational system<sup>pg 93,178</sup>.
    - Typically sits in a fact table but does not have a corresponding dimension table<sup>pg 94</sup> and reserved for operational transaction identifiers<sup>pg 178</sup>.
    - Plays an important role in a fact table's primary key<sup>pg 94</sup>.
    - The natural operational ticket number, such as the POS transaction number, sits by itself in the fact table without joining to a dimension table. Degenerate dimensions are very common when the grain of a fact table represents a single transaction or transaction line because the degenerate dimension represents the unique identifier of the parent<sup>pg 95</sup>.
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
    - Might be necessary to integrate product information sourced from different operation systems<sup>pg 173</sup>.
- Natural keys
    - Created by the operational source systems, subject to business rules outside the control of the DW/BI system, i.e. employee number<sup>pg 47</sup>.
    - When the data warehouse wants to have a single key for an entity (i.e. an employee), a new _durable key_ (sometimes known as _durable supernatural key_ must be created that is persistent, does not change and is independent of the original business process<sup>pg 48</sup>.
    - While multiple surrogate keys may be associated with a row as the profile changes (i.e. an employee), the durable key never changes<sup>pg 49</sup>.
    - Often modelled as an attribute in the dimension table<sup>pg 100</sup>.
    - In a dimension table with Type 2 slowly changing dimensions, it is important to have an identifier that uniquely and reliably indentifies the dimension entity across its attribute changes<sup>pg 101,265</sup>.
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
- Outrigger dimensions
    - Though outriggers are permissible, a dimensional model should not be littered with outriggers given the potentially negative impact. Outriggers should be the exception rather than the rule<sup>pg 107</sup>.
    - Outriggers almost always makes the user presentation more complex<sup>pg 243</sup>.
    - There are several reasons for bending the "no snowflake" rule:
        - If dimension data are available at significantly different grain than primary dimension data (i.e. demographic/socio-economic attributes relating to county of residence) and is not as analytically valuable<sup>pg 244</sup>.
        - Loaded at different times than the rest of the data<sup>pg 244</sup>.
        - Save significant space if the underlying dimension is large<sup>pg 244</sup>.
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
    - In the ideal case, you examine all the data sources and define a single comprehensive dimension which you attach to all the data sources. In the extreme integration world with dozens of related dimensions of different granularity and different quality, such a single comprehensive dimension is impossible to build<sup>pg 258</sup>. Fortunately you can implement a lighter-weight kind of conformed dimension<sup>pg 258</sup>.
    - The essential requirement for two dimensions to be conformed is they share one or more specically administered attributes that have the same column names and data values. Instead of requiring dozens of dimensions to be identical, you only require they share the specically administered conformed dimensions<sup>pg 258</sup>.
- Slowly Changing Dimensions
    - Type 0 Retain Original: attribute values never change so facts are always grouped by original value. Appropriate for any attribute labeled "original"<sup>pg 148</sup>.
    - Type 1 Overwrite: overwrite the old attribute value in the dimension table, replacing it with the current value. Always reflects most recent assignment. Fact table is untouched. Easy to implement but does not maintain any history of prior attributes.
    - Type 2 Add New Row:
        - Predominant technique for representing history. Create a new dimension row with a new single-column primary key to unqiuely identify the new profile<sup>pg 153</sup>.
        - Accurately tracking slowly changing dimension attributes, by adding a new row in dimension representing new history state<sup>pg 151</sup>. Should include administrative columns, such as 'effective' and 'expiration' dates<sup>pg 152</sup>.
        - Requires use of surrogate keys - because there will be multiple profile versions for the same natural key.
        - If being used to track customer dimension changes, you need to be careful to avoid overcounting because you may have multiple rows in the customer dimension for the same individual<sup>pg 243</sup>.
        - Current row indicators enable the most recent status to be retrieved quickly<sup>pg 266</sup>.
        - Changes can be embellished with a change reason. Because multiple dimension attributes may change concurrently and be represented by a single new row in the dimension, the change reason would be multi-valued. The multiple reason codes could be handled as a single text string attribute, or via a multi-valued bridge table<sup>pg 266</sup>.
    - Type 3 Add New Attribute:
        - Do not issue a new dimension row, but rather add a new column to capture the attribute change. Alter the dimension table to add a 'prior' attribute, and populate this new column with the _existing_ value, and treat original column as Type 1 (overwrite) with current value.
        - All existing reports immediately switch over to the newest value, but can still report on old values by querying 'prior' attributes.<sup>pg 154</sup>. Appropriate when there is a strong need to support two views of the world simultaneously (_alternative realities_). Suitable where dimension attributes change at a predictable rhythm - current attributes are overwritten, and attributes for previous values are added for every dimension row<sup>pg 156</sup>.
    - Type 4 Add Mini-Dimension: break off frequently analysed or frequently changing dimensions into a separate dimension<sup>pg 156</sup>. One row in the mini-dimension for each unique combination of age, purchase frequency, income level, etc encountered in the data (not one row per customer). Continuously variable attributes, such as income are converted to banded ranges.
    - Hybrid Techniques:
        - Type 5 Mini-Dimension: add a current mini-dimension key as an attribute in the primary dimension.
        - Type 6 Add 'Type 1' Attributes to 'Type 2' Dimension: issue a new row to capture the change (type 2) and add a new column to track the current assignment (type 3), where subsequent changes are overwritten (type 1)<sup>pg 161</sup>.
- Dimension role playing
    - Occurs when a single dimension (such as date) simultaneously appears several times in the same fact table<sup>pg 170</sup>.
    - The underlying dimension may exist as a single physical table, but each of the roles should be presented to tbhe BI tools as a separately labelled view<sup>pg 170</sup>.
    - Indicate the multiple roles within a single cell in the bus matrix<sup>pg 170</sup>.
- Junk dimensions
    - When modelling complex transactional source data, you often encounter a number of miscellaneous indicators and flags that are populated with a small range of discrete values<sup>pg 179</sup>.
    - There are several rather unappealing options to handling miscellaneous indicators and flags:
        - Ignore flags and indicators, but usually vetoed because someone occasionally needs them<sup>pg 179</sup>.
        - Leave flags and indicators unchanged on the fact row, but fact rows should not contain bulky descriptiors or cryptic indicators<sup>pg 179</sup>.
        - Make each flag and indicator into its own dimension, but should be avoided if the number of foreign keys is already more than 20 <sup>pg 179</sup>.
        - Store flags and indicators in a transaction header dimension, using transaction number as a regular dimension. Should be avoided because this dimension will be very large, and there should be orders of magnitude difference between size of fact table and associated dimensions<sup>pg 181</sup>.
    - Typically referred to as _transaction indicator_ or _transaction profile dimension_<sup>pg 180</sup>.
    - Grouping of low-cardinality flags and indicators<sup>pg 180</sup>.
    - Consideration needs to be given to whether junk dimension rows created for the full Cartesian product of all combinations beforehand or create junk dimension rows as they are encountered in data<sup>pg 180</sup>.
- Dimension attribute hierarchies
    - Fixed depth hierarchies:
        - Fixed set of labels, all with meaningful labels. Such as calendar hierarchies. Think of the levels as roll-ups. Can be represeted in a single dimension table<sup>pg 214</sup>.
        - Suited for highly predictable with fixed number of levels<sup>pg 245</sup>.
    - Slightly ragged variable depth hierarchies:
        - Propagate attribute values down to progressively complex hierarchies<sup>pg 215</sup>.
        - Duplicate lower-level values to populate the higher-level attributes<sup>pg 245</sup>.
    - Ragged variable depth hierarchies:
        - The classic way to represent a parent/child tree structure is by placing recursive pointers in the dimension table from each row to its parent<sup>pg 216</sup>.
        - The solution to the problem of representing arbitrary rollup structures is to build a special kind of _bridge table_ that is independent from the primary dimension table<sup>pg 216</sup>. The first column in the map table is the primary key of the parent, and the second column is the primary key of the child. A row must be constructed from each possible parent to each possible child, including a row that connects the parent to itself<sup>pg 217</sup> The 'highest parent' flag means the particular path comes from the highest parent in the tree, the 'lowest child' flag means the particular path ends in a "leaf node". Bridge tables are used by constraining the dimension table, joining to the bridge table, joined to the fact table<sup>pg 217</sup>.
        - An alternative approach is a pathstring attribute or 'modified preordered tree traversal' in the dimension table, but become complex if there are thousands of nodes in a tree, and difficult to maintain if hierarchy is changed<sup>pg 222</sup>.
- Bridge tables and positional design
    - Bridge tables have uses other than for dimension attribute hierarchies.
    - A fundamental tenet of dimensional modeling is to decide on the grain of the fact table, and then carefully add dimensions and facts to the design that are true to the grain<sup>pg 245</sup>. You do not want to change that grain<sup>pg 246</sup>.
    - When faced with multi-valued dimensions, there are two basic choices: a positional design or bridge table design<sup>pg 246</sup>.
        - Positional designs have named columns for each attribute. Attractive because the multivalued dimension is spread out into named columns that are easy to query and deliver fast query performance<sup>pg 275</sup>, but are not scaleable, and too many possibilities results in many nulls<sup>pg 246</sup>. Columnar databases are well suited to these kinds of designs because new columns can be added with minimal disruption<sup>pg 247</sup>. When the list of attributes is bounded and reasonably stable, a positional design is very effective<sup>pg 255</sup>.
        - Bridge tables are recommended when the number of different attributes grows beyond your comfort zone, if new attributes are added frequently and whenever the number of variables is open-ended and unpredictable<sup>pg 247</sup>. They are powerful and remove the scaleability and null value problems with positional designs because the rows in the bridge table exist only if they are needed, but the resulting table design requires a complex query that can be beyond the normal reach of BI tools<sup>pg 246</sup> and bridge tables should be buried within a canned BI application<sup>pg 274</sup>. 
    - Can also represent partial or shared ownership, in a 'percent ownership' attribute<sup>pg 219</sup>, and accommodate slowly changing hierarchies with the addition of start and end effective date/time attribute (making sure to constrain on these attributes to prevent double-counting)<sup>pg 220</sup>.
    - Disadvantages of bridge tables: require more ETL work to set up and more work when querying<sup>pg 223</sup>.
    - Advantages of bridge tables: alternative rollup structures can be selected at query time, shared ownership rollups, time varying ragged hierarchies, limited impact if a SCD, limited impact when tree structure is changed<sup>pg 223</sup>.
    - There is no silver bullet solution for handling complex structures in a simple and fast way<sup>pg 274</sup>.
- Behaviour tags and text strings
    - The recommended way to build an explicit time series of attributes in the dimension table. This is another example of positional design. Should not be stored as regular facts, and are used to formulate complex query patterns<sup>pg 241</sup>.
    - Remove many-many bridge tables by adding a dimension outrigger containing one long text string concatenating all keywords into a list key. You would need a special delimiter such as a backslash or pipe at the beginning of the string and after each value. The use of an outrigger dimension presumes a number of entities share a common list of values<sup>pg 276</sup>.
    - It would also be a good idea to create a single attribute with all the behaviour tags concatenated (such as CCCCDDDAAAABB) to support wildcard searches<sup>pg 242</sup>.
    - For example, can be used to look at customers' time series data such as RFV (Recency, Frequency, Value) quintile cube development over time<sup>pg 241<sup>.
- Aggregated facts as dimension attributes
    - You can store an aggregated fact as a dimension. These attributes are meant to be used for constraining and labeling; they are not to be used in numeric calculations<sup>pg 239</sup>.
    - The main burden falls on the back room ETL processes to ensure the attributes are accurate, up-to-date and consistent with the actual fact row<sup>pg 239</sup>.
- Step dimension
    - An abstract dimension defined in advance<sup>pg 251</sup>.
    - Useful to constrain on specific steps in a online checkout journey<sup>pg 252</sup>.
    - Another approach for modeling sequential behaviour takes advantage of specific fixed codes for each possible step such as a sequence of SKUS (not necessarily Product Dimension keys), i.e. 11254|45882|53340|74934<sup>pg 252</sup>.

### Kimball's DW/BI Architecture

- There are four separate and distinct components to consider in the DW/BI environment<sup>pg 18</sup>:
    - Operational source systems
        - Special purpose applications.
        - Intended for one-record-at-a-time queries.
        - Without any commitment to sharing common data such as product, customer or calendar with other operational systems.
        - Dimensional models contain data from many source systems, each with their own extract and transform programs<sup>pg 192</sup>.
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
