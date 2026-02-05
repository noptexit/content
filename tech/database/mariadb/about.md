---
title: MariaDB
subsection: mariadb
section: tech-database
description: Transactional SQL database, an enhanced drop-in replacement for MySQL.
---

# MariaDB

[MariaDB server](https://mariadb.org/en/about/) is a community-developed, open source relational database, and an enhanced, drop-in replacement for MySQL. To learn more about MariaDB, visit the [upstream feature page](https://mariadb.com/kb/en/mariadb-vs-mysql-features/), and to see the main differences from MySQL, see the [compatibility documentation](https://mariadb.com/kb/en/mariadb-vs-mysql-compatibility/).

MariaDB is the preferred MySQL implementation in Fedora. It can usually be used instead of MySQL as a drop-in replacement in most practical cases. However, if you need Community MySQL, it is available as the [mysql8.4 package](https://src.fedoraproject.org/rpms/mysql8.4) in Fedora repositories.

## Versions available in Fedora

Fedora ships only the LTS releases of MariaDB. Currently, several versions are available in parallel, each in its own versioned package (e.g. `mariadb10.11`, `mariadb11.8`).

Only one version in each Fedora release is set as the "distribution default", which provides the classic unversioned package names (e.g. `mariadb`, `mariadb-server`).

To see which MariaDB versions are available in your Fedora release:

```
$ dnf search mariadb | grep server
```

> **Note:** Available versions may differ between Fedora releases. Rawhide may offer newer versions than the current stable release.

## How to install MariaDB on Fedora

To install the distribution default version of MariaDB server:

```
$ sudo dnf install mariadb-server
```

In order to install a specific version, use the versioned package names:

```
$ sudo dnf install mariadb11.8-server
```

If you need to connect to the MariaDB server using a GUI, install phpMyAdmin:

```
$ sudo dnf install phpMyAdmin
```

Or install LibreOffice Base with the JDBC connector for MariaDB:

```
$ sudo dnf install libreoffice-base mariadb-java-client
```

For connecting to the MariaDB server using ODBC:

```
$ sudo dnf install mariadb-connector-odbc
```

## Getting started

MariaDB runs on port 3306 by default and creates a local unix socket at `/var/lib/mysql/mysql.sock`. Data is stored in `/var/lib/mysql` and logs are located under `/var/log/mariadb/`. When changing these directories, pay attention to use the correct SELinux context and owner.

The data directory is initialized automatically during the first start. To start MariaDB server:

```
$ sudo systemctl start mariadb
```

To start MariaDB automatically after reboot:

```
$ sudo systemctl enable mariadb
```

The root user has no password set by default and connects via unix socket authentication:

```
$ mysql -u root
```

## How to configure MariaDB

Configuration of both client and server is done by editing files `/etc/my.cnf`, `~/.my.cnf` and any files under `/etc/my.cnf.d/` with `.cnf` suffix. For more information, see [the upstream documentation](https://mariadb.com/kb/en/server-system-variables/).

### Local development setup

When developing an application that uses MariaDB, developers typically create a dedicated database and user:

```
$ sudo systemctl start mariadb
$ sudo systemctl enable mariadb
$ sudo mysql -u root
MariaDB [(none)]> CREATE DATABASE db1;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> CREATE USER 'valeria'@'localhost' IDENTIFIED BY 'secretpass';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> GRANT ALL ON db1.* TO 'valeria'@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
```

After that, access the database by running:

```
$ mysql -u valeria -p
```

By specifying `-p` without a password, you will be asked for the password interactively.

In various frameworks or libraries, you will usually use the username, password and database to access the database.

### Production deployment

When running MariaDB in production, pay extra attention to security:

 * Do not accept connections from all addresses unless absolutely necessary
 * **Always use a strong password**
 * Limit user permissions to those necessary for the application to run
 * Run SELinux in enforcing mode and set up the policy properly
 * Keep your system and MariaDB updated
 * Monitor your MariaDB and system logs

By default, MariaDB cannot be accessed from another computer. To allow remote access:

Allow connections on port 3306:

```
$ sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
$ sudo firewall-cmd --reload
```

Allow listening on all interfaces (or your preferred network interface) by adding this configuration option:

```
bind-address = 0.0.0.0
```

### Common configuration options

To change configuration, create a file under `/etc/my.cnf.d/`. The following example shows `/etc/my.cnf.d/myconfig.cnf` with commonly changed options:

```
# The maximum permitted number of simultaneous client connections:
max_connections = 20

# The minimum length of the word to be included in a FULLTEXT index:
ft_min_word_len = 3

# The maximum length of the word to be included in a FULLTEXT index:
ft_max_word_len = 20

# In case the native AIO is broken, it can be disabled by:
innodb_use_native_aio = 0

# Log slow queries:
slow_query_log = 1
slow_query_log_file = "/var/log/mariadb/slowquery.log"

# Log all queries (e.g. for debugging):
general_log = 1
general_log_file = "/var/log/mariadb/query.log"
```

After changing the configuration, restart the daemon: `$ sudo systemctl restart mariadb`.

## How to get an encrypted connection

For the proper TLS setup, you should first [add a TLS certificate](/en/docs/system-services/tls/certificates/) to your system.

## How to install MariaDB Galera on Fedora

MariaDB Galera Cluster is a synchronous multi-master cluster for MariaDB.

To install MariaDB Galera server:

```
$ sudo dnf install mariadb-server-galera
```

MariaDB Galera uses several packages from base MariaDB and provides the same service name. To start MariaDB Galera:

```
$ sudo systemctl start mariadb
```

## How to get a MariaDB container

```
$ podman pull quay.io/fedora/mariadb-118
```

For other versions, see available images at [quay.io/organization/fedora](https://quay.io/organization/fedora).

## How to extend MariaDB server by installing extensions available in Fedora

[MariaDB Connect Storage Engine](https://mariadb.com/kb/en/connect/) enables MariaDB to access external local or remote data (MED):

```
$ sudo dnf install mariadb-connect-engine
```

[The Open Query GRAPH computation engine (OQGraph)](https://mariadb.com/kb/en/oqgraph-storage-engine/):

```
$ sudo dnf install mariadb-oqgraph-engine
```

## MariaDB server available as a dynamic library

MariaDB server is also available as a dynamic library (`libmysqld.so`) in the package `mariadb-embedded`. Header files for building applications against this library are in `mariadb-embedded-devel`.

However, the use of the embedded library is discouraged. MySQL 8 dropped the embedded library and MariaDB is expected to follow in the same direction.

## See also

- [Official MariaDB documentation](https://mariadb.com/kb/en/)
- [MariaDB Release Notes](https://mariadb.com/kb/en/release-notes/)
- [MySQL documentation](https://dev.mysql.com/doc/) — mostly applicable to MariaDB as well
- [MariaDB vs MySQL compatibility](https://mariadb.com/kb/en/mariadb-vs-mysql-compatibility/)
