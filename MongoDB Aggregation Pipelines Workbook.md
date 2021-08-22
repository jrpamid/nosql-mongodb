# <span style="color:red"> MongoDB Aggregation Pipeline Usage </span>
```
MongoDB has about  30 aggregation pipeline operators/stages.
Useful for advanced data transformations, aggregations, deriving new data, adding new fields to existing docs, performing lookups etc beyind find() operator.
The aggregation pipeline supports operations on sharded collections. When aggregation operations run on multiple shards, the results are routed to the mongos to be merged. However if the pipeline includes the $out or $lookup stages, the merge runs on the primary shard.
Pipeline limitaions:-
1) Pipeline stage can use only 100 MB of RAM can be used per aggregation stage. Setting "allowDiskUse" option allows memory to disk swap.
   $db.<collection>.aggregate(pipeline, { allowDiskUse : true })
2) Documents returned by the aggregation query, either as a cursor or stored via $out() in another collection, are limited to 16MB.

## SYNTAX ##
@  pipeline is a list/array of stages.
pipeline = [
  { $match : { … } },
  { $group : { … } },
  { $sort : { … } },
  ...
]
$db.<collection>.aggregate(pipeline, options)
$db.<collection>.aggregate(pipeline, { allowDiskUse : true })
```


## <span style="color:green"> SQL & Aggregation stages  comparision
```
SQL Concept       <==>      MongoDB Aggregation Operators
WHERE                       $match
GROUP BY                    $group
HAVING                      $match
SELECT                      $project
ORDER BY                    $sort
LIMIT                       $limit
SUM()                       $sum
COUNT()                     $sum, $sortByCount
join                        $lookup
SELECT INTO NEW_TABLE       $out
MERGE INTO TABLE            $merge (Available starting in MongoDB 4.2)
UNION ALL                   $unionWith (Available starting in MongoDB 4.4)
```

###  <span style="color: magenta"> Sample data
```
@ persons.json 
{
    "_id": {
        "$oid": "6084c39af08f292db84fb5c8"
    },
    "gender": "male",
    "name": {
        "title": "mr",
        "first": "victor",
        "last": "pedersen"
    },
    "location": {
        "street": "2156 stenbjergvej",
        "city": "billum",
        "state": "nordjylland",
        "postcode": 56649,
        "coordinates": {
            "latitude": "-29.8113",
            "longitude": "-31.0208"
        },
        "timezone": {
            "offset": "+5:30",
            "description": "Bombay, Calcutta, Madras, New Delhi"
        }
    },
    "email": "victor.pedersen@example.com",
    "login": {
        "uuid": "fbb3c298-2cea-4415-84d1-74233525c325",
        "username": "smallbutterfly536",
        "password": "down",
        "salt": "iW5QrgwW",
        "md5": "3cc8b8a4d69321a408cd46174e163594",
        "sha1": "681c0353b34fae08422686eea190e1c09472fc1f",
        "sha256": "eb5251e929c56dfd19fc597123ed6ec2d0130a2c3c1bf8fc9c2ff8f29830a3b7"
    },
    "dob": {
        "date": "1959-02-19T23:56:23Z",
        "age": 59
    },
    "registered": {
        "date": "2004-07-07T22:37:39Z",
        "age": 14
    },
    "phone": "23138213",
    "cell": "30393606",
    "id": {
        "name": "CPR",
        "value": "506102-2208"
    },
    "picture": {
        "large": "https://randomuser.me/api/portraits/men/23.jpg",
        "medium": "https://randomuser.me/api/portraits/med/men/23.jpg",
        "thumbnail": "https://randomuser.me/api/portraits/thumb/men/23.jpg"
    },
    "nat": "DK"
}

# example from https://studio3t.com/knowledge-base/articles/mongodb-aggregation-framework/
@ university.json
{
  country : 'Spain',
  city : 'Salamanca',
  name : 'USAL',
  location : {
    type : 'Point',
    coordinates : [ -5.6722512,17, 40.9607792 ]
  },
  students : [
    { year : 2014, number : 24774 },
    { year : 2015, number : 23166 },
    { year : 2016, number : 21913 },
    { year : 2017, number : 21715 }
  ]
}
@ courses.json
{
  university : 'USAL',
  name : 'Computer Science',
  level : 'Excellent'
}
```

