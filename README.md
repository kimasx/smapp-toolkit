# SMAPP Twitter Toolkit
This is an user-friendly python package for social media analytics.

## Installation
Using pip:

`pip install -e git+gitL//..#egg-info=smapp_toolkit`

Or locally

    git clone ...
    cd ..
    python setyp.py install


## Usage

#### Using MongoDB as the backend

    from smapp_toolkit.twitter import MongoTweetCollection
    collection = MongoTweetCollection(address='mongodb-address', port='mongodb-port',
        username='mongodb-user', password='mongodb-password', dbname='database-name', collection_name='collection-name')

#### Count occurences of keywords

    collection.containing('#bieber').count()
    texts = collection.containing('#bieber').texts()

#### Tweets containing one of several keywords (#bieber OR #sexy)

    collection.containing('#bieber', '#sexy')

#### Select tweets from a certain time span

    from datetime import datetime
    collection.since(datetime(2014,1,30)).count()
    collection.since(datetime(2014,2,16)).until(datetime(2014,2,19)).containing('obama').texts()

#### Select tweets authored in a certain language

    collection.language('en').texts()