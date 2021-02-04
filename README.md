# Oracle database docker with full sample db

### Pull image

You can pull oracle database docker image with command:

```shell script
docker pull thanhnham/oracle-database:19.3.0-ee
```

### Running Oracle Database in a container

#### Running Oracle Database Enterprise in a container

To run your Oracle Database image use the `docker run` command as follows:

    docker run --name <container name> \
    -p <host port>:1521 -p <host port>:5500 \
    -e ORACLE_SID=<your SID> \
    -e ORACLE_PDB=<your PDB name> \
    -e ORACLE_PWD=<your database passwords> \
    -e INIT_SGA_SIZE=<your database SGA memory in MB> \
    -e INIT_PGA_SIZE=<your database PGA memory in MB> \
    -e ORACLE_EDITION=<your database edition> \
    -e ORACLE_CHARACTERSET=<your character set> \
    -v [<host mount point>:]/opt/oracle/oradata \
    thanhnham/oracle-database:19.3.0-ee
    
    Parameters:
       --name:        The name of the container (default: auto generated).
       -p:            The port mapping of the host port to the container port.
                      Two ports are exposed: 1521 (Oracle Listener), 5500 (OEM Express).
       -e ORACLE_SID: The Oracle Database SID that should be used (default: ORCLCDB).
       -e ORACLE_PDB: The Oracle Database PDB name that should be used (default: ORCLPDB1).
       -e ORACLE_PWD: The Oracle Database SYS, SYSTEM and PDB_ADMIN password (default: auto generated).
       -e INIT_SGA_SIZE:
                      The total memory in MB that should be used for all SGA components (optional).
                      Supported 19.3 onwards.
       -e INIT_PGA_SIZE:
                      The target aggregate PGA memory in MB that should be used for all server processes attached to the instance (optional).
                      Supported 19.3 onwards.
       -e ORACLE_EDITION:
                      The Oracle Database Edition (enterprise/standard).
                      Supported 19.3 onwards.
       -e ORACLE_CHARACTERSET:
                      The character set to use when creating the database (default: AL32UTF8).
       -v /opt/oracle/oradata
                      The data volume to use for the database.
                      Has to be writable by the Unix "oracle" (uid: 54321) user inside the container!
                      If omitted the database will not be persisted over container recreation.
       -v /opt/oracle/scripts/startup | /docker-entrypoint-initdb.d/startup
                      Optional: A volume with custom scripts to be run after database startup.
                      For further details see the "Running scripts after setup and on startup" section below.
       -v /opt/oracle/scripts/setup | /docker-entrypoint-initdb.d/setup
                      Optional: A volume with custom scripts to be run after database setup.
                      For further details see the "Running scripts after setup and on startup" section below.

Once the container has been started and the database created you can connect to it just like to any other database:

    sqlplus sys/<your password>@//localhost:1521/<your SID> as sysdba
    sqlplus system/<your password>@//localhost:1521/<your SID>
    sqlplus pdbadmin/<your password>@//localhost:1521/<Your PDB name>

The Oracle Database inside the container also has Oracle Enterprise Manager Express configured. To access OEM Express, start your browser and follow the URL:

    https://localhost:5500/em/

**NOTE**: Oracle Database bypasses file system level caching for some of the files by using the `O_DIRECT` flag. It is not advised to run the container on a file system that does not support the `O_DIRECT` flag.

#### Example command for my case

```shell script
docker volume create oracle_volume
docker run --name oracle -p 1521:1521 -p 5500:5500 -e ORACLE_PWD=admin -v oracle_volume:/opt/oracle/oradata thanhnham/oracle-database:19.3.0-ee
```

Wait for more than 20 minutes to complete...

### Unlock all schema & sample database

#### Copy `db-sample` folder to container

```sh
docker cp ./db-sample/ $(docker ps -a | grep "thanhnham/oracle-database:19.3.0-ee" | awk '{ print $1 }'):/home/oracle
```

#### Go inside container

```shell script
docker exec -ti $(docker ps -a | grep "thanhnham/oracle-database:19.3.0-ee" | awk '{ print $1 }') /bin/bash
```

#### Prepare to unlock

```shell script
cp -r db-sample/ $ORACLE_HOME/demo/schema/
```

#### connect to sql plus

```shell script
sqlplus sys/<your password>@//localhost:1521/<your SID> as sysdba
Alter session set container=<your PDB name>;
```

#### unlock all sample database & schema

```shell script
@?/demo/schema/db-sample/mksample <your password> <your password> <your password> <your password> <your password> <your password> <your password> <your password> users temp $ORACLE_HOME/demo/schema/log/ localhost:1521/<your PDB name>
```

### connect to SID

    - username: system
    - password: <your password>
    - hostname: localhost
    - port: <host port>
    - SID: <your SID>

### connect to HR/OE/PM/IX/SH schema

    - username: HR
    - password: <your password>
    - hostname: localhost
    - port: <host port>
    - SID: <your PDB name>

