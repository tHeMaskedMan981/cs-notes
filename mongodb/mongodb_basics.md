# MongoDB Basics 

## MongoDB export and dump 

### Mongodb export 
exports the documents of a certain collection in json format - 
```bash
mongoexport --uri "mongodb://localhost:27017/round" --collection mobikwik --out /home/akash/test.json

mongoexport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --collection=sales --out=sales.json

mongoimport --uri="mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies" --drop sales.json
```
while importing data, we use `--drop` to remove any existing data in the collection. If this is not included and the collection already contains overlapping data, we will get duplicate key error. 

### Mongodb dump 
dumps the whole db in bson format - 
```bash
mongodump --uri "mongodb://localhost:27017/round" 

mongodump --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies"

mongorestore --uri "mongodb+srv://<your username>:<your password>@<your cluster>.mongodb.net/sample_supplies"  --drop dump


```

### MongoDB connect via shell 

```bash 

mongo "mongodb+srv://m001-student:m001-mongodb-basics@sandbox.qgiqc.mongodb.net/admin"

mongo "mongodb+srv://sandbox.qgiqc.mongodb.net/admin" --username m001-student
```

both will work. 

## Basic commands 

- `show dbs`
- `show collections`
- `use databaseName`
- `db.zips.find({"state":"NY"})` - this command returns a cursor(iterator). If result set is large, only a small subset is shown initially. use 'it' to show more results, or iterate through the result set. 
- `db.zips.find({"state":"NY"}).count()` - number of results
- `db.zips.find({"state":"NY"}).pretty()` - displays formatted result 
- `db.inspections.findOne()`
- `db.inspections.insert([ { "test": 1 }, { "test": 2 }, { "test": 3 } ])`
- `db.inspections.insert({ "test": 3 })`
- `db.inspections.insert([{ "_id": 1, test": 1 }, { "_id": 1, "test": 2 }, { "_id": 3, "test": 3 }])` - as the ids of the first 2 documents are same, only the first one will get inserted. Also, insertion is by default ordered, and if an insertion error is encountered, the process stops and rest of the documents are not inserted.
- `db.inspections.insert([{ "_id": 1, "test": 1 },{ "_id": 1, "test": 2 }, { "_id": 3, "test": 3 }],{ "ordered": false })` - to avoid above issue, use ordered:false, which will also insert document with _id:3. 
- `db.zips.updateOne(query, update)` - update one document, which is the first result of query
- `db.zips.updateMany(query, update)` - update all the documents which result from the query
- `db.zips.updateMany({ "city": "HUDSON" }, { "$inc": { "pop": 10 } })` - use $inc to increase a particular field in many documents by a certain amount. We can also increase multiple fields by different amount in a single query. 
- `db.zips.updateOne({ "zip": "12534" }, { "$set": { "pop": 17630 } })` - use $set to directly update the value 
- `db.zips.updateOne({ "zip": "12534" }, { "$set": { "population": 17630 } })` - if the field does not exist already, a new field is added. 
- `db.grades.updateOne({ "student_id": 250, "class_id": 339 },{ "$push": { "scores": { "type": "extra credit", "score": 100 }} })` - use $push when entering an object into an array
- `db.collection.drop()` - delete collection 
- `db.inspections.deleteOne({ "test": 3 })`
- `db.inspections.deleteMany({ "test": 1 })`
- `db.iot.updateOne({ "sensor": r.sensor, "date": r.date,"valcount": { "$lt": 48 } }, { "$push": { "readings": { "v": r.value, "t": r.time } },        "$inc": { "valcount": 1, "total": r.value } }, { "upsert": true })` - finds a document with given sensor and date value, and with valcount < 48. If found, push the reading object in the readings array, increment valcount, add r.value to total field. If document for this sensor or for this date not found, or if the document present already have 48 readings, it creates a new document with the mentioned values. 

## Advanced CRUD operations

### Comparison Operators

- $eq, $neq, $lt, $lte, $gt, $gte - `{ field : { operator : value } }`
- Implicit operator is $eq
- `db.trips.find({ "tripduration": { "$lte" : 70 }}).pretty()`
- `db.trips.find({ "tripduration": { "$lte" : 70 }, "usertype": { "$ne": "Subscriber" } }).pretty()`
- `db.trips.find({ "tripduration": { "$lte" : 70 }, "usertype": "Customer" }).pretty()`

### Logical Operators 

- $nor, $or, $and - `{ operator : [{statement1}, {statement2},...] } `
- $not - `{ operator : {statement} }` 
- Implicit operator is $and
- `sample_training.inspections.find({$nor : [{"result":"Pass"}, {"result":"Fail"}]})` - find all documents with result neither pass nor fail
- all these queries are same for finding documents with student ids >25 and <100 in sample_training.grades - 
    - `{"$and" : [{"student_id" : { "$gt":25 }}, {"student_id" : { "$lt":100 }} ]}`
    - `{"student_id" : { "$gt":25 }}, {"student_id" : { "$lt":100 }}`
    - `{"student_id" : { "$gt":25, "$lt":100 }}` - as the field as same, can combine
- Can use $and when we need to include same operator more than once in a query - Using the routes collection find out how many CR2 and A81 planes come through the KZN airport - 
    - db.routes.find({ "$and": [ { "$or" :[ { "dst_airport": "KZN" },
                                    { "src_airport": "KZN" }
                                  ] },
                          { "$or" :[ { "airplane": "CR2" },
                                     { "airplane": "A81" } ] }
                         ]}).pretty()
    - If we dont include $and explicitly, it will also return documents which satisfies only one of the $or queries.  
