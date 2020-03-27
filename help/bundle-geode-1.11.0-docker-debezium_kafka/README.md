# Debezium-Kafka Geode/GemFire Connector

This bundle integrates Geode/GemFire with Debezium for ingesting initial data and CDC records from MySQL into a Geode/GemFire cluster via a Kafka sink connector included in the `geode-addon` distribution. It supports inserts, updates and deletes.

## Installing Bundle

```console
install_bundle -download bundle-imdg-3.12.6n4.0-docker-debezium_kafka.tar.gz
```

## Use Case

This use case ingests data changes made in the MySQL database into a Geode/GemFire cluster via Kafka connectors: the Debezium MySQL source connector and the `geode-addon` Debezium sink connector.

![Debezium-Kafka Diagram](/images/debezium-kafka.png)

## Required Software

- Docker
- Docker Compose
- Maven

## Optional Software

- jq

## Debezium Tutorial

The demo follows the Debezium Totorial steps shown in the link below.

https://debezium.io/documentation/reference/1.0/tutorial.html#registering-connector-monitor-inventory-database

All the commands provided in the tutorial are wrapped in the scripts found in the `bin_sh` directory. We'll use these scripts to simplify the demo.

## Building Demo

We must first build the demo by running the `build_app` command as shown below. This command compiles and packages the `VersionedPortable` data (domain) classes found in the source directory `src`. It also copies the Geode/GemFire and `geode-addon` jar files to the Docker container mounted volume in the `geode-addon` directory so that the Geode/GemFire Debezium Kafka connector can include them in its class path.

```console
cd_docker debezium_kafka; cd bin_sh
./build_app
```

Upon successful build, the `geode-addon` directory should have jar files similar to the following:

```console
cd_docker debezium_kafka
tree geode-addon
```

```console
geode-addon/
│   ├── etc
│   │   ├── cache.xml
│   │   ├── client-cache.xml
│   │   └── client-gemfire.properties
├── lib
│   ├── ...
│   ├── geode-addon-core-0.9.0-SNAPSHOT.jar
│   └── ...
├── log
└── plugins
    ├── debezium-demo-data-1.0-SNAPSHOT.jar
    └── geode-addon-core-0.9.0-SNAPSHOT-tests.jar
```


## Creating Geode/GemFire Docker Containers

Let's create a Geode/GemFire cluster to run on Docker containers as follows.

```console
create_docker -cluster geode -host host.docker.internal
cd_docker geode
```

If you are running Docker Desktop, then the host name, `host.docker.internal`, is accessible from the containers as well as the host machine. You can run the `ping` command to check the host name.

```console
ping host.docker.internal
```

If `host.docker.internal` is not defined then you will need to use the host IP address that can be accessed from both the Docker containers and the host machine. Run `create_docker -?` or `man create_docker` to see the usage.

```console
create_docker -?
```

If you are using a host IP other than `host.docker.internal` then you must also make the change in the Debezium Geode/GemFire connector configuration file as follows.

```console
cd_docker debezium_kafka
vi geode-addon/etc/client-cache.xml
```

Replace `host.docker.internal` in `geode-client.xml` with your host IP address.

```xml
<client-cache ...>
   ...
   <pool name="serverPool">
      <locator host="host.docker.internal" port="10334" />
   </pool>
   ...
</client-cache>
```

### Configuring Geode/GemFire Cluster

Copy the `cache.xml` and demo data jar files from the `debezium_kafaka` directory as follows.

```console
cd_docker geode

# Copy cache.xml
cp $GEODE_ADDON_WORKSPACE/docker/debezium_kafka/geode-addon/etc/cache.xml geode-addon/etc/

# Copy debezium-demo-data-1.0-SNAPSHOT.jar
cp $GEODE_ADDON_WORKSPACE/docker/debezium_kafka/geode-addon/plugins/debezium-demo-data-1.0-SNAPSHOT.jar geode-addon/plugins/
```

The `cache.xml` file defines the `inventory` regions that we will need for our demo and the`debezium-demo-data-1.0-SNAPSHOT.jar` file is needed for query services.

## Starting Docker Containers

