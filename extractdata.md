# Extracting data

Using Kafka connect

## design

Each data source type maps to a kafka topic, so all values flowing through 
a topic should essentially be the same.

The key for each topic entry has
- `projectId` (string) Project identifier. Null if unknown or the user is not enrolled in a project.
- `userId` (string) User Identifier created during the enrolment.
- `sourceId` (string) Unique identifier associated with the source.

(aggregate key also has timeStart, timeEnd; and the is a bulk RecordSet (usage??))

Topics & paritions may be split up between different SinkTask workers.
Paritions sub-divide topics into independent streams.
How are partitions determined in RADAR? 
Not sure! hopefully by project and/or user and/or source.

File output...
- project (id = name?)
- user (id)
- source (id) => (n:1) source type ~> (n:1) topic (that's what the CSV output does - see below)
- DATE_HOUR.csv (existing CSV approach?!) 

Note sure what time they use for the filename - see below (time or timeReceived?).

Columns include (existing CSV - see below):
- key.projectId
- key.userId
- key.sourceId
- value.time
- value.timeReceived
- value... (other value elements)

## Developing

[developing](https://docs.confluent.io/current/connect/devguide.html#connect-developing-simple-connector)

[SinkTask docs](https://docs.confluent.io/current/connect/javadocs/index.html?org/apache/kafka/connect/sink/SinkTask.html)

[file example](https://github.com/apache/kafka/tree/trunk/connect/file/src/main/java/org/apache/kafka/connect/file)
part of core [kafka source](https://github.com/apache/kafka)
[FileStreamSinkConnector.java](https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSinkConnector.java)
and 
[FileStreamSinkTask.java](https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSinkTask.java)

Basically the SinkTasks - a small number of them - do all the work.
Connector just gives them essential configuration.
Kafka connect framework divides all topics-partitions between the available SinkTasks.
The 
[SinkTask](https://docs.confluent.io/current/connect/javadocs/index.html?org/apache/kafka/connect/sink/SinkTask.html)
 is `start`ed and `stop`ped.
Once started, assigned topic-partitions are `open`ed (later `closed).
[SinkRecord](https://docs.confluent.io/current/connect/javadocs/org/apache/kafka/connect/sink/SinkRecord.html)s
are then `put` to the task which is periodically `flush`ed.

A SinkRecords has a topic, kafkaPartition, kafkaOffset, timestamp, key, keySchema, value and valueSchema.

Note 
[confluent vs kafka versions](https://docs.confluent.io/current/installation/versions-interoperability.html)
- confluent 4.x = kafka 1.x
- confluent 5.x = kafka 2.x
But should all interoperate(?!).

Mostly looks like RADAR-BASE is confluent 4.1 = kafka 1.1 at the moment,
which should use Java 1.8 u31 or later (and scala 2.11).

## Existing connectors

### kafka  -> hdfs

see
[wiki hadoop](https://radar-base.atlassian.net/wiki/spaces/RAD/pages/49512463/Guide+to+RADAR+HDFS+Connector)

```
docker-compose -f docker-compose-lite.yml -f optional-services-lite.yml up -d radar-hdfs-connector 
```
uses image 
[radarbase/radar-connect-hdfs-sinl](https://hub.docker.com/r/radarbase/radar-connect-hdfs-sink)
from
[source](https://github.com/RADAR-base/RADAR-HDFS-Sink-Connector)
with 
[Dockerfile](https://github.com/RADAR-base/RADAR-HDFS-Sink-Connector/blob/master/Dockerfile)

Note, two stage build creates jar then copies into image
`confluentinc/cp-kafka-connect-base:5.0.0` and customises
[launch](https://github.com/RADAR-base/RADAR-HDFS-Sink-Connector/blob/master/src/main/docker/launch)
a bit (e.g. classpath, property file config).

This is essentially the standard connector host process, connnect-standalone,
with (standard) connector.class io.confluent.connect.hdfs.HdfsSinkConnector
and (custom) format.class org.radarcns.sink.hdfs.AvroFormatRadar
which writes both key & value (I think value only is the default).

Connector base image [source](https://github.com/confluentinc/cp-docker-images/)
and 
[Dockerfile](https://github.com/confluentinc/cp-docker-images/blob/5.3.1-post/debian/kafka-connect-base/Dockerfile)
(`COMPONENT`=`kafka-connect`).
[base image](https://github.com/confluentinc/cp-docker-images/blob/5.3.1-post/debian/base/Dockerfile)
is based on debian jessie with zulu openjpk 8.

### kafka -> mongo

[source](https://github.com/RADAR-base/MongoDb-Sink-Connector)

`connector.class`=`org.radarcns.connect.mongodb.MongoDbSinkConnector`
and
`record.converter.class`=`org.radarcns.connect.mongodb.serialization.RecordConverterFactory`

[developer guide](https://github.com/RADAR-base/MongoDb-Sink-Connector#developer-guide)

[Dockerfile](https://github.com/RADAR-base/MongoDb-Sink-Connector/blob/master/Dockerfile)
is essentially the same as the HDFS connector (but an older base image version).

### hdfs -> local

[docs](https://docs.confluent.io/2.0.0/connect/connect-hdfs/docs/index.html)
says the default file output is something like 
`/topics/t1/partition=0/test_hdfs+0+0000000000+0000000002.avro`.
"The file name is encoded as `topic+partition+start_offset+end_offset.format`.

Avro tools can convert this to json, e.g. 
`hadoop jar avro-tools-1.7.7.jar tojson /topics/test_hdfs/partition=0/test_hdfs+0+0000000000+0000000002.avro`

From the wikiL
```
To extract data from HDFS
Create a directory to collect written data

sudo mkdir <dir-name>
Change current directory of command line to that directory

cd <dir-name>
Extract data from HDFS

hadoop fs -get /topics
```

### CSV extract

see
[post](https://radar-base.org/index.php/2019/02/14/accessing-the-data-collected-using-radar-base/)

`study->participant->data topics->date_hour.csv`

"data extraction is scheduled to be run every hour."

[csv format](https://radar-base.atlassian.net/wiki/spaces/RAD/pages/491880449/Data+Extraction)
essential CSV with (named) columns 
- key.projectId
- key.userId
- key.sourceId
- value.time
- value.timeReceived
- value... (other value elements)
