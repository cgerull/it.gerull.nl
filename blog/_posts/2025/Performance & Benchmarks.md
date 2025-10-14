---
layout: post
title: PGbench
description: "PostgreSQL benchmarkings."
tag: postgresql
category: Database
date: 2025-10-14 07:19:23
---
## PGbench

From postgresql.org:
-- pgbench is a simple program for running benchmark tests on PostgreSQL. It runs the same sequence of SQL commands over and over, possibly in multiple concurrent database sessions, and then calculates the average transaction rate (transactions per second). By default, pgbench tests a scenario that is loosely based on TPC-B, involving five `SELECT`, `UPDATE`, and `INSERT` commands per transaction. However, it is easy to test other cases by writing your own transaction script files.

### Example usage

#### Init

```bash
pgbench [-h host -U user] -i -s 100 <test-database>
```

This will initialize the database under test with 10.000.000 records, with a size of 1.6GB. (Default, or -s 1 is 16MB)

#### Run

```bash
pgbench [-h host -U user] -c 30 -j 4 -t 1000 <test-database>
```

This will run a benchmark with:
-c 30 -> 30 clients
-j 4 -> 4 workers
-t 1000 -> 1000 transactions per client

pgbench parameters:

```bash
-n : do not run vacuum before testing
-S : perform select-only transaction
-c : number of clients
-C : open a new connection for each transaction
-T : how many seconds the test will continue
-t : number of transactions
-j : number of threads (workers)
```

See also:

- [https://www.postgresql.org/docs/current/pgbench.html](https://www.postgresql.org/docs/current/pgbench.html)
- [https://www.cloudbees.com/blog/tuning-postgresql-with-pgbench](https://www.cloudbees.com/blog/tuning-postgresql-with-pgbench)
