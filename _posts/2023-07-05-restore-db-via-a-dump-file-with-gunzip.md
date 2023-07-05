---
layout: post
title: "Restore PostgreSQL DB via a dump.gz file with gunzip and psql"
description: "Restore PostgreSQL DB via a dump.gz file with gunzip and psql"
crawlertitle: "Restore PostgreSQL DB via a dump.gz file with gunzip and psql"
date: 2023-07-05 12:47:29 +0800
categories: Database
tags: Database
comments: true
---

If you have a compressed database dump file in `.dump.gz` format and want to restore it, you can use the `gunzip` command and PostgreSQL command-line utility to perform the restoration.

First, recreate the target database if it already exists, as stale data can lead to conflict:

```bash
psql -U {username} -c "DROP DATABASE IF EXISTS {database_name};" && createdb {database_name}
```

Replace `{username}` with your PostgreSQL username and `{database_name}` with the name of the database you want to restore, then enter your password when prompted before the restoration process begins.

Next, navigate to the directory where your `.dump.gz` file is located and extract it using the following command:

```bash
gunzip -c {your_file.dump.gz} | psql {database_name}
```

> `gunzip -c` will extract the compressed file to the standard output stream, **leaving files intact**.

Replace `{your_file.dump.gz}` with the path to your specific `.dump.gz` file, and `{database_name}` with the name of the database you want to restore.

This command will extract the compressed file and pipe it to the `psql` command to restore it to the specified database.

Once the restoration is complete, you can check the database to ensure that the data has been successfully restored.

That's it! You now have successfully restored your database from a compressed dump file using the `gunzip` command and PostgreSQL command-line utility.

## Refs.

- createdb: https://www.postgresql.org/docs/current/app-createdb.html
- gunzip: https://linux.die.net/man/1/gunzip
- psql: https://www.postgresql.org/docs/15/app-psql.html
