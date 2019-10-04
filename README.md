# <img align="right" src="http://jquerymy.com/kod/photon-github.png" /> couch-photon
Photon is Futon-inspired CouchDB admin panel with all Futon and most of Fauxton features. Photon is a single CouchDB design document with attachments, so it can be installed on any CouchDB without re-building or reconfiguring CouchDB itself.

Basic Photon features are shown in a 12 minute screencast at [youtu.be/M9ptWXfwMN8](https://youtu.be/M9ptWXfwMN8).

Photon is completely self-contained and is ok for restricted networks. Unless configured [differently](#configuring-access), users must have `_admin` or `app-photon` role to use Photon; this restriction only protects the app, it doesn’t add more security to CouchDB itself.

## Unique features

* Multiple DB backup to external Couch, prefixed and filtered
* Group operations with DBs, ACLs, docs and sync tasks
* Full text search for view results and JSON docs
* JSON tree editor understanding JS syntax
* Text editor for text-based attachments
* Document revisions structured diff
* View editor with JS, Erlang and Mango support
* Mango queries memoization for re-use
* Multiple file upload and rename
* Node and cluster level config manager
* Node and cluster stats
* Log viewer with search, for 1.x
* Local docs list, for 2.x
* Docs purge, for 2.3+.

Photon installation is basically putting `_design/photon` JSON doc into CouchDB. There are 3 ways: [command line](#installation-using-curl), [copy/paste](#installation-using-copypaste) and [replication](#installation-using-replication).

## Installation using curl

1. Create `photon` database, provide credentials: `curl -H "Content-Type: application/json" -X PUT http://admin:____@127.0.0.1:5984/photon`
2. Download source code and put it into a design document, provide credentials: `curl https://raw.githubusercontent.com/ermouth/couch-photon/master/photon.json | curl -H "Content-Type: application/json" -X PUT http://admin:____@127.0.0.1:5984/photon/_design/photon -d @-`.
3. Open `http://127.0.0.1:5984/photon/_design/photon/index.html` in the browser. 

Next time you can upgrade Photon directly from Photon itself. Just click the rightmost button of navbar, and click `Check for updates`.

## Installation using copy/paste
Download `photon.json` from [Github](https://raw.githubusercontent.com/ermouth/couch-photon/master/photon.json) or [AWS S3 CDN](https://s3-eu-west-1.amazonaws.com/cdn.cloudwall.me/photon/photon.json) and use one of the following ways:

a) Open JSON in any text editor. Create a doc in any CouchDB bucket, using Futon or Fauxton. Copy-paste JSON text into it. Save. Run `index.html` attachment, clicking it in Futon or Fauxton, or directly typing smth like `yourcouchdomain.xyz/db_with_photon/_design/photon/index.html`.

b) `curl -H 'Content-Type: application/json' -X PUT http://yourdomain.com:5984/somedb/_design/photon -d @photon.json`. Run `index.html`.

Next time you can upgrade Photon directly from Photon itself. Just click the rightmost button of navbar, and click `Check for updates`.

## Installation using replication
You can install Photon using native CouchDB replication. Since DB you will replicate from is of very limited capacity, please only replicate once, do not make sync continuous.

__For CouchDB 1.7.2 and earlier.__ Create new doc in your `_replicator` DB and copy-paste below JSON into it. Save – and you are done.
```json
{
  "_id":      "Photon",
  "source":   "https://cloudwall.smileupps.com/photon1/",
  "target":   "photon",
  "create_target": true,
  "doc_ids":  ["_design/photon"],
  "user_ctx": {"name":"admin", "roles":["_admin"]}
}
```
To make sure `user_ctx` section works properly, you must have CouchDB proxy auth turned on. By default it’s active in CouchDB 1.x.

__For CouchDB 2.x__ JSON is bit different (see below). You need to insert credentials since 2.x does not understand `user_ctx` param.
```json
{
  "_id":      "Photon",
  "source":   "https://cloudwall.smileupps.com/photon1/",
  "target":   "http://admin:__________@localhost:5984/photon",
  "create_target": true,
  "doc_ids":  ["_design/photon"]
}
```

Next time you can upgrade Photon directly from Photon itself, without repliction. Just click the rightmost button of navbar, and click `Check for updates` button.

## Dedicated host

Photon can run as a couchapp from a dedicated domain. To set up Photon for a dedicated host, 
you only need to configure CouchDB properly. First, [set up CORS](https://cloudwall.me/setup_couch#h-16ylld74) 
for the domain. Then set up two config keys:
```
[vhosts] 
photon.mydomain.xyz = /photon/_design/photon/_rewrite
[httpd]
secure_rewrites = false
```
Now typing `photon.mydomain.xyz` in browser runs Photon.

## Configuring access

By default, Photon only starts if a user has `_admin` or `app-photon` role. Allowed roles are listed in `.settings.roles` branch of the Photon design document. You can edit this branch, save ddoc and reload Photon. 

Modified settings are preserved during Photon updates, if update performed using `Check for updates` button.

## Photon on 5986 port

Photon is ok for viewing a cluster node internals using 5986 port. However, there are two main difficulties: how to ensure the Photon design ddoc is distributed over all cluster nodes, and how to determine the name of local shard containing Photon.

To ensure distribution, `photon` bucket must be created using `q=1`, and `n` equal to number of nodes. For 3-node cluster it looks like `curl -H "Content-Type: application/json" -X PUT http://admin:____@127.0.0.1:5984/photon?q=1&n=3`.

Setting `q=1` creates only a single shard for `photon` bucket, which makes it easy to locate shard name in `http://127.0.0.1:5986/_all_dbs` response.

Resulting URL will look like `http://127.0.0.1:5986/shards%2F00000000-ffffffff%2Fphoton.1500000000/_design/photon/index.html`.

## CouchDB performance test

Since 1.9.22 Photon includes superficial CouchDB performance test, located in `perftest.html` attachment to Photon design doc. The test needs clear browser tab, run it opening `_design/photon` and clicking the attachment name.

The test provides good insights how q,n cluster params impact performance. Also the test shows JS query server cost, and how it depends on sharding options.

## FAQ

__Where are source files?__

Photon never existed as source _files_, its sources are CouchDB _docs_. Photon is developed and built using specialized dev environment, CloudWall. You can explore [Photon source code](https://cloudwall.me/_demo/#cw/Manifest/!WyJlZGl0IiwiY3ctUGhvdG9uLTFjY2QiXQ--) right in browser, using CloudWall demo built-in IDE.

__What is underlying technology?__

Photon employs most conservative and bullet-proof approaches whenever possible: jQuery plus established plugins to render UI, and XMLHttpRequest to interact with CouchDB. Photon contains no libs, originating from corporate OSS. 

__Is it safe to update from CDN?__

Default CDN has logging turned off, so requests and IPs are not collected. Update is always explicit and never performed until you click the update button. 

---

(c) 2019 ermouth. Photon is MIT licensed. Photon works in all modern browsers, IE11, and on iPad. 
