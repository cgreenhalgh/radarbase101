# Extending

## install process

uses 
[bin/radar-docker](https://github.com/RADAR-base/RADAR-Docker/blob/master/dcompose-stack/radar-cp-hadoop-stack/bin/radar-docker)
which runs
[lib/perform-install.sh](https://github.com/RADAR-base/RADAR-Docker/blob/master/dcompose-stack/radar-cp-hadoop-stack/lib/perform-install.sh)

this is turn does radar-docker run of kafka-init

(note this also uses list_raw.sh and list_aggregated.sh in kafka-init to obtain topics names for hdfs to persist)

kafka-init is build in context images/radar-kafka-init. 
It also mounts ./etc/schema on /schema/conf and has environment variables for kafka, zookeeper and schema-registry processes.

kafka-init has jdk11, avro tools and unpacked versions of 
https://github.com/RADAR-base/RADAR-Schemas/releases/download/v${RADAR_SCHEMAS_VERSION}/radar-schemas-tools-${RADAR_SCHEMAS_VERSION}.tar.gz
and 
https://github.com/RADAR-base/RADAR-Schemas/archive/v${RADAR_SCHEMAS_VERSION}.tar.gz
the latter being full source of 
[RADAR-Schemas](https://github.com/RADAR-base/RADAR-Schemas)
and the former being build versions of the Java tools.

[topic_init.sh](https://github.com/RADAR-base/RADAR-Docker/blob/master/dcompose-stack/radar-cp-hadoop-stack/images/radar-kafka-init/topic_init.sh)
does `radar-schemas-tools create -p $KAFKA_NUM_PARTITIONS -r $KAFKA_NUM_REPLICATION -b $KAFKA_NUM_BROKERS "${KAFKA_ZOOKEEPER_CONNECT}" merged`
then `radar-schemas-tools register --force "${KAFKA_SCHEMA_REGISTRY}" merged`
(merged is a directory with all topics/schemas from build image overlaid with any from mounted volume ./etc/schema)

where KAFKA_ZOOKEEPER_CONNECT is zookeeper-1:2181,...
and KAFKA_SCHEMA_REGISTRY is http://schema-registry-1:8081
and ..._NUM_... depending on full/lite.

(init.sh is the entrypoint to kafka-init, and compiles all the avsc definitions into Java radar-schemas-commons-... for use by the radar-schemas-tools.)

merged is probably the catalogue root.
and there is probably a '-m TOPIC' option to limit activity.

Init.sh execs its arguments, so it should be possible to call radar-schemas-tools directly as args.
Main commands are 'validate', 'list', 'create' and 'register'

## Schemas

Confluence/kafka clients use AVRO to encode data for topics.
The AVRO data type schemas are stored in the (confluence) schema-registry, 
by default (and in RADAR) under the "subject"s `TOPIC-key` and `TOPIC-value`.

It seems to have full compatibility enabled, which means that a topic's schema
cannot change significantly (add optional values, only).

Internally (in kafka) I believe that schemas are identified by a (local)
schema-registry allocated ID. But it looks like the full schema definition can
be used externally (when data is put into the confluence/kafka REST gateway).

### schema API notes

registering schemas with the registry...
[docs](https://docs.confluent.io/1.0.1/schema-registry/docs/intro.html#quickstart)

Get IP from docker inspect of radar-cp-hadoop-stack_schema-registry-1_1
```
curl -X GET -i -H "Content-Type: application/vnd.schemaregistry.v1+json"     http:/
/192.168.176.3:8081/subjects
```

Gives TOPIC-key & TOPIC-value, e.g. "android_phone_contacts-key"

versions
```
curl -X GET -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    http://192.168.176.3:8081/subjects/android_phone_light-value/versions
```
e.g. `[1]`

```
curl -X GET -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    http://192.168.176.3:8081/subjects/android_phone_light-value/versions/1
```
e.g.
```
{"subject":"android_phone_light-value","version":1,"id":37,"schema":"{\"type\":\"record\",\"name\":\"PhoneLight\",\"namespace\":\"org.radarcns.passive.phone\",\"doc\":\"Data from the light sensor in luminous flux per unit area.\",\"fields\":[{\"name\":\"time\",\"type\":\"double\",\"doc\":\"Device timestamp in UTC (s).\"},{\"name\":\"timeReceived\",\"type\":\"double\",\"doc\":\"Device receiver timestamp in UTC (s).\"},{\"name\":\"light\",\"type\":\"float\","doc\":\"Illuminance (lx).\"}]}"}
```

config
```
curl -X GET -i -H "Content-Type: application/vnd.schemaregistry.v1+json" \
    http://192.168.176.3:8081/config
```