### <span style="color:blue"> $match pipeline stage
```
@ Filters the docs that are sent to the next stage in pipeline.
@ Must be as early as possible in pipeline. Will take advantage of indexes when used as first stage and also reduces the number  of docs to process.
@ $where in $match queries as part of the aggregation pipeline.
@ raw expressions are not accepted. use $expr query expression.
{ $match: { $expr: { <aggregation expression> } } }

# example 1:= Simple expressions. ind all males, age > 45 (persons.json)
$db.persons.aggregate([ {$match : { "gender" : "male"}} ]);
$db.persons.aggregate([ {$match : { "dob.age" : {$gt : 45}}} ]);

# combining these two into one expression. $and ,$or $not  etc can be used to vreate logical  expressions with multiple filter conditions.
aggregate_expr = [
    { $match : {
        $and : [
            { "gender" : "male"},
            { "dob.age" : { $gt : 45}}
        ]
     }
    }
]
db.persons.aggregate(aggregate_expr)

@ example 2:- compound expression. find all males between 40 - 45 and all females between 30 - 35
aggregate_expr = [
    {
        $match : {
            $or : 
            [   # condition 1
                { $and : [  
                            { "gender" : "male"},
                            { "dob.age" : { $gt : 40, $lt : 45 } } 
                        ]
                },
                # or condition 2
                { $and : [
                            { "gender" : "female" },
                            { "dob.age" : { $gt : 30 , $lt : 35 } }
                        ]
                }
            ]
        }
    }
]
db.persons.aggregate(aggregate_expr)
```


### <span style="color:blue"> $cond (if then else logic condition)
```
db.inventory.aggregate(
   [
      {
         $project:
           {
             item: 1,
             discount:
               {
                 $cond: { if: { $gte: [ "$qty", 250 ] }, then: 30, else: 20 }
               }
           }
      }
   ]
)
```


### <span style="color:blue"> $group
```
@ groups docs based on specified _id expression. Outputs a new  doc with a new  _id for each grouping.
@ group does not order the docs.
@ aggregation operations can also  be performed on the fields and return new fields.
@ $group stage has a limit of 100 megabytes of RAM.
@ when group  stage _id is null, all documents are considered for the aggregation.
{
  $group:
    {
      _id: <expression>, // Group By Expression
      <field1>: { <accumulator1> : <expression1> },
      ...
    }
 
@ accumulator operations that can be done as part of group operation. [ avg, max, min, sum, push,  first , last, ]

# example:- 1 group by having as in sql.
SELECT item,
   Sum(( price * quantity )) AS totalSaleAmount
FROM   sales
GROUP  BY item
HAVING totalSaleAmount >= 100

$ db.sales.aggregate([
    {
      $group :
        {
          _id : "$item",
          totalSaleAmount: { $sum: { $multiply: [ "$price", "$quantity" ] } }
        }
     },

     {
       $match: { "totalSaleAmount": { $gte: 100 } }
     }
   ])


## Example:- 2 Grouping by day and calculating total sales per day and avg quantity etc and sorting final result.
SELECT date,
       Sum(( price * quantity )) AS totalSaleAmount,
       Avg(quantity)             AS averageQuantity,
       Count(*)                  AS Count
FROM   sales
GROUP  BY Date(date)
ORDER  BY totalSaleAmount DESC

$ db.sales.aggregate([
  // matching all docs that are between jan 2014 and jan 2015
  {
    $match : { "date": { $gte: new ISODate("2014-01-01"), $lt: new ISODate("2015-01-01") } }
  },
  // grouping the docs in collection by their  date and calculating total, avg etc
  {
    $group : {
       _id : { $dateToString: { format: "%Y-%m-%d", date: "$date" } },
       totalSaleAmount: { $sum: { $multiply: [ "$price", "$quantity" ] } },
       averageQuantity: { $avg: "$quantity" },
       count: { $sum: 1 }
    }
  },
  // sorting the results in desc order
  {
    $sort : { totalSaleAmount: -1 }
  }
 ])

## Example:- 3 Grouping on 2 fields
@ use sample_training.zips collection, for each state, aggregating and grouping of  population by city.
var pipeline = [
 { // grouping  on state and city
      $group : { 
                _id : { city : "$city", state: "$state"} ,
                population : { $sum : "$pop" } 
            }
 },
 // sort the result on population field in ascending order
 { 
     $sort : { population : 1}
 },
 // grouping by state, notice how $_id.state is referred from previous stage
 {
     $group : {
         _id : { state : "$_id.state" },
         state_population : { $sum : "$population" },
         avg_population : { $avg : "$population" },
         highest_population : { $last : "$population" },
         lowest_population : { $first : "$population" },
         smallest_city : { $first : "$_id.city" },
         largest_city : { $last : "$_id.city" }
     }
 }
]
db.zips.aggregate(pipeline)
```

