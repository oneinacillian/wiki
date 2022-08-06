>Take note of ubuntu performance performance when configuring a Atomic instance

# The following will be covered here
- Log into postgresql
- Recover database backup
- Perform database backup

## Perform a database dump (backup) of your atomic database

> Connect using the postgres user

```
su - postgres
```

> List your databases to retrieve the exact name of your wax atomic database

```
psql
\l
```

> Results should look something like this

```
postgres-# \l+
                                                                       List of databases
           Name           |  Owner   | Encoding | Collate |  Ctype  |   Access privileges   |  Size   | Tablespace |                Description                 
--------------------------+----------+----------+---------+---------+-----------------------+---------+------------+--------------------------------------------
 api-wax-mainnet-atomic-1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 21 GB   | pg_default | 
 postgres                 | postgres | UTF8     | C.UTF-8 | C.UTF-8 |                       | 8601 kB | pg_default | default administrative connection database
 template0                | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8377 kB | pg_default | unmodifiable empty database
                          |          |          |         |         | postgres=CTc/postgres |         |            | 
 template1                | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +| 8545 kB | pg_default | default template for new databases
                          |          |          |         |         | postgres=CTc/postgres |         |            | 
(4 rows)
```

> Perform a database dump of your atomic database

```
pg_dump -Fc "api-wax-mainnet-atomic-1" > /data/postgres_backups/atomictest.dump
```
