# CouchCache
````
 _____                  _     _____            _          
/  __ \                | |   /  __ \          | |         
| /  \/ ___  _   _  ___| |__ | /  \/ __ _  ___| |__   ___ 
| |    / _ \| | | |/ __| '_ \| |    / _` |/ __| '_ \ / _ \
| \__/\ (_) | |_| | (__| | | | \__/\ (_| | (__| | | |  __/
 \____/\___/ \__,_|\___|_| |_|\____/\__,_|\___|_| |_|\___|
```                                                          

## Introduction
                                                          
If you want to produce a resilient Memcache cluster, it's harder than you think. If you happen to have a resilient CouchDB cluster to hand, such as the hosted CouchDB service provided by [Cloudant](https://cloudant.com/), you could use CouchDB as your cache without having to create and maintain another cluster of servers.

This project is software library for Node.js that provides Memcache-like functionality but uses CouchDB as the storage mechanism. This is slower than Memcache, but is persistant. 

## Installation

```                                                       
  npm install couchcache
```

## Hello World

You need to initialise CouchCache with the url of the your CouchDb instance:

```
  var couchcache = required("couchcache");
  couchcache.init("https://username:password@username.cloudant.com:443", function(err, data) {
    console.log("CouchCache nitialised");
  });                                                  
```

Then you can write items:

```
  couchcache.set("mykey", "myvalue", function(err, data) {
    console.log("CouchCache nitialised");
  });
```

and read them back:

```
  couchcache.get("mykey", function(err, data) {
    console.log(data);
  });
```

and delete them:

```
  couchcache.delete("mykey", function(err, data) {
    console.log(data);
  });
```

Occasionaly, you can ask CouchCache to purge old keys:

```
  couchcache.purge(function(err, data) {
    console.log("CouchCache purged");
  });
```

You can write arbitrary JSON objects:
Then you can write items:

```
  couchcache.set("mykey", { a: "myvalue", b: [1,2,3], c: true }, function(err, data) {
    console.log("CouchCache nitialised");
  });
```

## How does it work?

CouchCache simply stores the supplied key/value pairs in CouchDB documents. 

```
{
  "_id": "106e07e9cfebdf9e4da2d99931af4211",          // CouchDB auto-generated id
  "_rev": "1-96be08c1f3372ab8a1cea2913acaa7f8",       // CouchDB auto-generated revision
  "cacheKey": "mykey",                                // user supplied cache key
  "value": "myvalue",                                 // user supplied cache value
  "ts": 1390118630865                                 // expiry timestamp
}
```

Notice we don't store the supplied key in the document's "_id" field; this is because we want to be
able to overwrite keys simply by adding another document with the same *cacheKey* but with a later timestamp.

When we retrieve a cacheKey, we use a CouchDB view that fetches the document that matches the supplied cacheKey and has the newest timestamp.

On startup, a new CouchDB database is created and the CouchCache design document is installed which 
creates two views:
* _design/fetch/by_key - used to fetch individual cache keys
* _desing/fetch/by_ts - used to purge old cache documents 

## Other options

When initialising CouchCache, you can supply an optional "options" object which can contain:

```
var options = {
  expires: 60*60*24*1000,    // expire cacheKeys one day from now (in milliseconds) 
  turbo: false,              // whether to enable turbo mode
  autopurge: true,           // whether to purge old cacheKeys from the databse every hour
  dbname: "cache"            // the name of the CouchDB database to use
};
var couchcache = required("couchcache");
couchcache.init("https://username:password@username.cloudant.com:443", options, function(err, data) {
  console.log("CouchCache nitialised");
});  
```

The options listed above are the default values. The "turbo" option uses "stale=ok" when fetching cache keys. This yields a performance improvement, at the expense of the possibility of fetching an older version of the cache value. 

## Benchmarking

In test/benchmark.js, there is a script which calculates the average time to fetch a cache key. Here is how the results stack up:

1) From my machine to hosted CouchDB (Cloudant) over the internet - 158ms
2) From my machine to local CouchDB - 24ms
3) From a server to dedicated CouchDB (Cloudant) in same data centre - 