### <span style="color:blue"> $project
```
@ $project stage works similar to project in MQL. Used to include or  exclude  fields exposed to the next stage.
@ new fileds can also be added. 
@ _id must be manually excluded, its automalically includedby default.
@ fields must be specified if they have to  be included and passed to the next stage. (see $addFields also)
@ 
{ $project : {field1 : 1, _id : 0, field2 : 1, new_field3 : {transform_expression}} } 

sample data:-  ( https://raw.githubusercontent.com/tfogo/mongodb-workshop/master/pokedex.json )
{
    "_id": {
        "$oid": "60937e648e7f9a33a0a51e74"
    },
    "id": 1,
    "num": "001",
    "name": "Bulbasaur",
    "img": "http://www.serebii.net/pokemongo/pokemon/001.png",
    "type": ["Grass", "Poison"],
    "height": "0.71 m",
    "weight": "6.9 kg",
    "candy": "Bulbasaur Candy",
    "candy_count": 25,
    "egg": "2 km",
    "spawn_chance": 0.69,
    "avg_spawns": 69,
    "spawn_time": "20:00",
    "multipliers": [1.58],
    "weaknesses": ["Fire", "Ice", "Flying", "Psychic"],
    "next_evolution": [{
        "num": "002",
        "name": "Ivysaur"
    }, {
        "num": "003",
        "name": "Venusaur"
    }]
}

# transform the  height and weight string fields to numerical and convert metric units meters-> cms, kg -> gms :)

```


### <span style="color:blue"> $addfields
```
@ $addFields outputs documents containing all the existing fields from the input documents and any newly added fields.
@ no need to explicitly specify which fields are to be included and excluded.
@ To add field or fields to embedded documents (including documents in arrays) use the dot notation.
@ Specifying an existing field name in an $addFields operation causes the original field to be replaced.
@  $addFields along with $concatArrays expression can be used to add an element to an existing array field.
syntax :- { $addFields: { <newField>: <expression>, ... } }

sample scores data:-
{
  _id: 1,
  student: "Sam Geyser",
  homework: [ 10, 5, 10 ],
  quiz: [ 10, 8 ],
  project: 9,
  extraCredit: 0
},
{
  _id: 2,
  student: "John Tambo",
  homework: [ 5, 6, 5 ],
  quiz: [ 8, 8 ],
  extraCredit: 8,
  project: 0
}

$ var pipeline = [
    { $addFields : {
        homework_total : { $sum : "$homework"},
        quiz_total : { $sum : "$quiz" },
      }
    },
    {
        $addFields : {
            total_score : { $add : [ "$extraCredit", "$project", "$homework_total", "$quiz_total" ] }
        }
    }
]
$ db.scores.aggregate(pipeline);

# adding adding a  bonus homework score = 9 and project score = 7  to  all students .
$ db.scores.aggregate([
    { $match : {} },
    { $addFields: { homework : { $concatArrays: ["$homework", [9]] }}},
    { $addFields: { project : 9 }}
])

```

