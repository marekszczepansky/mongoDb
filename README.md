# MongoDb query examples
## Documentation
* [Collection methods 4.2](https://www.mongodb.com/docs/v4.2/reference/method/js-collection/), [4.4](https://www.mongodb.com/docs/v4.4/reference/method/js-collection/)
* [Query and Projection Operators](https://www.mongodb.com/docs/v4.2/reference/operator/query/), [4.4](https://www.mongodb.com/docs/v4.4/reference/operator/query/)
* [Upate operators](https://www.mongodb.com/docs/v4.2/reference/operator/update/), [4.4](https://www.mongodb.com/docs/v4.4/reference/operator/update/)
* [Updates with Aggregation Pipeline](https://www.mongodb.com/docs/v4.2/tutorial/update-documents-with-aggregation-pipeline/),  [4.4](https://www.mongodb.com/docs/v4.4/tutorial/update-documents-with-aggregation-pipeline/)
* [$objectToArray and other aggregation operators](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/objectToArray/), [4.4](https://www.mongodb.com/docs/v4.4/reference/operator/aggregation/objectToArray/)
* [$lookup and other aggregation stages](https://www.mongodb.com/docs/v4.2/reference/operator/aggregation/lookup/), [4.4](https://www.mongodb.com/docs/v4.4/reference/operator/aggregation/lookup/)
* [SQL to MongoDB Mapping Chart](https://www.mongodb.com/docs/manual/reference/sql-comparison/#sql-to-mongodb-mapping-chart)
* [SQL to Aggregation Mapping Chart](https://www.mongodb.com/docs/manual/reference/sql-aggregation-comparison/)

## User cases
[Push a document if not exist, update if exist in a nested array](https://stackoverflow.com/a/67490125)
```javascript
let user_id = "13";
let part_id = "P456";
db.collection.update(
  { user_id: user_id },
  [{
    $set: {
      stock: {
        $cond: [
          { $in: [part_id, "$stock.part_id"] },
          {
            $map: {
              input: "$stock",
              in: {
                $cond: [
                  { $eq: ["$$this.part_id", part_id] },
                  {
                    part_id: "$$this.part_id",
                    quantity: { $add: ["$$this.quantity", 1] }
                  },
                  "$$this"
                ]
              }
            }
          },
          { $concatArrays: ["$stock", [{ part_id: part_id, quantity: 1 }]] }
        ]
      }
    }
  }]
)
```

[How to filter some fields in objects and fetch a specific subject name value in MongoDB?](https://www.tutorialspoint.com/how-to-filter-some-fields-in-objects-and-fetch-a-specific-subject-name-value-in-mongodb)
```javascript
> db.demo507.aggregate([
    {$match: {"Information.SubjectName" : "MySQL" } },
    {$project: {
       _id:0,
       Information: {
          $filter: {
             input: '$Information',
             as: 'result',
             cond: {$eq: ['$$result.SubjectName', 'MySQL']}
          }
       }
    }
 },{$project: {Information: { SubjectName:1}}}
 ]);
```

[Can you specify a key for $addToSet in Mongo](https://stackoverflow.com/a/65112446)
```javascript
newSubDocs = [ {'name' : 'matt', 'options' : 0}, {'name' : 'nick', 'options' : 2} ];
db.coll.update( { _id:1 },
[ 
   {$set:  { profile_set:  {$concatArrays: [ 
      "$profile_set",  
      {$filter: {
             input:newSubDocs, 
             cond: {$not: {$in: [ "$$this.name", "$profile_set.name" ]}} 
      }}
   ]}}}
])
```

[MongoDB find value match for a property in array within array of objects](https://stackoverflow.com/questions/36936247/mongodb-find-value-match-for-a-property-in-array-within-array-of-objects)

```javascript
db.getCollection('test').aggregate(
    [{
        "$unwind": "$items"
    }, {
        "$match": {
            "items.votes.people.username": "user2"
        }
    }]
)
```

[Filter MongoDB Array Element Using $Filter Operator](https://kb.objectrocket.com/mongo-db/filter-array-elements-in-mongodb-1586#filter+mongodb+array+element+using+%24filter+operator)
```js
db.component.aggregate([
{
   "$match" : {
       "specification" : {
          "$elemMatch" : {
             "$and" : [
                { "frame_buffer" : "6 GB GDDR6" }
             ]
          }
       },
   }
},
{
   "$project" : {
       "sku" : 1, "maker" : 1,
       "specification" : {
          "$filter" : {
             "input" : "$specification",
             "as" : "specification",
             "cond" : {
                "$and" : [
                   { "$eq" : [ "$$specification.frame_buffer", "6 GB GDDR6" ] }
                ]
             }
          }
       }
   }
}
]).pretty();
````
[How to use $map over an object of objects in an aggregation? - Record<string, object>](https://stackoverflow.com/a/62283596)
```js
{
    $replaceRoot: {
        newRoot: {
            $mergeObjects: [
                { _id: "$_id", name: "$name" },
                {
                    $arrayToObject: {
                        $map: {
                            input: { $objectToArray: "$attribute_bag" },
                            in: [ "$$this.k", "$$this.v.value" ]
                        }
                    }
                }
            ]
        }
    }
}
```

[How to Map and reduce an array of Strings to a single object with multiple values - Record<string, string>](https://stackoverflow.com/a/71359816)
```json
[
  {
    "$project": {
      "provider": {
        "$arrayToObject": {
          "$map": {
            "input": "$value",
            "as": "v",
            "in": [
              {
                "$concat": [
                  "value",
                  {
                    "$toString": {
                      "$indexOfArray": [
                        "$value",
                        "$$v"
                      ]
                    }
                  }
                ]
              },
              "$$v"
            ]
          }
        }
      }
    }
  }
]
```

{Document to array of values](https://stackoverflow.com/a/71808874)
```js
db.collection.aggregate([
  {
    $project: {
      _id: 0,
      values: {
        $map: {
          input: {
            "$objectToArray": "$$ROOT"
          },
          as: "item",
          in: "$$item.v"
        }
      }
    }
  }
])
```

[Join array element with collection and project as extended element](https://stackoverflow.com/a/65047760)
```js
db.players.aggregate([
  {
    $addFields: {
      items: {
        $map: {
          input: "$items",
          in: {
            $mergeObjects: ["$$this", { itemId: { $toObjectId: "$$this.itemId" } }]
          }
        }
      }
    }
  },
  {
    $lookup: {
      from: "items",
      localField: "items.itemId",
      foreignField: "_id",
      as: "itemsCollection"
    }
  },
  {
    $project: {
      username: 1,
      items: {
        $map: {
          input: "$items",
          as: "i",
          in: {
            $mergeObjects: [
              "$$i",
              {
                $first: {
                  $filter: {
                    input: "$itemsCollection",
                    cond: { $eq: ["$$this._id", "$$i.itemId"] }
                  }
                }
              }
            ]
          }
        }
      }
    }
  }
])
```

[Remove a field from all elements in array in mongodb](https://stackoverflow.com/a/47836562)
```js
db.coll.update( {_id:235399}, {$unset: {"casts.crew.$[].withBase":""}} )
```

[Remove field found in any mongodb array](https://stackoverflow.com/a/55451053)

Case 1 - Remove the inner array elements where the Field is present
```js
db.collection.updateMany(
  { "scan": {
    "$elemMatch": {
      "$elemMatch": {
        "arrayToDelete": { "$exists": true }
      }
    }
  } },
  {
    "$pull": {
      "scan.$[a]": { "arrayToDelete": { "$exists": true } }
    }
  },
  { "arrayFilters": [
      {  "a": { "$elemMatch": { "arrayToDelete": { "$exists": true } } } }
    ]
  }
)
```
Case 2 - Just remove the matched field from the inner elements
```js
db.collection.updateMany(
  { "scan": {
    "$elemMatch": {
      "$elemMatch": {
        "arrayToDelete": { "$exists": true }
      }
    }
  } },
  { "$unset": { "scan.$[].$[].arrayToDelete": ""  } }
)
```
Case 3 - You Actually wanted to remove "Everything" in the array
```js
db.collection.updateMany(
  { "scan": {
    "$elemMatch": {
      "$elemMatch": {
        "arrayToDelete": { "$exists": true }
      }
    }
  } },
  { "$set": { "scan": []  } }
)
```
You could still use an arrayFilters and a positional filtered $\[<identifier>], but here it would be overkill.
```js
db.collection.updateMany(
  { "scan": {
    "$elemMatch": {
      "$elemMatch": {
        "arrayToDelete": { "$exists": true }
      }
    }
  } },
  { "$unset": { "scan.$[a].$[b].arrayToDelete": ""  } },
  {
    "arrayFilters": [
      { "a": { "$elemMatch": { "arrayToDelete": { "$exists": true } } } },
      { "b.arrayToDelete": { "$exists": true } },
    ]
  }
)
```

[How to Update Multiple Array Elements in mongodb](https://stackoverflow.com/a/46054172)
```js
db.collection.update(
  { "events.profile":10 },
  { "$set": { "events.$[elem].handled": 0 } },
  { "arrayFilters": [{ "elem.profile": 10 }], "multi": true }
)
```

[Map array primitiv elements to objects](https://stackoverflow.com/a/56766672)
```js
// {
//   title: "record 1",
//   fields: [
//     { _id: 1, items: [1] },
//     { _id: 2, items: [2, 3, 4] },
//     { _id: 3, items: [5] }
//   ]
// }
db.collection.update(
  {},
  [{ $set: {
       fields: { $map: {
         input: "$fields",
         as: "x",
         in: {
           _id: "$$x._id",
           items: { $map: {
             input: "$$x.items",
             as: "y",
             in: { item: "$$y", key: 0 }
           }}
         }
       }}
  }}],
  { multi: true }
)
// {
//   title: "record 1",
//   fields: [
//     { _id: 1, items: [ { item: 1, key: 0 } ] },
//     { _id: 2, items: [ { item: 2, key: 0 }, { item: 3, key: 0 }, { item: 4, key: 0 } ] },
//     { _id: 3, items: [ { item: 5, key: 0 } ] }
//   ]
// }
```

[Mongodb update pipeline update a field of a element inside a nested array](https://stackoverflow.com/a/68924820)
```js
await products.findOneAndUpdate(filters, [
  {
    $set: {
      "materials.m1.inventory": {
        $map: {
          input: {
            $range: [0, { $size: "$materials.m1.inventory" }]
          },
          in: {
            $cond: [
              { $eq: ["$$this", 0] }, // 0 position
              {
                $mergeObjects: [
                  { $arrayElemAt: ["$materials.m1.inventory", "$$this"] },
                  { amount: 100 } // input amount
                ]
              },
              { $arrayElemAt: ["$materials.m1.inventory", "$$this"] }
            ]
          }
        }
      }
    }
  }
]);
```

[Using MergeObjects without overwriting subdocuments - deep merge](https://stackoverflow.com/questions/61667255/using-mergeobjects-without-overwriting-subdocuments)
```js
db.collection.aggregate([
  {$addFields: {
      combined: {
        $concatArrays: [
          {$objectToArray: "$b"},
          {$objectToArray: "$c"}
        ]
      }
  }},
  {$unwind: "$combined"},
  {$group: {
      _id: {_id: "$_id", key: "$combined.k"},
      doc: {$first: "$$ROOT"},
      v: {$mergeObjects: "$combined.v"}
  }},
  {$group: {
      _id: "$_id._id",
      doc: {$first: "$doc"},
      combined: {$push: {
          k: "$_id.key",
          v: "$v"
      }}
  }},
  {$addFields: {"doc.b": {$arrayToObject: "$combined"}}},
  {$project: {
    combined: 0,
    "doc.c": 0
  }},
  {$replaceRoot: {newRoot: "$doc"}}
])
```

[MongoDB Aggregation Pipeline and combining array objects](https://stackoverflow.com/a/27111875)
```js
db.coll.aggregate([
    {
        $match:{_id:1}  
    },
     { 
         $project: {_id: 0 ,
                    combinedList: { $setUnion: [ "$deeper.list3","$list2", "$list1" ] }} 
     }
   ]
)
```

[How to flatten a subdocument into root level in MongoDB?](https://stackoverflow.com/q/22903849)
```js
db.collection.aggregate([
    { "$addFields": { "subdoc.a": "$a" } },
    { "$replaceRoot": { "newRoot": "$subdoc" }  }
])
```
```js
let pipeline = [
   {
      "$replaceWith": {
         "$mergeObjects": [ { "a": "$a" }, "$subdoc" ]
      }
   }
];

db.collection.aggregate(pipeline);
```
```js
let pipeline = [
    {
        "$replaceRoot": {
            "newRoot": {
                "$mergeObjects": [{ "a": "$a" }, "$subdoc" ]
            }
        }
    }
];

db.collection.aggregate(pipeline);
```
```js
{ $replaceWith: {
    $mergeObjects: [ "$$ROOT", "$subdoc" ]
  } 
}
```
```js
// { a: 1, subdoc: { b: 2, c: 3 } }
db.collection.aggregate([
  { $set: { "subdoc.a": "$a" } },
  { $replaceWith: "$subdoc" }
])
// { b: 2, c: 3, a: 1 }
```
[Find in Double Nested Array MongoDB](https://stackoverflow.com/a/29072062)

By ket field
```js
db.mycollection.find({
    "someArray.someNestedArray.name": "value"
})
```
By many fields
```js
db.mycollection.find({
    "someArray": { 
        "$elemMatch": {
            "name": "name1",
            "someNestedArray": {
                "$elemMatch": {
                    "name": "value",
                    "otherField": 1
                }
            }
        }
    }
})
```
By crazy :)
```js
db.mycollection.aggregate([
  { "$match": {
    "someArray": {
      "$elemMatch": {
         "name": "name1",
         "someNestedArray": {
           "$elemMatch": {
             "name": "value",
             "otherField": 1
           }
         }
       }
    }
  }},
  { "$addFields": {
    "someArray": {
      "$filter": {
        "input": {
          "$map": {
            "input": "$someArray",
            "as": "sa",
            "in": {
              "name": "$$sa.name",
              "someNestedArray": {
                "$filter": {
                  "input": "$$sa.someNestedArray",
                  "as": "sn",
                  "cond": {
                    "$and": [
                      { "$eq": [ "$$sn.name", "value" ] },
                      { "$eq": [ "$$sn.otherField", 1 ] }
                    ]
                  }
                }
              }             
            }
          },
        },
        "as": "sa",
        "cond": {
          "$and": [
            { "$eq": [ "$$sa.name", "name1" ] },
            { "$gt": [ { "$size": "$$sa.someNestedArray" }, 0 ] }
          ]
        }
      }
    }
  }}
])
```
