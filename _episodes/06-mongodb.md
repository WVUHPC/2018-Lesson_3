---
title: "No-SQL databases with MongoDB"
teaching: 15
exercises: 15
questions:
- "How to use a MongoDB database?"
objectives:
- "Get a first hand on using MongoDB"
keypoints:
- "MongoDB is a free and open-source cross-platform document-oriented database program. It stores data in JSON-like documents and fits easily for storing research data"
---

~~~
import pymongo
~~~
{: .source}

~~~
client = MongoClient('mongodb://mongo01.systems.wvu.edu', ssl=True, ssl_cert_reqs=pymongo.ssl_support.ssl.CERT_NONE)
client.server_info()
client.test.authenticate('training', 'training')
~~~
{: .source}

~~~
db=client['test']
db.collection_names()

coll=db['workshop']
~~~
{: .source}

~~~
ans = coll.insert_one({'a':1})
ans.acknowledged
ans.inserted_id
~~~
{: .source}

{% include links.md %}
