# SMAPP Twitter Toolkit
This is an user-friendly python package for interfacing with large collections of tweets. Developped at the SMaPP lab at New York University.

**Supports Python 2.7**

## Installation
Simplest: using `pip`:
```bash
pip install smapp-toolkit
```

To update to the latest version, if you have an older one installed:
```bash
pip install -U smapp-toolkit
```

Or download the source code using git
```bash
git clone https://github.com/SMAPPNYU/smapp-toolkit
cd smapp-toolkit
python setup.py install
```

or download [the tarball](https://github.com/SMAPPNYU/smapp-toolkit/tarball/master) and install.

#### Dependencies
The `smapp-toolkit` depends on the following packages, which will be automatically installed when installing `smapp-toolkit`:
* [pymongo](http://api.mongodb.org/python/current/), the Python MongoDB driver
* [smappPy](https://github.com/SMAPPNYU/smappPy), a utility library from SMaPP

## Usage

#### Using MongoDB as the backend
```python
from smapp_toolkit.twitter import MongoTweetCollection
collection = MongoTweetCollection(address='mongodb-address',
                                  port='mongodb-port',
                                  username='mongodb-user',
                                  password='mongodb-password',
                                  dbname='database-name')
```

#### Using a BSON file as the backend
```python
from smapp_toolkit.twitter import BSONTweetCollection
collection = BSONTweetCollection("path/to/file.bson")
```

#### Count occurences of keywords
```python
collection.containing('#bieber').count()
texts = collection.containing('#bieber').texts()
```

#### Tweets containing one of several keywords (#bieber OR #sexy)
```python
collection.containing('#bieber', '#sexy')
```

### Count occerences of multiple keywords over time
```python
collection.term_counts(['justin', 'miley'], count_by='days', plot=False)
Out[]:
{'2015-04-01': {'justin': 1312, 'miley': 837},
 '2015-04-02': {'justin': 3287, 'miley': 932}}
```


#### Random sample of tweets
```python
collection.containing('#bieber').sample(0.33).texts()
```

#### Select tweets from a certain time span
```python
from datetime import datetime
collection.since(datetime(2014,1,30)).count()
collection.since(datetime(2014,2,16)).until(datetime(2014,2,19)).containing('obama').texts()
```

#### Select tweets authored in a certain language
```python
collection.language('en').texts()
```

### Tweets in Russian OR Ukrainian
```python
collection.language('ru', 'uk')
```

### Tweets from users with their stated language preference to French OR German
```python
collection.user_lang_contains('de', 'fr')
```

#### Exclude retweets
```python
collection.excluding_retweets().count()
```

#### Only tweets where the user location indicates they are from new york
```python
collection.user_location_containing('new york', 'nyc')
```

#### Only tweets where the user cares about python data analysis
```python
collection.field_containing('user.description', 'python', 'data', 'analysis')
```

#### Only get geotagged tweets
```python
collection.geo_enabled()
```

#### Only get non-geotagged tweets
```python
collection.non_geo_enabled()
```

#### Sorting by time
```python
collection.sort('timestamp',-1)
```

#### Only get the latest 10 tweets
```python
collection.sort('timestamp',-1).limit(10).texts()
```

#### Visualizing the volume of tweets
```python
bins, counts = collection.containing('#sexy').tweets_over_time_figure(
    start_time,
    step_size=timedelta(minutes=1),
    num_steps=60,
    show=False)
plt.title('Tweets containing "#sexy"')
plt.show()
```

#### Vizualizing volume of selected terms over time
```python
collection.term_counts(['justin', 'miley'], count_by='days', plot=True, plot_total=True)
```

#### Iterate over the full tweet objects
```python
for tweet in collection.containing('#nyc'):
    print(tweet['text'])
```

#### Dumping tweets to a CSV file
```python
collection.dump_csv('my_tweets.csv')
```
This will dump a CSV with the following columns:

    'id_str', 'user.screen_name', 'timestamp', 'text'

The desired columns may be specified in the `columns=` named argument:

```python
collection.dump_csv('my_tweets.csv', columns=['id_str', 'user.screen_name', 'user.location', 'user.description', 'text'])
```

The full list of available fields from a tweet may be found on [the twitter REST-API documentation](https://dev.twitter.com/overview/api/tweets). In order to get nested fields (such as the user's location or the user's screen_name), use `user.location`, `user.screen_name`.

**tweet coordinates**
For geolocated tweets, in order to get the geolocation out in the csv, add `coordinates.coordinates` to the columns list. This will put the coordinates in [GeoJSON](http://geojson.org/geojson-spec.html#positions) (long, lat) in the column.
*Alternatively*¸ add `coordinates.coordinates.0` and `coordinates.coordinates.1` to the columns list. This will add two columns with the longitude and latitude in them respectively.

## Figures
Smapp-toolkit has some built-in plotting functionality. See the [example scripts](https://github.com/SMAPPNYU/smapp-toolkit/tree/master/examples), and check out the [gallery](http://philosoraptor.bio.nyu.edu:82/figure-gallery/)!

Currently implemented:
* barchart of tweets per time-unit (tweets_over_time_figure(...))
* barchart by language by day (languages_per_day_figure(...))
* line chart (tweets per day) with vertical event annotations (tweets_per_day_with_annotations_figure(...))

In order to get these to work, some extra packages (not automatically installed) need to be installed:
* `matplotlib`
* `seaborn`

## The MongoDB Data Model
SMAPP stores tweets in MongoDB databases, and splits the tweets across multiple MongoDB collections, because this gives better performance than a single large MongoDB collection. The MongoDB Database needs to have a `smapp_metadata` collection with a single `smapp-tweet-collection-metadata` document in it, which specifies the names of the tweet collections.

The `smapp-tweet-collection-metadata` document has the following form:

```json
{
  "document": "smapp-tweet-collection-metadata",
  "tweet_collections": [
    "tweets_1",
    "tweets_2",
    "tweets_3",
  ]
}
```

### Customization
The `MongoTweetCollection` object may still be used if the metadata collection and document have different names:

```python
collection = MongoTweetCollection(..., metadata_collection='smapp_metadata', metadata_document='smapp-tweet-collection-metadata')
```

#### Already have tweets in your own mongo and want to use the smapp-toolkit?
All you need to do is insert the following collection and document into your MongoDB database:

(from the mongo shell)

```
db.smapp_metadata.save({
  "document": "smapp-tweet-collection-metadata",
  "tweet_collections": [ "tweets" ]
})
```

and the default behavior will work as advertised.

-----------
Code and documentation &copy; 2014 New York University. Released under [the GPLv2 license](LICENSE).
