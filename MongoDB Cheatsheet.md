# <span style="color:red"> MongoDB CLI usage </span>

### <span style="color:blue"> Start and Stop MongoDB service
```
sudo service mongod start
sudo service mongod stop
```

### <span style="color:green"> MongoDB Tools/Utilities list
```
1) mongodump     -> utility for backing up mongodb database. dumps data and metadata in binary format
2) mongorestore  -> utility to restore from mongodump output
3) mongoexport   -> utility to export data to  json, csv,tsv format
4) mongoimport   -> utility to import data in json,csv,tsv format.
5) mongotop      -> overview of the time a mongod instance spends reading and writing data.
6) mongostat     -> quick overview of the status of a currently running mongod or mongos instance.
7) bsondump      -> utility to convert  bson dump files to  json. disgnostic tool only not for conversion or ingestion.
8) mongodb-compass-> GUI utility for managing/administering mongodb.
9) mongodb       ->  cli utility for  managing/administering mongodb
10) mongofiles   -> Supports manipulating files stored in your MongoDB instance in gridfs objects.
11) mongosh      -> mongodb shell cli utility.
```

### <span style="color:blue"> Connect to a MongoDB Standalone Cluster (using MongoDB as a service cluster here)
```
@connection via mongodb compass
secure uri = mongodb+srv://<username>:<password>@ymmcluster.ov2vm.mongodb.net/<database>
unsecure uri = mongodb://<username>:<password>@ymmcluster.ov2vm.mongodb.net/<database>

@ connection via mongo cli
$mongo mongodb+srv://ymmcluster.ov2vm.mongodb.net -u <username> -p <password>
```

### Connect to a MongoDB replicaset cluster
```

```

### Connect to a MongoDB Sharded cluster
```

```

### Getting help 
```
@ after connection to the mongodb cluster
[primary]> db.help;

```

### Show All Databases

```
[primary]> show dbs
sample_analytics  9.57 MB
sample_mflix      32.1 MB
sweetscomplete     270 kB
admin              344 kB
local             1.46 GB

# sample_analytics, sample_mflix and sweetscomplete are user databases.
# admin and local are default mongodb internal databases
```

### Check/Specify a Database

```
[primary]> db
```

### Create Or Switch Database

```
# if the database doesnt exist, its created but its only displayed with 'show dbs' only after an insert of  document and creation of a collection :)
[primary]> use sample_mflix
'switched to db sample_mflix'
```