### <span style="color:blue"> $set
```
@ alias for $addFields.
@ $set outputs documents that contain all existing fields from the input documents and newly added fields.
@ one or more $set stages can be included in a pipeline.
@ Use dot notation to add new fields to embedded documents.
@ Specifying an existing field name in a $set operation causes the original field to be replaced.

{ $set: { <newField>: <expression>, ... } }

# same example used in $addFields
$ db.scores.aggregate( [
   {
     $set: {
        totalHomework: { $sum: "$homework" },
        totalQuiz: { $sum: "$quiz" }
     }
   },
   {
     $set: {
        totalScore: { $add: [ "$totalHomework", "$totalQuiz", "$extraCredit" ] } }
   }
] )

# arrays concatenation
$ db.scores.aggregate([
   { $match: { _id: 1 } },
   { $set: { homework: { $concatArrays: [ "$homework", [ 7 ] ] } } }
])

```

### <span style="color:blue"> $unset
```
@ unset is an alias to $project operator. It removes fields from the document.
@  unset can also be used to remove embedded fields.

{ $unset : "field1" }
{ $unset: [ "<field1>", "<field2.nestedField", ... ] }

```

### <span style="color:blue"> $count
```
@ passes a document with the count of docs in this stage to the next stage.
{ $count: <string> }

# example:- performs the aggregations and assigns the count of docs to the field 'total_docs'
db.scores.aggregate([
    { $match : {} },
    { $addFields: { homework : { $concatArrays: ["$homework", [9]] }}},
    { $addFields: { project : 9 }},
    { $count : 'total_docs' }
])

```

### <span style="color:blue"> $limit 
```
@ limit the number of docs passed to next stage. lke top 100, lowest 10 etc
@ limit a=has not effect on the content of the docs.
{ $limit: <positive integer> }

@ usually used along with sort. When used with sort - sort has to be performed on a filed that contains unique values.
  sort on afiled that contains duplicate values gives in-consistent results.
```


### <span style="color:blue"> $sort
```
@ Sorts all input documents and returns them to the pipeline in sorted order.
@ sort can be performed on multiple fields.
@ fields that contain duplicate values, the docs containing those fields  may be returned in any order. results different for differnet executions.
@ for consistent sort : include at least one field in your sort that contains unique values ie _id. 
@ The $sort stage has a limit of 100 megabytes of RAM.Will error out if it exceeds 100 MB or must enable the allowDiskUse option.
{ $sort: { <field1>: <1 = ascending order> , <field2>: <-1 = descending order> , <field3> : { $meta : "textScore" } } }

$ db.scores.aggregate([ {$sort : { homework_total : 1 , age : -1 }} ])


@ for pipeline that includes a text based search using the $text operator the $meta : "textScore canbe used for soring based on the text search score.
db.users.aggregate(
   [
     { $match: { $text: { $search: "operating" } } },
     { $sort: { score: { $meta: "textScore" }, posts: -1 } }
   ]
) 
```


### <span style="color:blue"> $skip
```
@ Skips a specified number of documents that pass into the stage and passes the remaining documents to the next stage in the pipeline.
{ $skip: <# +ve number> }

@ when used with sort for consistent result, be sure to include at least one field in your sort that contains unique values.
```

### <span style="color:blue"> $sortByCount
```
@ Groups incoming documents based on the value of a specified expression, then computes the count of documents in each distinct group.
@ Each output document contains two fields: an _id field containing the distinct grouping value, and a count field containing the number of documents belonging to that grouping or category.
# syntax:-
{ $group: { _id: <expression>, count: { $sum: 1 } } }

# sample data
{
    _id: "1",
    "items" : [
     {
      "name" : "pens",
      "tags" : [ "writing", "office", "school", "stationary" ],
      "price" : NumberDecimal("12.00"),
      "quantity" : NumberInt("5")
     },
     {
      "name" : "envelopes",
      "tags" : [ "stationary", "office" ],
      "price" : NumberDecimal("1.95"),
      "quantity" : NumberInt("8")
     }
}

$ db.sales.aggregate([ 
    { $unwind: "$items" }, 
    { $unwind: "$tags" }, 
    { $sortByCount: "$items.tags" } ])
```

