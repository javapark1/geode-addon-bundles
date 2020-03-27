# Bundle: mysql

The `mysql` bundle is preconfigured to synchronize Geode/GemFire with MySQL running on localhost. It includes the `mysql` cluster and `perf_test_mysql` app to quickly start up a Gedoe/GemFire cluster and run the client app to write/read to/from Geode/GemFire and MySQL.

## Installing Bundle

```console
install_bundle -download bundle-geode-1.11.0-cluster-app-mysql-perf_test_mysql.tar.gz
```

## Use Case

In this use case, we synchronize the select Geode/GemFire regions with the MySQL database tables. The `CacheWriterLoaderPkDbImpl` plugin synchronizes the database tables with all region read and write operations, i.e., `get`, `getAll`, `put`, `putAll`, `destroy`, and `remove`.

![DB Sync Diagram](/images/db-sync.png)

## Configuring MySQL

The 'mysql' cluster has been preconfigured to connect to MySQL on localhost with the user name `root` and the password `MySql123`. Change the user name/password in `etc/hibernate.cfg-mysql.xml`.

```console
switch_cluster mysql
vi etc/hibernate.cfg-mysql.xml
```

### `nw` Schema

Open MySQL Workbench and create the `nw` schema. When you run the `test_group` script (see below), the `customers` and `orders` tables will automatically be created by Hibernate invoked by the `CacheWriterLoaderPkDbImpl` plugin.

## Building `perf_test_mysql`

You need to download the MySQL binary files by building `perf_test_mysql` as follows.

```console
cd_app perf_test_mysql; cd bin_sh
./build_app
```

## Running Cluster

First, add a locator and members to the `mysql` cluster. All bundles come without locators and members.

```console
add_locator
add_member
add_member
```

Run the cluster.

```console
start_cluster
```

## Running `test_group`

The `test_group` script creates mock data for `Customer` and `Order` objects and ingests them into the Geode/GemFire cluster which in turn writes to MySQL via the `CacheWriterLoaderPkDbImpl` plugin included in the `geode-addon` distribution. The same plugin is also registered to retrieve data from MySQL for cache misses in the cluster.

```console
cd_app perf_test_mysql; cd bin_sh
./test_group -prop ../etc/group-factory.properties  -run
```