### Dumping & Re-importing a database (bson)
```
@ Dumping -> if only databse is specified using -d flag all cllections are dumped to the dump/database folder.
@ only a specific collection can be dumped using -c flag.
example:- mongodump mongodb+srv://ymmcluster.ov2vm.mongodb.net -d sweetscomplete -c customers -u <username> -p <password>

$ cd /d/working/nosql-mongodb/data/dump
$ mongodump mongodb+srv://ymmcluster.ov2vm.mongodb.net -d sweetscomplete -u <username> -p <password>
2021-04-17T11:10:13.812+0530    writing sweetscomplete.purchases to dump\sweetscomplete\purchases.bson
2021-04-17T11:10:14.590+0530    [##......................]  sweetscomplete.purchases  101/830  (12.2%)
2021-04-17T11:10:14.753+0530    writing sweetscomplete.products to dump\sweetscomplete\products.bson
2021-04-17T11:10:14.774+0530    writing sweetscomplete.customers to dump\sweetscomplete\customers.bson
2021-04-17T11:10:15.262+0530    [########################]  sweetscomplete.purchases  830/830  (100.0%)
2021-04-17T11:10:15.263+0530    done dumping sweetscomplete.purchases (830 documents)
2021-04-17T11:10:15.450+0530    done dumping sweetscomplete.products (64 documents)
2021-04-17T11:10:15.493+0530    done dumping sweetscomplete.customers (76 documents)

$ ls -lRt dump
dump/sweetscomplete:
total 279
-rw-r--r-- 1 jrpam 197611  24146 Apr 17 11:10 customers.bson
-rw-r--r-- 1 jrpam 197611  25680 Apr 17 11:10 products.bson
-rw-r--r-- 1 jrpam 197611 226673 Apr 17 11:10 purchases.bson
-rw-r--r-- 1 jrpam 197611    156 Apr 17 11:10 customers.metadata.json
-rw-r--r-- 1 jrpam 197611    156 Apr 17 11:10 purchases.metadata.json
-rw-r--r-- 1 jrpam 197611    155 Apr 17 11:10 products.metadata.json

@ Reimporting -> while reimporting, --drop flag has tobe spcified to prevent errors due to _id clash.
@  No need to specify the db or collection id  specifying the dump folder.
@ other flags = --dryRun -v (verbose)
$ mongorestore mongodb+srv://ymmcluster.ov2vm.mongodb.net  -u <username> -p <password> --drop
2021-04-17T11:19:30.614+0530    using default 'dump' directory
2021-04-17T11:19:30.616+0530    preparing collections to restore from
2021-04-17T11:19:31.099+0530    reading metadata for sweetscomplete.purchases from dump\sweetscomplete\purchases.metadata.json
2021-04-17T11:19:31.381+0530    restoring sweetscomplete.purchases from dump\sweetscomplete\purchases.bson
2021-04-17T11:19:32.268+0530    reading metadata for sweetscomplete.products from dump\sweetscomplete\products.metadata.json
2021-04-17T11:19:32.398+0530    reading metadata for sweetscomplete.customers from dump\sweetscomplete\customers.metadata.json
2021-04-17T11:19:32.488+0530    no indexes to restore
2021-04-17T11:19:32.488+0530    finished restoring sweetscomplete.purchases (830 documents, 0 failures)
2021-04-17T11:19:32.512+0530    restoring sweetscomplete.products from dump\sweetscomplete\products.bson
2021-04-17T11:19:32.655+0530    restoring sweetscomplete.customers from dump\sweetscomplete\customers.bson
2021-04-17T11:19:32.988+0530    no indexes to restore
2021-04-17T11:19:32.988+0530    finished restoring sweetscomplete.products (64 documents, 0 failures)
2021-04-17T11:19:33.134+0530    [########################]  sweetscomplete.customers  23.6KB/23.6KB  (100.0%)
2021-04-17T11:19:33.177+0530    [########################]  sweetscomplete.customers  23.6KB/23.6KB  (100.0%)
2021-04-17T11:19:33.177+0530    no indexes to restore
2021-04-17T11:19:33.177+0530    finished restoring sweetscomplete.customers (76 documents, 0 failures)
2021-04-17T11:19:33.177+0530    970 document(s) restored successfully. 0 document(s) failed to restore.

@ individual collection restore using
$ mongorestore mongodb+srv://ymmcluster.ov2vm.mongodb.net  -u <username> -p <password> --nsInclude=sweetscomplete.customers
$ mongorestore mongodb+srv://ymmcluster.ov2vm.mongodb.net  -u <username> -p <password> --nsInclude=sweetscomplete.purchases
$ mongorestore mongodb+srv://ymmcluster.ov2vm.mongodb.net  -u <username> -p <password> --nsInclude=sweetscomplete.products
```

### Converting bson dump files to json 
```
@ previously dumped sweetscomplete database
$ ls -l /d/working/nosql-mongodb/data/dump/dump/sweetscomplete
total 279
-rw-r--r-- 1 jrpam 197611  24146 Apr 17 11:10 customers.bson
-rw-r--r-- 1 jrpam 197611    156 Apr 17 11:10 customers.metadata.json
-rw-r--r-- 1 jrpam 197611  25680 Apr 17 11:10 products.bson
-rw-r--r-- 1 jrpam 197611    155 Apr 17 11:10 products.metadata.json
-rw-r--r-- 1 jrpam 197611 226673 Apr 17 11:10 purchases.bson
-rw-r--r-- 1 jrpam 197611    156 Apr 17 11:10 purchases.metadata.json

@ location where the converted files  willbe copied to = /d/working/nosql-mongodb/data/dump/dumptojson
$ cd /d/working/nosql-mongodb/data/dump/dumptojson
$ bsondump --bsonFile /d/working/nosql-mongodb/data/dump/dump/sweetscomplete/sweetscomplete/purchases.bson --outFile purchases.json --pretty
$ bsondump --bsonFile /d/working/nosql-mongodb/data/dump/dump/sweetscomplete/sweetscomplete/customers.bson --outFile purchases.json --pretty
$ bsondump --bsonFile /d/working/nosql-mongodb/data/dump/dump/sweetscomplete/sweetscomplete/products.bson --outFile purchases.json --pretty
```