### <span style="color:blue"> $search
```
@ performs a full-text search on the specified field or fields.
@ search cannot be used in view definition, $lookup sub-pipeline, or $facet pipeline stages.

$ use sample_mflix;
$ db.movies.aggregate([
  { $search: 
    {
      "text": {
        "query": "baseball",
        "path": "plot"
      }
    }
  },
  {
    $limit: 5
  },
  {
    $project: {
      "_id": 0,
      "title": 1,
      "plot": 1
    }
  }
])

# search on multiple fields
$ db.movies.aggregate([
  {
    $search: {
      "text": {
        "query": "The Count of Monte Cristo",
        "path": { "value": "title", "multi": "keywordAnalyzer" }
      }
    }
  },
  {
    $project: {
      "title": 1,
      "year": 1,
      "_id": 0
    }
  }
])
```

### <span style="color:blue"> $lookup
```
@ Performs a left outer join to an unsharded collection in the same database.
@ The $lookup stage adds a new array field whose elements are the matching documents from the "joined" collection. 
@ Lookup cannot be performed on a shareded collection.
@ lookup performs an equality match on the local and foreign field.
@ In lookup, if the localField is an array, you can match the array elements against a scalar foreignField without needing an $unwind stage.
# syntax:-
{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}

db.classes.aggregate([
   {
      $lookup:
         {
            from: "members",
            localField: "enrollmentlist", // enrollmentList can be an array
            foreignField: "name",
            as: "enrollee_info"
        }
   }
])

```


### <span style="color: blue"> $merge (important)
```
@ Writes the results of the aggregation pipeline to a specified collection. 
@ $merge operator must be the last stage in the pipeline.
@ Can output to a collection in the same or different database.
@ For a sharded cluster, the specified output database must already exist.
@ Creates a new collection if the output collection does not already exist.
@ Can output to a sharded collection. Input collection can also be sharded.
@ Can incorporate results (insert new documents, merge documents, replace documents, keep existing documents, fail the operation, process documents with a custom update pipeline) into an existing collection.
@ $merge can incorporate the pipeline results into an existing output collection rather than perform a full replacement of the collection. This functionality allows users to create on-demand materialized views.
@if the _id field doesnt exist, $merge operation generates it automatically.
@ The $merge errors if the $merge results in a change to an existing document's _id value. To avoid the error, such as with a preceding $unset stage, etc.

## syntax 1
{ $merge: {
     into: <collection-name>,
     on: [ "identifier field1", "identifier field2", ...   ],  // Optional
     let: <variables>,                                         // Optional
     whenMatched: <replace|keepExisting|merge|fail|pipeline>,  // Optional
     whenNotMatched: <insert|discard|fail>                     // Optional
} }

## syntax 2
{ $merge: {
     into: { db: <output-db>, coll: <output-collection-name> },
     on: [ "identifier field1", "identifier field2", ...   ],  // Optional
     let: <variables>,                                         // Optional
     whenMatched: <replace|keepExisting|merge|fail|pipeline>,  // Optional
     whenNotMatched: <insert|discard|fail>                     // Optional
} }

@ "on" optional filed specifies the field or fields that act asunique identifier for the docs. Determines if the resultdoc already matches an existing doc.
  Order of the fields in the array/list doesnt matter.
  Same field must not be  specified multiple times in the list.
  The fields specified must exist in the pipeline result docs.
  A unique index must be present on the specified  fields.

@  whenMatched option, specifies the action to be taken when the result document and the existing doc in collection  have same values on  the specified fields. merge is the default operation. [ replace existing fields and add non-existing fields to the doc].

@ whenNotMatched option specifies what action to be taken for docs that dontmatch exisitng ones. "insert" if the default action. other actions include discard or fail.

# example :-
$ var pipeline = [
   { $group: 
        {   _id: { fiscal_year: "$fiscal_year", dept: "$dept" }, 
            salaries: { $sum: "$salary" } 
        } 
    },
   { $merge : 
        { into: { db: "reporting", coll: "budgets" }, 
          on: "_id",  
          whenMatched: "replace", 
          whenNotMatched: "insert" 
        } 
    }
]
$ db.salaries.aggregate(pipeline)

$ db.salaries.aggregate( [
   { $match : { fiscal_year:  { $gte : 2019 } } },
   { $group: { _id: { fiscal_year: "$fiscal_year", dept: "$dept" }, salaries: { $sum: "$salary" } } },
   { $merge : { into: { db: "reporting", coll: "budgets" }, on: "_id",  whenMatched: "replace", whenNotMatched: "insert" } }
] )

# $ merge can also be used with custom pipline to update existing docs in a collection.
$ db.votes.aggregate([
   { $match: { date: { $gte: new Date("2019-05-07"), $lt: new Date("2019-05-08") } } },
   { $project: { _id: { $dateToString: { format: "%Y-%m", date: "$date" } }, thumbsup: 1, thumbsdown: 1 } },
   { $merge: {
         into: "monthlytotals",
         on: "_id",
         whenMatched:  [
            { $addFields: {
                thumbsup: { $add:[ "$thumbsup", "$$new.thumbsup" ] },
                thumbsdown: { $add: [ "$thumbsdown", "$$new.thumbsdown" ] }
            } } ],
         whenNotMatched: "insert"
   } }
])
```

