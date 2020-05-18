# Ch 2 : Data Models and Query Languages

## Birth of NoSQL

Several driving forces behind the adoption of NoSQL databases are:
- A need for greater scalability than relational databases can easily achieve, including
very large datasets or very high write throughput
- A widespread preference for free and open source software over commercial
database products
- Specialized query operations that are not well supported by the relational model
- Frustration with the restrictiveness of relational schemas, and a desire for a more
dynamic and expressive data model

\
The JSON representation has better locality than the multi-table schema. All the data related to a certain object can be read in a single query, which will involve much complicated joins and multiple queries in relational model.

\
Removing duplication like storing location as an id from the location table instead of storing as a string is the
key idea behind **normalization** in databases.

\
Unfortunately, normalizing this data requires many-to-one relationships (many people live in one particular region, many people work in one particular industry), which
don’t fit nicely into the document model. In relational databases, it’s normal to refer to rows in other tables by ID, because joins are easy. In document databases, joins are
not needed for one-to-many tree structures, and support for joins is often weak.

If the database itself does not support joins, you have to emulate a join in application code by making multiple queries to the database, which increases the application code and generally reduces performance.

\
In a relational database, the **query optimizer** automatically decides which parts of the query to execute in which order, and which indexes to use. They are made automatically by the query optimizer, not by the application developer, thus increasing performance and reducing application code. 

## Document vs Relational Databases 
When it comes to representing many-to-one and many-to-many relationships, relational and document databases are not fundamentally different: in both cases, the related item is referenced by a unique identifier, which is called a *foreign
key* in the relational model and a *document reference* in the document model

The main arguments in favor of the document data model are schema flexibility, better performance due to locality, and that for some applications it is closer to the data
structures used by the application. The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.

\
schema based distinction - 
- document model - *schema-on-read* (the structure of the data is implicit, and only interpreted when the
data is read)
- relational model - *schema-on-write* (the traditional approach of relational databases, where the schema is explicit and the database ensures all written data conforms
to it)

\
The difference between the approaches is particularly noticeable in situations where an application wants to change the format of its data (initially had name field, later on need first name) - 
- In a document database, you would just start writing new documents with the new fields and have code in the application that handles the case when old documents are read. 
- In relational database, we will have to do a schema updation. 
    - Schema changes have a bad reputation of being slow and requiring downtime. This reputation is not entirely deserved: most relational database systems execute the
    ALTER TABLE statement in a few milliseconds. MySQL is a notable exception—it copies the entire table on ALTER TABLE, which can mean minutes or even hours of
    downtime when altering a large table.
    - Running the UPDATE statement on a large table is likely to be slow on any database, since every row needs to be rewritten. If that is not acceptable, the application can
    leave first_name set to its default of NULL and fill it in at read time, like it would with a document database

The schema-on-read approach is advantageous if the items in the collection don’t all have the same structure for some reason (i.e., the data is heterogeneous) for example,
because:
- There are many different types of objects, and it is not practical to put each type of object in its own table.
- The structure of the data is determined by external systems over which you have no control and which may change at any time.

In situations like these, a schema may hurt more than it helps, and schemaless documents can be a much more natural data model. But in cases where all records are expected to have the same structure, schemas are a useful mechanism for documenting and enforcing that structure

\
PostgreSQL since version 9.3 [8], MySQL since version 5.7, and IBM DB2 since version10.5 [30] have support for JSON documents. On the document database side, RethinkDB supports relational-like joins in its query language, and some MongoDB drivers automatically resolve database references.

A hybrid of the relational and document models is a good route for databases to take
in the future

\
## Query languages for Data
**Imperative language** - An imperative language tells the computer to perform certain operations in a certain
order. You can imagine stepping through the code line by line, evaluating conditions, updating variables, and deciding whether to go around the loop one more time.

**Declaritive** - In a declarative query language, like SQL or relational algebra, you just specify the pattern of the data you want—what conditions the results must meet, and how you want the data to be transformed (e.g., sorted, grouped, and aggregated)—but not how to achieve that goal. It is up to the database system’s query optimizer to decide which indexes and which join methods to use, and in which order to execute various parts of the query.

declarative languages often lend themselves to parallel execution. Today, CPUs are getting faster by adding more cores, not by running at significantly higher
clock speeds than before [31]. Imperative code is very hard to parallelize across multiple cores and multiple machines, because it specifies instructions that must be performed
in a particular order. Declarative languages have a better chance of getting faster in parallel execution because they specify only the pattern of the results, not the algorithm that is used to determine the results. The database is free to use a parallel implementation of the query language, if appropriate.

## Map Reduce Querying

A limited form of MapReduce is supported by some NoSQL datastores, including MongoDB and CouchDB, as a
mechanism for performing read-only queries across many documents

MapReduce is neither a declarative query language nor a fully imperative query API, but somewhere in between: the logic of the query is expressed with snippets of code, which are called repeatedly by the processing framework

\
\
TO DO :
## Graph like data-models
## The Foundation - Datalog