### Exporting & Importing files to database (json/csv format)
```
@ Exporting existing database/collection as json files. Each collection must be exported seperately.
$ cd /d/working/nosql-mongodb/data/json/sample_analytics
$ mongoexport mongodb+srv://ymmcluster.ov2vm.mongodb.net -u <username> -p <password> -d sample_analytics -c transactions -o transactions.json
2021-04-17T12:33:25.283+0530    connected to: mongodb+srv://ymmcluster.ov2vm.mongodb.net
2021-04-17T12:33:26.548+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:27.549+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:28.547+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:29.549+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:30.547+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:31.548+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:32.548+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:33.547+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:34.548+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:35.549+0530    [........................]  sample_analytics.transactions  0/1746  (0.0%)
2021-04-17T12:33:35.633+0530    [########################]  sample_analytics.transactions  1746/1746  (100.0%)

@ Exporting  as csv file
$ mongoexport mongodb+srv://ymmcluster.ov2vm.mongodb.net -u <username> -p <password> -d sample_analytics -c customers --type=csv --fields="_id,username,name,address,email,birthdate,accounts,tier_and_details"


@ Importing a json file into database.
$ mongoimport mongodb+srv://ymmcluster.ov2vm.mongodb.net -u <username>  -p <password> -d sample_analytics -c customers --type=json customers.json --drop
2021-04-17T12:48:06.589+0530    connected to: mongodb+srv://ymmcluster.ov2vm.mongodb.net
2021-04-17T12:48:06.829+0530    dropping: sample_analytics.customers
2021-04-17T12:48:08.394+0530    500 document(s) imported successfully. 0 document(s) failed to import.

```

### Droping a mongodb database
```
[primary]> use sweetscomplete;
[primary]> db.dropDatabase()
```

### Create a new collection
```
@ under sample_analytics database, a new  collection "compliants" is created
[primary]> use sample_analytics;
[primary]> db.createCollection('compliants')

```

### Show Collections
```
[primary]> use sample_analytics;
[primary]> show collections

```

### MongoDB cluster stats info
```
@ provides mongod and mongos daemons stats details, similar to the UNIX/Linux file system utility vmstat
$mongostat mongodb+srv://ymmcluster.ov2vm.mongodb.net -u <username> -p <password>

                                          host insert query update delete getmore command dirty used flushes vsize res qrw arw net_in net_out conn                  set repl                time
ymmcluster-shard-00-00.ov2vm.mongodb.net:27017     *0    *0     *0     *0       0     1|0                  0    0B  0B 0|0 0|0   272b   2.28k    7 atlas-2pamp4-shard-0  SEC Apr 17 12:50:25.597
ymmcluster-shard-00-01.ov2vm.mongodb.net:27017     *0    *0     *0     *0       0     2|0                  0    0B  0B 0|0 0|0   420b   3.28k   31 atlas-2pamp4-shard-0  PRI Apr 17 12:50:26.051
ymmcluster-shard-00-02.ov2vm.mongodb.net:27017     *0    *0     *0     *0       0     1|0                  0    0B  0B 0|0 0|0   263b   3.24k    7 atlas-2pamp4-shard-0  SEC Apr 17 12:50:25.366

ymmcluster-shard-00-00.ov2vm.mongodb.net:27017     *0    *0     *0     *0       0     1|0                  0    0B  0B 0|0 0|0   263b   2.20k    7 atlas-2pamp4-shard-0  SEC Apr 17 12:50:26.596
ymmcluster-shard-00-01.ov2vm.mongodb.net:27017     *0    *0     *0     *0       0     0|0                  0    0B  0B 0|0 0|0   262b   2.22k   31 atlas-2pamp4-shard-0  PRI Apr 17 12:50:27.051
ymmcluster-shard-00-02.ov2vm.mongodb.net:27017     *0    *0     *0     *0       0     0|0                  0    0B  0B 0|0 0|0   262b   2.20k    7 atlas-2pamp4-shard-0  SEC Apr 17 12:50:26.367
```

