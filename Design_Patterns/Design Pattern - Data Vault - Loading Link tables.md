# Design Pattern - Data Vault - Loading Link tables

## Purpose
This design pattern describes the loading process for �Link� tables in the Data Vault concept.

## Motivation
The Link concept in Data Vault provides the flexibility of this data modeling approach. Links are sets of (hub) keys that indicate that a relationship between those Hubs has existed at some point in time. A Link table is similar in concept to the Hub table, but only stores key pairs. The structure of the relationship tables (including the Link table) is documented in the Integration FrameworkA120 � Integration Layer document of the Outline Architecture. Even though the Data Vault concept allows for adding attributes in the Link table it is strongly recommended (for flexibility reasons) to only store the generated hashkey (meaningless key), the Hub surrogate hashkeys and the date/time information. Doing this will ensure compatibility with both stationary facts (time dependent facts such as balances) and pure transactions.
Link to file Eru Marumaru
Also known As
Relationship table

## Applicability
This pattern is only applicable for loading processes from the Staging Area into the Integration Area and from the Integration Area to the Interpretation Area. The pattern varies slightly for the type of Link table specified (such as Transactional Link, Same-As Link, Hierarchical Link or Low-Value Links) or whether it contains a degenerate attribute. In most cases the Link table will contain the default attributes (Link Key, Hub Keys and metadata attributes) but in the case of a pure transactional Link table it can contain the transaction attributes as well.

## Structure
The ETL process can be described as an �insert only� set of the unique combination of Data Warehouse keys. Depending on the type of source table, the process will do the following:
Source Area to Integration Area: the process executes a SELECT DISTINCT query on business keys and performs key lookups (outer join) on the corresponding Hub tables to obtain the Hub Data Warehouse keys. The resulting key combination is then verified using a key lookup against the target Link table to verify if that specific combination of Data Warehouse keys already exists. If it exists, the row can be discarded, if not it can be inserted.
Integration Area to Interpretation Area: the process executes a SELECT DISTINCT query on Data Warehouse keys (likely after combining multiple tables first) and performs a key lookup against the target Link table to verify if that specific combination of Data Warehouse keys already exists. If it exists, the row can be discarded, if not it can be inserted.
The maintenance of the Interpretation Area can also be done as part of an (external) process or through Master Data Management. In this context, Link tables between Integration and Interpretation Area tables are very similar to cross-referencing tables.
The following diagram displays the ETL process for Link tables;
Business Insights > Design Pattern 010 - Data Vault - Loading Link tables > image2015-4-29 16:24:14.png
This image needs updating to reflect DV 2.0 (hashkey) useage Eru Marumaru
In a pure relational Link it is required that a dummy key is available in each corresponding Link-Satellite to complete the timelines. This is handled as part of the Link-Satellite processing as a Link can contain multiple Link-Satellites. Dummy records are only required to be inserted for each driving key as a view in time across the driving key is ultimately required. Inserting a dummy record for every Link key will cause issues in the timeline. This is explained in more detail in the Link-Satellite Design Pattern.

## Implementation Guidelines
Use a single ETL process, module or mapping to load the Link table, thus improving flexibility in processing. Every ETL process should have a distinct function.
Multiple passes of the same source table or file are usually required. The first pass will insert new keys in the Link table; the other passes are needed to populate the Link Satellite tables (if any).
By default, create a sequence / meaningless key for each unique key combination in a Link table.
Link tables can be seen as the relationship equivalent of Hub tables; only distinct new key pairs are inserted.
Date/time information is copied from the Staging Area tables and not generated by the ETL process.
The logic to create the initial (dummy) Satellite record can both be implemented as part of the Link ETL process, as a separate ETL process which queries all keys that have no corresponding dummy or as part of the Link-Satellite ETL process. This depends on the capabilities of the ETL software since not all are able to provide and reuse sequence generators or able to write to multiple targets in one process. The default and arguably most flexible way is to incorporate this concept as part of the Link-Satellite ETL since it does not require rework when additional Link-Satellites are associated with the Link. This means that each Link-Satellite ETL must perform a check if a dummy record exists before starting the standard process (and be able to roll back the dummy records if required).
Depending on how the Link table is modelled (what kind of relationship it manages) the Link table may contains a relationship type attribute. If a link table contains multiple, or changing, relationships (types) this attributes is moved to the Link-Satellite table.
Ending /closing relationships is always done in the Link-Satellite table, typically using a separate ETL process.

## Considerations and Consequences
Multiple passes on source data is likely to be required. In extreme cases a single source table might be used (branch out) to Hubs, Satellites, Links and Link Satellites.
 Known Uses
This type of ETL process is to be used for loading all link tables in both the Integration Area as well as the Interpretation Area. This is because the Link table is also used to relate raw (Integration Area) data and cleansed (Interpretation Area) data together.

## Related Patterns
Design Pattern 006 � Generic � Using Start, Process and End Dates
Design Pattern 008 � Data Vault � Loading Hub tables
Design Pattern 009 � Data Vault � Loading Satellite tables
Design Pattern 013 � Data Vault � Loading Link Satellite tables
Discussion items (not yet to be implemented or used until final)
None.