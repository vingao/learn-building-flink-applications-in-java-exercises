# tutorial
`
https://developer.confluent.io/courses/flink-java/get-started-exercise/
`

# code repo
```bash
https://github.com/vingao/learn-building-flink-applications-in-java-exercises
/Users/a202898/workspace/DEP/PLAYGROUND/FlinkProjects/learn-building-flink-applications-in-java-exercises
```

# Create a New Environment
```bash

## login to confluent cloud web
https://confluent.cloud/environments

## command line login
confluent login --save
Email: ******003@gmail.com

confluent environment create building-flink-applications-in-java
confluent environment list
```
```bash
       ID      |                Name                  
---------------+--------------------------------------
    env-xr65qx | default                              
  * env-dgw817 | apache-flink-101                     
    env-dgwnmo | building-flink-applications-in-java  
```
confluent environment use env-dgwnmo

# Run Flink Cluster and Taskmanger Locally
```bash
# Note:  Flink only works with java8 & 11;  Make sure JAVA_HOME is set to java 11 before running the following:
itermocil local-tiles

http://localhost:8081
```

# Execute DataGeneratorJob 
```bash
cd exercises
mvn clean package

# Note: need to disconnect from VPN
../flink-1.17.1/bin/flink run target/travel-itinerary-0.1.jar

view messages from skyone topic on https://confluent.cloud/environments/env-dgwnmo/clusters/lkc-r020op/topics/skyone/message-viewer
```

# Creating a Flink Kafka Data Source and a Log Sink
- Execute DataGeneratorJob as main entry point for the application (same as above) 
- Execute FlightImporterJob specifically:  
```bash
../flink-1.17.1/bin/flink run -c flightimporter.FlightImporterJob target/travel-itinerary-0.1.jar

## if FlightImporterJob produces to flightdata topic:
view messages from flightdata topic on https://confluent.cloud/environments/env-dgwnmo/clusters/lkc-r020op/topics/flightdata/message-viewer
```

### Cancel the job
```bash
../flink-1.17.1/bin/flink list
../flink-1.17.1/bin/flink cancel <jobID>
```

# sql client for local cluster
```bash
# Note:  For flink sql-client to work with flink's kafka connector, 
# 1. put the following jar under <flink_home/lib>: kafka-clients-3.6.0.jar and flink-sql-connector-kafka-1.17.1.jar
# 2. restart the flink cluster and sql-client
flink; ./sql-client.sh

CREATE TABLE flightdata_kafka (
  `emailAddress` STRING,
  `flightNumber` STRING,
  `departureAirportCode` STRING
) WITH (
  'connector' = 'kafka',
  'topic' = 'flightdata',
  'properties.group.id' = 'demoGroup',
  'scan.startup.mode' = 'earliest-offset',
  'properties.bootstrap.servers' = 'BOOTSTRAP_SERVER',
  'properties.security.protocol' = 'SASL_SSL',
  'properties.sasl.mechanism' = 'PLAIN',
  'properties.sasl.jaas.config' = 'org.apache.kafka.common.security.plain.PlainLoginModule required username="API_KEY" password="API_SECRET";',
  'value.format' = 'json',
  'sink.partitioner' = 'fixed'
);

select * from flightdata_kafka;
select count(*) total_departure, departureAirportCode from flightdata_kafka group by departureAirportCode;
select substring(flightNumber, 1, 3) airline, count(*) total from flightdata_kafka group by substring(flightNumber, 1, 3);
```