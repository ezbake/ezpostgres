EzPostgres
===
EzPostgres is a secure wrapper around PostgreSQL that provides row level security and metering/auditing.

Requirements
===
 * PostgreSQL 9.3.4
 * ezbake_visibility.so and dependencies compiled and linked in the Postgres lib folder (see below)
 * ezbake_visibility extension config files in Postgres extensions directory (see below)

Installing
===
 1. First you should build the project: `mvn clean install`

 1. Next you need to make sure the ezbake_visibility.so is built and placed in the correct spot.
    1. In the visibility-extensions directory run `mvn clean package`. This should build the shared object.
    1. `sudo cp visibility-extension/target/nar/ezbake-postgresql-visibility-<version>-amd64-Linux-gpp-shared/lib/amd64-Linux-gpp/shared/libezbake-postgresql-visibility-<version>.so
       /usr/pgsql-9.3/lib/ezbake_visibility.so` (or whereever your Postgres lib directory is).
    1. `sudo cp visibility-extension/src/main/resources/ezbake_visibility* /usr/pgsql-9.3/share/extension` (or
       whereever your Postgres extensions directory is).
    1. Copy libezbake-permission-utils-<version>.so, libthrift-0.9.1-SNAPSHOT.so, and
       libezbake-base-thrift-<version>.so from the target/**/... folders into /usr/lib64
    1. Now run `sudo /sbin/ldconfig` to update the linking information for the shared objects.

 1. If the deployer deployed the application using this PostgreSQL should be configured; if not perform the below:
    1. Connect to postgres as a superuser and run the following SQL:

        ```sql
        CREATE ROLE deployerdba WITH SUPERUSER NOINHERIT;
        CREATE ROLE appName_user WITH PASSWORD '<password>';
        CREATE DATABASE appName OWNER appName_user;
        CREATE SCHEMA IF NOT EXISTS appName;
        ALTER ROLE appName_user SET search_path = appName,public;
        GRANT deployerdba TO appName_user;
        ALTER DATABASE appName SET search_path TO appName,public;
        GRANT ALL PRIVILEGES ON SCHEMA appName TO appName_user;
        GRANT ALL PRIVILEGES ON SCHEMA public TO appName_user;
        CREATE EXTENSION IF NOT EXISTS postgis SCHEMA public;
        CREATE EXTENSION IF NOT EXISTS postgis_topology;
        CREATE EXTENSION IF NOT EXISTS "uuid-ossp" SCHEMA public;
        CREATE EXTENSION IF NOT EXISTS ezbake_visibility;
        ```

 1. Now start PostgreSQL and run EzPostgres with ezbake-thrift-runner.

Usage
===
EzPostgres is a wrapper around PostgreSQL 9.3 and thus supports most of the syntax that Postgres supports.  There are
a few exceptions where EzPostgres rejects syntax that is not currently supported.  Those exceptions are:

 * AGGREGATE, FUNCTION, TRIGGER, and OPERATOR are not supported and by proxy none of the commands that use these are supported.

By default all tables in EzPostgres must contain a column named 'visibility' of type 'varchar' (any precision, but must
be large enough to hold a base64 encoded serialized Thrift Visibility object).

Building EzBake Visibility Extension
===
The EzBake visibility PostgreSQL extension is a native library and is distributed as an RPM. To build, the following
packages and their dependencies are required for RHEL-like distributions:

    * postgresql93-devel
    * gcc-cpp
    * rpm-build

Packages may vary for other distributions.
