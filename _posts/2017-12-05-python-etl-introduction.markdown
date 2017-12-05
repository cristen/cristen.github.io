---
layout: post
title: "Python ETL introduction"
date: 2017-12-03 18:01:00 GMT+1
categories: Datascience Python
---
So you’re probably here because you heard about the wonders you can make with Python and want to make your own ETL. I’ll assume you have little knowledge in SQL to go further (at least what is a column).

## But what is an ETL ?

You probably already know the popular ones (Talend or SAS for instance) but what is it all about ?

Popularized as a software, ETL is more than that, in truth it doesn’t refer to a software but rather to a process made of three stages : Extract, Transform and Load

## Do I need an ETL ?
When you’re dealing with a single source of data it’s common and you’ll probably won’t need one.
But have you ever thought what would happen if you had more than one source ? You could say that you would put each into a database… Think higher, wouldn’t that be overkill ? Managing all these databases for the same type of content ?

**“Ok so let’s just put all the sources into a single database”**

That’s the idea, but it’s not that simple because **different sources** also means **different structures** of data.
To insert data into any database there are “rules” to follow (even on NoSQL databases), and by rules, I mean you can’t fit a square into a circle (you can try but you’ll probably break everything).

The solution is to extract the data from your sources, transform the data on the fly and inject this into a database.

## The three stages of the ETL
You can view the ETL process as a pipeline constisting of three linear steps (executed in this particular order).

### Extract
This stage gathers the data from your various sources and that’s all.
### Transform
This step takes care of any transformation performed on your data. For instance changing commas to dots for decimal values (12,33 => 12.33).
### Load
This step just injects the data into your database using a predefined structure that the database will expect.

Here’s a simple representation of the process : 

<div style="text-align:center">
{% include etl-process.svg %}
</div>

As you can see on this example, all of our three sources are extracted. Then they are transformed to be consistent. Finally they are loaded into a new database.

## Dive into the practical example

Let’s see how this works in a practical context. For this example I had set up a Neo4J database with a MongoDB one.  
The example is overdocumented for the sake of better understanding.

Here’s the configuration :
**connection.cfg**

{% highlight yaml %}
[Neo4J]
url = bolt://****
port = 24786
login = awards
password = ****

[MongoDBMovies]
url = ****
port = 27017
mechanism = SCRAM-SHA-1
db = movies
collection = movies
login = ****
password = ****
{% endhighlight %}

And here’s the code : **cube.py**

{% highlight python %}
import configparser
import os

import pandas as pd
# Those are the libs to connect respectively to neo4j and mongodb databases
from neo4j.v1 import GraphDatabase, basic_auth
from pymongo import MongoClient

config = configparser.ConfigParser()
config.read('connection.cfg')

# We define some global variables that are used to connect to the databases
NEO4J = config['Neo4J']
MONGODBMOVIES = config['MongoDBMovies']

NEO4J_REQUEST = (
    'MATCH (a:Award)-[:FOR_MOVIE]->(m:Movie) '
    'RETURN m.title, count(*) as nb_award ORDER BY nb_award DESC;')
MONGODB_REQUEST = {}


def neo4j_connection():
    """
    Neo4J connection

    """
    url = NEO4J.get('url')
    login = NEO4J.get('login')
    password = NEO4J.get('password')
    driver = GraphDatabase.driver(url, auth=basic_auth(login, password))
    return driver


def neo4j_request(driver, query=None):
    """
    Neo4J request

    """
    query = query or NEO4J_REQUEST
    session = driver.session()
    result = session.run(query)
    return {'headers': result.keys(), 'records': list(result.records())}


def mongo_connection():
    """
    MongoDB connection

    """
    dbconfig = MONGODBMOVIES
    url = dbconfig.get('url')
    port = dbconfig.getint('port')
    mechanism = dbconfig.get('mechanism')
    login = dbconfig.get('login')
    password = dbconfig.get('password')
    db = dbconfig.get('db')
    collection = dbconfig.get('collection')
    client = MongoClient(url, port)
    client.movies.authenticate(login, password, mechanism=mechanism)
    db = getattr(client, db)
    return getattr(db, collection)


def mongo_request(collection, query=None):
    """
    MongoDB request

    """
    query = query or MONGODB_REQUEST
    dbmongo_results = list(collection.find(query))
    return dbmongo_results


if __name__ == '__main__':
    # ** LOAD STEP ** #
    # ---------- NEO4J ---------- #
    records = neo4j_request(neo4j_connection(), NEO4J_REQUEST)
    # Turn the Neo4j request into a dataframe
    neo4j_df = pd.DataFrame([r.values() for r in records.get('records')],
                            columns=records.get('headers'))

    # --------- MONGODB --------- #
    # Turn the MongoDB request into a dataframe too
    dbmongo_results = list(mongo_connection().find(MONGODB_REQUEST))
    dbmongo_results = mongo_request(mongo_connection())
    dbmongo_df = pd.DataFrame(dbmongo_results)

    # ** TRANSFORM STEP ** #
    # Here we merge the 2 dataframes, this is the transform step
    # turns out the two databases have the title column in common.
    neo4j_df.rename(columns={'m.title': 'title'}, inplace=True)
    merged_df = pd.merge(neo4j_df, dbmongo_df, on=['title'])
    # Still transforming the data by deleting the duplicate values
    clean_df = merged_df.sort_values(by='year').drop_duplicates(
        'title', keep='last')
    
{% endhighlight %}

And now that’s all, your data is coherent and you can insert it into your favorite DB system as you’ll usually do.

**⚠ This code will probably won’t work out of the box as I made some adjustments and hid most of it. But it should give you a vague idea of how you can make your own ETL using Python. ⚠**

Have fun !
