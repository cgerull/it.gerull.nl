---
layout: post
title: SQL Snippets
description: "Collection of often used SQL fragments."
tag: postgresql
category: Database
date: 2025-10-14 07:22:23
---

Examples and snippets for frequently used database operations.

## Setup a simple database and user

```sql
-- Run as superuser
CREATE ROLE "demo-user" WITH NOSUPERUSER INHERIT NOCREATEROLE CREATEDB LOGIN NOREPLICATION NOBYPASSRLS PASSWORD 'demo-user';

CREATE DATABASE "demo" WITH TEMPLATE = template0 ENCODING = 'UTF8' LOCALE_PROVIDER = libc LOCALE = 'en_US.UTF-8';

ALTER DATABASE "demo" OWNER TO "demo-user";
```
