## Storage

## Types of Data

Types of Data

That is not a surprise now but data is everywhere in several different formats. We have three main categories of data we may deal with:

- ***Structured data*** also referred as relational data. Data has a strict schema that defines field names and their data types but also relations between the tables thank to keys. It is easy to use and query because all the data follow the same format.
- ***Semi-structured data*** as XML, JSON and YAML documents. Here the data is not organized as table but they do have structure (tree-structure typically). They can have a schema though but is not strictly enforced as in RDBMS. This gives more flexibility to update the data by adding or removing fields.
- ***Unstructured data*** do not have any kind of schema or structure. This concerns files (pdf, word), audio, videos, images.

### Databases

#### Relational databases

To store structured data we typically use relational databases. Relational databases impose a tabular structure to the data (tables or *relations*). Table are called relations or entities, a row is a record (tuple) and columns are attributes of the data. You can have relationships between two tables by using keys.

![]({{ site.baseurl }}/images/2021-07-27-01-data-storage.png)

On RDBMS you can perform *transactions*, a set of operation to update data in your database. Typically transactions conform to the **atomic, consistent, isolated, durable** (ACID) model for updating these information.

- *Atomicity*: if a transaction consists of several sub-operations to update the data, these sub operations will be referred as **ONE UNIT**: if all sub-operations succeed then the transaction succeeds. If one or more sub-operation fails the transaction roll-back.
- *Atomicity*: the transaction cannot bring the database to an invalid state. Once the transaction is committed or rolled-back, the rules for each record will still apply and future transaction will see the effect of previous transactions.
- *Isolation*: executing several transactions has the same effect as executing them sequentially. Each transaction is stored in a queue.
- *Durability*: any committed transaction is written to non-volatile storage and it cannot be lost.

#### Non-relational databases

These databases do not use the traditional tabular schema found in RDBMS. They instead use a storage model optimized for the data being stored. Among the NoSQL landscape you can find *document data store* (MongoDB), *columnar data store* (MariaDB, AWS Redshift, Azure SQL Data Warehouse), *key/value pairs* (Redis, HBase) stores and graph databases (Neo4J).