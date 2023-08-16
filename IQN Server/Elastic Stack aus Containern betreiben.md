Serverlogs wachsen schnell über den Kopf, vor allem bei im großen Maßstab ausgerollten Applikationen. Der Elastic Stack bietet ausgereifte Komponenten für das Einsammeln und zentrale Auswerten von Logmessages, in dessen Zentrum die Big Data-Suchmaschine Elasticsearch steht. Dieser Artikel zeigt, wie Sie sich eine einfache Logshipping-Pipeline mit Elastic-Elementen aus Docker-Containern heraus aufsetzen.

&lt;span data-mce-type='bookmark' style='display: inline-block; width: 0px; overflow: hidden; line-height: 0;' class='mce_SELRES_start'&gt;

Logmeldungen sind kostbare Datensammlungen für die Fehleranalyse, Funktions- und Zugriffskontrolle von Computersystemen. Auf einem einzelnen Linux-Server gibt es bereits eine ganze Reihe von Log-Erzeugern: Eine daraufbetriebene Applikation gibt normalerweise laufend Statusmeldungen aus, die eingesetzte Middleware tut das, und das Betriebssystem selber mit Kernel und Services auch. Bei einer einfachen WordPress-Installation genügen beispielsweise einfache Administrations-Bordmittel wie _grep_, _cut_ und _tail_ eventuell noch, um die bereits hier aufkommende, nicht geringe Menge an Logdaten zu bewältigen. Bei im großen Maßstab und auf vielen Knoten ausgerollten Applikationen ist es allerdings nicht mehr möglich, sich etwa bei einem Systemausfall der Reihe nach auf einzelnen Maschinen einzuloggen und dort unter Zeitdruck, mit Texttools nach relevanten Logmeldungen zu suchen. Anstatt sich Werkzeuge für das zentrale Einsammeln und Auswerten von Logs selber zu basteln, kommen für die Log-Aggregation heutzutage professionelle Lösungen zum Einsatz, die auf Enterprise-Niveau arbeiten können und zu denen der [Elastic Stack](https://www.elastic.co/de/products/) gehört.

## Elastic Stack

Der Elastic Stack (auch „ELK-Stack“) ist eine Sammlung von einzelnen Komponenten für die Logverarbeitung, in deren Mittelpunkt die hochperformante Suchmaschine Elasticsearch steht. Sie ist in Java implementiert, setzt auf die Volltextsuche-Bibliothek [Apache Lucene](https://lucene.apache.org/) auf, und ist für ein massives Datenaufkommen im Bereich Big Data ausgelegt. Zusammen mit der Visualisierung-Plattform Kibana bildet Elasticsearch ein potentes Gespann für die Analyse von Daten im NoSQL- beziehungsweise JSON-Format, deren Funktionalität auch über den speziellen Anwendungszweck der zentralen Sammlung und Auswertung von Server-Logdaten hinausgeht. Um eine Logshipping-Pipeline aufzusetzen, also Systemmeldungen von Servern zu einzusammeln und in Elasticsearch hineinzubekommen, gibt es im Elastic Stack weitere Komponenten.

Da gibt es zunächst einmal das mittlerweile in JRuby aufgesetzte [Logstash](https://www.elastic.co/products/logstash), das Sie als Einsammler von Logdaten (als „Shipper“) wie etwa auf Servern einsetzen können. Darübe rhinaus können Sie es mit seinen vielen eingebauten Filter-Plugins aber auch als zentralen „Indexer“ betreiben, um damit verschiedenartig strukturierte Logmessage-Formate aus unterschiedlichen Quellen zu harmonisieren und in Elasticsearch einheitlich zu integrieren. Logstash ist umfangreich und kann nahezu alles auswerten, was regelmäßige Meldungen absetzt. Dazu gehören beliebige Logdateien und -Ströme, Messaging-Systeme, Syslog-Meldungen, Twitter, aber auch Sensor- und Signaldaten wie etwa aus Kraftfahrzeugen und von Wetterstationen.

&lt;span data-mce-type='bookmark' style='display: inline-block; width: 0px; overflow: hidden; line-height: 0;' class='mce_SELRES_start'&gt;&lt;/span&gt;

Eine relativ neue Entwicklung im Elastic Stack sind die [sogenannten „Beats“](https://www.elastic.co/de/products/beats). Dabei handelt es sich um eine Gruppe von in Google Go aufgesetzten, leichtgewichtigen Logshippern, mit denen Sie auf Servern Logmessages direkt nachdem sie entstanden sind, einsammeln und an Logstash oder – auch ganz ohne Logstash-Instanz in der Pipeline – direkt an Elasticsearch weiterschicken können. Dazu gehört das hier im Folgenden besprochene Filebeat für Serverlogs, aber beispielsweise auch der Packetbeat für Netzwerkdaten und Metricbeat für Metriken, wobei einige Beats auch Aufgaben aus dem Bereich des Monitorings übernehmen. Der Trend tendiert dazu, dass die Beats weiter ausgebaut werden und immer mehr von der Funktionalität von Logstash übernehmen. So kann Filebeat zum Beispiel nun auch schon Multiline-Logmessages zusammenfassen. Zudem indizieren die eingebauten Module für bestimmte Standard-Applikationen wie Apache2, Nginx oder MongoDB Logmessages direkt, ohne dass Sie einen Logstash-Filter dafür aufsetzen müssen – allerdings auf Kosten der Flexibilität. Das Beats-Protokoll für den Transport von eingelesenen Logmessages im ELK-Stack ist robust und so wird die Übertragungsleistung zum Beispiel automatisch gedrosselt, wenn Kapazitätsengpässe aus der Pipeline zurückgemeldet werden, und es ist bei den Komponente dafür gesorgt, dass auch bei Hochlast keine Logmeldungen (in Elastic-Sprech „event messages“ genannt) verloren gehen.

Die Elastic Stack-Komponenten sind Open Source, sie stehen in ihrer einfachen Fassung unter der Apache 2.0-Lizenz und sind ohne Lizenzkosten einsetzbar – freilich ohne Anspruch auf Support vom Hersteller. Die aktuelle Versionsnummer von Elasticsearch (aktuell: 7.2) bestimmt immer die Version des gesamten Stacks mit seinen Komponenten. Es gibt allerdings einen gewissen Spielraum bei der Kombination von unterschiedlichen Versionen, so dass sie ein bestehendes Setup schrittweise updaten können.

**Listing 1: docker-compose.yml**

```java
version: '2.4'

services:
    nginx:
      image: nginx:latest
      ports:
        - "8080:80"
      volumes:
        - ./log:/var/log/nginx

    filebeat:
      image: docker.elastic.co/beats/filebeat:7.2.0
      volumes:
        - ./log/:/var/log/nginx
      command: >
        ./filebeat -e -c /etc/motd
        -E "filebeat.inputs=[{type:log,paths:['/var/log/nginx/access.log']}]"
        -E "output.logstash.hosts=['logstash:5044']"

    logstash:
      image: docker.elastic.co/logstash/logstash:7.2.0
      expose:
        - "5044"
      volumes:
        - ./logstash:/usr/share/logstash/pipeline

    elasticsearch:
      image: docker.elastic.co/elasticsearch/elasticsearch:7.2.0
      ports:
        - "9200:9200"
      environment:
        http.host: 0.0.0.0
        discovery.type: single-node
        ES_JAVA_OPTS: "-Xms750m -Xmx750m"

    kibana:
      image: docker.elastic.co/kibana/kibana:7.2.0
      ports:
        - "5601:5601"

```

## Beispiel-Setup

Ein einfaches Beispiel-Setup soll demonstrieren, wie Sie mit Filebeat und Logstash eine Logshipping-Pipeline aufsetzen können, um die Zugriffslogs eines Nginx-Webservers (_access.log_) in Elasticsearch hineinzubekommen und dort dann auf sie zugreifen können. Der Elastic Stack ist beim Hersteller in verschiedenen Formaten wie zum Beispiel DEB- und RPM-Paketen verfügbar, in diesem Artikel soll es aber darum gehen, wie Sie die Komponenten für eine solche Pipeline aus Docker-Containern heraus betreiben. Das Setup ist eine Konfigurationsdatei für [Docker Compose](https://docs.docker.com/compose/) (Listing 1), in dem alle beteiligten Services inklusive Nginx definiert sind. Sie können das gesamte Gebilde ganz einfach mit dem Befehl _docker-compose up -d_ hochziehen und in Betrieb nehmen.

Obwohl hier aus Platzgründen auf Kibana leider nicht ausführlich eingegangen werden kann, umfasst das Beispiel-Setup auch den Container dieser Auswertungs-Plattform. Die Elastic-Komponenten werden – falls lokal noch nicht vorhanden – beim Hochfahren aus der Container-Registry bei Elastic _docker.elastic.co_ gezogen. Der Nginx-Container kommt aus dem Docker Hub. Docker Compose setzt für den Betrieb des Verbandes praktischer Weise ein eigenes virtuelles Netzwerk auf, in welchem die Container die DNS-Namen tragen, die als Namen der _services_ in dem Compose-Sheet jeweils eingestellt sind.

Filebeat betreiben Sie grundsätzlich mit auf demselben Server wie die Applikation, deren Logs Sie damit einlesen wollen. Die anderen Komponenten würden Sie in der Praxis aber eher auf eigenen Knoten unterbringen. Logstash als eigenständigen Indexer einzusetzen, macht eigentlich nur dann richtig Sinn, wenn Sie heterogene Logdatenströme von einer Vielzahl von Shippern damit einsammeln und verarbeiten möchten; bei einem einzigen Beat und Logmessages in einem Standard-Format wie im Beispiel könnten Sie auch auf den Einsatz von Logstash verzichten. Wenn benötigt, und im größeren Maßstab mit einer großen Anzahl von Applikationsknoten in der Produktion würde es sich  sogar empfehlen, können Sie Zwecks der Lastverteilung und der Ausfallsicherheit, die Anzahl der Logstash-Instanzen erhöhen. Filebeat bietet für diesen Zweck extra einen eingebauten Load-Balancer für die Kommunikation mit vervielfältigten Logstash-Instanzen. Elasticsearch ist als Big Data-Applikation eigentlich für den Betrieb in einem Cluster gedacht, und mit Kibana würden Sie in diesem Fall auch von einer eigenen Instanz aus daraufzugreifen.

Nichtsdestotrotz handelt es sich bei dem Beispiel-Setup mit allen Komponenten auf demselben Rechner, um eine voll funktionstüchtige Elastic Stack-Pipeline, die sich für Experimente, für Testzwecke und die Entwicklung von Konfigurationsdateien ohne Weiteres eignet. Die vorgestellten Komponenten verhalten sich im kleineren Maßstab, wie hier im Beispiel aus Anwender-Perspektive, nicht anders als im größeren Rahmen. Auch der Umgang mit ihnen in Containern lässt sich so gut nachvollziehen.

**Listing 2: logstash.conf**

```java
input {
    beats {
        port => 5044 }
    }

filter {
    grok {
        match => {
	    "message" => '%{IPORHOST:clientip} - %{DATA:auth} \[%{HTTPDATE:timestamp}\] "%{WORD:verb} %{DATA:request} HTTP/%{NUMBER:httpversion}" %{NUMBER:response:int} %{NUMBER:size:int} "%{DATA:referrer}" "%{DATA:platform}"' }
		}
    date {
        match => ["timestamp", "dd/MMM/YYYY:HH:mm:ss Z"]
        remove_field => ["timestamp"] }
	}

output {
    file {
        path => "/tmp/events" }
	}

output {
    elasticsearch {
        hosts => ["elasticsearch:9200"]
	    index => "nginx-%{+YYYY.MM}" }
	}

```

## Filebeat

Der Filebeat-Container startet das Binary _/usr/share/filebeat/filebeat_ mit der YAML-Konfigurationsdatei _filebeat.yml_ im selben Verzeichnis. Wenn Sie den laufenden Container mit _docker exec -ti elk_filebeat_1 /bin/bash_ zu Wartungszwecken betreten, dann landen Sie direkt in diesem Folder. Per Standardeinstellung liest Filebeat keine Logs ein, aber ein Ausgangskanal zu Server mit dem Namen _elasticsearch_ und Port 9200 ist voreingestellt, und der Beat lässt immer nur eine Output-Methode zu. Sie können den Filebeat-Container ganz einfach umkonfigurieren, indem Sie auf dem Host eine eigene _filebeat.yml_-Konfigurationsdatei bereithalten, und diese beim Launch des Containers mit einen Volume-Mount über _/usr/share/filebeat/filebeat.yml_ drüberlegen. Eine einfache Konfiguration genügt für die Beispiel-Pipeline und sieht so aus:

```java
filebeat.inputs:
- type: log
  paths:
  - /var/log/nginx/access.log

output.logstash:
  hosts: ["logstash:5044"]

```

Als Eingang geben Sie die Logdatei an, die der Nginx-Server als Zugriffsprotokoll generiert und als Ausgangskanal die Instanz mit dem DNS-Namen _logstash_ auf Port 5044, welche der mit Docker Compose gestartete Filebeat-Service dann im Docker-Netzwerk vom Beispiel findet. Filebeat liest die Logdatei dann zunächst einmal komplett ein, überwacht sie danach weiter und nimmt jede neu darin auftauchende Zeile auf und transportiert sie an Logstash zur weiteren Verarbeitung. Sie können in der Pfadangabe auch Wildcards verwenden, um mehrere Dateien und ganze Verzeichnisse zu überwachen, und Filebeat kommt auch mit Logrotation zurecht.

Diese Konfiguration können Sie wie in Listing 1 aber auch mittels der Kommandozeilenoption _-E_ für _filebeat_ mit kondensierten YAML-Objekten beim Start des Containers mitgeben, damit ist ein Volume-Mounting von _filebeat.yml_ überflüssig. In dem Fall liest Filebeat allerdings zunächst die Default-Konfigurationsdatei ein, und der dort als erstes auftauchende Elasticsearch-Ausgangskanal gewinnt, und blockiert Logstash als Ausgangskanal. Als kleinen Workaround geben Sie mit -c als Konfigurationsdatei einfach eine leere Datei an, und im Beispiel hält _/etc/motd_ von CentOS im Filebeat-Container dafür her.

Bei aktuelleren Filebeat-Versionen gibt es eine [Autodiscover-Funktion für Docker-Container](https://www.elastic.co/guide/en/beats/filebeat/current/configuration-autodiscover.html), die praktisch ist und gut funktioniert. Sie können damit den Shipper so konfigurieren, dass er Ausschau hält, wann ein Container mit einem bestimmten Namen auf dem Server auftaucht, und dann automatisch damit beginnt, die Logs daraus einzulesen. Aufgenommen wird dabei immer die Default-Logausgabe eines Containers, die Sie selbst mit _docker logs_ abfragen können. Dieser Kanal transportiert den Inhalt der Linux-Ausgabekanäle STDOUT und STDERR, auf welche bei Containern gemeinhin die Logausgabe von den in ihnen betriebenen Applikationen umgelenkt ist. So ist im Nginx-Container _/var/log/nginx/access.log_ so auch ein Link zu _/dev/stdout_. Den Autodiscovery-Input für einen Container mit dem Namen „nginx“ konfigurieren Sie in Filebeat zum Beispiel folgendermaßen:

```java
filebeat.autodiscover:
  providers:
    - type: docker
      templates:
        - condition:
            equals.docker.container.image: nginx
          config:
             - type: docker
               enabled: true
               containers.ids:
                 - "${data.docker.container.id}"

```

Wenn Filebeat aber selbst auch in einem Container läuft, gibt es dabei allerdings ein Problem. Denn Sie müssen in dem Fall einige Docker-Internas (_/var/run/docker.sock_ und _/var/lib/docker/containers_) in den Filebeat-Container mounten, damit das Scannen nach anderen Containern auf dem Host aus dem eigenen Container heraus auch funktioniert. Darüber hinaus müssen Sie den Filebeat-Container mit _user=root_ laufen lassen, und _filebeat_ benötigt dann die Option _strict.perms=false_, wenn Sie den Shipper mit einer eigenen _filebeat.yml_-Datei konfigurieren. Es kann sein, das Ihnen das zu kompliziert ist, oder es in der Produktion aus Sicherheits- und Compliance-Gründen einfach nicht geht, die Applikation im Container mit Root-Rechten laufen zu lassen.

Es gibt allerdings aber auch noch den Weg wie im Beispiel, dass Sie zunächst ein Logverzeichnis auf dem Host anzulegen, dieses dann in den Nginx-Container Volume-mounten um _access.log_ darin als Datei schreiben zu lassen, und dasselbe Verzeichnis auch noch in den Filebeat-Container mounten, um die Logdatei daraus zu lesen. Der zusätzliche Vorteil dabei ist, dass die Logs dabei auf dem Host geschrieben werden, und damit auch über die Lebensdauer des Nginx-Containers hinaus zur Verfügung stehen. In Listing 1 dient als Transferordner _./log_ im Arbeitsverzeichnis (von dem aus Sie _docker-compose_ starten), das als Volume-Mount auf _/var/log/nginx_ im Nginx-Container (siehe unter _volumes_) die den voreingestellten Links auf STDOUT überlagert. Dieses Setup kommt also auch ohne Umkonfiguration des Nginx-Containers aus.

**Listing 3: Logstash-Funktionstests**

```java
$ cat log/access.log | tail -n 1
192.168.10.44 - - [04/Jun/2015:07:06:35 +0000] "GET /downloads/product_1 HTTP/1.1" 404 334 "-" "Debian APT-HTTP/1.3 (0.8.16~exp12ubuntu10.22)"

$ docker exec -ti elk_logstash_1 cat /tmp/events | wc -l
51462

$ docker exec -ti elk_logstash_1 cat /tmp/events | tail -n 1 | jq .
{ "@timestamp": "2015-06-04T07:06:35.000Z",
  "agent": {
    "type": "filebeat",
    "hostname": "52bf7d0b6b8d"
},
  "@version": "1",
  "input": {
    "type": "log"
  },
  "response": 404,
  "referrer": "-",
  "log": {
    "offset": 6991433,
    "file": {
      "path": "/var/log/nginx/access.log"
    }
  },
  "auth": "-",
  "request": "/downloads/product_1",
  "httpversion": "1.1",
  "host": {
    "name": "52bf7d0b6b8d"
  },
  "message": "192.168.10.44 - - [04/Jun/2015:07:06:35 +0000] \"GET /downloads/product_1 HTTP/1.1\" 404 334 \"-\" \"Debian APT-HTTP/1.3 (0.8.16~exp12ubuntu10.22)\"",
  "size": 334,
  "platform": "Debian APT-HTTP/1.3 (0.8.16~exp12ubuntu10.22)",
  "clientip": "192.168.10.44",
  "verb": "GET" }

```

## Logstash

Das Arbeitsverzeichnis von Logstash im Container ist _/usr/share/logstash_. Die Konfiguration erfolgt über das Verzeichnis _/usr/share/logstash/pipeline_, über das Sie ein auf dem Host bereitgestelltes Verzeichnis (im Beispiel _./logstash_) mit einer oder mehreren Konfigurationsdateien beim Launch des Containers als als Volume-mounten können. Logstash wird mit Ruby-artiger Syntax konfiguriert und benötigt mindestens einen Eingangs- und einen Ausgangskanal, aber auch mehrere sind möglich. Dazwischen können Sie nach Bedarf Transformationsfilter mit einem oder mehreren Filter-Plugins auf die eintrudelnden Eventmessages anwenden – das ist innerhalb von Logstash mit „Pipeline“ gemeint. Die Logstash-Konfiguration in Listing 2 definiert als Eingangskanal das Beats-Protokoll auf dem dafür vorgesehenen Port 5044, den Sie auch noch im Docker-Netzwerk exponieren müssen (siehe expose in Listing 1). Zwei Ausgangskanäle sind eingestellt.  Einmal zu Wartungszwecken die Datei _/tmp/events_ im Logstash-Container, und gleichzeitig den Elasticsearch-Server über Port 9200, der beim Beispiel-Setup unter dem DNS-Namen _elasticsearch_ als Docker-Service im Netzwerk läuft. Sie können die Elemente Ihrer Konfiguration auch auf mehrere Dateien, für den pipeline-Ordner, mit grundsätzlich beliebigen Dateinamen aufteilen, die Logstash dann beim Starten aneinanderfügt. Da die Reihenfolge der Elemente der Logstash-Pipeline (input-filter-output) einzuhalten ist, müssen Sie dabei selbst auf die richtige Sortierreihenfolge der Dateinamen achten, und deshalb Namen wie etwa _001-beats.conf_, _004-grok.conf_, _005-date.conf_ usw. verwenden, damit beim automatischen Zusammenfügen nicht etwa _filter_ vor _input_ landet.

In Listing 2 sind zwei der eingebauten Filter-Plugins von Logstash verwendet, und zwar [Grok](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html) und _date_. Der Grok-Filter ist dafür da, die einzelnen Logmessages beziehungsweise -Zeilen wie im Beispiel von Nginx (siehe die Rückgabe des ersten Befehls in Listing 3) zu strukturieren, weshalb beim Plugin praktische Regex-Konstanten wie _IPORHOST_ und _HTTPDATE_ zur Verfügung stehen. Damit können Sie die in den kompakten Logzeilen (im JSON-Dokument in Listing 3 unter _message_) steckenden Einzelinformationen extrahieren, also die IP-Adresse des jeweiligen Besuchers des Nginx-Webservers, welcher Endpunkt mit welcher Software angefragt worden ist usw. Diese isolierten Einzelinformationen fügt Grok dann zu der jeweiligen Eventmessage hinzu, damit Sie diese später in Elasticsearch gezielt abfragen können. Die neu dazugekommenen Datenfelder sind zum Beispiel _clientip_ und _request_ und _platform_, siehe die JSON-Eventmessage in Listing 3. Die Namen der Datenfelder sind dabei beliebig.

Die letzten beiden Funktionstests in Listing 3 (die Gesamtanzahl der verarbeiteten Logmessages und die letzte verarbeitete Logmessage ausgeben) greifen mit _docker exec cat_ auf die als zusätzlichen Ausgabekanal definierte Datei _/tmp/events_ im Logstash-Container zu. Zur anschaulichen Ausgabe von JSON-Dokumenten verwenden einige Beispiele in diesem Artikel das [sed-ähnliche Tool _jq_](https://stedolan.github.io/jq/), dass unter anderem eine formatierte Ausgabe liefert. Die von _docker-compose_ generierten Namen der Container lautet bei Ihnen eventuell anders, benutzen Sie die Umgebungsvariable _$COMPOSE_PROJECT_NAME_ um das Präfix der Namen des  
Projektes (im Beispiel „elk“) festzulegen.

Das Datenfeld _@timstamp_ in der Eventmessage ist von Filebeat erzeugt worden und trägt zunächst den Zeitstempel des Zeitpunkts, an welchem die Logmessage gelesen („harvested“) worden ist. Das ist aber meistens nicht erwünscht, denn Sie möchten in Elasticsearch und Kibana doch lieber angezeigt bekommen, was zu einem gewissen Zeitpunkt beim Betrieb einer Applikation vorgefallen ist. Das betrifft also den Zeitstempel für den Zeitpunkt, an dem die Logmessage generiert worden ist, und dieser Zeitstempel ist aus dem Feld _message_ verfügbar. Das Filter-Plugin date kann _@timestamp_ entsprechend anpassen, und dafür wird in Listing 2 das vorher mit dem Grok-Filter aus der Logmessage generierte Datenfeld _timestamp_ (ohne Klammeraffe) verwendet, welches vom selben Filter danach direkt wieder gelöscht wird (siehe im Beispiel _remove_field_), weil nicht länger benötigt. So steht in Listing 3 in _@timestamp_ nicht mehr der Zeitstempel, an dem die Eventmessage bei der Abfassung dieses Artikels in 2019 generiert worden ist, sondern ein Zeitpunkt im Jahr 2015, aus dem für diesen Artikel verarbeiteten historische Logdaten stammen. Gleichzeitig hat der Filter das Format des Zeitstempels auf das für _@timstamp_ in Elastic Stack gültige Format angepasst.

**Listing 4: Elasticsearch-Abfragen**

```java
$ curl -s 'http://localhost:9200/_cat/indices?v'
health status index                uuid                   docs.count store.size
yellow open   nginx-2015.06        2pBvQj58Qb2v8AfWV0K7_w       9422      4.1mb
yellow open   nginx-2015.05        ZRDny7rnThikLjpUrrhBMQ      42040     16.3mb
green  open   .kibana_task_manager GApLHdsKTRKbHD60pbExMQ          2     12.7kb
green  open   .kibana_1            PSR2sl1_RiWYuKUoart-2w          2     11.3kb

$ curl -s 'localhost:9200/_search?q=platform:ansible-httpget&_source=message' | jq .hits.hits
  "_index": "nginx-2015.05",
  "_type": "_doc",
  "_id": "gzpPIGwB_oZeSPL2lojQ",
  "_score": 27.762356,
  "_source": {
    "message": "192.168.25.10 - - [27/May/2015:03:05:14 +0000] \"GET /downloads/product_2 HTTP/1.1\" 200 17403440 \"-\" \"ansible-httpget\"" }

```

## Elasticsearch

Elasticsearch als Kernstück läuft genauso gut aus einem Container, wie die anderen Komponenten der gezeigten Logshipping-Pipline auch. Mehr als bei den in Golang geschriebenen Beats, die einzelne statische kompilierte Binaries sind und demnach keine Abhängigkeiten benötigten, spielen Docker-Container bei Java-Applikationen wie Elasticsearch ihre Stärke voll aus, indem über diesen Weg alles mitgeliefert wird, was Sie für den Betrieb benötigen. Den Port 9200 müssen Sie auf den Host durchschleifen (siehe Listing 1), damit Sie Elasticsearch über die eingebaute REST-API darüber vom Host aus abfragen können. Konfigurationseinstellungen für die Suchmaschine und das Java Runtime Environment stellen Sie als Umgebungsvariablen ein, die Sie mit _environment_ für den Launch des Containers beziehungsweise des Docker-Services mitgeben. Im Beispiel ist das die Freischaltung für Abfragen von beliebigen Adressen (_http.host_) und die Einstellung für den Betrieb als eingeschränkten Cluster mit nur einem Knoten (_single-node_). Wenn der Start des Elasticsearch-Containers wegen mangelnden Speicherplatzes abbricht ist es eventuell notwendig, dass Sie mit der Umgebungsvariable _$ES_JAVA_OPTS_ Anpassungen die Größe des Prozessspeichers im Container für die Java-Engine (im Beispiel mit Xms und Xmx) vornehmen. Geben Sie dem Container nach dem Start einen Moment Zeit um die Applikation vollständig hochzufahren, danach können Sie einen „Alive“-Test mit curl _http://localhost:9200_ durchführen, wodurch Informationen über den laufenden Cluster (im Beispiel mit nur einem Knoten) zurückgegeben werden.

Wie viele Logmeldungen in der Suchmaschine gelandet sind können Sie jederzeit einsehen, indem Sie die vorhandenen Indices abfragen (siehe _docs.count_ in Listing 4). Im Beispiel werden neben zwei von Kibana angelegten, intern verwendeten Indices (daran können Sie schon erkennen, dass Kibana sich erfolgreich verbunden hat) zwei weitere ausgegeben, nämlich _nginx-2015.05_ und _nginx-2015.06_. Diese beiden Indices sind vom Output von Logstash generiert worden (siehe in Listing 2 _index_ beim Elasticsearch-Outputkanal) und beinhalten Logmessages aus Mai und Juni 2015. Diese Indices sind mit Health-Status _yellow_ gekennzeichnet, was bedeutet das Replicas nicht alloziert werden konnten; das ist aber bei Single Cluster-Setups ganz normal, weil nun einmal keine anderen Knoten zum Replizieren von Daten zur Verfügung stehen. Sie können bei Logstash auch _YYYY.MM.dd_ einstellen, um die Indices nach den _Tagen_, an welchen Logmessages entstanden sind noch feiner zu granulieren. Oder Sie können etwa auch den Hostnamen des Servers verwenden, von welchem Messages herstammen (der wird unter _agent.hostname_ von Filebeat mitgeliefert, siehe in Listing 3). Für einen guten Überblick über die Logmessages in Kibana machen kleiner geschnittene Indices auf jeden Fall mehr Sinn, in Elasticsearch können Sie grundsätzlich aber über alle Indices abfragen, wie bei dem zweiten Kommando in Listing 4.

Diese Abfrage sucht unter allen gespeicherten JSON-Dokumenten mit Besuchen des Nginx-Webservers diejenigen mit „ansible-httpget“ als _platform_ (das ist der Identifier des Moduls _get_ur_l von Ansible), und es stellt sich heraus, dass es bei den 51462 geloggten Besuchen nur einen einzigen mit dieser Software gegeben hat. Elasticsearch gibt den gesamten Datensatz als umfangreiches JSON-Dokument zurück, die Ausgaben können Sie aber wie im Beispiel mit __source=message_ auf bestimmte Teile wie die abgespeicherte ursprüngliche Logmessage von Nginx verkürzen, und hier auch wieder _jq_ einsetzen. Ein Datensatz ist in Elasticsearch immer durch Index, Dokumententyp und ID genau bezeichnet, und wie das Beispiel zeigt wird als Typ per Default __doc_, und ein zufälliger Hashwert als __id_ bei Aufnahme in die Datenbestände der Suchmaschine vergeben.

Der Kibana-Container im Beispiel-Setup kommt völlig ohne zusätzliche Konfiguration aus. Nach dem Starten verbindet sich die Visualisierungs-Plattform automatisch mit dem im Netzwerk erreichbarer Server, beziehungsweise dem im virtuellen Netzwerk vorhandenen Docker-Service mit dem Namen „elasticsearch“ über Port 9200. Stellen Sie noch den Port 5601 aus dem Container durch auf den Host (Listing 1), und greifen Sie dann mit einem Webbrowser über _http://localhost:5601_ bequem auf Kibana, und darüber auf die in Elasticsearch aufbewahrten Daten zu (**Abbildung 1**).

##   
![](https://s3.eu-west-1.amazonaws.com/redsys-prod/articles/63ee4add5129e20604bf1b70/images/kibana.png)

Beim Elastic Stack handelt es es sich um ein Logshipping-System für den Transport von Logmessages in die Suchmaschine Elasticsearch plus Kibana, der auch mit einem sehr hohen Logaufkommen bei im großen Maßstab ausgerollten Applikationen zurechtkommt. Die Software agiert auf hohem Niveau und die beteiligten Komponenten sind ausgereift, sie lassen sich auf verschiedene Weisen kombinieren und so an den eigenen Bedarf anpassen.

Das hier vorstellte Beispiel ist eine einfache Variante des Elastic Stack, die sich gegenüber einem Setup in der Produktion unter anderem auch dadurch unterschiedet, dass die Kanäle für die Übertragung von Logs nicht verschlüsselt sind, und der Zugriff auf die Suchmaschine und Kibana ohne Authentifizierung möglich ist.

Docker-Container für den Betrieb von Elastic-Komponenten einzusetzen ist ein praktikabler Weg. Nur Filebeat bleibt dabei eher eingesperrt, so dass sich hierbei vielleicht eher ein Linux-Paket als Installations- und Betriebsgrundlage anbietet, oder Sie auch vielleicht das unkomplizierte Go-Binary selbst im Applikationscontainer einspielen und dort betreiben.