### <span style="color:blue"> $out
```
@ Writes the result of a  pipeline to a collection.
@ From mongoDB 4.4 a database can be specified.
@ must be the last stage in a pipeline.
@ $out operator lets the aggregation pipeline return result sets of any size.
@ For ReplicaSet/Standalone setup, if the database specified doesnt exist, it will be created.
@ If the specified collection doesnt exist, it will be created but only after successful completion of the aggregation pipeline.
@ For a Sharded Cluster, the database must already exist.
@ If db parameter is not provided, mongo will write to collection in the same database.
{ $out: { db: "<database-name>", coll: "<output-collection-name>" } }
{ $out: "<output-collection-name>"  }

@ $out stage cannot be used  to write results to a capped collection.

@ If the collection specified by the $out operation already exists, then upon completion of the aggregation, the $out stage atomically replaces the existing collection with the new results collection. The $out operation does not change any indexes that existed on the previous collection.

## Restrictions:-
@ Pipeline cannot use $out inside transactions.
@ $out stage is not allowed as part of a view definition. 
@ $out stage cannot be included in the $lookup stage's nested pipeline.
@ $facet stage's nested pipeline cannot include the $out stage.
@ $unionWith stage's nested pipeline cannot include the $out stage.

# creates a new collection "authors" with author as id and the list of books by the author.
$ db.books.aggregate( [
    { $group : { _id : "$author", books: { $push: "$title" } } },
    { $out : "authors" }
] )

# writes the output of pipline to a new database "reports" and collection name "authors"
db.books.aggregate( [
    { $group : { _id : "$author", books: { $push: "$title" } } },
    { $out : { db: "reports", coll: "authors" } }
] )

```

### <span style="color:green"> $merge vs $out comparision
```
MongoDB (4.2 onwards) provides two stages, $merge and $out, for writing the results of the aggregation pipeline to a collection. 

$out                                               ||           $merge
1) Available starting in MongoDB 2.6                            Available starting in MongoDB 4.2
2) collection in the same or different database.                Can output to a collection in the same or different database.
3) Creates a new collection if does not already exist.          Creates a new collection if the output collection does not already exist.

4) Replaces the collection completely if it already exists.    
                                                               Can incorporate results (insert new documents, merge documents, replace documents, keep existing documents, fail the operation, process documents with a custom update pipeline) into an existing collection.
                                                               Can replace the content of the collection but only if the aggregation results contain a match for all existing documents in the collection.

5) Cannot output to a sharded collection. 
Input collection, however, can be sharded.
                                                               Can output to a sharded collection. Input collection can also be sharded. Corresponds to SQL statements:

```