There are numerous Docker containers to this demo. We'll first start the Geode/GemFire cluster containers and then proceed with the Debezium containers. For this demo, we intentionally made each container to run in the foreground so that you can view the log events. You will need to launch a total of eight (8) terminals.

### Start Geode/GemFire Containers

Start the `geode` Geode/GemFire cluster containers.

```console
cd_docker geode
docker-compose up
```

### Start Debezium Containers

Launch six (6) terminals and run each script from their own terminal as shown below. Each script must be run from their own terminal as they will block and display log messages.

```console
cd_docker debezium_kafka; cd bin_sh

# 1. Start Zookeeper
./start_zookeeper

# 2. Start Kafka
./start_kafka

# 3. Start MySQL database
./start_mysql

# 4. Start MySQL CLI
./start_mysql_cli

# 5. Start Kafka Connect
./start_kafka_connect

# 6. Start topic watcher
./watch_topic_customers
```

### Register Kafka Connect via REST API

There are two (2) Kafka connectors that we must register. The MySQL connector is provided by Debezium and the Geode/GemFire connector is part of the `geode-addon` distribution. 

```console
cd_docker debezium_kafka; cd bin_sh
./register_mysql_connector
./register_debezium_geode_connector
```

### Check Kafka Connect

```console
# Check status
curl -Ss -H "Accept:application/json" localhost:8083/ | jq

# List registered connectors 
curl -Ss -H "Accept:application/json" localhost:8083/connectors/ | jq
```

The last command should display the inventory connector that we registered previously.

```console
[
  "debezium-geode-sink",
  "inventory-connector"
]
```

### MySQL CLI

Using the MySQL CLI, you can change table contents. The changes you make will be captured in the form of change events by the Debezium source connector. The Geode/GemFire sink connector in turn receives the change events, transforms them into data objects and updates (or deletes) the assigned map, i.e., `inventory/customers`.

```console
use inventory;
SELECT * FROM customers;
UPDATE customers SET first_name='Anne Marie' WHERE id=1004;
-- Delete - First, delete the row in addresses that references the foreign key
DELETE FROM addresses WHERE customer_id=1004;
DELETE FROM customers WHERE id=1004;

INSERT INTO customers VALUES (default, "Sarah", "Thompson", "kitt@acme.com");
INSERT INTO customers VALUES (default, "Kenneth", "Anderson", "kander@acme.com");
```

### View Region Contents

To view the map contents, run the `read_cache` command as follows:

```console
./read_cache /inventory/customers
```

**Output:**

```console
Region Values [/inventory/customers]:
        [email=annek@noanswer.org, firstName=Anne Marie, id=1004, lastName=Kretchmar]
        [email=gbailey@foobar.com, firstName=George, id=1002, lastName=Bailey]
        [email=sally.thomas@acme.com, firstName=Sally, id=1001, lastName=Thomas]
        [email=ed@walker.com, firstName=Edward, id=1003, lastName=Walker]
```

### Browse Region Contents using `gfsh`

You can also use `gfsh` to browse region contents.

```console
docker container exec -it geode_locator_1 gfsh
```

Connect to the `geode` cluster and execute the query as shown below.

```console
gfsh>connect --locator=localhost[10334]
gfsh>query --query="select * from /inventory/customers order by id"
```

**Output:**

```console
Result : true
Limit  : 100
Rows   : 4

 id  |  firstName   |  lastName   | email
---- | ------------ | ----------- | -----------------------
1001 | "Sally"      | "Thomas"    | "sally.thomas@acme.com"
1002 | "George"     | "Bailey"    | "gbailey@foobar.com"
1003 | "Edward"     | "Walker"    | "ed@walker.com"
1004 | "Anne Marie" | "Kretchmar" | "annek@noanswer.org"
```

### Browse Region Contents from Pulse

Pulse URL: http://localhost:7070/pulse

![Pulse Screenshot](/images/pulse-inventory-customers.png)

## Tearing Down

```console
# Shutdown Debezium containers
cd_docker debezium_kafka; cd bin_sh
./cleanup

# Shutdown Geode/GemFire containers
cd_docker geode
docker-compose down

# Prune all stopped containers 
docker container prune
```
