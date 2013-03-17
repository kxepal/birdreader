# BirdReader

## Introduction

In March 2013, Google announced that Google Reader was to be closed. I used Google Reader every day so I set out to find a replacement.
I started with other online offerings, but then I thought "I could build one". So I created BirdReader which I have released to the world
in its unpolished "alpha".

BirdReader is designed to be installed on your own webserver or laptop, running Node.js. e.g.

* on an old PC
* on a cloud server e.g. AWS Micro server (free!)
* on a [Raspberry Pi](http://www.raspberrypi.org/)

## Features

* import your old Google Reader subscriptions 
* fetches RSS every 15 minutes
* web-based aggregated newsfeed
* - mark articles as read
* - delete articles without reading
* - 'star' articles
* - add a new feed
* - sorted in newest-first order
* - bootstrap-based, responsive layout

## How does it work?

BirdReader doesn't store anything locally other than its source code and your configuration. The data is stored in a Cloudant (CouchDB) database in the cloud.
You will need to sign up for a free Cloudant account (disclaimer: other hosted CouchDB services are available, and this code should work with any
CouchDB server e.g. your own).

Two databases are used:

### feeds database

The 'feeds' database stores a document per RSS feed you are subscribed to e.g.

```
{
    "_id": "f1cf38b2f6ffbbb69e75df476310b3a6",
    "_rev": "8-6ad06e42183368bd696aec8d25eb03a1",
    "text": "The GitHub Blog",
    "title": "The GitHub Blog",
    "type": "rss",
    "xmlUrl": "http://feeds.feedburner.com/github",
    "htmlUrl": "http://pipes.yahoo.com/pipes/pipe.info?_id=13d93fdc3d1fb71d8baa50a1e8b50381",
    "tags": ["OpenSource"],
    "lastModified": "2013-03-14 15:06:03 +00:00"
}
```

This data is directly imported from the Google Checkout OPML file and crucially stores:

* the url which contains the feed data (xmlUrl)
* the last modification date of the newest article on that feed (lastModified)

### articles database

The 'articles' database stores a document per article e.g. :

```
{
    "_id": "3c582426df29863513500a736111fa4e",
    "_rev": "1-b49944fd0edf8f50fc17c6562d75169e",
    "feedName": "BBC Entertainment",
    "tags": ["BBC"],
    "title": "VIDEO: Iran planning to sue over Argo",
    "description": "Best Picture winner Argo has been criticised by the Iranian authorities over its portrayal of the 1979 Iran hostage crisis.",
    "pubDate": "2013-03-15T15:20:31.000Z",
    "link": "http://www.bbc.co.uk/news/entertainment-arts-21805140#sa-ns_mchannel=rss&ns_source=PublicRSS20-sa",
    "pubDateTS": 1363360831,
    "read": false,
    "starred": false
}
```

The _id and _rev fields are generated by CouchDB. The feedName and tags come from the feed where the article originated. 
The rest of the fields come form the RSS article itself apart from 'read' and 'starred' which we add to record whether an 
article has been consumed or favourited.

### Views

Cloudant/CouchDB only allows data to be retrieved by its "_id" unless you define a "view". We have one view on the articles database called "byts"
that allows us to query our data: 

* unread article, sorted by timestamp
* read articles, sorted by timestamp
* starred articles, sorted by timestamp
* counts of the number of read/unread/starred articles 

The view creates a "map" function which emits keys like

```
  ["string",123455]
```

where "string" can be "unread", "read" or "starred", and "12345" is the timestamp of the article.

### Scraping articles

Every so often, BirdReader fetches all the feeds using the [feedparser](https://npmjs.org/package/feedparser). Any articles newer than
the feed's previous newest article is saved to the articles database.

### Adding new feeds

New feeds can be added by filling in a web form with the url of the page that has an RSS link tag. We use the [extractor](https://npmjs.org/package/extractor)
library to pull back the page, find the title, meta description and link tags and add the data to our feeds database.

## What does it look like?

The site is built with [Bootstrap](http://twitter.github.com/bootstrap/) so that it provides a decent interface on desktop and mobile browsers

![mobile screenshot](https://github.com/glynnbird/birdreader/raw/master/public/images/mobile.png "Mobile screenshot")

## Key technologies

* [Node.js](http://nodejs.org/) - Server side Javascript
* [Express](http://expressjs.com/) - Application framework for node
* [feedparser](https://npmjs.org/package/feedparser) - RSS feed parser 
* [Cloudant](https://cloudant.com/) - Hosted CouchDB
* [async](https://npmjs.org/package/async) - Control your parallelism in Node.js
* [Bootstrap](http://twitter.github.com/bootstrap/) - Twitter responsive HTML framework
* [sax](https://npmjs.org/package/sax) - XML parser for Node.js
* [extractor](https://npmjs.org/package/extractor) - HTML scraper, to find RSS links in HTML pages

## Installation

You will need Node.js and npm installed on your computer. Unpack the BirdReader repository and install its dependencies e.g.

```
  git clone git@github.com:glynnbird/birdreader.git
  cd birdreader
  npm install
```

Copy the sample configuration into place
```
  cd includes
  cp _config.json config.json
```

Edit the sample configuration to point to your CouchDB server.

Run Birdreader with

```
  node birdreader.js
```

See the website by pointing your browser to port 3000:
```
  http://localhost:3000/
```

## Importing Google Reader subscriptions

You can export your Google Reader subscriptions using [Google Takeout](https://www.google.com/takeout/). Download the file, unpack it 
and locate the subscriptions.xml file.

You can import this into BirdReader with

```
  node import_opml.js subscriptions.xml
```







