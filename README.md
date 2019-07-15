# RADAR-BASE "101" 

i.e. some initial explorations.

Chris Greenhalgh, The University of Nottingham, 2019

## RADAR-BASE

[Main site](https://radar-base.org/)

## Install

[how to install](https://radar-base.org/index.php/2019/02/13/how-to-install-radar-base-using-radar-docker/)

pre-reqs:
- linux server
- docker, docker-compose, git
- also java for keytool

Ports:
- 80 - http (webserver)
- 443 - https (webserver, if enabled)

see [Vagrantfile](Vagrantfile)

install steps
```
sudo usermod -G docker ${USER}
git clone https://github.com/RADAR-base/RADAR-Docker.git
cd RADAR-Docker/dcompose-stack/radar-cp-hadoop-stack/
cp etc/env.template .env
```
Edit .env - see docs. e.g. standalone VM test
- `ENABLE_HTTPS` no
- `FROM_EMAIL`...

```
cp etc/smtp.env.template etc/smtp.env
```
Edit & config (remove all config => SMTP server, not relay)
```
cp etc/radar-backend/radar.yml.template etc/radar-backend/radar.yml

mkdir -p /usr/local/var/lib/docker
sudo chown ${USER} /usr/local/var/lib/docker

bin/radar-docker install XXX
```
Note, don't just install with no arguments unless your machine is real powerful - 
instead try a minimal subset of services.

Preferably change the nginx (webserver) config so that will still start
without the extra services.


problems - see end of doc

## Concepts

The Management Portal is the main web UI.

See [concepts in management portal](https://radar-base.org/index.php/2019/02/13/concepts-in-management-portal/)

- `Project` = study
- `Subject` = participant
- `Source` = a data collection device, i.e. smartphone or tracker
- `Source Type` = type of Source, versioned via a `catalog`
- `Source Data` = individual data stream from a `Source Type`

Subjects are explicitly created in the Management Portal. 
Typically Sources (individual devices) are also explicitly created in the Portal, associated with specific Subjects, and a QRCode reading step is used in the Subject's smartphone app to link to the specific device.

There are currently two main smartphone apps,
- `pRMT`, which monitors smartphone sensors and also acts as a bridge to other sensor devices (communicated with over Bluetooth?!)
- `aRMT`, which handles questionnaires (explicit user reports)

Authentication seems to be token based; 
tokens seem to be issued by management portal (only).


## Components

NB "cp" = "Confluent Platform" - built on Kafka

Web front-end
nginx:1.14.0-alpine, ports 80, 443, radar-cp-hadoop-stack_webserver_1, 
config in etc/webserver/....conf
- /kafka/ -> http://gateway/radar-gateway/ (various sub-paths denied - consumers, brokers, topics/...partition)
- /schema/ -> http://schema-registry-1:8081/ (get only?)
- /dashboard/ -> http://dashboard:80/
- /portainer/ -> http://portainer:9000/ (ip access controlled; includes websocket)
- /api/ -> http://rest-api:8080/api/ (w. CORS)
- /managementportal/ -> http://managementportal-app:8080/managementportal/ (w. CORS)
- /kafkamanager/ -> http://kafka-manager:9000 (basic auth kafka-manager.htpasswd)
- /redcapint/ (optional) -> http://radar-integration:8080/redcap/
- /rest-sources/authorizer/ (optional) -> http://radar-rest-sources-authorizer:80/
- /rest-sources/backend/ (optional) -> http://radar-rest-sources-backend:8080/

Zookeeper (dyn config), on zookeeper network
required by kafka
confluentinc/cp-zookeeper:4.1.0, radar-cp-hadoop-stack_zookeeper-1_1 ...-2_1 ...-3_1

Kafka, depends on Zookeeper; on zookeeper & kafka networks
confluentinc/cp-schema-registry:4.1.0, radar-cp-hadoop-stack_schema-registry-1_1

schema registry, depends on Kafka,
has SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL: zookeeper-1:2181
confluentinc/cp-kafka:4.1.0, radar-cp-hadoop-stack_kafka-1_1 ...-2_1 ...-3_1

Kafka (data collection?) REST API/proxy, depends on Kafka & schema resistry
has KAFKA_REST_ZOOKEEPER_CONNECT: zookeeper-1:2181...
required by gateway
confluentinc/cp-kafka-rest:4.1.0, radar-cp-hadoop-stack_rest-proxy-1_1

Kafka init, depends on Kafka, schema registry and zookeeper
build images/radar-kafka-init tagged with schemas version
has SCHEMAS_VERSION: ${RADAR_SCHEMAS_VERSION}
radarbase/kafka-init:0.4.3, radar-cp-hadoop-stack_kafka-init_1

(Backend) rest API - fronts data visualisation, e.g. dashboard ?!
depends on hotstorage, managementportal-app
required by dashboard
has RADAR_IS_CONFIG_LOCATION: usr/local/conf/radar/rest-api/radar-is.yml
> All the endpoints are secured and a valid user or a client registered with ManagementPortal with appropriate permissions/scopes can access the endpoints. For more details see [Management Portal] and [radar-auth]
> For accessing the end-points of this API, you will need JWT tokens from the [Management Portal] 
[git](https://github.com/RADAR-base/RADAR-RestApi)
Definitely extracts data from Mongo.
radarbase/radar-restapi:0.3, radar-cp-hadoop-stack_rest-api_1
See also [radar-commons](https://github.com/RADAR-base/radar-commons) ? 

Dashboard (visualisation ex?), depends on rest-api
> An Angular and D3 web application to manage and monitor research data from the RADAR-base Platform.
[git](https://github.com/RADAR-base/RADAR-Dashboard)
radarcns/radar-dashboard:2.1.0, radar-cp-hadoop-stack_dashboard_1

SMTP server/relay, used from backend monitor (to report compliance, etc.) and management portal
namshi/smtp:latest, radar-cp-hadoop-stack_smtp_1

"hot" storage connector, depends on zookeeper, kafka, schema-registry, rest-proxy, kafka-init and hotstorage
> The MongoDB-Sink-Connector is a Kafka-Connector for scalable and reliable data streaming from a Kafka topic or number of Kafka topics to a MongoDB collection or number of MongoDB collections. It consumes Avro data from Kafka topics, converts them into Documents and inserts them into MongoDB collections.
[git](https://github.com/RADAR-base/MongoDb-Sink-Connector)
radarbase/kafka-connect-mongodb-sink:0.2.2, radar-cp-hadoop-stack_radar-mongodb-connector_1

"Hot" storage, mongodb
required by rest api and hot storage connector
radarbase/radar-hotstorage:0.1, radar-cp-hadoop-stack_hotstorage_1 

"Cold" storage/HDFS connector, depends on zookeeper, kafka, schema-registry, rest-proxy, kafka-init and hdfs
[git](https://github.com/RADAR-base/RADAR-HDFS-Sink-Connector)
> The connector periodically polls data from Kafka and writes them to HDFS. The data from each Kafka topic is partitioned by the provided partitioner and divided into chunks. Each chunk of data is represented as an HDFS file with topic, kafka partition, start and end offsets of this data chunk in the filename. 
[parent docs](https://docs.confluent.io/current/connect/kafka-connect-hdfs/index.html)
radarbase/radar-connect-hdfs-sink:0.2.0, radar-cp-hadoop-stack_radar-hdfs-connector_1

"cold" storage = HDFS
radarbase/hdfs:3.0.3-alpine, radar-cp-hadoop-stack_hdfs-namenode-1_1
radarbase/hdfs:3.0.3-alpine, radar-cp-hadoop-stack_hdfs-datanode-1_1 ...-2_1 ...-3_1

"Backend" streams, depends on zookeeper, kafka, schema registry, kafka-init
radarbase/radar-backend:0.4.0, radar-cp-hadoop-stack_radar-backend-stream_1
> RADAR-backend implements Kafka Streams, to compute aggregated values of published data based on various time windows and to monitor data quality and compliance. The aggregated values are stored on separate topics and consumed by a sink-connector.
[docs](https://radar-base.org/index.php/documentation/introduction/concepts-and-components-of-radar-base/)
[git](https://github.com/RADAR-base/RADAR-Backend)

"Backend" monitor, depends on zookeeper, kafka, schema registry, kafka-init, smtp
Same codebase/repo as Backend streams.
radarbase/radar-backend:0.4.0 radar-cp-hadoop-stack_radar-backend-monitor_1
> The monitors are used to validate data quality and compliance and to provide email alerts for early interventions. For example, a battery level monitor is provided, which alerts the user/researcher by sending an email notification when battery level is found low according to given criteria.
[docs](https://radar-base.org/index.php/documentation/introduction/concepts-and-components-of-radar-base/)
[git](https://github.com/RADAR-base/RADAR-Backend)

Management portal (interfaces to "Metadata roles" - postgres??)
depends on radarbase-postgresql, smtp, catalog-server
[git](https://github.com/RADAR-base/ManagementPortal)
[docs](https://radar-base.github.io/ManagementPortal/)
There is discussion of oauth to access portal, but not entirely clear on use case (app??) or relation to gateway
radarbase/management-portal:0.5.3, radar-cp-hadoop-stack_managementportal-app_1

RDBMS (postgres) - management portal persistence, build ./images/postgres
postgres with single account
radarbase/postgres:10.6-alpine-1, radar-cp-hadoop-stack_radarbase-postgresql_1

Kafka manager (generic web UI for kafka), depends on zookeeper, kafka
Source from https://github.com/yahoo/kafka-manager/
radarbase/kafka-manager:1.3.3.18, radar-cp-hadoop-stack_kafka-manager_1

(external) Gateway, depends on (generic kafka) rest-proxy
adds authentication
> Gateway to the Confluent Kafka REST Proxy. It does the authentication and authorization, content validation and decompression if needed. 
> The RADAR-Auth library is used for authentication and authorization of users. 
[git](https://github.com/RADAR-base/RADAR-Gateway)
Note, radar-auth is in [management portal git](https://github.com/RADAR-base/ManagementPortal/tree/dev/radar-auth)
radarbase/radar-gateway:0.3.3, radar-cp-hadoop-stack_gateway_1

Catalog = access to schema information ?!
has zookeeper and schema-registry URLs (but does not depend on them)
build context: images/radar-kafka-init
command: radar-schemas-tools serve /schema/merged
volume ./etc/schema
see https://github.com/RADAR-base/RADAR-Schemas
radarbase/kafka-init:0.4.3, radar-cp-hadoop-stack_catalog-server_1

Generic docker management web UI
has access to docker socket
portainer/portainer:1.19.1, radar-cp-hadoop-stack_portainer_1

Optional
Fitbit rest connect 
[git](https://github.com/RADAR-base/RADAR-REST-Connector)

### Process list

Processes:
- nginx:1.14.0-alpine, ports 80, 443, radar-cp-hadoop-stack_webserver_1, config in etc/webserver/....conf
- radarbase/radar-connect-hdfs-sink:0.2.0, radar-cp-hadoop-stack_radar-hdfs-connector_1
- radarcns/radar-dashboard:2.1.0, radar-cp-hadoop-stack_dashboard_1
- radarbase/radar-restapi:0.3, radar-cp-hadoop-stack_rest-api_1
- radarbase/hdfs:3.0.3-alpine, radar-cp-hadoop-stack_hdfs-datanode-1_1 ...-2_1 ...-3_1
- radarbase/radar-gateway:0.3.3, radar-cp-hadoop-stack_gateway_1
- radarbase/kafka-connect-mongodb-sink:0.2.2, radar-cp-hadoop-stack_radar-mongodb-connector_1
- radarbase/radar-backend:0.4.0, radar-cp-hadoop-stack_radar-backend-stream_1
- radarbase/radar-backend:0.4.0 radar-cp-hadoop-stack_radar-backend-monitor_1
- radarbase/management-portal:0.5.3, radar-cp-hadoop-stack_managementportal-app_1
- radarbase/hdfs:3.0.3-alpine, radar-cp-hadoop-stack_hdfs-namenode-1_1
- radarbase/radar-hotstorage:0.1, radar-cp-hadoop-stack_hotstorage_1
- confluentinc/cp-kafka-rest:4.1.0, radar-cp-hadoop-stack_rest-proxy-1_1
- radarbase/kafka-manager:1.3.3.18, radar-cp-hadoop-stack_kafka-manager_1
- radarbase/kafka-init:0.4.3, radar-cp-hadoop-stack_kafka-init_1
- namshi/smtp:latest, radar-cp-hadoop-stack_smtp_1
- radarbase/kafka-init:0.4.3, radar-cp-hadoop-stack_catalog-server_1
- radarbase/postgres:10.6-alpine-1, radar-cp-hadoop-stack_radarbase-postgresql_1
- confluentinc/cp-schema-registry:4.1.0, radar-cp-hadoop-stack_schema-registry-1_1
- confluentinc/cp-kafka:4.1.0, radar-cp-hadoop-stack_kafka-1_1 ...-2_1 ...-3_1
- confluentinc/cp-zookeeper:4.1.0, radar-cp-hadoop-stack_zookeeper-1_1 ...-2_1 ...-3_1
- portainer/portainer:1.19.1, radar-cp-hadoop-stack_portainer_1



## install problems

### Problem 1

bin/radar-docker install
```
...
Compiling schemas...
===> Independent schemas:
merged/commons/active/notification/notification.avsc
...
===> Dependent schemas:
merged/commons/active/thincit/thinc_it_code_breaker.avsc
merged/commons/active/thincit/thinc_it_pdq.avsc
merged/commons/active/thincit/thinc_it_spotter.avsc
merged/commons/active/thincit/thinc_it_symbol_check.avsc
merged/commons/connector/fitbit/fitbit_activity_log_record.avsc
merged/commons/stream/aggregator/aggregate_list.avsc
### FAILURE ###
```
running
/RADAR-Docker/dcompose-stack/radar-cp-hadoop-stack/bin/radar-docker install
calls
/RADAR-Docker/dcompose-stack/radar-cp-hadoop-stack/lib/perform-install.sh
on stage '==> Setting up topics' ("initializing kafka)
sudo-linux bin/radar-docker run --rm kafka-init

exits with status 1
context: images/radar-kafka-init

bash-4.4# java -jar "${AVRO_TOOLS}" compile -string schema ${INDEPENDENT} ${DEPE
NDENT} java/src
Error: Invalid or corrupt jarfile /usr/share/java/avro-tools.jar
bash-4.4# file /usr/share/java/avro-tools.jar
bash: file: command not found
bash-4.4# ls -l /usr/share/java/avro-tools.jar
-rw-r--r--    1 root     root           357 Jul 15 08:36 /usr/share/java/avro-tools.jar

=>
<p>The requested URL /avro/avro-1.8.2/java/avro-tools-1.8.2.jar was not found on this server.</p>

from Dockerfile 
```
RUN curl -#o /usr/share/java/avro-tools.jar \
 "$(curl -s http://www.apache.org/dyn/closer.cgi/avro/\?as_json \
  | jq --raw-output ".preferred")avro/avro-1.8.2/java/avro-tools-1.8.2.jar"
```

Workaround, in /RADAR-Docker/dcompose-stack/radar-cp-hadoop-stack/images/radar-kafka-init/Dockerfile
```
RUN curl -#o /usr/share/java/avro-tools.jar \
  http://archive.apache.org/dist/avro/avro-1.8.2/java/avro-tools-1.8.2.jar
```

### Problem 2

```
Databases created
Admin password for ManagementPortal is not set in .env.
Password:
--> Generating keystore to hold EC keypair for JWT signing
bin/keystore-init: line 17: keytool: command not found
--> Generating keystore to hold RSA keypair for JWT signing
bin/keystore-init: line 28: keytool: command not found
chmod: cannot access 'etc/managementportal/config/keystore.p12': No such file or
 directory
### FAILURE ###
### FAILURE ###
```
bin/radar-docker install
calls lib/perform-install.sh
calls bin/keystore-init

bin/keystore-init does indeed use keytool...

Install openjdk-8-jre-headless
```
sudo apt-get install -y openjdk-8-jre-headless
```

### Problem 3

Super-sluggish on a VM... can't do anything, pretty much

immediately after install:
```
### SUCCESS ###
vagrant@ubuntu-bionic:~/RADAR-Docker/dcompose-stack/radar-cp-hadoop-stack$ dock
er ps
CONTAINER ID        IMAGE                                        COMMAND
           CREATED              STATUS                                 PORTS
                                  NAMES
74aed96ac4bc        nginx:1.14.0-alpine                          "nginx -g 'daem
on of…"    About a minute ago   Up 4 seconds                           0.0.0.0:8
0->80/tcp, 0.0.0.0:443->443/tcp   radar-cp-hadoop-stack_webserver_1
0a5f7f22ce7e        radarbase/radar-connect-hdfs-sink:0.2.0      "/etc/confluent
/dock…"    About a minute ago   Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_radar-hdfs-connector_1
aaaadab454c4        radarcns/radar-dashboard:2.1.0               "./init.sh"
           About a minute ago   Up About a minute (healthy)
                                  radar-cp-hadoop-stack_dashboard_1
2562ebe02946        radarbase/radar-restapi:0.3                  "radar-restapi"
           About a minute ago   Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_rest-api_1
6db4866d7f5a        radarbase/hdfs:3.0.3-alpine                  "entrypoint.sh
datan…"    About a minute ago   Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_hdfs-datanode-2_1
c1051e00a598        radarbase/hdfs:3.0.3-alpine                  "entrypoint.sh
datan…"    About a minute ago   Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_hdfs-datanode-3_1
0bb2f2bbd62f        radarbase/hdfs:3.0.3-alpine                  "entrypoint.sh
datan…"    About a minute ago   Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_hdfs-datanode-1_1
62bec0f23e19        radarbase/radar-gateway:0.3.3                "radar-gateway
/etc/…"    2 minutes ago        Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_gateway_1
79bdadb3c186        radarbase/kafka-connect-mongodb-sink:0.2.2   "/etc/confluent
/dock…"    2 minutes ago        Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_radar-mongodb-connector_
1
5105641e6b38        radarbase/radar-backend:0.4.0                "radar-backend-
init …"    2 minutes ago        Up About a minute
                                  radar-cp-hadoop-stack_radar-backend-stream_1
97f0f6814aec        radarbase/radar-backend:0.4.0                "radar-backend-
init …"    2 minutes ago        Up About a minute
                                  radar-cp-hadoop-stack_radar-backend-monitor_1
e98c7d115681        radarbase/management-portal:0.5.3            "/bin/sh -c 'ec
ho \"T…"   2 minutes ago        Up About a minute (health: starting)   5701/udp,
 8080/tcp                         radar-cp-hadoop-stack_managementportal-app_1
207efd149f27        radarbase/hdfs:3.0.3-alpine                  "entrypoint.sh
namen…"    2 minutes ago        Up About a minute (health: starting)
                                  radar-cp-hadoop-stack_hdfs-namenode-1_1
34ad98cdf151        radarbase/radar-hotstorage:0.1               "/entrypoint.sh
 ./in…"    2 minutes ago        Up 2 minutes (healthy)
                                  radar-cp-hadoop-stack_hotstorage_1
6f0192d1bc66        confluentinc/cp-kafka-rest:4.1.0             "/etc/confluent
/dock…"    2 minutes ago        Up 2 minutes (health: starting)
                                  radar-cp-hadoop-stack_rest-proxy-1_1
1fa66348986c        radarbase/kafka-manager:1.3.3.18             "./entrypoint.s
h"         2 minutes ago        Up 17 seconds (health: starting)
                                  radar-cp-hadoop-stack_kafka-manager_1
78c28af5f778        radarbase/kafka-init:0.4.3                   "init.sh topic_
init.…"    2 minutes ago        Up 2 minutes
                                  radar-cp-hadoop-stack_kafka-init_1
63c9ce2485dd        namshi/smtp:latest                           "/bin/entrypoin
t.sh …"    2 minutes ago        Up 2 minutes                           25/tcp
                                  radar-cp-hadoop-stack_smtp_1
f6203949152a        portainer/portainer:1.19.1                   "/portainer --a
dmin-…"    2 minutes ago        Up 2 minutes
                                  radar-cp-hadoop-stack_portainer_1
cbfbf7dd88d9        radarbase/kafka-init:0.4.3                   "init.sh radar-
schem…"    2 minutes ago        Up 2 minutes (health: starting)
                                  radar-cp-hadoop-stack_catalog-server_1
7c340e8e5e9d        radarbase/postgres:10.6-alpine-1             "docker-entrypo
int.s…"    5 minutes ago        Up 5 minutes (healthy)
                                  radar-cp-hadoop-stack_radarbase-postgresql_1
d07f7ae8b635        confluentinc/cp-schema-registry:4.1.0        "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_schema-registry-1_1
aad4c469e11a        confluentinc/cp-kafka:4.1.0                  "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_kafka-3_1
ed3cfe035865        confluentinc/cp-kafka:4.1.0                  "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_kafka-2_1
1a2097c7fa4d        confluentinc/cp-kafka:4.1.0                  "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_kafka-1_1
70cf7e98c94b        confluentinc/cp-zookeeper:4.1.0              "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_zookeeper-2_1
04ef4750cfbe        confluentinc/cp-zookeeper:4.1.0              "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_zookeeper-1_1
032a8417dff9        confluentinc/cp-zookeeper:4.1.0              "/etc/confluent
/dock…"    About an hour ago    Up About an hour (healthy)
                                  radar-cp-hadoop-stack_zookeeper-3_1
```

Note, [confluent](https://docs.confluent.io/current/platform.html)
offer a commercial stack on kafka. 
The community verison includes the schema registry and associated support 
(building on apache AVRO serialisation).
The community version also includes a HTTP proxy for kafka, and some
standard "connectors" for other platforms. 

