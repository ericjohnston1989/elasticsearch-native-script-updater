Native Script Updater Plugin for Elasticsearch
==============================================

This plugin define the native script "updater".

It can be used during update requests as an alternative to the dynamic scripts
documented here: http://www.elasticsearch.org/guide/reference/api/update/

Motives
-------
Here are the reasons why one would want to use this instead of the
standard update built-in with Elasticsearch:

* Security: it does not require dynamic scripting which is disabled by default
  since elasticsearch-1.2.0
* In a single execution all types of manipulations of the document are
  done
* Array manipulations are easier than MVEL scripts: arrays are
  merged, after a removal empty ancestors are deleted recursively.
* Performance is at least as good as MVEL: no wrapping/unwrapping we stay in pure
  compiled java land here.

Usage:
------
Here is a show-case example all the manipulations supported:
```
curl -XPOST 'localhost:9200/test/type1/1/_update' -d '{
  "script": "updater", "lang": "native",
  "params": {
    "doc" : {
      "name" : "First article",
      "my" : {
        "tags" : [ "elasticsearch", "search" ]
      },
      "address" : {
        "street" : "Trocadero",
        "city" : "Paris",
        "country": [ "France", "England" ]
      }
    },
    "remove": [ "address.street", "name" ],
    "set"   : { "something.tags" : [ "sport", "entertainment", "game" ] }
    "removeItems": { "address.country" : [ "England" ], "my.tags": ["foo"] },
    "appendItems": { "my.tags" : [ "elasticsearch" ] },
    "mergeItems"   : { "my.tags" : [ "bonsai", "search" ] }
  }
}'
```

`doc`: partial document to merge onto the existing document. Note that
arrays are also merged with existing arrays. Values are checked for
uniqueness before they are added to the existing arrays.

`remove`: a list of paths to the properties to be removed from the
`source` document. A single string path is also valid.

`set`: Sets path associated to values. Could be done using `doc`.
Required ancestor properties will be lazily created when they don't exist.

`removeItems`: Paths associated to list of values. The values are
removed from the document's corresponding arrays when the array exist.

`appendItems`: Paths associated to a list of values to append to the
existing values. Ancestor properties are lazily created if they don't
exist.

`mergeItems`: Same than `appendItems` but uniqueness of the values is
checked so existing values are not added again.

Installation
------------
```
bin/plugin -install native-script-updater --url https://github.com/sutoiku/elasticsearch-native-script-updater/releases/download/1.0.0/elasticsearch-native-script-updater-1.0.0.zip
```

When not to use it
------------------
When the built-in updates is enough, no need to use the script:
* merge a document where arrays overrides each other and no removals take place
It would work fine though.

Features not supported:
* Increment values and other scripted updates
* Return a the \_source or a set of fields in the response

License
-------
Same than Elasticsearch: ASL-2.0

Contributions
-------------
Contributions are welcome, please file issues and PR on Github; tests are
highly regarded.
