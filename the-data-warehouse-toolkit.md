# The Data Warehouse Toolkit
###### Ralph Kimball, Margy Ross

## DW/BI Overview

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
- Contains the same information as a normalized model, but provides data in a way that is *understandable* to users (not operational systems)<sup>pg 8</sup>.
- Normalization in the dimensional model negatively impacts the model's twin objective of *understandability* and *performance*<sup>pg 301</sup>.
- Plan to reevaluate models every 12 to 24 months<sup>pg 306</sup>.

## Fact and Dimension Table Overview

### Fact Tables for Measurements

- Stores the performance measurements resulting from an organisations business process events.
- Each row in a fact table corresponds to the most atomic measurement event<sup>pg 10</sup>.
- The data on each row is at a specific level of detail (_grain_)<sup>pg 10</sup>.
- All the measurement rows in a fact table must be at the same grain.
- Facts are often described as _continuously valued_ to help sort out what is a fact and what is a dimension attribute<sup>pg 11</sup>.
- Put textual data into dimensions<sup>pg 12</sup>.
- Fact tables tend to go deep in terms of number of rows, but narrow in terms of number of columns<sup>pg 12</sup>.
- All fact tables have two or more _foreign keys_ that relate to the dimension tables' _primary keys_.
- When all the keys in the fact table correctly match their respective primary keys in the corresponding dimension tables, the tables satisfy _referential integrity_.
- Fact table _primary keys_ are composed of a subset of the foreign keys, called a _composite key_

### Dimension Tables for Descriptive Context

- Provide the "who, what, where, when, why and how: context surrounding a business process event.<sup>pg 40</sup>
- Dimension tables are the heart of the data warehouse. They provide the context for the fact tables and hence for all measurements.<sup>pg 463</sup> They are the context for the measured activity (fact records).<sup>pg 478</sup>
- Contain the entry points and descriptive labels that enable the DW/BI system to be leveraged for analysis.<sup>pg 40</sup>
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

## Fact and Dimension Table Techniques

### Fact Table Techniques

- Fact table structure
    - Contains numeric measures produced by an operational measurement event in the real world<sup>pg 41</sup>.
    - Based on a _physical activity_ and is not influenced by the eventual reports that may be produced<sup>pg 41</sup>. A dimensional model is based solidly on the physics of a measurement process and is independent of how a user chooses to define a report.<sup>pg 400</sup>
    - Fact records are 'measured activity'.<sup>pg 478</sup>
    - Atomic data should be the foundation for every fact table design to withstand business users' ad hoc attacks.
    - _Continuously valued_ numeric observations are almost always facts.
    - If the numeric attributes are summarized rather than simply constrained upon, they belong in a fact table.<sup>pg 265</sup>
    - Fact tables are typically limited to foreign keys, degenerate dimensions and numeric facts<sup>pg 278</sup>.
    - You should prohibit text fields, including cryptic indicators and flags, from the fact table<sup>pg 301</sup>. Textual comments should not be stored in a fact table directly because they waste space and rarely participate in queries<sup>pg 350</sup>. If textual characteristics are used for query filtering or report labeling, then they belong in a dimension<sup>pg 383</sup>.
    - The fact table's granularity determines what constitutes a fact table row. In other words, what is the measurement event being recorded?<sup>pg 342</sup>.
    - There are trade-offs between creating separate fact table for each natural cluster of transaction types versus lumping the transactions into a single fact table<sup>pg 378</sup>.
    - The natural conflict between the detailed transaction view and the snapshot perspective almost always requires building both kinds of fact tables un the warehouse<sup>pg 378</sup>.
- There are just three fundamental types of fact tables; transaction, periodic snapshot and accumulating snapshot<sup>pg 119</sup>. All three serve a useful purpose, and often need two to complement each other. Whenever a source business process is considered for inclusion in the DW/BI system, there are three essential grain choices<sup>pg 342</sup>.
    - Transaction fact tables are the most fundamental view of operations, at the individual transaction line level<sup>pg 120</sup>. A row exists in the fact table only if a transaction event occurred. Naturally most atomic and enables analysis at extreme detail. However you cannot survive on transactions alone.
    - The transaction grain is the most fundamental<sup>pg 342</sup>. Transaction fact tables are useful for answering a wide range of questions, however the blizzard of transactions makes it difficult to quickly determine the status or financial value of a sale/policy/project at a given point in time <sup>pg 385</sup>.
    - There is more to life than transactions alone. Some form of a snapshot table to give a more cumulative view of a process often complements a transaction fact table<sup>pg 117</sup>. Even with a robust transaction schema, there is a whole class of urgent business questions that cannot be answered using only transaction detail<sup>pg 392</sup>.
    - Periodic snapshot are needed to see the cumulative performance of an operation at regular and predictable intervals<sup>pg 120</sup>, and cover all facts at a given snapshot date<sup>pg 113</sup>. You take a 'picture' of the activity at the end of a given day, week, month. Stacked consecutively into the fact table<sup>pg 120</sup>. Represents an aggregation of the transactional activity that occurred during a time period. Can include non-events. Good for long-running scenarios<sup>pg 139,342</sup>.
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
        - This kind of design is successful when 90 percent or more of the events progress through the same steps without any unusual exceptions, but there is no good way to reveal what happened when an event deviates from the standard scenario<sup>pg 255</sup>. An accumulating snapshot does a great job of presenting a workflow's current state, but it obliterates the intermediate states<sup>pg 394</sup>. Unusual departures from the standard scenario are described by a Status dimension on the accumulating snapshot fact table: tag the fact row with the Status 'Weird', and join back onto fact table using companion Keys<sup>pg 256</sup>. Accumulating snapshot does not attempt to fully describe unusual situations, so companion transaction schemas inevitably will be needed<sup>pg 343</sup>. Accumulating snapshot fact tables are typically appropriate for predictable workflows with well-established milestones with usually 10 key milestone dates representing the pipeline's start, completion and key events in between<sup>pg 393</sup>. Consider adding effective and expiration dates to the accumulating snapshot<sup>pg 394</sup> where new rows are added that preserves the state of a row for a span of time<sup>pg 394</sup>.
        - A single row represents the complete history of a workflow or pipeline instance<sup>pg 326</sup>.
        - Facts often include metrics corresponding to each milestone, plus status counts and elapsed duration<sup>pg 326</sup>.
        - Each row is revisited and updated whenever the pipeline instance changes; both foreign keys and measured facts may be changed during the fact row updates<sup>pg 326</sup>. They do not preserve counts and statuses at critical points, hence analysts might also want to retain snapshots at several important cut-off dates<sup>pg 329</sup>.
        - The row represents the accumulated history of the line item from the moment of creation to the current state<sup>pg 343</sup>.
    - Transactions and snapshots are the yin and yang of dimensional designs<sup>pg 122</sup>. Each provide different vantage points on the same story.
    - After the base processes have been built, it may be useful to design complementary schemas, such as summary aggregations, accumulating snapshots that look across a workflow of processes, consolidated fact tables that combine facts from multiple processes to a common granularity, or subset fact tables that provide access to a limited subset of fact data for security or data distribution purposes<sup>pg 300</sup>. You can almost never add enough facts to a snapshot schema to do away with the need for a transaction schema, and vice versa<sup>pg 387</sup>.
- Supertype and subtype schemas
    - A family of supertype and subtype fact tables are needed when a business has heterogeneous products that have naturally different facts and descriptors, but a single customer base that demands an integrated view<sup>pg 295</sup>.
    - Business users typically require two different perspectives that are difficult to represent in a single fact table<sup>pg 293</sup>. These are common where an organisation sells heterogenous products/services where some products have common dimensions, there may be special attributes not shared by other products (such as financial services).
    - The first perspective is the global view (supertype), including the ability to slice and dice common dimensionsm regardless of their product type. Crosses all lines of business to provide insight into the complete business. Can present only a limited number of facts that make sense for virtually every line of business. You cannot accommodate incompatible facts in the supertype fact table. Similarly, the supertype dimensions must be restricted to the subset of common attributes<sup>pg 293</sup>.
     - The second perspective is the line-of-business view (subtype) that focuses on the in-depth details of one business. There are special facts and attributes that make sense only for the particular line-of-business, and cannot be included in the supertype fact table<sup>pg 293</sup>. Subtype schemas must also contain the common supertype facts and attributes to avoid joining tables from the supertype and subtype schemas for the complete set of facts and attributes. The keys of the subtype dimensions are the same keys used in the supertype dimension, which contains all possible keys<sup>pg 294</sup>.
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
    - Events are modeled as fact tables containing a series of keys, each representing a participating dimension in the event. Event tables sometimes have no variable measurement facts associated with them and hence are called factless fact tables<sup>pg 332</sup>.
    - Each fact table typically has 5 to approximately 20 foreign key columns, followed by one to potentially several dozen numeric, continuously valued, preferably additive facts<sup>pg 329</sup>.
    - The facts can be regarded as measurements taken at the intersection of the dimension key values. From this perspective, the facts are the justification for the fact table, and the key values are simply administrative structure to identify the facts<sup>pg 330</sup>.
    - A fact table represents the robust set of many-many relationships among dimensions; it records the collision of dimensions at a point in time and space<sup>pg 331</sup>.
     - If a measurable fact does surface during the design, it can be added to the schema assuming it is consistent with the grain<sup>pg 332</sup>.
