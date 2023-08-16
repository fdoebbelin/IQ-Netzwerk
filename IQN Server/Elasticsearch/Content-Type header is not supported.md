
```
Content-Type header [application/vnd.elasticsearch+json; compatible-with=8] is not supported
```

Fehler erscheint bei unterschiedlicher Client- und Serverversion


```
You shouldn't use the latest version (8.1.0) of the Elasticsearch client against a 6.x version of the Elasticsearch server. Instead you should use the same major version client (6.x) against the major version of Elasticsearch that you're using.
```
