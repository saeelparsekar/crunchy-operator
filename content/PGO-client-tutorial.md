---
title: Crunchy Data PostgreSQL Operator Tutorial
description: This tutorial explains how to use Crunchy PostgreSQL Operator (PGO)
---

### Introduction

The PostgreSQL Operator Client, aka pgo, is the most convenient way to interact with the PostgreSQL Operator. pgo provides many convenience methods for creating, managing, and deleting PostgreSQL clusters through a series of simple commands.

Note: Terms 'cluster' and 'database' are interchangeable. Creating a PostgreSQL cluster creates a PostgreSQL database with the same name.

### Check the PostgreSQL DB Operator 

```execute
kubectl get pods -n pgo
```

You should see a pod starting with 'postgres-operator' with Ready value '4/4' and Status 'Running'.

### PGO Setup

For user to create PostgreSQL database Cluster using Crunchy PostgreSQL DB Operator:
```execute
cd /home/student/projects/crunchy-postgres-scripts && export PGO_OPERATOR_NAMESPACE=pgo 
```

Set the default namespace for PGO commands (if you skip this step you will need to add "-n pgo" to all the commands to specify the namespace):
```execute
export PGO_NAMESPACE=pgo
```

### Create a database

Set name of the cluster to be created
```execute
export pgo_cluster_name=my-db
```

Set the location of your image repository
```execute
export cluster_image_prefix=registry.developers.crunchydata.com/crunchydata
```

Create a PostgreSQL cluster (database) :
```execute
kubectl apply -f pgcluster.yaml
```

### Test database availability

Test connectivity to the created database. Note the IP (including the port) of the primary service that can be used to connect to this database.
The database will be available when both the status of the primary service and of the primary instance will be **UP**.
```execute
pgo test my-db
```

To test the availability of all the databases:
```execute
pgo test --all
```

### Get database info

To get info about the created database. The service field shows the IP that can be used to connect to this database:
```execute
pgo show cluster my-db
```

To get info about all the databases:
```execute
pgo show cluster --all
```

### How to use PostgreSQL client (psql)

Connect to database using PostgreSQL client (psql). Replace the 'primary-service-IP' with the correct value from 
'pgo test' or 'pgo show cluster' commands (don't include the port number).
If needed replace the database (-d flag), user (-U flag) or password (PGPASSWORD).
```execute
PGPASSWORD=password psql -h primary-service-IP -U pguser -d my-db
```

To enter interactively the password use this command. If needed replace the database (-d flag) or user (-U flag):
```execute
psql -h primary-service-IP -U pguser -d my-db
```

Using the PostgreSQL client (psql) you can create tables, insert data into them and execute queries.

Create a table:
```execute
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL,
   JOIN_DATE	  DATE
);
```

Insert Values into created table:
```execute
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,SALARY,JOIN_DATE) VALUES (1, 'Paul', 32, 'California', 20000.00,'2001-07-13');
```

The following example is to insert a row; here salary column is omitted and therefore it will have the default value:
```execute
INSERT INTO COMPANY (ID,NAME,AGE,ADDRESS,JOIN_DATE) VALUES (2, 'Allen', 25, 'Texas', '2007-12-13');
```

Fetch the data from the table:
```execute
SELECT * FROM company;
```

To Exit from PostgreSQL client (psql):
```execute
\q
```

### Working with database users

Create a database 'test' user with the specified password for my-db database:
```execute
pgo create user --username=test --password=password my-db
```

Create a database 'test2' user with an autogenerated password for my-db database:
```execute
pgo create user --username=test2 my-db
```

Show the list of users:
```execute
pgo show user my-db
```

Delete 'test2' user:
```execute
pgo delete user --username=test2 my-db
```

### Clone a database

Clone 'my-db' database into a new database called 'my-db-copy'. A default 'testuser' user will be created. Note the username and the password required to connect to the 'my-db-copy' database.
```execute
pgo create cluster my-db-copy --restore-from=my-db 
```

You can specify a database user for the new database. This will clone 'my-db' database into a new 'my-db-copy' database and the 'pguser' database user with 'password' password will be created for accessing the created database:
```execute
pgo create cluster my-db-copy --restore-from=my-db --username pguser --password password
```

### Backup and restore a database

Backup 'my-db' database. By default the created backup is an incremental backup. A full backup is created when the database is created.
Note that the command will just start the backup process. It will take a while until the backup will be completed and will appear in the list of backups.

```execute
pgo backup my-db
```

To show the backups of 'my-db' database:
```execute
pgo show backup my-db
```

You can restore a database. During the restore the database will be down. To see when the cluster is available use 'pgo test' command.

Usually you will restore a database at a specific point in time. For example, to restore to November 02, 2019 at 8:00am:
```execute
pgo restore my-db --pitr-target="2020-11-02 08:00:00.000000+00" --backup-opts="--type=time"
```

Rarely you will want to execute a full restore. To execute a full restore of a database:
```execute
pgo restore my-db
```

### Delete a database

This will delete 'my-db' database. This command will just start the process of deleting the database.
To see when the database was deleted use 'pgo test --all' command.

```execute
pgo delete cluster my-db
```

