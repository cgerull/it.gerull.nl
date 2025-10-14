---
layout: post
title: PostgreSQL troubleshooting
description: "Useful troubleshooting commands."
tag: postgresql
category: Database
date: 2025-10-14 07:24:23
---

Shamelessly copied from [https://data-nerd.blog/2018/12/30/postgresql-diagnostic-queries/](https://data-nerd.blog/2018/12/30/postgresql-diagnostic-queries/)

## POSTGRESQL SYSTEM CONFIGURATION

### Version

```sql
SHOW server_version;
```

### Get product version, server IP and port number

```sql
-- query server version (standard major.minor.patch format)
SELECT Version()          AS "Postgres Version",
        Inet_server_addr() AS "Server IP",
       setting            AS "Port Number"
FROM   pg_settings
WHERE  name = 'port';
```

### Get selected server properties

```sql
-- Server up time
SELECT Inet_server_addr()
       AS
       Server_IP --server IP address
       ,
       Inet_server_port()
       AS Server_Port --server port
       ,
       Current_database()
       AS Current_Database --Current database
       ,
       current_user
       AS Current_User --Current user
       ,
       Pg_backend_pid()
       AS ProcessID --Current user pid
       ,
       Pg_postmaster_start_time()
       AS Server_Start_Time --Last start time
       ,
       current_timestamp :: TIMESTAMP - Pg_postmaster_start_time() :: TIMESTAMP
       AS
       Running_Since;
```

### Get selected postgres configuration parameter values

```sql
-- Option 1: PG_SETTINGS
-- This gives you a lot of useful info about postgres instance
SELECT name, unit, setting FROM pg_settings WHERE name ='port'
UNION ALL
SELECT name, unit, setting FROM pg_settings WHERE name ='shared_buffers'        -- shared_buffers determines how much memory is dedicated for caching data
UNION ALL
SELECT name, unit, setting FROM pg_settings WHERE name ='work_mem'              -- work memory required for each incoming connection
UNION ALL
SELECT name, unit, setting FROM pg_settings WHERE name ='maintenance_work_mem'  -- work memory of maintenace type queries "VACUUM, CREATE INDEX etc."
UNION ALL
SELECT name, unit, setting FROM pg_settings WHERE name ='wal_buffers'           -- Sets the number of disk-page buffers in shared memory for WAL
UNION ALL
SELECT name, unit, setting FROM pg_settings WHERE name ='effective_cache_size'  -- used by postgres query planner
UNION ALL
SELECT name, unit, setting FROM pg_settings WHERE name ='TimeZone'              -- server time zone;

-- Option 2: SHOW ALL
-- The SHOW ALL command displays all current configuration setting of in three columns
SHOW all;

-- Option 3: PG_FILE_SETTINGS
-- To read what is stored in the postgresql.conf file itself, use the view pg_file_settings.
SELECT * FROM pg_settings;

-- Get the significant tuning parameters
SELECT name, setting, unit  FROM pg_settings
WHERE name in ('max_connection', 'shared_buffers', 'effective_cache_size', 'maintenance_work_mem', 'wal_buffers', 'default_statistics_target', 'work_mem', 'max_worker_processes', 'max_parallel_workers_per_gather', 'max_parallel_workers', 'max_parallel_maintenance_workers', 'checkpoint_completion_target');

```

### Get Operating System (OS) details

```sql
-- Get OS Version
SELECT version();

| OS     | Wiki References                                       |
| ------ | ----------------------------------------------------- |
| RedHat | wikipedia.org/wiki/Red_Hat_Enterprise_Linux           |
| Windows| wikipedia.org/wiki/List_of_Microsoft_Windows_versions |
| Mac OS | wikipedia.org/wiki/MacOS                              |
| Ubuntu | wikipedia.org/wiki/Ubuntu_version_history
```

### Get location of data file and log file (WAL) directory (this is where postgres stores the database files)

```sql
SELECT NAME,
       setting
FROM   pg_settings
WHERE  NAME IN ( 'data_directory', 'log_directory' );
--OR
SHOW data_directory;
SHOW log_directory;
```

### Get data and log (WAL) size for all databases

```sql
-- Cumulative size of all databases
SELECT Pg_size_pretty(Sum(Pg_database_size(datname))) AS total_database_size
FROM   pg_database;

-- Cumulative size of all Write-Ahead Log (WAL) files
SELECT Pg_size_pretty(Sum(size)) AS total_WAL_size
FROM   Pg_ls_waldir();
```

### List all databases along with their creation date and size

```sql
-- List database by name with creation date and size
SELECT datname AS database_name,
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size,
       (Pg_stat_file('base/'
              ||oid
              ||'/PG_VERSION')).modification as create_timestamp
FROM   pg_database
WHERE  datistemplate = false;
```


### Get an overview of current server activity

```sql
SELECT
    pid
    , datname
    , usename
    , application_name
    , client_addr
    , to_char(backend_start, 'YYYY-MM-DD HH24:MI:SS TZ') AS backend_start
    , state
    , wait_event_type || ': ' || wait_event AS wait_event
    , pg_blocking_pids(pid) AS blocking_pids
    , query
    , to_char(state_change, 'YYYY-MM-DD HH24:MI:SS TZ') AS state_change
    , to_char(query_start, 'YYYY-MM-DD HH24:MI:SS TZ') AS query_start
    , backend_type
FROM
    pg_stat_activity
ORDER BY pid;
```

### Connections

### Get connections configuration

```sql
-- The maximum amount of connections is configured in the configuration file with max_connections.  Default is 100
show max_connections;

-- To calculate the connections that are really available, one has also to check the reserved connections for superusers configured in superuser_reserved_connections. Default is 3
show superuser_reserved_connections;

-- With PostgreSQL 16 arrived a new parameter to reserve connections to certain roles, reserved_connections. Default is 0
show reserved_connections;```
### Get active v/s inactive connections
```sql
-- Get active vs inactive connections
SELECT state,
       Count(pid)
FROM   pg_stat_activity
GROUP  BY state,
          datname
HAVING datname = '<your_database_name>'
ORDER  BY Count(pid) DESC;

-- One row per server process, showing database OID, database name, process ID, user OID, user name, current query, query's waiting status, time at which the current query began execution
-- Time at which the process was started, and client's address and port number. The columns that report data on the current query are available unless the parameter stats_command_string has been turned off.
-- Furthermore, these columns are only visible if the user examining the view is a superuser or the same as the user owning the process being reported on
```

### Get more connection info

```sql
-- Get more connection information

```
### Get total usable connections
```sql
-- Get total usable connections
SELECT sum(   CASE name
                  WHEN 'max_connections' THEN
                      setting::int
                  ELSE
                      setting::int * (-1)
              END
          ) AS available_connections
FROM pg_settings
WHERE name IN ( 'max_connections', 'superuser_reserved_connections', 'reserved_connections' );
```

### Terminate connections

```sql
SELECT pg_terminate_backend(pg_stat_activity.pid)
FROM pg_stat_activity
WHERE pg_stat_activity.datname = 'TARGET_DB' -- ← change this to your DB
  AND pid <> pg_backend_pid();
```
##  Database specific queries

### Get database size (pretty size)

```sql
SELECT Current_database(),
       Pg_size_pretty(Pg_database_size(Current_database()));
```

### Get top 20 objects in database by size

```sql
SELECT nspname                                        AS schemaname,
       cl.relname                                     AS objectname,
       CASE relkind
         WHEN 'r' THEN 'table'
         WHEN 'i' THEN 'index'
         WHEN 'S' THEN 'sequence'
         WHEN 'v' THEN 'view'
         WHEN 'm' THEN 'materialized view'
         ELSE 'other'
       end                                            AS type,
       s.n_live_tup                                   AS total_rows,
       Pg_size_pretty(Pg_total_relation_size(cl.oid)) AS size
FROM   pg_class cl
       LEFT JOIN pg_namespace n
              ON ( n.oid = cl.relnamespace )
       LEFT JOIN pg_stat_user_tables s
              ON ( s.relid = cl.oid )
WHERE  nspname NOT IN ( 'pg_catalog', 'information_schema' )
       AND cl.relkind <> 'i'
       AND nspname !~ '^pg_toast'
ORDER  BY Pg_total_relation_size(cl.oid) DESC
LIMIT  20;
```

### Get size of all tables

```sql
SELECT *,
       Pg_size_pretty(total_bytes) AS total,
       Pg_size_pretty(index_bytes) AS INDEX,
       Pg_size_pretty(toast_bytes) AS toast,
       Pg_size_pretty(table_bytes) AS TABLE
FROM   (SELECT *,
               total_bytes - index_bytes - COALESCE(toast_bytes, 0) AS
               table_bytes
        FROM   (SELECT c.oid,
                       nspname                               AS table_schema,
                       relname                               AS TABLE_NAME,
                       c.reltuples                           AS row_estimate,
                       Pg_total_relation_size(c.oid)         AS total_bytes,
                       Pg_indexes_size(c.oid)                AS index_bytes,
                       Pg_total_relation_size(reltoastrelid) AS toast_bytes
                FROM   pg_class c
                       LEFT JOIN pg_namespace n
                              ON n.oid = c.relnamespace
                WHERE  relkind = 'r') a) a;
```

### Get table metadata

```sql
SELECT relname,
       relpages,
       reltuples,
       relallvisible,
       relkind,
       relnatts,
       relhassubclass,
       reloptions,
       Pg_table_size(oid)
FROM   pg_class
WHERE  relname = '<table_name_here>';
```

### Get table structure (i.e. describe table)

```sql
SELECT column_name,
       data_type,
       character_maximum_length
FROM   information_schema.columns
WHERE  table_name = '<table_name_here>';

-- Does the table have anything unusual about it?
-- a. contains large objects
-- b. has a large proportion of NULLs in several columns
-- c. receives a large number of UPDATEs or DELETEs regularly
-- d. is growing rapidly
-- e. has many indexes on it
-- f. uses triggers that may be executing database functions, or is calling functions directly
```

## Locking

### Get Lock connection count

```sql
SELECT Count(DISTINCT pid) AS count
FROM   pg_locks
WHERE  NOT granted;
```

### Get locks_relation_count

```sql
SELECT   relation::regclass  AS relname ,
         count(DISTINCT pid) AS count
FROM     pg_locks
WHERE    NOT granted
GROUP BY 1;
```

### Get locks_statement_duration

```sql
SELECT a.query                                     AS blocking_statement,
       Extract('epoch' FROM Now() - a.query_start) AS blocking_duration
FROM   pg_locks bl
       JOIN pg_stat_activity a
         ON a.pid = bl.pid
WHERE  NOT bl.granted;
```

## Indexing

### Get missing indexes

```sql
SELECT
    relname AS TableName
    ,seq_scan-idx_scan AS TotalSeqScan
    ,CASE WHEN seq_scan-idx_scan > 0
        THEN 'Missing Index Found'
        ELSE 'Missing Index Not Found'
    END AS MissingIndex
    ,pg_size_pretty(pg_relation_size(relname::regclass)) AS TableSize
    ,idx_scan AS TotalIndexScan
FROM pg_stat_all_tables
WHERE schemaname='public'
    AND pg_relation_size(relname::regclass)>100000
        ORDER BY 2 DESC;
```

### Get Unused Indexes

```sql
SELECT indexrelid::regclass AS INDEX ,
       relid::regclass      AS TABLE ,
       'DROP INDEX '
              || indexrelid::regclass
              || ';' AS drop_statement
FROM   pg_stat_user_indexes
JOIN   pg_index
using  (indexrelid)
WHERE  idx_scan = 0
AND    indisunique IS false;
```

### Get index usage stats

```sql
SELECT t.tablename                                                         AS
       "relation",
       indexname,
       c.reltuples                                                         AS
       num_rows,
       Pg_size_pretty(Pg_relation_size(Quote_ident(t.tablename) :: text))  AS
       table_size,
       Pg_size_pretty(Pg_relation_size(Quote_ident(indexrelname) :: text)) AS
       index_size,
       idx_scan                                                            AS
       number_of_scans,
       idx_tup_read                                                        AS
       tuples_read,
       idx_tup_fetch                                                       AS
       tuples_fetched
FROM   pg_tables t
       left outer join pg_class c
                    ON t.tablename = c.relname
       left outer join (SELECT c.relname   AS ctablename,
                               ipg.relname AS indexname,
                               x.indnatts  AS number_of_columns,
                               idx_scan,
                               idx_tup_read,
                               idx_tup_fetch,
                               indexrelname,
                               indisunique
                        FROM   pg_index x
                               join pg_class c
                                 ON c.oid = x.indrelid
                               join pg_class ipg
                                 ON ipg.oid = x.indexrelid
                               join pg_stat_all_indexes psai
                                 ON x.indexrelid = psai.indexrelid) AS foo
                    ON t.tablename = foo.ctablename
WHERE  t.schemaname = 'public'
ORDER  BY 1, 2;
```

## Query Performance

### Get top 10 costly queries

```sql
-- Get top 10 most costly queries
SELECT   r.rolname,
         Round((100 * total_time / Sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu ,
         Round(total_time::numeric, 2)                                            AS total_time,
         calls,
         Round(mean_time::numeric, 2) AS mean,
         Substring(query, 1, 800)     AS short_query
FROM     pg_stat_statements
JOIN     pg_roles r
ON       r.oid = userid
ORDER BY total_time DESC limit 10;
```

## Caching

### Get TOP cached tables & indexes

```sql
-- Measure cache hit ratio for tables
SELECT relname AS "relation",
       heap_blks_read AS heap_read,
       heap_blks_hit AS heap_hit,
       COALESCE((( heap_blks_hit * 100 ) / NULLIF(( heap_blks_hit + heap_blks_read ), 0)),0) AS ratio
FROM   pg_statio_user_tables
    ORDER BY ratio DESC;

-- Measure cache hit ratio for indexes
SELECT relname AS "relation",
    idx_blks_read AS index_read,
    idx_blks_hit AS index_hit,
    COALESCE((( idx_blks_hit * 100 ) / NULLIF(( idx_blks_hit + idx_blks_read ), 0)),0) AS ratio
FROM pg_statio_user_indexes
    ORDER BY ratio DESC;
```

## Autovacuum & Data bloat

### Last Autovaccum, live & dead tuples

```sql
SELECT relname AS "relation",
       Extract (epoch FROM CURRENT_TIMESTAMP - last_autovacuum) AS since_last_av,
       autovacuum_count AS av_count,
       n_tup_ins,
       n_tup_upd,
       n_tup_del,
       n_live_tup,
       n_dead_tup
FROM   pg_stat_all_tables
WHERE  schemaname = 'public'
ORDER  BY relname;
```

## Partitioning

### List all table partitions (as parent/child relationship)

```sql
SELECT nmsp_parent.nspname AS parent_schema,
       parent.relname      AS parent,
       child.relname       AS child,
       CASE child.relkind
         WHEN 'r' THEN 'table'
         WHEN 'i' THEN 'index'
         WHEN 'S' THEN 'sequence'
         WHEN 'v' THEN 'view'
         WHEN 'm' THEN 'materialized view'
         ELSE 'other'
       END                 AS type,
       s.n_live_tup        AS total_rows
FROM   pg_inherits
       JOIN pg_class parent
         ON pg_inherits.inhparent = parent.oid
       JOIN pg_class child
         ON pg_inherits.inhrelid = child.oid
       JOIN pg_namespace nmsp_parent
         ON nmsp_parent.oid = parent.relnamespace
       JOIN pg_namespace nmsp_child
         ON nmsp_child.oid = child.relnamespace
       JOIN pg_stat_user_tables s
         ON s.relid = child.oid
WHERE  child.relkind = 'r'
ORDER  BY parent,
          child;
```

### List ranges for all partitions (and sub-partitions) for a given table

```sql
SELECT pt.relname AS partition_name,
       Pg_get_expr(pt.relpartbound, pt.oid, TRUE) AS partition_expression
FROM   pg_class base_tb
       join pg_inherits i
         ON i.inhparent = base_tb.oid
       join pg_class pt
         ON pt.oid = i.inhrelid
WHERE  base_tb.oid = 'public.table_name ' :: regclass;
```

### Postgres 12 – pg_partition_tree()
Alternatively, can use new PG12 function **`pg_partition_tree()`** to display information about partitions.

```sql
SELECT relid,
       parentrelid,
       isleaf,
       level
FROM   Pg_partition_tree('<parent_table_name>');
```

## Security Roles and Privileges

### Checking if user is connected is a “superuser”

```sql
SELECT usesuper
FROM   pg_user
WHERE  usename = CURRENT_USER;
```

### List all users (along with assigned roles) in current database

```sql
SELECT usename AS role_name,
       CASE
         WHEN usesuper
              AND usecreatedb THEN Cast('superuser, create database' AS
                                   pg_catalog.TEXT)
         WHEN usesuper THEN Cast('superuser' AS pg_catalog.TEXT)
         WHEN usecreatedb THEN Cast('create database' AS pg_catalog.TEXT)
         ELSE Cast('' AS pg_catalog.TEXT)
       END     role_attributes
FROM   pg_catalog.pg_user
ORDER  BY role_name DESC;
```

## Extensions

### List available extensions

```sql
-- pg_available_extensions is a system catalogue view listing all available extensions.
SELECT *
FROM   pg_available_extensions
```

### List installed extensions

```sql
SELECT pge.extname    AS extension_name,
       pge.extversion AS extension_version,
       pge.extowner   AS extension_owner,
       pgu.usename    AS owner_name,
       pgu.usesuper   AS is_super_user
FROM   pg_extension pge
       JOIN pg_user pgu
         ON pge.extowner = pgu.usesysid;
```