### Insert a Row

```
db.posts.insert({
  title: 'Post One',
  body: 'Body of post one',
  category: 'News',
  tags: ['news', 'events'],
  user: {
    name: 'John Doe',
    status: 'author'
  },
  date: Date()
})
```

### Insert Multiple Documents

```
db.posts.insertMany([
  {
    title: 'Post Two',
    body: 'Body of post two',
    category: 'Technology',
    date: Date()
  },
  {
    title: 'Post Three',
    body: 'Body of post three',
    category: 'News',
    date: Date()
  },
  {
    title: 'Post Four',
    body: 'Body of post three',
    category: 'Entertainment',
    date: Date()
  }
])
```
## MongoDB Query Language


### Get All Documents
```
@ actually prints the first 20 docs. ie a cursor is returned. The next batch of 20 docs can be printed by "it" iterate command
[primary]> use sweetscomplete;
db.products.find()
```

### Get any arbitraty single document/row and pretty print.
```
[primary]> db.products.findOne()

{
  _id: ObjectId("5b4c232accf2ea73a85ed2c6"),
  sku: 'C21000',
  title: 'Chocolate Eclair',
  description: 'Magna libris id has, solet impetus cu pri. Aliquando assueverit contentiones ius ei, accusata imperdiet vix at, eum labore mediocrem no. Eum viris exerci et. Vix ea quas perfecto assueverit, mei id simul prodesset. Et munere scaevola erroribus quo, sed posse mollis iracundia ea, aeque mucius in mei. In nostrum vulputate usu, an quo solum mucius.\\n',
  price: '2.10'
}

```

### Get All Documents and pretty print themin json format.

```
db.products.find().pretty()
```

### MongoDB commonly used filtering operators
```
@ Comparision operators
$eq	  =>  Matches values that are equal to the given value.
$gt	  =>  Matches if values are greater than the given value.
$lt	  =>  Matches if values are less than the given value.
$gte  => 	Matches if values are greater or equal to the given value.
$lte  => 	Matches if values are less or equal to the given value.
$in	  => Matches any of the values in an array.
$ne	  =>  Matches values that are not equal to the given value.
$nin	=>  Matches none of the values specified in an array.

@ Logical Operators
$and	=>  Joins two or more queries with a logical AND and returns the documents that match all the conditions.
$or	  =>  Join two or more queries with a logical OR and return the documents that match either query.
$nor	=>  The opposite of the OR operator. The logical NOR operator will join two or more queries and return documents that do not match conditions.
$not	=>  Returns the documents that do not match the given query expression.

@ Document field operators
$exists	=>  Matches documents that have the specified field.
$type	  =>  Matches documents according to the specified field type. These field types are specified BSON types and can be defined either by type number or alias.

@ Array operators
$all	        =>  Matches arrays that contain all the specified values in the query condition.
$size	        =>  Matches the documents if the array size is equal to the specified size in a query.
$elemMatch	  =>  Matches documents that match specified $elemMatch conditions within each array element.


@ Evaluation operators - a bit complex usecases where java script expression, regex, json schema can be used for validation.
$jsonSchema	=>  Validate the document according to the given JSON schema.
$mod	      =>  Matches documents where a given field’s value is equal to the remainder after being divided by a specified value.
$regex	    =>  Select documents that match the given regular expression.
$text	      =>  Perform a text search on the indicated field. The search can only be performed if the field is indexed with a text index.
$where	    =>  Matches documents that satisfy a JavaScript expression.
```

