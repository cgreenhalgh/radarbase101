# Extending

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

