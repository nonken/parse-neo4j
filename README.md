[![npm version](https://badge.fury.io/js/parse-neo4j.svg)](https://www.npmjs.com/package/parse-neo4j)
# parse-neo4j
Neo4j's http endpoint produces result that contains complete query information.

`parse-neo4j` helps those who only want what they've returned in the query as normal JSON.

## Install

```bash
npm install parse-neo4j --save
```

## Usage
You can connect to neo4j's http endpoint via one of the many drivers available, e.g. official [`neo4j-driver`](https://github.com/neo4j/neo4j-javascript-driver)
```javascript
var neo4j = require('neo4j-driver').v1;
var driver = neo4j.driver("bolt://localhost", neo4j.auth.basic("neo4j", "neo4j"));
var session = driver.session();
var result = session
    .run("MERGE (james:Person {name : {nameParam} }) RETURN james.name", { nameParam:'James' })
    .then(function(result){
        result.records.forEach(function(record) {
            console.log(record._fields);
        });
        // Completed!
        session.close();
    })
    .catch(function(error) {
        console.log(error);
    });
```
Then you can parse the results:
```javascript
var parse = require('parse-neo4j');
var parsedResult = result
    .then(parse)
    .then(function(parsed){
        parsed.forEach(function(parsedRecord) {
            console.log(parsedRecord);
        });
    })
    .catch(function(parseError) {
        console.log(parseError);
    });
```

## Example
This:
```JSON
{
   "records":[
      {
         "keys":[
            "key1",
            "key2"
         ],
         "length":2,
         "_fields":[
            {
               "identity":{
                  "low":143258,
                  "high":0
               },
               "labels":[
                  "Label1",
                  "Label2"
               ],
               "properties":{
                  "field1":"...",
                  "number":{
                     "low":42,
                     "high":0
                  },
                  "date":1460183280000
               }
            },
            {
               "identity":{
                  "low":117186,
                  "high":0
               },
               "labels":[
                  "Another",
                  "Node"
               ],
               "properties":{}
            }
         ],
         "_fieldLookup":{
            "key1":0,
            "key2":1
         }
      },
      {
         "keys":[
            "key1",
            "key2"
         ],
         "length":2,
         "_fields":[
            {
               "identity":{
                  "low":143260,
                  "high":0
               },
               "labels":[
                  "Label1",
                  "Label2"
               ],
               "properties":{
                  "field1":"...",
                  "number":{
                     "low":2,
                     "high":0
                  },
                  "date":1460183280000
               }
            },
            {
               "identity":{
                  "low":117186,
                  "high":0
               },
               "labels":[
                  "Another",
                  "Node"
               ],
               "properties":{}
            }
         ],
         "_fieldLookup":{
            "key1":0,
            "key2":1
         }
      }
   ],
   "summary":{
      "statement":{
         "text":"MATCH (a:Label1)\nMATCH (a)-[:Another]->(au)\nRETURN a AS key1, au AS key2\nORDER BY a.date DESC\nSKIP {skip} LIMIT {limit}",
         "parameters":{
            "skip":{
               "low":10,
               "high":0
            },
            "limit":{
               "low":2,
               "high":0
            }
         }
      },
      "statementType":"r",
      "updateStatistics":{
         "_stats":{
            "nodesCreated":0,
            "nodesDeleted":0,
            "relationshipsCreated":0,
            "relationshipsDeleted":0,
            "propertiesSet":0,
            "labelsAdded":0,
            "labelsRemoved":0,
            "indexesAdded":0,
            "indexesRemoved":0,
            "constraintsAdded":0,
            "constraintsRemoved":0
         }
      },
      "plan":false,
      "profile":false,
      "notifications":[

      ]
   }
}
```
Becomes:
```JSON
[
    {
        "key1": {
            "field1": "...",
            "number": "42",
            "date": 1460183280000
        },
        "key2": {}
    },
    {
        "key1": {
            "field1": "...",
            "number": "2",
            "date": 1460183280000
        },
        "key2": {}
    },
]
```
