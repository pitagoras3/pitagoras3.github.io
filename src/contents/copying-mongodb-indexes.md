---
author: Szymon Marcinkiewicz
datetime: 2023-02-26T08:30:00Z
title: "Copying MongoDB indexes programmatically"
slug: copying-mongodb-indexes
featured: true
draft: false
tags:
  - mongodb
  - kotlin
ogImage: ""
description: How to copy MongoDB indexes from one collection to another using MongoDB Java Driver?
---

## Rationale

When I was working on a project at work I needed to _copy_ MongoDB indexes from one collection, and apply them on another collection (running on different MongoDB instance). Unfortunately, MongoDB Java driver doesn't have `MongoCollection.copy(...)` method, so I needed to find a way to make it work. I have stumbled upon a [question on Stack Overflow](https://stackoverflow.com/q/62895863/6026621) with exact same problem, sadly - without an answer at the time.

## Copy & Modify & Paste JSON magic

If I would have this problem in JavaScript or Python, then solving it would be pretty easy (with help from Google and Stack Overflow). Quick look into [similar StackOverflow question answer](https://stackoverflow.com/a/43163958/6026621) leaves us with a code snippet which copies indexes from `source` collection to `destination` collection with JavaScript:
```javascript
var indexes = db.source.getIndexes(); // 1️⃣

indexes.forEach(function(index){
  delete index.v; // 2️⃣
  delete index.ns; // 3️⃣
  var key = index.key;
  delete index.key
  var options = [];
  for (var option in index) {
    options.push(index[option]);
  }
  db.destination.createIndex(key, options); // 4️⃣
});
```

But _why_ it works?

Firstly, let's see how an index definition is represented in MongoDB. Below you can find example indexes definitions (result of running `db.source.getIndexes() - 1️⃣` command on my _source_ MongoDB 3.6 database):

```js
db> db.source.getIndexes()

[
  { v: 2, key: { _id: 1 }, name: '_id_', ns: 'db.source' },
  {
    v: 2,
    key: { _fts: 'text', _ftsx: 1 },
    name: '$**_text',
    ns: 'db.source',
    weights: { '$**': 1 },
    default_language: 'english',
    language_override: 'language',
    textIndexVersion: 3
  }
]
```

Secondly, let's answer the question - how to create an index in MongoDB? There is a [createIndex(keys, options)](https://www.mongodb.com/docs/manual/reference/method/db.collection.createIndex/#db.collection.createindex--) method, which takes **keys** and **options** arguments, where:

> keys - A document that contains the field and value pairs where the field is the index key and the value describes the type of index for that field.

> options - A document that contains a set of options that controls the creation of the index.

One last thing which we need to know is what are `v - 2️⃣` and `ns - 3️⃣` fields in each MongoDB index. 
- `v` represents index version,
- `ns` represents namespace of a collection on which index is created.

Those fields are not valid [index options](https://www.mongodb.com/docs/manual/reference/method/db.collection.createIndex/#options-for-all-index-types) for index creation, so they should be omitted in process of copying (with one addition - since MongoDB 4.4 `ns` field is not returned from `db.collection.getIndexes()`, it doesn't need to be omitted when copying indexes from newer MongoDB versions).

After preparing all required parameters, you can create index on destination collection by calling `db.destination.createIndex(key, options) - 4️⃣`.

## How to do it with MongoDB Java Driver?

I have managed to implement approach described above with Kotlin and optimise it, to not call `createIndex` for each individual index, rather to create all indexes at once with one call. The trick requires using [`runCommand`](https://mongodb.github.io/mongo-java-driver/4.9/apidocs/mongodb-driver-sync/com/mongodb/client/MongoDatabase.html#runCommand(org.bson.conversions.Bson)) method from MongoDB Java driver, along with [createIndexes](https://www.mongodb.com/docs/manual/reference/command/createIndexes/#mongodb-dbcommand-dbcmd.createIndexes) command. The final result is:

```kotlin
import com.mongodb.client.MongoClients
import com.mongodb.client.MongoDatabase
import org.bson.Document

fun main() {
  MongoClients.create("mongodb://localhost:27017").use { mongoClient ->
    val db = mongoClient.getDatabase("db")
    val indexes = getIndexes(db, "source")
    createIndexes(db, "destination", indexes)
  }
}

private fun getIndexes(
  sourceDb: MongoDatabase,
  collectionName: String
) = sourceDb.getCollection(collectionName).listIndexes()
      .toList()
      .filterNot { it["name"] == "_id_" } // There is no need to copy _id index
      .map {
        it.apply {
          remove("v")
          remove("ns") // Not needed when copying from MongoDB > 4.4
        }
      }

private fun createIndexes(
  destinationDb: MongoDatabase,
  collectionName: String,
  indexes: List<Document>
) {
  destinationDb.runCommand(
    Document().append("createIndexes", collectionName)
      .append("indexes", indexes)
  )
}
```