- 'Centipede' fact tables
    - A very large number of dimensions (more than 25) typically are a sign that several dimensions are not completely independent and should be combined into a single dimension. For example, it is a mistake to model a fact table with separate keys for `Date`, `Week`, `Month`, `Quarter`, `Year` dimensions or `Product`, `Product Category`, `Package Type` dimensions - these are clearly related attributes so should be included in the same dimension tables<sup>pg 108</sup>.
    - If the fact table has more than 20 or so foreign keys, you should look for opportunities to combine or collapse dimensions<sup>pg 302</sup>.
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
- Single versus multiple transaction fact tables
    - A common design quandry that surfaces in many transactional situations, is should you build a blended transaction fact table with a transaction type dimension to view all procurement transactions together, or do you build separate fact tables for each transaction type? There is no simple formula to make the definite determination of whether to use a single fact table or multiple fact tables<sup>pg 144</sup>.
    - When faced with a design decision, the following considerations help sort out the options:
        - What are the users' analytic requirements? And which approach most naturally aligns with their business-centric perspective?<sup>pg 144</sup>
        - Are there really multiple unique business processes?<sup>pg 144</sup>
        - Are multiple source systems capturing metrics with unique granularities? Separate source systems suggests separate fact tables<sup>pg 144</sup>.
        - What is the dimensionality of the facts? If dimensions are applicable to some transaction types but not to others, this would again lead you to separate fact tables<sup>pg 144</sup>.
    - A simple way to consider these trade-offs is to draft a bus matrix, including two additional columns identifying the atomic granularity and metrics for each row (i.e. business process)<sup>pg 145</sup>
    - Multiple fact tables enable richer, more descriptive dimensions and attributes. The single fact table approach would have required generalized labelling for some dimensions. This generalization reduces the legibility of the resulting dimensional model<sup>pg 145</sup>.
    - Multiple fact tables may require more time to manage and adminster because there are more tables to load, index and aggregate. However loading the operational data from separate source systems into separate fact tables likely requires less complex ETL processing than attempting to integrate data from the multiple sources into a single fact table<sup>pg 145</sup>.
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

### Dimension Table Techniques

- Dimension table structure
    - Every dimension table has a single primary key column, which is also embedded in any associated fact tables<sup>pg 46</sup>.
    - Wide, flat, denormalised<sup>pg 46</sup>.
    - Populated with verbose, descriptive columns<sup>pg 46,172</sup>. Cryptic abbreviations, operational codes or true/false flags should be replaced with full text words<sup>pg 48</sup>.
    - Rather than decoding flags into understandable labels in the BI application, we prefer that decoded values be stored in the database so they are consistently available to all users regardless of their BI reporting environment<sup>pg 82</sup>. Columns in a dimension are the sole source of query constraints and report labels so should be legible<sup>pg 173</sup>. Quality check the attribute values to ensure no misspellings, impossible values or multiple variations, and are completely populated (SQL will produce another line in a report if the attribute values varies in any way based on trivial punctuation or spelling differences)<sup>pg 173</sup>.
    - Separate heirarchies can gracefully coexist in the same dimension table<sup>pg 48</sup>. Hierarchical data should be presented in a single, flattened denormalized table<sup>pg 172</sup>.
    - Millions of rows in a dimension table is considered large<sup>pg 176</sup>.
    - 25 dimensions in a single dimensional model is considered large, and model should consider combining dimensions<sup>pg 176</sup>. Most dimensional models end up with between 5 and 20 dimensions<sup>pg 284</sup>.
    - You certainly do not want to have as many rows in a fact table as you do in a related dimension table<sup>pg 264</sup>. You should avoid creating dimensions with the same number of rows as the fact table<sup>pg 392</sup>.
    - If the cardinality of a dimension table is less than the number of transactions in the fact table, attribute values should be captured in a separate dimension table. If there is a unique attribute value for every event, it is treated as a transaction-grained dimension attribute<sup>pg 264</sup>.
    - Flatten many-to-one hierarchies: i.e. SKUs roll up to brands, brands roll up to categories, categories roll up to departments. Each of these is a many-to-one relationship. Dimension tables can contain more than one explicit hierarchy.<sup>pg 85</sup>
- Degenerate dimensions
    - Some fact tables have dimension keys for dimension tables that bear no context, i.e. invoice number. It is acknowledged there is no associated dimension table with this key<sup>pg 47</sup>.
    - These can be useful for grouping purposes (i.e. grouping across basket transactions), and also linking back to operational system<sup>pg 93,178</sup>.
    - Typically sits in a fact table but does not have a corresponding dimension table<sup>pg 94</sup> and reserved for operational transaction identifiers<sup>pg 178</sup>.
    - Plays an important role in a fact table's primary key<sup>pg 94</sup>.
    - The natural operational ticket number, such as the POS transaction number, sits by itself in the fact table without joining to a dimension table. Degenerate dimensions are very common when the grain of a fact table represents a single transaction or transaction line because the degenerate dimension represents the unique identifier of the parent<sup>pg 95</sup>.
    - Transaction numbers are best treated as degenerate dimensions<sup>303</sup>.
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
    - Every fact table should have at least one foreign key to an explicit date dimension<sup>pg 302</sup>.
    - "Date dimension" means a _daily_ grained dimension table<sup>pg 80</sup>.
    - Even 20 years' worth of days is only approximately 7,300 rows<sup>pg 80</sup>.
    - Explicit date dimensions are preferred (rather than a date key in the fact table as a date type column):
        1. Most database engines are efficient enough at resolving dimensional queries (it is not necessary to avoid joins like the plague)<sup>pg 81</sup>.
        2. Average business users are not versed in SQL date semantics, so provide a wide range of meaningful calendar groupings<sup>pg 81</sup>.
        3. Calendar logic belongs in a dimension table, not in the application code<sup>pg 82</sup>.
        4. There are many date attributes not supported by the SQL date function<sup>pg 82</sup>.
    - Time-of-day should be handled as a simple date/time fact in the fact table<sup>pg 83</sup>.
    - Date dimension smart keys: it is common to assign the primary key of a date dimension, and foreign key of a fact table, as a meaningful integer such as yyyymmdd. Although not intended to provide BI applications with a mechanism to bypass a join between fact table and date dimension (and directly query the fact table), it is useful for partitioning fact tables. It allows old data to be removed gracefully<sup>pg 102</sup>.
    - On accumulating snapshot fact tables where a workflow event has not occurred yet, surrogate date keys must not be null and must point to a date dimension row reserved for a To Be Determined date<sup>pg 345</sup>.
- Conformed dimensions
    - Shared across business process fact tables<sup>pg 130</sup>.
    - Built once in ETL system and replicated either logically or physically throughout the enterprise DW/BI environment, a policy mandated by CIO<sup>pg 130</sup>.
    - Enable you to combine performance measurements from different business processes in a single report ("drilling across fact tables")<sup>pg 130</sup>. Query each dimensional model separately and then outer-join the query results based on a common dimension attribute. Full outer-join ensures all rows are included in the report, even if they only appear in one set of query results. Also known as multipass, multi-select, multi-fact, or stitch queries, because metrics from different facts tables are brought together.
    - Identical conformed dimensions mean the same thing with every possible fact table to which they are joined. Dimension built once in ETL system and duplicated outward to each dimensional model<sup>pg 131</sup>.
    - Shrunken rollup conformed dimensions conform to the base atomic dimension if the attributes are a strict subset of the atomic dimension's attributes<sup>pg 132</sup>.
    - In the ideal case, you examine all the data sources and define a single comprehensive dimension which you attach to all the data sources. In the extreme integration world with dozens of related dimensions of different granularity and different quality, such a single comprehensive dimension is impossible to build<sup>pg 258</sup>. Fortunately you can implement a lighter-weight kind of conformed dimension<sup>pg 258</sup>.
    - The essential requirement for two dimensions to be conformed is they share one or more specically administered attributes that have the same column names and data values. Instead of requiring dozens of dimensions to be identical, you only require they share the specically administered conformed dimensions<sup>pg 258</sup>.
    - Without conformed dimensions, you inevitably perpetuate incompatible stovepipe views of performance across the organisation<sup>pg 304</sup>.
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
    - Type 4 Add Mini-Dimension:
        - Break off frequently analysed or frequently changing dimensions into a separate dimension<sup>pg 156</sup>.
        - Break off the browseable and changeable attributes into multiple mini-dimensions, such as credit bureau and demographics mini-dimensions, whose keys are included in the fact table<sup>pg 289</sup>.
        - Mini-dimensions enable you to slice and dice the the fact table, while readily tracking attribute changes over time, even though they may be updated at different frequencies<sup>pg 289</sup>.
        - Even though extremely powerful, be careful to avoid overusing the technique or you end up with too many dimensions for the fact table<sup>pg 289</sup>.
        - One row in the mini-dimension for each unique combination of age, purchase frequency, income level, etc encountered in the data (not one row per customer). Continuously variable attributes, such as income are converted to banded ranges.
    - Hybrid Techniques:
        - Type 5 Mini-Dimension: add a current mini-dimension key as an attribute in the primary dimension.
        - Type 6 Add 'Type 1' Attributes to 'Type 2' Dimension: issue a new row to capture the change (type 2) and add a new column to track the current assignment (type 3), where subsequent changes are overwritten (type 1)<sup>pg 161</sup>.