### <span style="color:blue"> $unwind
```
@ Deconstructs an array field from the input documents to output a document for each element.
@ if an array contains 5 elements, one document is genearted for each element, rest of the fields are same for the docs generated.
$unwind:
    {
      path: <field path>,
      includeArrayIndex: <string>,
      preserveNullAndEmptyArrays: <boolean>
    }

# sample_supplies.sales collection has one customer document with the purchases/sales as array objects with  price/object and quantity.
# lets find  out the
$ use sample_supplies
$ var pipeline = [
    {   # generates individual doc for each array element with other fields being  same 
        $unwind : {
        path: "$items",
        includeArrayIndex: "i",
        preserveNullAndEmptyArrays: true
        }
    },
    {  # just some simple projections not really needed here
        $addFields : {
            quantity : "$items.quantity",
            price : "$items.price",
            email : "$customer.email"
        }
    },
    {  # grouping by each customer email, adding all the sales total for each  customer.
        $group : {
            id : "$email",
            total : { $sum : { $multiply : ["$price" ,"$quantity"] } }
        }

    }
]
$ db.sales.aggregate(pipeline)

@ Unwinding Embedded Arrays can be done in multile unwind stages.
sample data:-  items is an array of embedded docs, and tags field is an array in each  sub document. 
TO calculate aggregations on the categries (tags) ..
{
    _id: "1",
    "items" : [
     {
      "name" : "pens",
      "tags" : [ "writing", "office", "school", "stationary" ],
      "price" : NumberDecimal("12.00"),
      "quantity" : NumberInt("5")
     },
     {
      "name" : "envelopes",
      "tags" : [ "stationary", "office" ],
      "price" : NumberDecimal("1.95"),
      "quantity" : NumberInt("8")
     }
}

$ pipeline = [
    {
        $unwind : { path : "$items" }
    }
    {
        $unwind : { path : "$items.tags" }
    }
    {
        $group : {
            id : "$items.tags",
            total : {  $multiply: [ "$items.price", "$items.quantity" ] }
        }
    }
]
```

### <span style="color:blue"> MongoDB Views.
```
@ A queryable object whose contents are defined by an aggregation pipeline, collection or a view.
@ A view  name is immutable. It cannot be changed.
@ computed on demand when a client queries a view.
@ clients must have permissions to query the view.
@ Views are read-only. ops that re supported:- find(), findOne(), count(),  distinct(), aggregate(), countDocuments().
@ Views use the Indexes of the source collection.
@ useful to redact personal information for the main docs from  users and create user specific view.
@ useful to create a view that joins multiple collections while apps can query the data without any idea of the underlying complex pipeline/stages.
@ command thats list the collections db.getCollection() also lists the views.
@ to modify a view sue collMod() or  drop it and recreate it.

@ Note:- View's underlying aggregation pipeline is subject to the 100 MB memory limit for blocking sort and blocking group operations. 
@ Views do not support $geoNear stage in pipeline, $text operator or mapReduce()
syntax:-
$ db.createView(
  "<viewName>",
  "<source>",
  [<pipeline>],
  {
    "collation" : { <collation> }
  }
)

```


### <span style="color:blue"> MongoDB on-demand Materialized Views.
```
@ In version 4.2, onwards $merge stage in pipeline can merge the results of a pipeline to a collection insted of replacing the collection.
@ this allows to  create materialized views that get updated with each pipleine  run. 
example :-
From a sales collection, a new collection (materialized  view) montlySalesReport can be created and updated everymonth with the pipeline run.

# pipeline definition

$ var monthlyReport = [
    { $match: { date: { $gte: startDate } } },
    { $group: { _id: { $dateToString: { format: "%Y-%m", date: "$date" } }, sales_quantity: { $sum: "$quantity"}, sales_amount: { $sum: "$amount" } } },
    { $merge: { into: "monthlySalesReport", whenMatched: "replace" } }
]

# pipeline execution every month
$ var startDate = new ISODate("2019-01-01")
$ db.bakesales.aggregate(monthyReport);
```

### <span style="color:blue"> $bucket
```
@ Categorizes incoming documents into groups/buckets, based on a specified expression and bucket boundaries and outputs.
@ useful for getting/understanding the distribution of data.
@ if "default" bucket not specified then the document must fall into one of the groupBy categories.
@ boundaries = lower bound is inclusive, upper bound is exclusive.
            [0,5,15] 0-5 = o inclusive,  5 exclusive; 5-15 = 5 inclusive, 15 exclusive.
@ the _id of the result will be  the lower bound of the bucket.

# Syntax :-
{
  $bucket: {
      groupBy: <expression>,
      boundaries: [ <lowerbound1>, <lowerbound2>, ... ],
      default: <literal>,
      output: {
         <output1>: { <$accumulator expression> },
         ...
         <outputN>: { <$accumulator expression> }
      }
   }
}

# group the persons, by age, count the people in each group and list their names.

$ var pipeline = [ {
    $bucket : {
        groupBy : "$dob.age",
        boundaries : [0,18,30,50,80,100],
        default : "unspecified",
        output : {
            names : { $push : "$name"},
            count : { $sum : 1 }
            averageAge : { $avg : "$dob.age"}
        }
    }
}] 
$ db.persons.aggregate(pipeline);

```

