# Lab 8

## Student information

* Full name: Arun Chacko
* E-mail: achac021@ucr.edu
* UCR NetID: achac021
* Student ID: 862298328

## Answers

* (Q1) What is the schema of the file? Copy it to the README file and keep it for your reference.

```
[Stage 0:>                                                                                              root  
 |-- hashtags: array (nullable = true)
 |    |-- element: string (containsNull = true)
 |-- id: long (nullable = true)
 |-- lang: string (nullable = true)
 |-- place: struct (nullable = true)
 |    |-- country_code: string (nullable = true)
 |    |-- name: string (nullable = true)
 |    |-- place_type: string (nullable = true)
 |-- text: string (nullable = true)
 |-- time: string (nullable = true)
 |-- user: struct (nullable = true)
 |    |-- followers_count: long (nullable = true)
 |    |-- statuses_count: long (nullable = true)
 |    |-- user_id: long (nullable = true)
 |    |-- user_name: string (nullable = true)

```

* (Q2) What is your command to import the `tweets.json` file?

    ```shell
    mongoimport --db mydatabase --collection tweets --file /home/cs167/cs167/tweets.json
    ```

* (Q3) What is the output of the import command?

    ```text
    2025-02-17T17:02:35.773-0800    connected to: mongodb://localhost/
    2025-02-17T17:02:35.820-0800    1000 document(s) imported successfully. 0 document(s) failed to import.
    ```

* (Q4) What is your command to count the total number of records in the `tweets` collection and what is the output of the command?

    ```javascript
    db.tweets.countDocuments()
    The output is 1000.
    ```

* (Q5) What is your command for this query?

    ```javascript
    db.tweets.find(
  {
    "place.country_code": "JP",
    "user.statuses_count": { $gt: 50000 }
  },
  {
    "user.user_name": 1,
    "user.followers_count": 1,
    "user.statuses_count": 1,
  }
  ).sort({ "user.followers_count": 1 })
    ```

* (Q6) How many records does your query return?
- 16 records are returned by my query.
* (Q7) What is the command that retrieves the results without the _id field?

    ```javascript
    db.tweets.find(
  {
    "place.country_code": "JP",
    "user.statuses_count": { $gt: 50000 }
  },
  {
    "user.user_name": 1,
    "user.followers_count": 1,
    "user.statuses_count": 1,
    "_id": 0
  }
  ).sort({ "user.followers_count": 1 })

    ```

* (Q8) What is the command to insert the sample document? What is the result of running the command?

    ```javascript
    db.tweets.insertOne({
  id: NumberLong("921633456941125634"),
  place: { 
    country_code: "JP", 
    name: "Japan", 
    place_type: "city" 
  },
  user: {
    user_name: "xyz2",
    followers_count: [2100, 5000],  // Array of follower counts
    statuses_count: 55000
  },
  hashtags: ["nature"],
  lang: "ja"
  })

    
    ```
This is the result of running the command:
```
{
  acknowledged: true,
  insertedId: ObjectId('67b3def8dab253a501e43269')
}
```

* (Q9) Does MongoDB accept this document while the followers_count field has a different type than other records?
- Yes, MongoDB accepts this document even though the followers_count has a different type than other recoirds.
* (Q10) What is your command to insert this record?

    ```javascript
    db.tweets.insertOne({
  id: NumberLong("921633456941121354"),
  place: { 
    country_code: "JP", 
    name: "Japan", 
    place_type: "city" 
  },
  user: {
    user_name: "xyz3",
    followers_count: { 
      last_month: 550, 
      this_month: 2200 
    },
    statuses_count: 112000
  },
  hashtags: ["art", "tour"],
  lang: "ja"
  })

    ```


* (Q11) Where did the two new records appear in the sort order?
- User name "xyz2" appears between followers_count {4578} and {6745}. User name "xyz3" is the first record that appears in the sort order.
* (Q12) Why did they appear at these specific locations?
- According to the MongoDB documentation: "A descending sort compares the largest elements of the array according to the reverse BSON type sort order.' This is hy 'xyz2' appears where it does because the value {5000} is being used to compare with the follower counts.
- When sorting with MongoDB, it uses the first field in nested objects for comparison. In this case, it's using the value 550. That is why 'xyz3' appears first in the sort order.

* (Q13) Where did the two records appear in the ascending sort order? Explain your observation.
- User name "xyz2" appears between followers_count {2049} and {2197}. User name "xyz3" is the last record that appears in the sort order.
- According to the MongoDB documentation: "An ascending sort compares the smallest elements of the array according to the BSON type sort order.' This is why 'xyz2' appears where it does because the value {2100} is being used to compare with the follower counts. 


- When sorting with MongoDB, it uses the first field in nested objects for comparison. In this case, it's using the value 550. That is why 'xyz3' appears last in the sort order, since we are using ascending order this time.

* (Q14) Is MongoDB able to build the index on that field with the different value types stored in the `user.followers_count` field?
- Yes, MongoDB was able to create the index, but with limitations. MongoDB will index only values it can sort properly. Documents with non-sortable types (objects, arrays) may be ignored or behave unexpectedly in queries.

* (Q15) What is your command for building the index?

    ```javascript
    db.tweets.createIndex({ "user.followers_count": 1 })
    ```

* (Q16) What is the output of the create index command?

    ```text
    "user.followers_count_1"
    ```

* (Q17) What is your command for this query?

    ```javascript
    db.tweets.find(
  {
    "hashtags": { $in: ["job", "hiring", "IT"] }
  },
  {
    "text": 1,
    "hashtags": 1,
    "user.user_name": 1,
    "user.followers_count": 1,
    "_id": 0
  }
  ).sort({ "user.followers_count": 1 })
    ```

* (Q18) How many records are returned from this query?

   24 records were returned from this query/

* (Q19) What is your command for this query?
    ```javascript
    db.tweets.aggregate([
  { $group: { _id: "$place.country_code", tweets_count: { $sum: 1 } } },
  { $sort: { tweets_count: -1 } },
  { $limit: 5 },
  { $project: { country_code: "$_id", tweets_count: 1, _id: 0 } }
])

    ```

* (Q20) What is the output of the command?
```
[
  { tweets_count: 153, country_code: 'US' },
  { tweets_count: 107, country_code: 'JP' },
  { tweets_count: 89, country_code: 'GB' },
  { tweets_count: 65, country_code: 'TR' },
  { tweets_count: 56, country_code: 'IN' }
]
```
* (Q21) What is your command for this query?
    ```javascript
    db.tweets.aggregate([
  { $unwind: "$hashtags" },
  { $group: { _id: "$hashtags", count: { $sum: 1 } } },
  { $sort: { count: -1 } },
  { $limit: 5 }
])

    ```

* (Q22) What is the output of the command?
```
[
  { _id: 'ALDUBxEBLoveis', count: 56 },
  { _id: 'FurkanPalalı', count: 31 },
  { _id: 'no309', count: 31 },
  { _id: 'LalOn', count: 31 },
  { _id: 'job', count: 19 }
]
```

* (Q23) Are there any existing indexes? Explain your answer.
- Yes, there is an existing index. The default index is present but there are no additional custom indexes since none have been added.

* (Q24) What's the running time of the second query? Comparing to Q23, it is faster or slower?
* The running time of the second query is 163ms which, when compared to Q23, it is faster.

  
