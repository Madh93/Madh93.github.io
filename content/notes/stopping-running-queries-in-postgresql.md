---
title: Stopping Running Queries in PostgreSQL
date: 2025-06-19
summary: Find active queries with pg_stat_activity, then cancel them gracefully with pg_cancel_backend(pid) or terminate them forcefully with pg_terminate_backend(pid).
categories: [PostgreSQL, Database, SQL]
---

{{< admonition type="tip" title="TL;DR" >}}
Find active queries with `pg_stat_activity`, then cancel them gracefully with `pg_cancel_backend(pid)` or terminate them forcefully with `pg_terminate_backend(pid)`.
{{< /admonition >}}

Occasionally, a query in PostgreSQL may run for too long, consume excessive resources, or become stuck. When this happens, you need a way to stop it without restarting the entire database. The process involves two main steps: finding the Process ID (PID) of the runaway query and then using a command to cancel or terminate it.

## 1. Find the Query's Process ID (PID)

You can view all currently running activities by querying the `pg_stat_activity` view. To see only the active queries, you can filter by the `state` column.

```sql
SELECT pid, query_start, now() - query_start AS duration, query, state
FROM pg_stat_activity
WHERE state = 'active';
```

This command will list all active queries, showing their process ID (`pid`), when they started, their total `duration`, and the query text itself. Identify the `pid` of the query you wish to stop.

## 2. Cancel the Query (Graceful Method)

The safest way to stop a query is to cancel it using `pg_cancel_backend(pid)`. This function sends a `SIGINT` signal to the backend process, which attempts to stop the specific query gracefully while keeping the database connection alive. The user's session is preserved, and they can run other queries.

```sql
SELECT pg_cancel_backend(<pid of the process>);
```

This should always be your first choice.

## 3. Terminate the Backend (Forceful Method)

If `pg_cancel_backend()` doesn't work or the process is stuck in a state where it cannot be canceled (e.g., idle in a transaction), you can use the more forceful `pg_terminate_backend(pid)`. This function sends a `SIGTERM` signal, which will terminate the entire backend process and close the associated connection.

```sql
SELECT pg_terminate_backend(<pid of the process>);
```

Use this command with caution, as it abruptly ends the entire session, which might cause the client application to see a dropped connection and could lead to transaction rollbacks.