### Examples of Queries for documents bassed on search/filter operations
```
@ filtering - allows simple filters based on docuemnt field values ex - product price=5.00
[primary]> db.products.find( { "price" : "5.00"})
[
  {
    _id: ObjectId("5b4c232accf2ea73a85ed2e3"),
    sku: 'P50000',
    title: 'Pumpkin Ice Cream',
    description: 'Nec ei elitr accusam torquatos. Eu eos dicit pertinacia. Eu quando torquatos quo, quod tollit phaedrum te vis. An quo assum iudico, duo probo atomorum et. Est eros partem corrumpit ad, est ei zril choro.\\n',
    price: '5.00'
  }
]

@ filtering  - checking if a values equals something using $eq operator 
[primary]> db.products.find({ sku : { "$eq" : "C21000" } })
[{
    "_id": {
        "$oid": "5b4c232accf2ea73a85ed2c6"
    },
    "sku": "C21000",
    "title": "Chocolate Eclair",
    "description": "Magna libris id has, solet impetus cu pri. Aliquando assueverit contentiones ius ei, accusata imperdiet vix at, eum labore mediocrem no. Eum viris exerci et. Vix ea quas perfecto assueverit, mei id simul prodesset. Et munere scaevola erroribus quo, sed posse mollis iracundia ea, aeque mucius in mei. In nostrum vulputate usu, an quo solum mucius.\\n",
    "price": "2.10"
}]

@ filtering - allows regular expressions - example all products that contain word pecan and case insensitive
[primary]> db.products.find( { "title" : /pecan/i })
[
  {
    _id: ObjectId("5b4c232accf2ea73a85ed2ea"),
    sku: 'P57000',
    title: 'Pecan Praline Ice Cream',
    description: 'Nam ornatus accusata similique ut, quo ignota dolores cu. Offendit invidunt sit ut, has ne quot decore. Usu ad sumo illud luptatum, dicat nostrud urbanitas pro te. Mel epicuri aliquando ei, audiam scripserit vis id. Veri simul et sit. Affert salutatus id cum, idque labore equidem ei mei.\\n',
    price: '5.70'
  },
  {
    _id: ObjectId("5b4c232accf2ea73a85ed2dd"),
    sku: 'P44000',
    title: 'Pecan Pie',
    description: 'At viris doming assueverit vim, rebum graeci omnesque cum cu, ea doctus dissentias nec. Ex eros animal sententiae est, meis volumus mnesarchum mel te. Sed et utroque perfecto, eum in nonumy convenire neglegentur. Inermis omittam ad vix. Eu quo eruditi intellegat efficiendi. Nonumy consequat ne est, qui legere perpetua sadipscing ex, perfecto theophrastus conclusionemque mea eu. Volumus detraxit in mea.\\n',
    price: '4.40'
  }
]

@ filtering - comparision operators
1) $gt , $gte (greater than/greater than equals)
$ db.products.find( { "price" : { "$gt" : "6.00" }} )  # all products with price > 6.00
$ db.products.find( { "price" : { "$gte" : "6.00" }} ) # all products with price >= 6.00

2) $lt , $lte (less than / less than equals)
$ db.products.find( { "price" : { "$lt" : "0.20" }} )  # all products with price < 0.20
$ db.products.find( { "price" : { "$lte" : "0.20" }} ) # all products with price <= 0.20>

@ filtering - if a value exists or  not in a list/array

@ filtering - by if a field exists in a document using $exists operator
example - if the field age exists and if its gte 30
$ db.customers.find( { "age" : { "$exists": true } } )
$ db.customers.find( { "age" : { "$exists" : true, "$gte" : 30 }})

@ filtering/searching docs using regex operator.
$ db.products.inventory({"name": {$regex: '.Packed.'}})
$ db.products.find({"title": {$regex: '.Soufl.'}})

```

### Text based search on document fileds using $text operator
```
@ if there is no Index created on the field on which text based search wil be created a "text type index" must be created.
$ db.inventory.createIndex({ "name": "text"})
$ db.inventory.find({ $text: { $search: "Non-Fat"}})
```


### Filtering/Searching based on array operators
```
@ find all products that contain "healthy and  organic" in the catgory array list
$ db.inventory.find({ "category": { $all: ["healthy", "organic"]}}).pretty()

@ find documents where the category array field has two elements.
$ db.inventory.find({ "category": { $size: 2}}).pretty()

@ Find documents where at least a single element in the “daily_sales” array is less than 200 and greater than 100.
$ db.promo.find({ "daily_sales": { $elemMatch: {$gt: 100, $lt: 200}}}).pretty()
```
### Sort Documents

```
# ascending
db.products.find().sort({ title: 1 }).pretty()
# descending
db.products.find().sort({ title: -1 }).pretty()
```

