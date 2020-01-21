---
layout: post
title: "Case insensitive search in Oracle database"
description: "Case insensitive search in Oracle database"
crawlertitle: "Case insensitive search in Oracle database"
date: 2020-01-20 23:59:59 +0800
categories: Database
tags: Database SQL Note
comments: true
---

Scenario: In an Oracle database, when we want to search table which name includes "foo",
but not sure the case of table name.

- Option 1:

```sql
SELECT table_name
FROM user_tables
WHERE regexp_like(table_name, 'foo', 'i')
```

- Option 2:

```sql
SELECT table_name
FROM user_tables
WHERE upper(table_name) LIKE '%FOO%'
```
