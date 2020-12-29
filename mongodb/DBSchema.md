# Mongo DB Schema Design Patterns 

## Relational vs. MongoDB Schema Design Approaches

### Relational Schema Design
- Model data independent of queries 
- Normalize in the 3rd form - Don't duplicate data

### MongoDB Schema Design
- No Rules, No Process, No Algorithm
- Design a schema that will work well for a given application
- Things to consider - 
    - How to store the data
    - Query Performance
    - Reasonable amount of hardware

eg - 
```python
{
 first_name: "Paul",
 surname: "Miller",
 cell: "447557505611",
 city: "London",
 location: [45.123,47.232],

}
profession: ["banking", "finance", "trader"],
cars: [
 {
 model: "Bentley",
 year: 1973
 },
 {
 model: "Rolls Royce",
 year: 1965
 }
]
```

## Embedding vs. Referencing

## Embedding 

This is similar to JOIN in relational databases, combining data of multiple tables together. 

JOIN is an expensive operation in relational databases as it requires all the data to be loaded into the memory. It is both memory and time intensive.

eg - 
```python 
{
 _id : ObjectId('AAA'),
 name: 'Kate Monster',
 ssn: '123-456-7890',
 addresses: [
 {
 street: '123 Sesame St’,
 city: 'Anytown', cc: ‘USA'
 },
 {
 street: '123 Avenue Q',
 city: 'New York',
 cc: 'USA'
 }
 ]
}
```

**Pros** - 
- Retrieve all data with a single query
- Avoids expense JOINs or $lookup
- Update all data with a single atomic operation

**Cons** - 
- Large docs === more overhead
- 16 MB Document size limit

## Referencing

Break up a document into smaller documents which are connected via their object Ids.

**Pros** - 
- Smaller documents
- Less likely to reach 16 MB limit
- No duplication of data
- Infrequently accessed data not accessed on every query

**Cons** - 
- Two queries or $lookup required to retrieve all data

eg - 
```python 

 name : 'left-handed smoke shifter',
 manufacturer : 'Acme Corp',
 catalog_number: 1234,
 parts : [
 ObjectID('AAAA'),
 ObjectID('BBBB'),    
 ObjectID('CCCC')
 ]
}
,
{
_id : ObjectID('AAAA'),
partno : '123-aff-456',
name : '#4 grommet',
qty: 94,
cost: 0.94,
price: 3.99
}
```

## Types of Relationships

## One to One 

Directly add key value pair in the document.

## One to few 

Prefer Embedding. therefore embed in the document itself. 

for eg - 
```python 
{
_id: ObjectId("AAA"),
name: "Joe Karlsson",
company: "MongoDB",
twitter: "@JoeKarlsson1",
twitch: "joe_karlsson",
tiktok: "joekarlsson",
website: "joekarlsson.com",
addresses: [
{ street: "123 Sesame St", city: "Anytown", cc: "USA" },
{ street: "123 Avenue Q", city: "New York", cc: "USA" }
]
}
```

Multiple addresses in the above example are directly embedded in the document.

## One to Many

In this case, it may cause excessive overlead to store all the data embedded in the documents, and may even reach the document size limit. Therefore instead of storing the document embedded, seperate it out and only store its reference in the parent document. 

eg -
```python 
Products : 

{
_id: ObjectId("123"),
name: "left-handed smoke shifter",
manufacturer: "Acme Corp",
catalog_number: 1234,
parts: [
ObjectId("AAA"),
ObjectId("BBB"),
ObjectId("CCC"),
]
}

Parts : 
{
_id: ObjectId("AAA"),
partno: "123-ABC-456",
name: "#4 grommet",
qty: 94,
cost: 0.54,
price: 2.99,
}
```


## One to Squillions

When the number of sub documents increase to a very large number, it doesnt make sense to even store the object ids. Instead, store the parent data in the sub documents, therefore performing a reverse-id lookup. 

eg (In case of log messages) - 

```python 
Host : 
{
_id: ObjectId("AAA"),
name: "goofy.example.com",
ipaddr: "127.66.66.66",
}

Log Messages : 
{
_id: ObjectId("123"),
time: ISODate("2014-03-28T09:42:41.382Z"),
message: "The CPU is on fire!!!",
host: ObjectId("AAA"),
},
{
_id: ObjectId("456"),
time: ISODate("2014-03-28T09:42:41.382Z"),
message: "Drive is hosed",
host: ObjectId("AAA"),
}

```



## Many to Many 

Example of this can be a todo list application, where a user can have multiple tasks, but a single task can be in the list of multiple users. 

eg - 
```python 
Person - 
{
_id: ObjectId("AAF1"),
name: "Joe Karlsson",
tasks: [
ObjectId("ADF9"),
 ObjectId("AE02"),
 ObjectId("ZDF2"),
]
}

Tasks - 
{
_id: ObjectId("ADF9"),
description: "Learn MongoDB",
due_date: ISODate("2014-03-28T09:42:41.382Z"),
owner: ObjectId("AAF1"),
},
{
_id: ObjectId("AE02"),
description: "Write lesson plan",
due_date: ISODate("2014-03-28T09:42:41.382Z"),
owner: ObjectId("AAF1"),
},

```


## Rules 

**Rule 1** - Favor embedding unless there is a compelling reason not to.

**Rule 2** - Needing to access an object on its own is a compelling reason not to
embed it.

**Rule 3** - Avoid JOINs and $lookups if
they can be, but don’t be afraid if they can provide a better schema design.


**Rule 4** - Arrays should not grow without bound.

**Rule 5** - How you model your data
 depends – entirely – on your  particular application’s data  access patterns.
