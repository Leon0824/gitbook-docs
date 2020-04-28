# 27.1. Standard Unix Tools

On most Unix platforms, PostgreSQL modifies its command title as reported by `ps`, so that individual server processes can readily be identified. A sample display is

```text
$ ps auxww | grep ^postgres
postgres  15551  0.0  0.1  57536  7132 pts/0    S    18:02   0:00 postgres -i
postgres  15554  0.0  0.0  57536  1184 ?        Ss   18:02   0:00 postgres: writer process
postgres  15555  0.0  0.0  57536   916 ?        Ss   18:02   0:00 postgres: checkpointer process
postgres  15556  0.0  0.0  57536   916 ?        Ss   18:02   0:00 postgres: wal writer process
postgres  15557  0.0  0.0  58504  2244 ?        Ss   18:02   0:00 postgres: autovacuum launcher process
postgres  15558  0.0  0.0  17512  1068 ?        Ss   18:02   0:00 postgres: stats collector process
postgres  15582  0.0  0.0  58772  3080 ?        Ss   18:04   0:00 postgres: joe runbug 127.0.0.1 idle
postgres  15606  0.0  0.0  58772  3052 ?        Ss   18:07   0:00 postgres: tgl regression [local] SELECT waiting
postgres  15610  0.0  0.0  58772  3056 ?        Ss   18:07   0:00 postgres: tgl regression [local] idle in transaction
```

\(The appropriate invocation of `ps` varies across different platforms, as do the details of what is shown. This example is from a recent Linux system.\) The first process listed here is the master server process. The command arguments shown for it are the same ones used when it was launched. The next five processes are background worker processes automatically launched by the master process. \(The “stats collector” process will not be present if you have set the system not to start the statistics collector; likewise the “autovacuum launcher” process can be disabled.\) Each of the remaining processes is a server process handling one client connection. Each such process sets its command line display in the form

```text
postgres: user database host activity
```

The user, database, and \(client\) host items remain the same for the life of the client connection, but the activity indicator changes. The activity can be `idle` \(i.e., waiting for a client command\), `idle in transaction` \(waiting for client inside a `BEGIN` block\), or a command type name such as `SELECT`. Also, `waiting` is appended if the server process is presently waiting on a lock held by another session. In the above example we can infer that process 15606 is waiting for process 15610 to complete its transaction and thereby release some lock. \(Process 15610 must be the blocker, because there is no other active session. In more complicated cases it would be necessary to look into the [`pg_locks`](https://www.postgresql.org/docs/10/static/view-pg-locks.html)system view to determine who is blocking whom.\)

If [cluster\_name](https://www.postgresql.org/docs/10/static/runtime-config-logging.html#GUC-CLUSTER-NAME) has been configured the cluster name will also be shown in `ps` output:

```text
$ psql -c 'SHOW cluster_name'
 cluster_name
--------------
 server1
(1 row)

$ ps aux|grep server1
postgres   27093  0.0  0.0  30096  2752 ?        Ss   11:34   0:00 postgres: server1: writer process
...
```

If you have turned off [update\_process\_title](https://www.postgresql.org/docs/10/static/runtime-config-logging.html#GUC-UPDATE-PROCESS-TITLE) then the activity indicator is not updated; the process title is set only once when a new process is launched. On some platforms this saves a measurable amount of per-command overhead; on others it's insignificant.

## Tip

Solaris requires special handling. You must use `/usr/ucb/ps`, rather than `/bin/ps`. You also must use two `w` flags, not just one. In addition, your original invocation of the `postgres`command must have a shorter `ps` status display than that provided by each server process. If you fail to do all three things, the `ps` output for each server process will be the original `postgres` command line.