### Count Documents

```
db.posts.find().count()
db.posts.find({ category: 'news' }).count()
```

### Limit Documents

```
db.posts.find().limit(10)
```

### Chaining - filter + limiting + sorting result + pretty printing as json 

```
@ find the top 10 products that are more than 5.00
db.products.find( {"price" : { "$gt" : "5.00" }} ).limit(10).sort( {price:-1} ).pretty()
```

### Foreach

```
db.posts.find().forEach(function(doc) {
  print("Blog Post: " + doc.title)
})
```

### Find Specific Fields

```
db.posts.find({ title: 'Post One' }, {
  title: 1,
  author: 1
})
```

### Update Row

```
db.posts.update({ title: 'Post Two' },
{
  title: 'Post Two',
  body: 'New body for post 2',
  date: Date()
},
{
  upsert: true
})
```

### Update Specific Field

```
db.posts.update({ title: 'Post Two' },
{
  $set: {
    body: 'Body for post 2',
    category: 'Technology'
  }
})
```

### Increment Field (\$inc)

```
db.posts.update({ title: 'Post Two' },
{
  $inc: {
    likes: 5
  }
})
```

### Rename Field

```
db.posts.update({ title: 'Post Two' },
{
  $rename: {
    likes: 'views'
  }
})
```

### Delete Row

```
db.posts.remove({ title: 'Post Four' })
```

### Sub-Documents

```
db.posts.update({ title: 'Post One' },
{
  $set: {
    comments: [
      {
        body: 'Comment One',
        user: 'Mary Williams',
        date: Date()
      },
      {
        body: 'Comment Two',
        user: 'Harry White',
        date: Date()
      }
    ]
  }
})
```

### Find By Element in Array (\$elemMatch)

```
db.posts.find({
  comments: {
     $elemMatch: {
       user: 'Mary Williams'
       }
    }
  }
)
```

### Add Index

```
db.posts.createIndex({ title: 'text' })
```

### Text Search

```
db.posts.find({
  $text: {
    $search: "\"Post O\""
    }
})
```

### Greater & Less Than

```
db.posts.find({ views: { $gt: 2 } })
db.posts.find({ views: { $gte: 7 } })
db.posts.find({ views: { $lt: 7 } })
db.posts.find({ views: { $lte: 7 } })
```
db.collection.find(<query>)	Find the documents matching the <query> criteria in the collection. If the <query> criteria is not specified or is empty (i.e {} ), the read operation selects all documents in the collection.The following example selects the documents in the users collection with the name field equal to "Joe":copycopiedcoll = db.users; coll.find( { name: “Joe” } ); For more information on specifying the <query> criteria, see Specify Equality Condition.
db.collection.find(<query>, <projection>)	Find documents matching the <query> criteria and return just specific fields in the <projection>.The following example selects all documents from the collection but returns only the name field and the _id field. The _id is always returned unless explicitly specified to not return.copycopiedcoll = db.users; coll.find( { }, { name: true } ); For more information on specifying the <projection>, see Project Fields to Return from Query.
db.collection.find().sort(<sort order>)	Return results in the specified <sort order>.The following example selects all documents from the collection and returns the results sorted by the name field in ascending order (1). Use -1 for descending order:copycopiedcoll = db.users; coll.find().sort( { name: 1 } );
db.collection.find(<query>).sort(<sort order>)	Return the documents matching the <query> criteria in the specified <sort order>.
db.collection.find( ... ).limit( <n> )	Limit result to <n> Documents. Highly recommended if you need only a certain number of Documents for best performance.
db.collection.find( ... ).skip( <n> )	Skip <n> results.
db.collection.count()	Returns total number of documents in the collection.
db.collection.find(<query>).count()	Returns the total number of documents that match the query.The count() ignores limit() and skip(). For example, if 100 records match but the limit is 10, count() will return 100. This will be faster than iterating yourself, but still take time.
db.collection.findOne(<query>)	Find and return a single document. Returns null if not found.The following example selects a single document in the users collection with the name field matches to "Joe":copycopiedcoll = db.users; coll.findOne( { name: “Joe” } ); Internally, the findOne() method is the find() method with a limit(1).