- Dimension role playing
    - Occurs when a single dimension (such as date) simultaneously appears several times in the same fact table<sup>pg 170</sup>.
    - The underlying dimension may exist as a single physical table, but each of the roles should be presented to tbhe BI tools as a separately labelled view<sup>pg 170</sup>.
    - Indicate the multiple roles within a single cell in the bus matrix<sup>pg 170</sup>.
    - Date foreign keys should not join to a single instance of the date dimension table. Instead, create views on the single underlying date dimension table, and join the fact table separately to these views<sup>pg 345</sup>.
- Junk dimensions
    - When modelling complex transactional source data, you often encounter a number of miscellaneous indicators and flags that are populated with a small range of discrete values<sup>pg 179</sup>.
    - There are several rather unappealing options to handling miscellaneous indicators and flags:
        - Ignore flags and indicators, but usually vetoed because someone occasionally needs them<sup>pg 179</sup>.
        - Leave flags and indicators unchanged on the fact row, but fact rows should not contain bulky descriptiors or cryptic indicators<sup>pg 179</sup>.
        - Make each flag and indicator into its own dimension, but should be avoided if the number of foreign keys is already more than 20 <sup>pg 179</sup>.
        - Store flags and indicators in a transaction header dimension, using transaction number as a regular dimension. Should be avoided because this dimension will be very large, and there should be orders of magnitude difference between size of fact table and associated dimensions<sup>pg 181</sup>.
    - Typically referred to as _transaction indicator_ or _transaction profile dimension_<sup>pg 180</sup>.
    - Grouping of low-cardinality flags and indicators<sup>pg 180</sup>. For example, one row per unique combination of profile attributes<sup>pg 392</sup>.
    - Consideration needs to be given to whether junk dimension rows created for the full Cartesian product of all combinations beforehand or create junk dimension rows as they are encountered in data<sup>pg 180</sup>.
- Audit dimensions
    - Each audit dimension row describes the data lineage of the fact row, including the time of the extract, source table, and extract software version<sup>pg 383</sup>.
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
        - Positional designs have named columns for each attribute. Attractive because the multi-valued dimension is spread out into named columns that are easy to query and deliver fast query performance<sup>pg 275</sup>, but are not scaleable, and too many possibilities results in many nulls<sup>pg 246</sup>. Columnar databases are well suited to these kinds of designs because new columns can be added with minimal disruption<sup>pg 247</sup>. When the list of attributes is bounded and reasonably stable, a positional design is very effective<sup>pg 255</sup>.
        - Bridge tables are recommended when the number of different attributes grows beyond your comfort zone, if new attributes are added frequently and whenever the number of variables is open-ended and unpredictable<sup>pg 247</sup>. They are powerful and remove the scaleability and null value problems with positional designs because the rows in the bridge table exist only if they are needed, but the resulting table design requires a complex query that can be beyond the normal reach of BI tools<sup>pg 246</sup> and bridge tables should be buried within a canned BI application<sup>pg 274</sup>. An open-ended, many-valued attribute can be associated with a dimension row by using a bridge table to associate the many-valued attributes with the dimension<sup>pg 288</sup>.
    - Can also represent partial or shared ownership, in a 'percent ownership' attribute<sup>pg 219</sup>, and accommodate slowly changing hierarchies with the addition of start and end effective date/time attribute (making sure to constrain on these attributes to prevent double-counting)<sup>pg 220</sup>.
    - Disadvantages of bridge tables: require more ETL work to set up and more work when querying<sup>pg 223</sup>.
    - Advantages of bridge tables: alternative rollup structures can be selected at query time, shared ownership rollups, time varying ragged hierarchies, limited impact if a SCD, limited impact when tree structure is changed<sup>pg 223</sup>.
    - There is no silver bullet solution for handling complex structures in a simple and fast way<sup>pg 274</sup>.
- Behaviour tags and text strings
    - The recommended way to build an explicit time series of attributes in the dimension table. This is another example of positional design. Should not be stored as regular facts, and are used to formulate complex query patterns<sup>pg 241</sup>.
    - Remove many-many bridge tables by adding a dimension outrigger containing one long text string concatenating all keywords into a list key. You would need a special delimiter such as a backslash or pipe at the beginning of the string and after each value. The use of an outrigger dimension presumes a number of entities share a common list of values<sup>pg 276</sup>.
    - It would also be a good idea to create a single attribute with all the behaviour tags concatenated (such as CCCCDDDAAAABB) to support wildcard searches<sup>pg 242</sup>.
    - For example, can be used to look at customers' time series data such as RFV (Recency, Frequency, Value) quintile cube development over time<sup>pg 241<sup>.
    - Unbounded text comments should either be stored in a separate comments dimension or treated as attributes in a transaction event dimension<sup>pg 350</sup>.
