# PROBLEM
## How do mongodb indices work, together?

I have a users collection with two fields, structured as follows.

"email":"dog@gmail.com"
"identities":[{"uid":"terrible","provider":"even_worse"}].
I have created the following indexes on the collection. Basically I have indices

1.An index on "_id" : the default

2.An index on "email" : alone

3.An index on "identities" : alone

4.An index on "_id" + "identities"

5.An index on "email" + "identities"

```
rs0:PRIMARY> db.users.getIndexes()
[
{
    "v" : 1,
    "key" : {
        "_id" : 1
    },
    "name" : "_id_",
    "ns" : "test_development.users"
},
{
    "v" : 1,
    "key" : {
        "email" : 1
    },
    "name" : "email_index",
    "ns" : "test_development.users"
},
{
    "v" : 1,
    "key" : {
        "identities.uid" : 1,
        "identities.provider" : 1
    },
    "name" : "identities_index",
    "ns" : "test_development.users"
},
{
    "v" : 1,
    "key" : {
        "_id" : 1,
        "identities.uid" : 1,
        "identities.provider" : 1
    },
    "name" : "id_and_identities_index",
    "ns" : "test_development.users"
},
{
    "v" : 1,
    "key" : {
        "email" : 1,
        "identities.uid" : 1,
        "identities.provider" : 1
    },
    "name" : "email_and_identities_index",
    "ns" : "test_development.users"
}
]
```

i perform the following query with explain() turned on:


```
db.users.find({ "email":"test@gmai.com","identities":{$elemMatch : {"uid":"cat", "provider": "dog"}}}).explain()
```

the results of explain indicated that only the email index is used, and that the identities indices are never queried. 

```
{
"cursor" : "BtreeCursor email_index",
"isMultiKey" : false,
"n" : 0,
"nscannedObjects" : 0,
"nscanned" : 0,
"nscannedObjectsAllPlans" : 0,
"nscannedAllPlans" : 0,
"scanAndOrder" : false,
"indexOnly" : false,
"nYields" : 0,
"nChunkSkips" : 0,
"millis" : 0,
"indexBounds" : {
    "email" : [
        [
            "test@gmai.com",
            "test@gmai.com"
        ]
    ]
},
"server" : "dragon:27017",
"filterSet" : false
}
```

=========================

# Answer

Here's how this works:

Mongodb is using the indexes correctly.
It all depends on how many docs match your query
Going by the example that I posted, here is how the flow goes:

a.If both the fields that are searched have more than one doc id in the index, then the combined index will be used. 
What that means is:

```
Document A: {"email":"tagger@gmail.com","identities":[{"uid":"test","provider":"facebook"}]}

Document B: {"email":"raggy@gmail.com","identities":[{"uid":"test","provider":"google"}]}
```

If we ran my query on a collection with these two documents, the "email" index will be used, because the emails in the collection can limit the documents scanned to a just one. The identities collection will not be used, and neither will the combined "email" and "identities" index.

Suppose that in the above two documents, the emails were the same, but the identities were different, then the "identities" index would be used, ignoring both the "email" index and the combined "identities" + "email" index.

Now suppose that we add a third document into the fray:

```
Document C: {"email":"tagger@gmail.com", "identities":[{"uid":"test","provider":"google"}]}
```

This document shares the email of document A, and the identities of document B. In order to answer my query, MongoDb will use the combined "email" + "identities" index, because both the indexed fields, have more than one document in the index, and the only way to find a match is to narrow results down both ways.
