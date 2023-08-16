### Solr

The Apache Solr PHP extension is an extremely fast, light-weight, feature-rich library that allows PHP applications to communicate easily and efficiently with Apache Solr server instances using an object-oriented API.

If this module is not in your stack, install it manually following these steps:

-   Install and setup required system packages:
    
```
apt-get update
apt-get install vi build-essential libtool autoconf pkg-config unzip wget libcurl4-openssl-dev libxml2-dev
ln -s /usr/include/x86_64-linux-gnu/curl /usr/include/curl
```
    
-   Download the latest source code from the [web page](http://pecl.php.net/package/solr) and uncompress it:
    
```
wget http://pecl.php.net/get/solr-2.6.0.tgz
tar xzf solr-2.6.0.tgz
cd solr-2.6.0
```
    
-   Compile the module:
    
    ```
    phpize
    ./configure --enable-solr
    make
    make install
    ```
    
-   Enable the module in the _php.ini_ file:
    
```
...
vi /opt/bitnami/php/etc/php.ini
extension=solr.so
...
```

The option to create a new core in the Admin web page that you are using is a bit misleading, that option assumes you already have folders with some of the required data.

If you have access to the terminal on the machine where Solr is running you can easily create a core with a command like this:

```

$ solr create -c moodle
```

ab 77 hinter Erklärung dir-option

```
<lib dir="${solr.install.dir:../../../..}/modules/extraction/lib" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/modules/clustering/lib/" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/modules/langid/lib/" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/modules/ltr/lib/" regex=".*\.jar" />
<lib dir="${solr.install.dir:../../../..}/modules/scripting/lib/" regex=".*\.jar" />
```

ab 802 hinter Solr Cell Update Request Handler

```
  <requestHandler name="/update/extract"
                  startup="lazy"
                  class="solr.extraction.ExtractingRequestHandler" >
    <lst name="defaults">
      <str name="lowernames">true</str>
      <str name="uprefix">ignored_</str>

      <!-- capture link hrefs but ignore div attributes -->
      <str name="captureAttr">true</str>
      <str name="fmap.a">links</str>
      <str name="fmap.div">ignored_</str>
    </lst>
  </requestHandler>
```

  

```bash
curl 'http://iqn.bwsa.de:8983/solr/test/update/extract?literal.id=doc1&commit=true' -F "myfile=solr-word.pdf"
```