-  $not operator only affects other operators and cannot check fields and documents independently. So, use the $not operator for logical disjunctions and the $ne operator to test the contents of fields directly.
- `db.inventory.find( { price: { $not: { $gt: 1.99 } } } )` returns documents where - 
    - the price field value is less than or equal to 1.99 or
    - the price field does not exist

### Expressive Query Operator

- $expr - used to write more complicated queries, like when we are not comparing the fields to a particular value, but comparing multiple fields in the same document with each other 
- $field_name - adding $ in front of field name gives out the value of this field
- Find number of trips where end station and start station are same - 
    `db.trips.find({ "$expr": { "$eq": [ "$end station id", "$start station id"] } }).count()`
- Find all documents where the trip lasted longer than 1200 seconds, and started and ended at the same station - 
    `db.trips.find({ "$expr": { "$and": [ { "$gt": [ "$tripduration", 1200 ]}, { "$eq": [ "$end station id", "$start station id" ]} ]}}).count()`
- Here `$gt` is used for aggregation.
- syntax for comparison operators using aggregation - `{ operator : { field : value } }`
- Using the $expr operator is the reason why we have to use the aggregation syntax for the comparison operator. 

### Array Operators 

- lets say we have documents with an ameneties array. 
- `{ "ameneties":"Shampoo" }` - returns documents with "ameneties" field having "Shampoo" as one of the elements in the array. 
- `{ "ameneties":[ "Shampoo" ] }` - returns documents with "ameneties" having only one single element as "Shampoo", ie it performs exact match. When specifying array for matching, order of elements matter as well. 
- `{ array_field : { "$all" : [value1, value2, value3,...] } }` - returns documents with array_field having all the values specified under $all operator. Here order doesn't matter. 
- `{ array_field : { "$size" : 20 } }` - returns documents with array_field having size of exactly 20
- `{ array_field : { "$elemMatch" : { field : value } } }` - returns those documents that contain an array field with at least one element that matches the specified query creteria
- `db.grades.find({ "scores": { "$elemMatch": { "type": "extra credit" } } }).pretty()` - returns documents where student had an extra credit score
- `db.grades.find({ "class_id": 431 }, { "scores": { "$elemMatch": { "score": { "$gt": 85 }}} }).pretty()` - Find all documents where the student in class 431 received a grade higher than 85 for any type of assignment. This way $elemMatch can also be used in projection. 
- `db.companies.find({ "relationships.0.person.last_name": "Zuckerberg" }, { "name": 1 }).pretty()` - find companies where first element in relationships array have last name as Zuckerberg. Can use . (dot) operator to query subdocuments.
- `db.companies.find({ "relationships": { "$elemMatch": { "is_past": true, "person.first_name": "Mark" } } }, { "name": 1 }).pretty()`
    - returns companies where atleast 1 element in relationships array have left the company and have first name as Mark.

### Projection
- second argument in find command. 
- syntax - `{"field1":1, "field2":1}` - only includes field1 and field2 in the result.
- use 1 for inclusion, 0 for exclusion
- don't mix 0s and 1s, except when excluding _id - `{"field1":1, "field2":1, "_id":0}`


## Aggregation Framework

- Aggregation framework exceeds the capabilities of MQL. 
- We can process data in the form of a pipeline, where data passes from 1 stage to other. 
- The order of operators matters in aggregation. 
- Find all documents that have Wifi as one of the amenities. Only include price and address in the result - `db.listingsAndReviews.aggregate([
                                  { "$match": { "amenities": "Wifi" } },
                                  { "$project": { "price": 1,
                                                  "address": 1,
                                                  "_id": 0 }}]).pretty()`
- Project only the address field value for each document, then group all documents into one document per address.country value - `db.listingsAndReviews.aggregate([ { "$project": { "address": 1, "_id": 0 }},
                                  { "$group": { "_id": "$address.country" }}])`
- Project only the address field value for each document, then group all documents into one document per address.country value, and count one for each document in each group - 
`db.listingsAndReviews.aggregate([
                                  { "$project": { "address": 1, "_id": 0 }},
                                  { "$group": { "_id": "$address.country",
                                                "count": { "$sum": 1 } } }
                                ])`


## Limit and Sort

- limit() and sort() are cursor methods - are applied on the result of find()
- `db.zips.find().sort({ "pop": 1 }).limit(1)` - sort in ascending order, only return 1 result
- `db.zips.find().sort({ "pop": -1 }).limit(1)` - sort in descending order with -1
- `db.zips.find().sort({ "pop": 1, "city": -1 })` - can apply multiple sorting criterias
- `limit().sort()` means `sort().limit()`, mongodb does this internally

## Indexes 

- Indexes help to optimize the queries. 
- creating an index sorts the data using that field so that any further queries can be performed faster
- `db.trips.createIndex({ "birth year": 1 })` - single index 
- `db.trips.createIndex({ "start station id": 476, "birth year": 1 })` - compound index (multiple fields)


## Important Points 

- _id is a compulsory unique field in every mongodb document. It is of type ObjectId(). If we have another data or key which is unique, we can use that for _id field as well. 