### <span style="color:blue"> $bucketAuto
```
@ Categorizes incoming documents into groups/buckets, based on a specified expression automatically.
@ tries  to evenly distribute the docs between the buckets.
@ granulatity is available only if all the groupBY fields are numbers.
@ _id in the o/p specifies the bucket range.

# syntax:-
{
  $bucketAuto: {
      groupBy: <expression>,
      buckets: <number>,
      output: {
         <output1>: { <$accumulator expression> },
         ...
      }
      granularity: <string>
  }
}

example:-
$ db.artwork.aggregate( [
   {
     $bucketAuto: {
         groupBy: "$price",
         buckets: 4
     }
   }
] )
```


### <span style="color:blue"> $facet
```
@ Categorizes incoming documents into groups/buckets, based on a specified expression and bucket boundaries and outputs.

```

### <span style="color:blue"> $redact
```
@ Restricts the contents of the documents based on information stored in the documents themselves.

$$DESCEND => $redact returns the fields at the current document level, excluding embedded documents. To include embedded documents and embedded documents within arrays, apply the $cond expression to the embedded documents to determine access for these embedded documents.

$$PRUNE => $redact excludes all fields at this current document/embedded document level, without further inspection of any of the excluded fields. This applies even if the excluded field contains embedded documents that may have different access levels.

$$KEEP => $redact returns or keeps all fields at this current document/embedded document level, without further inspection of the fields at this level. This applies even if the included field contains embedded documents that may have different access levels.

# example:- A user must be able to see the data for docs that contain "STLW, or G as tags.
# The pipeline stage , checks the $tags field and performs a set intersection and if it contains the allowed tags, the rest of the doc is displayed,
# else skips that section of the doc.

var userAccess = [ "STLW", "G" ];
db.forecasts.aggregate(
   [
     { $match: { year: 2014 } },
     { $redact: {
        $cond: {
           if: { $gt: [ { $size: { $setIntersection: [ "$tags", userAccess ] } }, 0 ] },
           then: "$$DESCEND",
           else: "$$PRUNE"
         }
       }
     }
   ]
);

# sample data with user and his credit card info.each data is categorzed with a "level" field. $ redact can be used to prune the cc info by using the level field.
[{
   "_id": 1,
   "level": 1,
   "acct_id": "xyz123",
   "cc": {
      "level": 5,
      "type": "yy",
      "num": 0,
      "exp_date": "2015-11-01T00:00:00.000Z",
      "billing_addr": {
         "level": 5,
         "addr1": "123 ABC Street",
         "city": "Some City"
      },
      "shipping_addr": [
         {
            "level": 3,
            "addr1": "987 XYZ Ave",
            "city": "Some City"
         },
         {
            "level": 3,
            "addr1": "PO Box 0123",
            "city": "Some City"
         }
      ]
   },
   "status": "ACTIVE"
},

{
   "_id": 2,
   "level": 1,
   "acct_id": "xyz123",
   "cc": {
      "level": 3,
      "type": "yy",
      "num": 0,
      "exp_date": "2015-11-01T00:00:00.000Z",
      "billing_addr": {
         "level": 5,
         "addr1": "123 ABC Street",
         "city": "Some City"
      },
      "shipping_addr": [
         {
            "level": 3,
            "addr1": "987 XYZ Ave",
            "city": "Some City"
         },
         {
            "level": 5,
            "addr1": "PO Box 0123",
            "city": "Some City"
         }
      ]
   },
   "status": "ACTIVE"
} ]


$ db.accounts.aggregate(
  [
    { $match: { status: "ACTIVE" } }, // get all the customers who are ACTIVE
    {
      $redact: {
        $cond: {
          if: { $eq: [ "$level", 5 ] }, // if level is set to 5 for any data, PRUNE it. @ cc level (highest its set to 5) 
          then: "$$PRUNE",
          else: "$$DESCEND"
        }
      }
    }
  ]
);
```



