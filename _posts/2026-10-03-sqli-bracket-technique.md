---
layout: post
title: "SQL Injection Without Spaces Using Brackets"
tags: SQL Injection
excerpt_separator: <!--more-->
---

In some SQL injection vulnerabilities, the environment prevents spaces in a payload. This can happen because of URL encoding (spaces become `%20`) or other input constraints. This article shows you how to get around this using a novel escaping technique.<!--more-->

## Common Bypasses

SQL injection references often recommend SQL comments to replace whitespace:

```sql
SELECT/*avoid-spaces*/password/**/FROM/**/Members
```

## A Bracketing Technique

Parentheses can also separate parts of a query without spaces. The previous example can be rewritten as:

```sql
(SELECT(password))FROM(Members)
```

## Example

During a penetration test, I encountered a host that rewrote URL parameters and effectively blocked spaces. I used an error-based SQL injection payload with an XPath expression and the bracketing technique:

```sql
index'AND'1'=(extractvalue(rand(),concat(0x3a,(select(substr(group_concat(user),1,10))from(mysql.user)))))AND'1'='1"
```

![A space-free payload returning database user data through an error-based SQL injection](/assets/images/posts/sql-injection-without-spaces-bracket-technique/grafik-11.png)

The bracketing technique can be combined with other encodings and injection payloads when spaces or comments are restricted.
