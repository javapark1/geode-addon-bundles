# geode-addon-bundles

This repo contains readily runnable geode-addon bundles. It is an experimental repo and subject to change.

A geode-addon bundle is a tarball that includes all the necessary files to activate a Geode/GemFire environment in which you can start an IMDG cluster and/or run Geode/GemFire apps. You can find README.md that provides detailed installation, build, and execution instructions in the cluster and app directories.

To use this repo, you must first install `geode-addon`:

[https://github.com/javapark1/geode-addon](https://github.com/javapark1/geode-addon)

## Bundle Catalog - Use Cases

The following link provides a compiled list of bundle use cases.

**[Go To Bundle Catalog](Bundle-Catalog.md)**

## Bundle Naming Conventions

A bundle may contain configuration files pertaining to cluster, app or both. 

### Cluster

A cluster bundle contains only cluster files. It may contain more than one cluster.

```console
bundle-geode|gemfire-<version>-cluster-<cluster-name>[-bin].tar.gz
```
#### Cluster Example

```console
bundle-geode-1.11.0-cluster-mysql.tar.gz
```

### App

An app bundle contains only app files. It may contain more than one app.

```console
bundle-geode|gemfire-<version>-app-<app-name>.tar.gz
```

#### App Example

```console
bundle-geode-1.11.0-app-perf_test_mysql.tar.gz
```

### Cluster and App Combo

A cluster and app combo cluster contains both cluster and app files. It may contain multiple clusters and apps.

```console
bundle-geode|gemfire-<version>-cluster-app-<cluster-name>-<app-name>.tar.gz
``` 

#### Combo Example

```console
bundle-geode-1.11.0-cluster-app-mysql-perf_test_mysql.tar.gz
```

### Binary Bundles

The names of the bundles that include binary files end with `-bin.tar.gz`. A binary bundle includes a partial or complete set of binary files. If it contains a complete set of binary files then the build step is typically not required. See the `README.md` file included in the bundle for instructions.

:pushpin: Because the binary files tend to become obsolete over time, they may not always work especially when you have upgraded `geode-addon` or Geode/GemFire products. For this reason, it is recommended that you should instead install and build non-binary bundles where possible. Note that a binary bundle may also include the build script so that you can wipe out the included binaries and build from scratch as needed.

#### Binary Bundle Example

```console
bundle-geode-1.11.0-cluster-app-mysql-perf_test_mysql-bin.tar.gz
```

## Version Compatibility

The version number in the bundle name indicates that the bundle has been created in the workspace which was configured to run that particular version of Geode or GemFire. `geode-addon` currently supports Geode 1.x and GemFire 9.x. Both products are compatible and therefore bundles should run on both. You may see a warning message if you try to install a `geode` bundle on `gemfire`, or vice versa. In most cases, if not all, you can ignore the warning message.

## Installing Bundles

To install bundles, run the `install_bundle` command included in `geode-addon`. Before installing a bundle, make sure you are in the workspace configured with the compatible Geode or GemFire version. Always execute the `switch_workspace` command first to enter the workspace of your choice before installing a bundle.

### Listing Remote Bundles

The bundles in this repo can be listed using the `install_bundle` command. Simply run the command with or without the `-list` option.

```console
# List remote bundles
install_bundle
```

### Downloading Bundle

To install remote bundles hosted by this repo, specify the `-download` option.

```console
install_bundle -download bundle-geode-1.11.0-cluster-app-mysql-perf_test_mysql.tar.gz
```

### Previewing Bundle

Before you install a bundle, you can preview the contents of the bundle by specifying the `-preview` option as shown in the following example. You can also specify the `-download` option to preview a remote bundle.

```console
install_bundle -download -preview bundle-geode-1.11.0-cluster-app-mysql-perf_test_mysql.tar.gz
```

### Bundle Installation Example

The example below shows typical bundle installation steps. Each bundle includes `README.md` with instructions in the cluster and/or app directories.

:exclamation: For this example to work, you will need to first create the **`nw`** schema in MySQL before starting the cluster. Open MySQL Workbench and create the **`nw`** schema.

```console
# Switch to myws
switch_workspace myws

# Download and install bundle in the myws workspace
install_bundle -download -preview bundle-geode-1.11.0-cluster-app-mysql-perf_test_mysql.tar.gz

# Switch cluster to the 'mysql' cluster you just installed.
switch_cluster mysql

# View README.md
less README.md

# You must now add a locator and members. All bundles come without locators and members.
add_locator
add_member
add_member

# The 'mysql' cluster has been preconfigured to connect to MySQL on localhost
# with the user name 'root' and the password 'MySql123'. Change the user name/password
# in the following file.
vi etc/hibernate.cfg-mysql.xml

# For the 'mysql' bundle, you must first build the perf_test_mysql app.
cd_app perf_test_mysql; cd bin_sh
./build_app

# Upon successful build, start the 'mysql' cluster
start_cluster

# Run the app to write/read mock data to/from the Geode and MySQL.
cd_app perf_test_mysql; cd bin_sh
./test_group -prop ../etc/group-factory.properties  -run