- Aggregated facts as dimension attributes
    - You can store an aggregated fact as a dimension. These attributes are meant to be used for constraining and labeling; they are not to be used in numeric calculations.<sup>pg 239</sup> 
    - It is possible to store a fact as a dimension value. [Link](https://www.kimballgroup.com/2007/12/design-tip-97-modeling-data-as-both-a-fact-and-dimension-attribute/)
    - The main burden falls on the back room ETL processes to ensure the attributes are accurate, up-to-date and consistent with the actual fact row<sup>pg 239</sup>.
    - Continuously valued numeric quantities are treated as facts. In the dimension table, you could store a more descriptive value range for grouping and filtering<sup>pg 382</sup>.
- Step dimension
    - An abstract dimension defined in advance<sup>pg 251</sup>.
    - Useful to constrain on specific steps in a online checkout journey<sup>pg 252</sup>.
    - Another approach for modeling sequential behaviour takes advantage of specific fixed codes for each possible step such as a sequence of SKUS (not necessarily Product Dimension keys), i.e. 11254|45882|53340|74934<sup>pg 252</sup>.
- Combining correlated dimensions
    - If a many-many relationship exists between two groups of dimension attributes, they should be modeled as separate dimensions with separate foreign keys in the fact table. Sometimes, however, you encounter situations where these dimensions can be combined into a single dimension rather than treating them as two separate dimensions with two separate foreign keys in the fact table<sup>pg 318</sup>.
- Multi-valued dimension attributes
    - Normally the dimensions surrounding a fact table take on a single value in the context of a fact event. However, there are situations where multivaluedness is natural and unavoidable<sup>pg 345</sup>.
    - Some dimension attributes take on multiple values for the fact table's declared grain<sup>pg 333</sup>. There are several options. Also see _Transaction facts at different granularity_:
        - Alter the grain of the fact table, but could lead to unnatural granularity that would be prone to overstated count errors.
        - Add a bridge table with a group key.
        - Concatenate the names into a single, delimited attribute. Enables easy report labelling but would not support analysis of events by specific dimension characteristics.
    - Weighting factors in multi-valued bridge tables provide an elegant way to prorate numeric facts to produce correctly weighted reports. However, these weighting factors are by no means required in a dimensional design. If there is no agreement or enthusiasm within the business community for the weighting factors they should be left out. Also, in a schema with more than one multi-valued dimension, it is not worth trying to decide how multiple weighting factors would interact<sup>pg 346</sup>.

## Kimball's DW/BI Architecture

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
- An alternative DW/BI architecure is the indepdent data mart:
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

---

## Kimball Modelling Techniques

### Gather Business Requirements and Data Realities

- Do not focus on a set of required reports or dashboard gauges. Instead constantly ask about the business process measurement events producing that report or dashboard<sup>pg 32</sup>
- Understand the source data and business requirements<sup>pg 38</sup>.
- Dimensional models should be designed based on a blended understanding of the business's needs, along with the operational source system's data realities. While requirements are collected from the business users, the underlying source data should be profiled. Models driven solely by requirements inevitably include data elements that cannot be sourced. Meanwhile, models driven solely by source system data analysis inevitably omit data elements that are critical to the business's analytics<sup>pg 300</sup>.
- Objectives based on KPIs, decision making processes and analytic needs<sup>pg 38</sup>.
- Dimensional models should be designed to mirror an organization's primary business process events<sup>pg 300</sup>.

### Four-Step Dimensional Design Process

1. Select the business process
    - i.e. taking an order, processing an insurance claim, etc.
    - Business process events generate or capture performance metrics hat translate into facts in a fact table. Most fact tables focus on the results of a single business process<sup>pg 39</sup>.
    - Frequently expressed as action verbs<sup>pg 70</sup>.
    - Typically supported by an operational system<sup>pg 70</sup>.
    - A series of processes results in a series of fact tables<sup>pg 70</sup>.
    - The performance measurements users want to analyse in the DW/BI system result from business process events<sup>pg 70</sup>.
    - The definition of the lowest level of granularity possible depends on the business process being modeled<sup>pg 301</sup>.
2. Declare the grain
    - The first question to ask during a design review is, "What is the grain of the fact table?"<sup>pg 300</sup>.
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
    - Each of the dimensions associated with a fact table should take on a single value with each row of fact table measurements<sup>pg 301</sup>.
4. Identify the facts
    - A single fact row has a one-one relationship to a measurement event<sup>pg 40</sup>.
    - Result from a single business process event and are almost always numeric<sup>pg 40</sup>.
    - New facts (consistent with the grain of an existing fact table) can be added by creating new columns<sup>pg 41</sup>.
    - "What is the process measuring?"<sup>pg 72</sup>.
    - Facts that clearly belong to a different grain must be in a separate fact table<sup>pg 72</sup>.
    - Consider both business users' requirements and the realities of the operational source data in tandem<sup>pg 72</sup>.
    - After the fact table granularity has been established, facts should be identified that are consistent with the grain declaration<sup>pg 301</sup>.
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

### Kimball DW/BI Lifecycle

#### Project Planning

- Assessing Readiness
    - Is there a strong executive business sponsor with a clear vision for the DW/BI system's potential impact?<sup>pg 406</sup>
    - Is there strong compelling business motivation? i.e. competition
    - Are real data in real operational source systems being collected to support the business reqirements<sup>pg 407</sup>
    - Is the scope known?
    - What is the justification for a DW/BI project? "Delivering a single version of the truth" is not a sufficient financial justification. What are the increased profit opportunities?<sup>pg 407</sup>
    - Staffing (roles, not not people - it is common for the same person to fill multiple roles on the team). See full list on pg 408.
- Business Requirements Definitions
    - Requirements preplanning: requirements sessions are usually interwoven with source system expert data discover sessions. Interview business representatives about what they do, why they do it, how they make decisions and how they hope to make decisions in the future<sup>pg 410</sup>. Talk to business people representing a reasonable horizontal spectrum of the organisation. Do some homework and do not presume you know it all. Identify lead inteviewer who asks great open-ended questions and scribe to take copious notes<sup>pg 411</sup>. Ask interviewees to bring copies of important reports and analyses to the session<sup>pg 412</sup>.
    - Collecting business requirements: the objective of an interview is to get business users to talk about what they do and why they do it<sup>pg 412</sup>. Ask analytical staff about the types of analysis currently generated. Ask executives about better leveraging information throughout the organization<sup>pg 413</sup>. Ask each interviewee to articulate specific success critera for the project<sup>pg 413</sup>.
    - Conducting data-centric interviews: intersperse interviews with sessions with source system data gurus or subject matter experts to evaluate the feasibility of supporting the business needs<sup>pg 413</sup>. Do some initial data profiling to ensure you are not standing on quicksand (a more complete data audit will occur during the dimensional modeling process)<sup>pg 414</sup>.
    - Documenting requirements: interview team should debrief and review notes shortly after interview, consolidating everything in a findings document, organised around key business processes<sup>pg 414</sup>.
    - Prioritising requirements: prioritise business processes to tackle using a prioritisation grid (feasibility on x-axis, potential business impact on y-axis)<sup>pg 415</sup>.
    - This step usuall determines the first step of the Dimensional Design Process - _identify the business process_<sup>pg 434</sup>.

#### Lifecycle Technology Track

- Like a blueprint for a home, technical architecture consists of a series of models that unveil greater detail regarding each major component, and allows you to catch problems on paper. Identifies immediately required components versus those that will be incorporated at a later date. A communication tool between all parties involved<sup>pg 416</sup>.
- Establish an architecure task force: typically technical architect, ETL designer and BI application designer to ensure both back room and front room representation<sup>pg 417</sup>.
- Collect and document architecture-related requirements, and create architecure model: list each business requirement impacting the architecture, along with a list of architectural implications<sup>pg 417</sup>. Architecture requirements are grouped into major components such as ETL, BI, metadata and infrastructure<sup>pg 418</sup>.
- Design and specify the subsystems. Define in enough detail to evaluate whether subsystem can be bought or built<sup>pg 418<sup>.

#### Lifecycle Data Track

- Dimensional modelling
- Physical design: dimensional models developed via preliminary source-target mapping need to be translated into a physical database<sup>pg 420</sup>
    - Develop naming and database standards
    - Develop physical database model, including staging tables to support ETL, and auditing tables for ETL processing and data quality.
    - Develop initial index plan, applying indexes to dimension tables' single column primary key. 
    - Design aggregations, considering business users' access patterns and statistical distributions of data.
    - Finalise physical storage details, i.e. partition tables by date.
- ETL design and development
    - Resist temptations to defer complexities to the BI applications in an effort to streamline the ETL processing. Remember the goal is to trade off ETL processing complexity for simplicity and predictability at the BI presentation layer.<sup>pg 431</sup> The primary mission of the ETL system is the hand off of the dimension and fact tables in the [delivery step](#delivering-prepare-for-presentation).<sup>pg 463</sup> We believe it is irresponsible to hand off data to the BI application in such a way to increase the complexity of the application.<sup>pg 448</sup>
    - ETL consumes a disproportionate share of the time and effort required to build a DW/BI environment due to the many outside constraints putting pressure on the design, such as source data realities, budget, processing windows, and skills of available staff<sup>pg 443</sup>.
    - Round up the requirements:
        - Business needs: maintain a list of the KPIs uncovered during the business requirements sessions<sup>pg 444</sup>.
        - Compliance: list all data and final reports subject to compliance restrictions. List those data inputs and transformation steps for which you must maintain the "chain of custody"<sup>pg 445</sup>.
        - Data quality: list all data elements whose quality is known to be unacceptable, and whether an agreement has been reached with the source systems to correct the data before extraction<sup>pg 445</sup>.
        - Security: seek guidance from senior management as to what aspects of the DW/BI system carry extra security sensitivity. Likely to overlap with Compliance<sup>pg 446</sup>.
        - Data integration: the "360 degree view of the enterprise" is a familiar name for data integration and usually takes the form of conforming dimensions and conforming facts in the data warehouse. Generate a priority list for conforming dimensions<sup>pg 446</sup>.
        - Data latency: if the data latency requirements are sufficiently urgent, the ETL system's architecture must convert from batch to microbatch or streaming<sup>pg 447</sup>.
        - Archiving and lineage: data should be staged (written to disk) after each major activity of the ETL pipeline - after it has been extracted, cleaned and conformed, and delivered. All staged data should be archived unless a conscious decision is made that specific data sets will never be recovered in the future<sup>pg 447</sup>. List the data sources, retention policies, and compliance, security and privacy constraints<sup>pg 448</sup>.
        - BI delivery interfaces: the ETL team, working closely with the modeling team, must take responsibility for the content and structure of the data that makes the BI applications simple and fast. The ETL team and data modelers need to closely work with the BI application developers to determine the exact requirements for the data handoff<sup>pg 448</sup>.
        - Available skills: inventory your department's operating system, ETL tool, scripting language, programming language, SQL, DBMS and OLAP skills<sup>pg 449</sup>.
        - Legacy licenses: list your legacy all as above, and add whether their exclusive use is mandated or recommended<sup>pg 449</sup>.
    - [See 34 subsystems of ETL here](#34-subsystems-of-etl)

#### Lifecycle BI Applications Track

- While a data warehouse provides an ad-hoc self service query environment, delivering parameter-driven BI applications will satisfy a large percentage of the business communities needs<sup>pg 422</sup>.
- BI application specification and development: identify a start set of approximately 10-15 BI reports and analyses, focus on standards (naming conventions, calculations, libraries and coding standards). Application development can begin when the database design is complete, BI tools and metadata are installed, and a subset of historical data have been loaded. BI developers will quickly find needling problems in the data haystack despite QA performed by ETL application<sup>pg 423</sup>.

#### Lifecycle Wrap-up Activities

- Deployment: Technology, data and BI tracks must converge. Preparing data in the ETL phase is the most unpredictable task<sup>pg 424</sup>.
- Maintenance and growth: the DW/BI system needs to be treated as a production environment with service level agreements. You do not want to rely on the business community to tell you that performance/quality has degraded, so it should be proactively monitored<sup>pg 425</sup>.

### Dimensional Modeling Process and Tasks

- The primary goals are to create a model that meets the business requirements, verify that data is available to populate the model, and provide the ETL team with a solid starting source-to-target mapping<sup>pg 430</sup> and a clear direction<sup>pg 441</sup>.

- Dimensional models unfold through a series of design sessions, with each pass resulting in a more detailed and robust design. A typical design requires _three to four weeks_ for a single business process dimensional model<sup>pg 430</sup>.
- {image of dimensional modeling process flow diagram}

#### Get Organised

- Identify participants, especially business representatives: data modelers facilitiate the process, but subject matter experts should be actively involved because they are often the individuals who have historically figured out how to get out of source systems and turned into valuable analytic information<sup>pg 431</sup>.
- Initiate a data governance and stewardship program (if not already exists)<sup>pg 431</sup>. It is difficult to get people in different corners of the business to agree on common attribute names, definitions and values, but that is the crux of unified, integrated data<sup>pg 432</sup>.
- Review business requirements from Project Planning phase and translate into dimensional model. Some designers are tempted to skip the requirements review and move directly into the design, but the resulting models are typically driven exclusively by the souce data without considering added value required by the business community<sup>pg 432</sup>.
- Document dimensional model using a spread sheet to help the DBA forward engineer the model into the database<sup>pg 433</sup>.
- Data profiling: the modeling team needs an ever-increasing understanding of the source data's structure, content and relationships. _Data profiling_ uses query capabilities to explore the actual content and relationships of the source system rather than relying on incomplete or outdated documentation<sup>pg 433</sup>. The source system needs to be investigated at great depth, so DBAs may need to set up a static snapshot of the database for the ETL team.<sup>pg 503</sup>
- Establish naming conventions (if not already established.) Part of the process of designing a dimensional model is agreeing on common definitions and labels<sup>pg 433</sup>.
- Coordinate calendars. Schedule design sessions for 2-3 hours for 3-4 days each week. The design team can use unschedule time to research the source data and confirm requirements<sup>pg 434</sup>.

#### Design the Dimensional Model

- Reach consensus. The initial task in the design session is to create a high-level dimensional model diagram for the target business process, often using a bubble chart. This entity-level graphical model clearly identifies the grain of the fact table and its associated dimensions to a non-technical audience<sup>pg 435</sup>. The bubble chart must be rooted in the realities of available physical data sources<sup>pg 436</sup>. See sample model diagram [here](http://www.kimballgroup.com/wp-content/uploads/2014/03/Ch07-High-Level-Model-diagram.ppt).

![Bubble chart diagram](/images/dwtk-ch18-bubblechart.png)
- Develop the detailed dimensional model:
    - Identify dimensions and their attributes: start with dimension tables; conformed dimensions are defined and agreed across business users<sup>pg 437<sup>.
    - Identify the facts: declaring the grain crystallises the discussion about the fact table's measurements<sup>pg 437</sup>.
    - Identify slowy changing dimension techniques: for each dimension table attribute, define how source system data changes will be reflected in the dimension table<sup>pg 437</sup>.
    - Document the detailed table designs: the main deliverable of detailed dimensional models are the design worksheets, capturing detail for communication between business users, BI application developers and ETL developers tasked with populating the design<sup>pg 437</sup>. Each dimension and fact table should be documented in a separate worksheet, or at least attribute/fact names, descriptions, example values and SCD type for every dimension attribute<sup>pg 438</sup>. Worksheets downloaded from [here](http://www.kimballgroup.com/wp-content/uploads/2014/03/Ch08-Physical-Model-Template.xls).
    - Track issues, definitions, transformation rules and data quality challenges discovered during design process should be captured in an issue tracking log<sup>pg 439</sup>.
    - Keep bus matrix updated as new discoveries lead to new fact tables or splitting/combining dimensions<sup>pg 439</sup>.
    - Ongoing analysis of the source system and data profiling helps the team better understand the realities of the underlying source data<sup>pg 436</sup>.
- Review and validate the model: conduct reviews with IT who are intimately familiar with the target business process, because they probably wrote or manage the system that runs it. Conduct education sessions with business users to illustrate how the dimensional model supports their business requirements<sup>pg 440</sup>.
- Final design documentation should include: brief description of the project, high-level data model diagrams, detailed dimensional design worksheet for each fact and dimension table, open issues<sup>pg 441</sup>.

### 34 Subsystems of ETL

- ETL is more than simply extract, transform and load; it is a host of complex and important tasks.<sup>pg 496</sup>. There are really 4 components:
    - [Extracting](#extracting): gathering raw data from source systems and usually writing to disk in the ETL environment before any significant restructuring of the data takes place<sup>pg 450</sup>.
    - [Cleaning and conforming](#cleaning-and-conforming): sending source data through a series of processing steps to improve the quality of the data received from the source<sup>pg 450</sup>.
    - [Delivering](#delivering-prepare-for-presentation): physically structuring and loading the data into the presentation server's target dimensional models<sup>pg 450</sup>.
    - [Managing](#managing-the-etl-environment): managing the related systems and processes of the ETL environment in a coherent manner<sup>pg 450</sup>.

#### Extracting

- Subsystem 1 - _Data Profiling_: technical analysis of data to describe its content, consistency and structure. The profiling step provides the ETL team with guidance as to how much data cleaning machinery to invoke and protects them from missing major project milestones due to unexpected diversion of building systems to deal with dirty data. Use the data profiling results to set the business sponsors' expectations in development schedules, source limitations and the need to invest in better data capture practises<sup>pg 451</sup>.
- Subsystem 2 - _Change Data Capture System_: isolating the latest source data is called change data capture (CDC), and just transfer the data that has changed since the last load<sup>pg 451</sup>. You must carefully evaluate your stragegy for each data source<sup>pg 452</sup>.
    - Audit columns: source systems sometimes include audit columns that store the date and time a record was added or modified, populated by database triggers that are fired automatically as records are inserted or updated<sup>pg 452</sup>.
    - Timed extracts: select all rows where the create or modified date fields equal `SYSDATE-1`, meaning all of yesterday's records. However this is unreliable if the process fails for any reason<sup>pg 452</sup>.
    - Full diff compare: keep a full snapshot of yesterday's data, and compare it record by record, against today's data to find what changed.Thorough but resource-intensive. Try to do comparison on source machine to avoid transferring entire table/database to the ETL environment<sup>pg 453</sup>.
    - Database log scraping: scraping the log for transactions is probably the messiest of all techniques. Takes a snapshot of the database redo log at a scheduled point in time and scours it for transactions affecting the tables of interest in the ETL load<sup>pg 453</sup>.
    - Message queue monitoring: monitor queue for all transactions against the tables of interest<sup>pg 453</sup>.
- Subsystem 3 - _Extract System_: each source might be in a different system, environment and/or DBMS, presenting a variety of challenges. There are two primary methods for getting data from a source system: as a file (extract to the file, move the file to the ETL server, transform the file contents, and load into a staging database) or a stream (data flows out of the source system, through the transformation engine, and into the staging datatbase as a single process)<sup>pg 454</sup>.

#### Cleaning and Conforming

- These are the steps where the ETL system adds value to the data. Cleaning and conforming subsystems actually change the data and enhance its value to the organisation<sup>pg 455</sup>.

- Subsystem 4 - _Data Cleansing System_: striking a balance between fixing dirty data and providing an accurate picture of the source as it was captured by production systems. Quality screens/tests are diagnostic filters in the data flow pipelines<sup>pg 457</sup>. Each quality screen has to decide what happens when an error is thrown (halt process, send to suspense file for later processing, tag as faulty and pass to next step in pipeline).
    - Column screens test the data within a single column. Does a column contain null values? Does a value fall outside of a prescribed range? Does a value fail to adhere to a required format?<sup>pg 457</sup>
    - Structure screens test foreign key/primary key relationships between tables.<sup>pg 457</sup>
    - Business rule screens test aggregate threshold quality tests (i.e. is there a statistically improbable number of counts) and test business rule logic (i.e. are there any 'platinum' customers that do not meet attributes required for platinum status?)<sup>pg 457</sup>
- Subsystem 5 - _Error Event Schema_: dimensional schema to record every error event thrown by a quality screen/tests anywhere in the ETL pipeline. Consists of a fact table with grain of every error thrown by a quality screen.<sup>pg 458</sup>
- Subsystem 6 - _Audit Dimension Assembler_: the audit dimension contains the metadata context at the moment when a specific fact row is created. For fact rows being loaded that fail a quality screen/test, an audit dimension row flagging the condition would be attached to the fact row.<sup>pg 460</sup>
- Subsystem 7 - _Deduplication System (Survivorship)_: Dimensions are often derived from several sources (i.e. customer information may need to be merged from several lines of business and outside sources.) Sometimes the data can be matched through identical values in some column, other times a fuzzy match is needed. However common columns in different sources may contradict one another, requiring a decision on which data should survive. Survivorship is the process of combining matched records into a unified image that combines the highest quality columns from the matched records. This requires a priorty sequence from all source systems into the best-survived attributes. If the dimensional design is fed from multiple systems, you must maintain natural keys to all participating sources used to construct the dimension row.<sup>pg 461</sup> 
- Subsystem 8 - _Conforming System_: Conforming consists of all the steps required to align the content of some or all the columns in a dimension with columns in similar or identical dimensions in other parts of the data warehouse.<sup>pg 461</sup> Incoming data from multiple systems needs to be combined and integrated so it is structurally identical, deduplicated, filtered of invalid data, and standardised in terms of content rows in a conformed image.<sup>pg 463</sup>

#### Delivering: Prepare for Presentation

- This stage involves the handoff of the dimension and fact tables.<sup>pg 463</sup>

- Subsystem 9 - _Slowly Changing Dimension Manager_: the ETL system must determine how to handle an attribute value that has changed from the value already stored in the data warehouse.
    - Type 1 Overwrite: take the revised data from the CDC and overwrite the dimension table contents. Type 1 is appropriate when correcting data or when there is no business need to keep history of previous values. Invalidates any aggregates built<sup>pg 465</sup>
    - Type 2 Add New Row: standard technique for accurately tracking changes in dimensions and associating them correctly with fact rows. There should be 5 housekeeping columns: changed date, row effective date/time, row end date/time, reason for change, current/expired flag. Does not invalidate any aggregates.<sup>pg 466</sup>
    - Type 3 Add New Attribute: "soft" attribute changes that allow a user to refer either to the old value of the attribute or the new value. Push the existing column values into the newly created column and populate the original column with new values. Invalidates any aggregates.<sup>pg 467</sup>
    - Type 4 Add Mini-Dimension: when a group of attributes in a dimension change sufficiently rapidly, they can be split off into a mini-dimension. Both the primary key of the main dimension and the primary key of the mini-dimension must appear in the fact table.<sup>pg 467</sup>
    - Type 5 Add Mini-Dimension and Type 1 Outrigger: builds on Type 4 but embeds a type 1 reference to the mini-dimension in the primary dimension. Allows for accessing the current values in the mini-dimension directly from the base dimension without linking through a fact table<sup>pg 468</sup>.
    - Type 6 Add Type 1 to Type 2 Dimension: has an embedded attribute that is an alternate value of a normal type 2 attribute in the base dimension.<sup>pg 468</sup>
- Subsystem 10 - _Surrogate Key Generator_: if the DBMS is used to assign surrogate keys, it is preferable for the ETL process to directly call the database sequence generator. The surrogate key generator should independently generate keys for every dimension.<sup>pg 469</sup>
- Subsystem 11 - _Hierarchy Manager_: multiple hierarchies simply coexist in the same dimension as dimension attributes. Normalised data structures are not recommended for the presentation level but may be appropriate in the ETL staging area to assist with populating/maintaining hierarchies in dimension tables.<sup>pg 470</sup>
- Subsystem 12 - _Special Dimension Manager_: placeholder in the ETL architecure for supporting organisation-specific dimensional design characteristics.
    - Date/Time dimensions: typically specified at the beginning of a data warehouse project.<sup>pg 470</sup>
    - Junk dimensions: creating new junk dimension rows on-the-fly requires assembling the junk dimension attributes and comparing them to the existing junk dimension rows to see if the row already exists. If not, a new dimension row must be assembled and surrogatae key created.<sup>pg 471</sup>
    - Mini-dimensions: technique used to track dimension attribute changes in a large dimension where the type 2 technique is infeasible, such as a customer dimension.<sup>pg 471</sup> ETL system must maintain a multicolumn surrogate key lookup table to identify base dimension member and appropriate mini-dimension row.<sup>pg 472</sup>
    - Shrunken subset dimensions: conformed dimensions that are a subset of rows and/or columns of one of your base dimensions, so ETL data flow should build conformed shrunken dimensions from the base dimension rather than independently to assure conformance.<sup>pg 472</sup>
    - Small static dimensions: created entirely by ETL system, i.e. lookup dimensions where an operational code is translated into words.<sup>pg 472</sup>
    - User maintained dimensions: dimensions with no formal system of record, i.e. custom descriptions, groupings and hierarchies created by the business for reporting and analysis. Ideally the appropriate busines user department agree to own the maintenance of these attributes and the DW/BI team provides a maintenance user interface.<sup>pg 473</sup>
- Subsystem 13 - _Fact Table Builders_: the ETL requirements to effectively build the transaction, perioid snapshot and accumulating snapshot fact tables.
    - Transaction fact table loader: transaction grain represents a measurement event defined at a particular instant. Bulk load new rows into the fact table, with the target fact table partitioned by time.<sup>pg 474</sup>
    - Periodic snapshot fact table loader: facts should describe only measures appropriate to the timespan defined by the period. Typically loaded en masse at the end of the appropriate period (daily, weekly or monthly).<sup>pg 474</sup>
    - Accumulating snapshot fact table loader: each time something happens, the snapshot fact row is destructively modified. Date foreign keys are overwritten and various facts are updated.<sup>pg 475</sup>
- Subsystem 14 - _Surrogate Key Pipeline_: step for replacing operational natural keys in the incoming fact table with the appropriate dimension surrogate keys.<sup>pg 475</sup> A surrogate key lookup needs to occur to substitute the operational natural keys in the fact table recird with the proper current surrogate key. Use the actual dimension table as the source for the most current value of the surrogate key corresponding to each natural key. Look up all the rows in the dimension with the natural key equal to the desired value and select the surrogate key that aligns with the historical context of the fact row using the begin and end effective dates.<sup>pg 476</sup> Late arriving facts need to be processed by finding the surrogate key where the fact transaction date is between the key's effective start and end date.<sup>pg 477</sup>
- Subsystem 15 - _Multivalued Dimension Bridge Table Builder_: the ETL system has the choice of creating a unique group for each set of observations, or reusing groups when an identical set of observations occurs.<sup>pg 477</sup>
- Subsystem 16 - _Late Arriving Data Handler_: for late arriving facts, you have to search back in history to decide which dimension keys were in effect when the activity occurred.<sup>pg 478</sup> For late arriving dimensions, support type 2 columns by adding the revised row to the dimension with a new surrogate key and then go and destructively modify any subsequent fact rows' foreign key. In the meantime, assign a new surrogate key with a set of dummy attributes until dimension values are available.<sup>pg 479</sup>
- Subsystem 17 - _Dimension Manager System_: prepares and publishes conformed dimensions to the data warehouse community. Implements the common descriptive labels; adds new rows to the conformed dimension for the new source data and generate new surrogate keys; add new rows for type 2 changes and generate new surrogate keys; modify rows in place for type 1 and type 3 changes; maintain dimension version numbers (for type 1 or 3); release dimension simultaneously to all fact table providers.<sup>pg 480</sup>
- Subsystem 18 - _Fact Provider System_: fact table providers are responsible for receiving conformed dimensions from the dimension manager, and creating, maintaining and using one or more fact tables. Add all new rows to fact tables after replacing their natural keys with correct surrogate keys; modify rows in fact tables for errors/accumulating snapshots/late arriving dimensions; remove/recalculate aggregates and quality check; and bring fact and dimension tables online.<sup>pg 481</sup>
- Subsystem 19 - _Aggregate Builder_: like indexes, specific data structures created to improve performance. Build and use aggregates without causing distraction or consuming extraordinary resources. User feedback on slow-running queries provides input to designing aggregations. Populate and maintain aggregate fact table rows and shrunken dimension tables where needed by aggregate fact tables. Aggregates need to be taken off-line when they are not consistent with the base data.<sup>pg 481</sup>
- Subsystem 20 - _OLAP Cube Builder_: relational databases are best at providing storage and management, and the foundation of OLAP cubes (if part of architecture).<sup>pg 482</sup>
- Subsystem 21 - _Data Propagation Manager_: presenting conformed, integrated enterprise data from the data warehouse presentation layer to other environments for special purposes, i.e. sharing with business partners, customers and vendors. Since most third parties cannot directly query the existing data warehouse, data need to be extracted from presentation layer and loaded into proprietary structures required by third party. Data propagation is part of the ETL system.<sup>pg 482</sup>

#### Managing the ETL Environment

- One of the goals for the DW/BI system is to build a reputation for providing timely, consistent, and reliable data to empower the business. It should be operated in a _professional manner_ with approprite SLAs per subsystem. There are three criteria to achieve this goal:<sup>pg 483</sup>
  - Reliability: ETL processes must consistently run to completion on time.
  - Availability: data warehouse should be available as promised.
  - Manageability: a successful data warehouse constantly grows along with the business, and ETL processes need to evolve as well.

- Subsystem 22 - _Job Scheduler_: ETL schedulers should be managed through a single metadata driven job control environment, needs to be aware of and control the relationships and dependencies between ETL jobs, and recognise when a table is ready to be processed.<sup>pg 483</sup> It needs:
  - Job definition: define a series of steps as a job and specify some relationship among jobs.<sup>pg 484</sup>
  - Job scheduling: provides time- and event-based scheduling, including ability to monitor database flags, check for the existence of files, and compare creation dates.<sup>pg 484</sup>
  - Metadata capture: needs to capture information about what step the load is on, what time it started, and how long it took, by having each step write to a log file.<sup>pg 484</sup> 
  - Logging: collecting information about the entire ETL process and logging to a database (text file at a minimum). Support recovery and restarting of a process.<sup>pg 484</sup>
  - Notification: ETL processes should run without human intervention, without fail. But when problems do occur, control systems need to interface to the problem notification system.<sup>pg 484</sup>
- Subsystem 23 - _Backup System_: Backup and recovery processes are often designed as part of the ETL system, including backing up of intermediate staging data necessary to restart failed ETL jobs.
  - Backup: usually images of the database at a certain point in time, including indexes and physical layout. Need to be high performance (backups that do not impact performance); simple to administer (identify objects to backup, create schedules and logs, and maintain backup verifcation); automated (storage management, automated schedling).<sup>pg 485</sup>
  - Archive and retrieval: do not throw data away but put it somewhere that costs less but is still accessible. Audit trails for access and alterations to archived data need to be kept.<sup>pg 486</sup> Archives should include copies of the ETL system alongside the data.<sup>pg 495</sup>
- Subsystem 24 - _Recovery and Restart System_: used for either resuming a job that has halted (ETL system needs reliable checkpoint functionality to restart jobs at exactly the right point) or for backing out the whole job and restarting from the beginning. Fact table surrogate keys can be used to help identify where jobs need to be restarted from/which rows need to be backed out.<sup>pg 487</sup>
- Subsystem 25 - _Version Control System_: snapshotting capability for archiving and recovering all the _logic and metadata_ of the ETL pipeline. Each subsystem, ETL modules and jobs need to be version controlled.<sup>pg 488</sup> Data may need to be reprocessed using exact versions of an ETL system.<sup>pg 495</sup>
- Subsystem 26 - _Version Migration System_: ETL jobs designed and developed need to be bundled and migrated to the next environment - from development to test to production. Everything done to the production system should have been designed in development and the deployment script tested on the test environment. Should interface with the version control system to back out a migration. Every operation should go through this process.<sup>pg 488</sup> Effectively "CI/CD".
- Subsystem 27 - _Workflow Monitor_: the ETL system must be constantly monitored to ensure ETL processes are operating efficiently and the warehouse is being loaded on a consistently timely basis. Monitor job status for all jobs initiated, including number of records processed and summaries of errors.<sup>pg 489</sup>
- Subsystem 28 - _Sorting System_: sometimes needed for aggregating and joining flat file sources. Sorting can produce aggregates where each break row of a given sort is a row for the aggregate table, and sorting plus counting is often a good way to diagnose data quality issues.<sup>pg 490</sup>
- Subsystem 29 - _Lineage and Dependency Analysis_:
  - Lineage: beginning with a specific data element in a BI report, identify the _source_ of that data element, upstream intermediate tables, and all transformations applied. The ETL system must display the physical sources and all subsequent transformations of any selected data element.<sup>pg 490</sup> Show where a final piece of data came from, and fully document all the transformations.<sup>pg 495</sup>
  - Dependency: beginning with a specific data element in a source table, identify all downstream intermediate tables and final BI reports containing that data element, and all transformations applied to that data element.<sup>pg 490</sup> Important when assessing changes to a source system and the downstream impacts on the data warehouse and ETL system.<sup>pg 491</sup> Show where an original source data element was ever used.<sup>pg 495</sup>
- Subsystem 30 - _Problem Escalation System_: ETL teams develop the ETL processes and the QA teams test them thoroughly before they are turned over to those responsible for day-day operations. First level support for ETL system should be a group dedicated to monitoring production applications (usually a helpdesk); the ETL development team gets involved only if operational support cannot solve. If a problem does occur, the ETL process should automatically notify the problem escalation system, with each notification event should be written to a database to analyse the types of problems that arise.<sup>pg 491</sup>
- Subsystem 31 - _Parallelizing/Pipelining System_: enables the ETL system to take advantage of multiple processors to process large amounts of data. Highly desirable that parallelizing and pipelining be automatically invoked for every ETL process. Extraction processes can be parallelized by logically partitioning on ranges of an attribute.<sup>pg 492</sup>
- Subsystem 32 - _Security System_: adminster role-based security on all data and metadata in the ETL system. Enforce comprehensive authorised access to all ETL data and metadata by individual and role, and strictly controlled access to the production data warehouse, but privileged access to the development environments. Maintain historical records of access logs. Ability to show who has accessed or modified data and transforms.<sup>pg 495</sup> Ensure data moved across networks are encrypted.<sup>pg 493</sup>
- Subsystem 33 - _Compliance Manager_: tracking exact condition and content of data at any point in time that may have been under the control of the data warehouse; archives of data as it was originally received; who had authorized access.<sup>pg 493</sup>
- Subsystem 34 - _Metadata Repository Manager_: capture ETL metadata, including the process metadata, technical metadata and business metadata. Make sure somebody on the ETL team is assigned the role of metadata manager.<sup>pg 495</sup>

### ETL System Design and Development

- Before beginning ETL system design for a dimensional model, logical designs, high-level architectures and source-target mappings should have been completed. Should transform processing burden be hosted on source system, target system or its own platform?<sup>pg 497</sup>
- Develop the high-level ETL plan:
  - Step 1 - _Draw the High-Level Plan_: start with a simple schematic of the sources and targets.<sup>pg 498</sup>
  - Step 2 - _Choose an ETL Tool_: read from a range of sources (flat files, ODBC, OLE DB, native database drivers), define transformations, write into a variety of target formats. This can be achieved by developing internally or using an ETL tool.<sup>pg 499</sup>
  - Step 3 - _Develop Default Strategies_: default strategies for common activities in ETL system, including determining the default method of extraction from each source system, lifecycle policy on extracted/staged data, policing data quality, managing changes to dimension attributes, when each source becomes available, audit information to be appended to every row that enters the system, how data are staged (written to disk for a later ETL step).<sup>pg 500</sup>
  - Step 4 - _Drill Down by Target Table_: after overall strategies have been defined, drill into detailed transformations needed to populate each target table.<sup>pg 500</sup> Develop detailed table schematics.<sup>pg 501</sup>> {insert image from pg 501}
  - Step 5 - _Populate Dimension Tables with Historic Data_: more often than not you build separate ETL processes for the historic and ongoing loads, although significant functionality can often be reused from one to the other.<sup>pg 503</sup> Start building the simplest dimension tables.<sup>pg 503</sup>.
    - For Type 1 dimensions, extract the current value for each dimension attribute from the source.<sup>pg 504</sup> 
    - For Type 2 dimensions, you need to reconstruct history for dimension attributes that are managed as Type 2. Business users will want history going back in time, not just from the date the data warehouse is implemented. Less suited to SQL.<sup>pg 507</sup>
    - Dimension transformations: data type conversions; combine data from separate sources to derive dimensions; decode production codes (lookups often stored in a staging database table); validate many-one and one-one relationships (i.e. rollup paths); dimension surrogate key assignment (i.e. assign surrogate keys when confident dimension tables have one row for each true unique dimension member).<sup>pg 506</sup>
    - Dimension table loading: bulk load where possible.<sup>pg 507</sup>
    - Populate static dimensions: date dimensions, budget scenario dimensions holding Actual and Budget.<sup>pg 508</sup>
- Step 6 - _Populate Fact Tables with Historic Data_: can take several days due to sheer volumes of data.<sup>pg 508</sup>
  - Establish credibility of data warehouse by cross-checking integrity, i.e. gathering audit statistics such as sums and counts that you can compare between data warehouse and source systems.<sup>pg 509</sup>
  - Fact table transformations: ensure null values are stored as `null` so sums and averages "do the right thing".<sup>pg 509</sup> Exchange natural keys for the surrogate keys (dimension table processing must be complete before the fact data enters the surrogate key pipeline), i.e. by joining fact staging table to each dimension table on the natural keys and returning the facts and surrogate keys from each dimension table.<sup>pg 511</sup> When all the fact source keys have been replaced with surrogate keys, the fact row is ready to load.<sup>pg 511</sup> Assign audit dimension keys to each fact row to describe characteristics of the load, i.e. source table, quality checks passed.<sup>pg 511</sup>
  - Fact table loading: look for fast-loading featuers in database, and experiment with different batch sizes.<sup>pg 512</sup>
- Step 7 - _Dimension Table Incremental Processing_: incremental ETL system development begins with the dimension tables.<sup>pg 512</sup>
  - Pull the current snapshots of the dimension tables in their entirety and let the transformation step determine what has changed, although this can take a long time to look up each entry in a dimension table. Preferably the extracts would contain only rows that have changed. <sup>pg 513</sup>
  - Find new dimension members by performing a lookup from the incoming stream to the master dimension, comparing on natural key. Any rows that fail the lookup are new dimension members and should be inserted into the dimension table. Then determine if the incoming dimension row has changed by comparing column by column between incoming data and current corresponding member stored in the master dimension table.<sup>pg 513</sup> You can add two new housekeeping columns to the dimension table: hash of concatenated Type 1 attributes, and similarly for Type 2, then compute hashes on the incoming rowset and compare to the stored values, which can be more efficient than pair-wise comparisons on dozens of columns.<sup>pg 514</sup>
  - Process changes to dimension attributes by deciding if you already have that row. If all the incoming dimensional information matches the corresponding row in the dimension table, no further action is required. If the dimensional information has changed, then you can apply changes to the dimension, such as Type 1 or Type 2.<sup>pg 514</sup>
- Step 8 - _Fact Table Incremental Processing_: it is more efficient to incrementally load only the records that have been added or updated since the previous load.<sup>pg 515</sup>
  - Untransformed data should be written to the staging area to support: archive for auditability; starting point after data quality verification; starting point for restarting the process.<sup>pg 516</sup>
  - Perform the surrogate key lookups against a query, view or physical table that subsets the dimension table. Handle referential integrity errors automatically. <sup>pg 517</sup>
  - Late arriving facts must be associated with the version of the dimension member that was in effect when the fact occurred.<sup>pg 517</sup>
  - Incrementally load transaction fact tables with inserts on small to medium systems.<sup>pg 517</sup> Accumulating snapshot fact tables are typically smaller in size but more expensive to maintain than transaction and periodic snapshot fact tables.<sup>pg 518</sup>
  - Processing only data that has been changed is one way to speed up the ETL cycle. Shift to more frequent processing cycles to process less data. Parallelize the ETL process by having multiple steps running in parallel, or a single step running in parallel.<sup>pg 518</sup>
- Step 9 - _Aggregate Table and OLAP Loads_: aggregate tables are simply the results of a really big aggregate query stored as a table.<sup>pg 519</sup> This gets complex when an aggregate is along the date dimension - the current month must be updated, or dropped and re-created to incorporate the current day's data. Same goes for any Type 1 attribute.<sup>pg 519</sup>
- Step 10 - _ETL System Operation and Automation_:
  - Job schedulers should contain functionality to schedule a job to kick off at a certain time, and conditionally execute a second task if the first task is successful.<sup>pg 520</sup>
  - Should have comprehensive error handling to make sure jobs run to completion.<sup>pg 520</sup>
  - Assign sequential surrogate keys to new fact table rows to allow the load to resume from a certain reliable point, or back out the load by constraining on a range of the surrogate keys.<sup>pg 520</sup> 
- Real-time Implications
  - Real-time design challenges can be divided into: daily, intra-day, and instantaneous. Daily updated data usually involves reading a batch file prepared by the source system or performing an extract query at a convenient time on the source system. Intra-day data usually involves tapping into a message queue between applications, or low level database triggers whenever something happens in the transactional system, and means data are updated many times per day but not guaranteed to be the absolute current truth. Instantaneous is usually implemented as an Enterprise Information Integration (EII) solution.<sup>pg 521</sup>
  - Real-time delivery should not compromise overall ETL solution goals of quality, integration, security, compliance, backup, recovery and archiving. Replacing batch files with reading from message queues can lead to incomplete or incorrect data because additional transactional information may arrive later. [Column screen](#cleaning-and-conforming) tests on single fields should survive, but structure and business rule screens by definition require multiple fields, records and tables may not be possible.<sup>pg 523</sup>
  - If the real-time system cannot wait for the dimensions to be resolved, then old copies of the dimensions must be used. When revised versions of the dimensions are received, the data warehouse may decide to post these to a hot partition, or delay processing until the end of the day. Hence there may be an ephermeral window of time where the dimensions do not exactly describe the facts.<sup>pg 523</sup>
  - One design solution for responding to real-time data is to build a real-time partition as an extension of the conventional, static data warehouse.<sup>pg 524</sup>

### Common Dimensional Modeling Mistakes to Avoid<sup>pg 397</sup>

- Place text attributes in a fact table. The process of creating a dimensional model is always a kind of triage. The numeric measurements delivered from an operational business process source belong in the fact tablel; the descriptive textual attributes comprising the context of the measurements go in dimension tables<sup>pg 397</sup>.
- Limit verbose descriptors to save space. Dimension tables are geometrically smaller than fact tables - our job as designers of easy-to-use dimensional models is to supply as much verbose descriptive context in each dimension as possible<sup>pg 398</sup>.
- Split hierarchies into multiple dimensions. Your job is to present the hierarchies in the most natural and efficient manner in the eyes of the users, not in the eyes of the data modeler. Resist the urge to snowflake a hierarchy<sup>pg 398</sup>.
- Ignore the need to track dimension changes. Three basic techniques track slowly moving attribute changes, do not rely on Type 1 exclusively. Business users often want to understand the impact of changes on a subset of the dimension table's attributes<sup>pg 398</sup>.
- Solve all performance problems with more hardware. Building aggregates, partitioning, creating indices, choosing query-efficient DBMS software, increasing memory size, increasing CPU speed, adding parallelism can be achieved first<sup>pg 399</sup>.
- Use operational keys to join dimensions and facts. Do not declare the dimension key to be the operational key. Dimension keys should be replaced with simple integer surrogate keys sequentially numbered from 1 to _N_ where _N_ is the total number of rows in the dimension table<sup>pg 399</sup>.
- Neglect to declare and comply with the fact grain. All dimensional designs should begin by declaring the business process that generates the numeric performance measurements, followed by the exact granularity of the data (building fact tables at the most atomic, granular level will gracefully resist the ad hoc attack), followed by surrounding those measurements with dimensions that are true to the grain. Each different measurement grain demands its own fact table<sup>pg 399</sup>.
- Use a report to design the dimensional model. A dimensional model has nothing to do with an intended report. Rather, it is a model of a measurement process. Numeric measurements form the basis of fact tables; the dimensions appropriate for a given fact table are the context that describes the circumstances of the measurements. A dimensional model is independent of how a user chooses to define a report<sup>pg 400</sup>.
- Expect users to query normalised atomic data. Normalized models may be helpful for preparing data in the ETL kitchen, but they should never be used for presenting the data to business users<sup>pg 400</sup>.
- Fail to conform facts and dimensions. The single most important design technique in the dimensional modeling arsenal is conforming dimensions. When you conform dimensions across fact tables, you can drill across separate data sources because the constraints and row headers mean the same thing and match at the data level<sup>pg 401</sup>.

#### Lifecycle Pitfalls<sup>pg 426</sup>

- Do not become overly enamored with technology rather than focusing on business requirements.
- Fail to embrace an influential and accessible senior manager as a sponsor.
- Do not tackle multi-year projects. Instead pursue iterative development efforts.
- Do not neglect front room query performance by focussing too heavily on back room operational performance.
- Do not populate models without regard to a data architecture.
- Do not load only summaryd data into the dimensional models.
- Do not presume business requirements are static.
- Do not forget that DW/BI success is tied directly to business acceptance and improved decision making.

### Big Data Analytics

- Structured, semi-structured, un-structured and raw data in many different formats. Conventional RDBMSs and SQL simply cannot store or analyze the wide range of data and use-cases.<sup>pg 528</sup>
- RDBMSs must allow the new data types to be processed within the DBMS inner loop by means of specially crafted user-defined functions (UDFs).<sup>pg 530</sup>
![Extended RDBMS Architecture](/images/dwtk-ch21-extendedrdbms.jpg)
- MapReduce is a UDF execution framework, where the "F" can be extraordinarily complex.<sup>pg 530</sup> A flexible, general purpose environment for many forms of ETL processing.<sup>pg 532</sup>
- Best practises for big data:<sup>pg 531</sup>
  - Choose data sources from business needs.
  - Focus on user interface and performance.
  - Divide the world into dimensions and facts. Dimensions can be used to integrate data sources.<sup>pg 538</sup> 
  - Track time variance with slowly changing dimensions.
  - Use durable surrogate keys. Durable means there is no busines rule that can change the key.<sup>pg 539</sup> Surrogate means the keys are simple integers assigned in sequence or generated by a robust hashing algorithm that guarantees uniqueness.<sup>pg 539</sup> Much big data will never end up in a relational database; rather it will stay in Hadoop or object storage. But it can still be combined with conformed dimensions and surrogate keys in single analyses.<sup>pg 539</sup>
  - Be prepared to re-program and re-implement data science proofs of concept in a standard of set of technologies. Be tolerant to the range of technologies data scientists use.<sup>pg 532</sup>
  - Declare data structures at time of loading into Hadoop or a data grid. This conflicts with traditional RDBMS methodologies which puts a lot of emphasis on modeling the data carefully before loading. Loading data as simple name-value pairs into RDBMS named columns can provide a valuable ETL step.<sup>pg 540</sup>
  - Prototype using data virtualisation, a powerful way to prototype data structures and make rapid alterations or provide distinct alternatives. Expect to materialise the virtual schemas.<sup>pg 541</sup>
  - For most forms of analysis, personal details must be masked and data aggregated enough not to allow identification of individuals. Hadoop does not manage updataes very well so data should be masked or encrypted on write.<sup>pg 542</sup>