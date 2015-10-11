Title: Introduction To MongoDB
Date: 2015-01-15
Category: Mongo
Tags: mongo, deep-dive
Slug: introduction-to-mongodb
Authors: Tushar Tyagi
Summary: Introduction to mongodb, mongo shell, and mongo database driver

## Intro to mongodb

MongoDB is a document based, schemaless database, and unlike relational
database, there are no tables, values are stored as JSON documents.

At the highest level, we have database server, which stores a number of
database. Each database is made up of collections, which are equivalent to
tables in relational database. And inside every collection, we have a number of
documents, which are equivalent to tuples/rows in a relational database.

The database is schemaless, which means that there are no fixed structure of
documents inside the collection, and each can have different structure.

For instance, the following is a valid document in mongodb:

	:::javascript
	{ name : "Oscar Wilde", age : 46 }
	

### Inserting Values

* In order to start working, start the mongodb shell using the command `mongo`
* A database can be created using the `use` command. If it's present it is
  selected, else it is created. 
		
		mongo> use learn-mongo 
    
	Previous statement will create a new database called learn-mongo, given that
	it's not present.

	Once a database is selected, the variable `db` refers to it.
	
		mongo> db
		learn-mongo

* A collection can be created using `db.createCollection('name')`.

		mongo> db.createCollection('test')
		{ "ok" : 1 }

		mongo> show collections
		test
	
	This is the	explicit way of creating a collection. A collection is also
	created when a document is saved under some collection using
	`db.collectionName.save()` command.

	The following command  will create a new collection called persons and
	insert a new record in the collection. 

		mongo> db.persons.save({ name: "Oscar", lastName: "Wilde" })
		WriteResult({ "nInserted" : 1 })

* `db.collectionName.find()` method can be used to list all the documents in
  the collection.

		mongo> db.persons.find()
		{ "_id" : ObjectId("54aecbd3d2576a194767a357"), "name" : "Oscar", lastName: "Wilde" }

	`_id` is internal value, akin to primary key.

* `save()` method is a wrapper around insert and update so it provides the 
	functionality of both of these, i.e. it inserts the document if not already
	present of updates it. 


*  In order to find a specific document, pass the filter criteria:
  
		mongo> db.persons.find({ name : "Oscar" })
		{ "_id" : ObjectId("54ae48a94038c8255bdbb862"), "name" : "Oscar", "lastName" : "Wilde" }

*  The syntax of find command is: `db.collectionName.find(criteria, projection)`
   where criteria is the filter criteria and projection is the list of values
   which you want to get. The primary key value, `_id` is always get by default
   unless supressed.

*  And to display only selected values from the document, pass another object, 
  with the fields which you want to show having non-zero values
  
    	mongo> db.persons.find({ name : "Oscar" }, { name : 1 })
		{ "_id" : ObjectId("54ae48a94038c8255bdbb862"), "name" : "Oscar" }
    
    	mongo> db.persons.find({ name : "Oscar" }, { name : 1, _id : 0 })
		{ "name" : "Oscar" }
    
  
*  Insertion can be one object at a time, or we can pass an array of objects.
  
    	mongo> var algy = { firstName : "Algy", lastName: "Moncrieff", hobbies: ["Eating", "Bunburying", "Flirting"] }
    	mongo> var jack = { firstName : "Jack", lastName : "Worthing" }
    	mongo> var bracknells = [{ firstName : "Augusta", lastName : "Bracknell", hobbies : ["Gossiping", "Husband hunting"] },
                        { firstName : "Gwendolen", lastName : "Fairfax", hobbies: ["Loving Earnest"]}];
    
    	mongo> db.persons.save(algy);
		WriteResult({ "nInserted" : 1 })
    
    	mongo> db.persons.save(jack);
		WriteResult({ "nInserted" : 1 })
    
    	mongo> db.persons.save(bracknells);
		BulkWriteResult({
			"writeErrors" : [ ],
			"writeConcernErrors" : [ ],
			"nInserted" : 2,
			"nUpserted" : 0,
			"nMatched" : 0,
			"nModified" : 0,
			"nRemoved" : 0,
			"upserted" : [ ]
		})
    
    	mongo> db.persons.find();
		// ... values
    
    
    
### Updating Values

We already know how to insert values using `save`, and find values using `find`
command. Now we have to see how we can update the values we've already inserted
using `update` method, which has the following syntax:

    db.collection.update(query, update, options)
    query: filter query
    update: the modifications to apply
    options: { upsert : default false,  // Creates a document if none found
               multi  : default false,  // Updates multiple docs if true, else updates only one document
               writeConcern : document
             }


    

  
