---
layout: post
title: 'Fuzzy text searching and case insensitive matching in Oracle database'
description: 'Fuzzy text searching and case insensitive matching in Oracle database'
crawlertitle: 'Fuzzy text searching and case insensitive matching in Oracle database'
date: 2020-01-20 23:59:59 +0800
categories: Database
tags: Database SQL
comments: true
---

## Scenario

Fuzzy text searching with case-insensitive matching.

The default behavior of `LIKE` and the other comparison operators such as `=` are case-sensitive, unlike PostgresSQL has a nice `ILIKE` keyword (typically a bit slower), work with Oracle database requires a little bit of effort to get this job done.

## Solution

### Option 1: `LIKE` condition

```sql
SELECT table_name
  FROM user_tables
 WHERE LOWER(table_name) LIKE '%foo%'
```

The `LOWER` function here takes a character expression as a parameter, converts all alpha character to lower case and return.

The `LIKE` condition allows you to use wildcards to specify a test involving pattern matching.
Whereas the equality operator (`=`) exactly matches one character value to another.

  - **Wildcard character**  
    The wildcard character is used to substitute one or more characters in a string:
    - "`%`" the percent sign can match 0 or more characters, except `null`.
    - "`_`" the underscore sign in the pattern matches exactly 1 character.

### Option 2: [REGEXP_LIKE Condition](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Pattern-matching-Conditions.html#GUID-D2124F3A-C6E4-4CCA-A40E-2FFCABFD8E19)

```sql
SELECT table_name
  FROM user_tables
 WHERE regexp_like(table_name, 'foo', 'i')
```

The `REGEXP_LIKE` is similar to the `LIKE` condition, except `RREGEXP_LIKE` performs regular expression matching instead of the simple pattern matching performed by `LIKE`.

This condition complies with the POSIX regular expression standard and the Unicode Regular Expression Guidelines,
for more information, refer to [Oracle Regular Expression Support](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Oracle-Regular-Expression-Support.html#GUID-969230D6-FC1A-4C75-BF2A-6B1BE909DED6).

## References

- [REGEXP_LIKE Condition](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Pattern-matching-Conditions.html#GUID-D2124F3A-C6E4-4CCA-A40E-2FFCABFD8E19) - SQL Language Reference from [Oracle Documentation](https://docs.oracle.com/en/)
- [Oracle Regular Expression Support](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/Oracle-Regular-Expression-Support.html#GUID-969230D6-FC1A-4C75-BF2A-6B1BE909DED6) - [Oracle Documentation](https://docs.oracle.com/en/)
