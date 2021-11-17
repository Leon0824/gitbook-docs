# 22.2. Creating a Database

In order to create a database, the PostgreSQL server must be up and running (see [Section 18.3](https://www.postgresql.org/docs/13/server-start.html)).

Databases are created with the SQL command [CREATE DATABASE](https://www.postgresql.org/docs/13/sql-createdatabase.html):

```
CREATE DATABASE name;
```

where _`name`_ follows the usual rules for SQL identifiers. The current role automatically becomes the owner of the new database. It is the privilege of the owner of a database to remove it later (which also removes all the objects in it, even if they have a different owner).

The creation of databases is a restricted operation. See [Section 21.2](https://www.postgresql.org/docs/13/role-attributes.html) for how to grant permission.

Since you need to be connected to the database server in order to execute the `CREATE DATABASE` command, the question remains how the _first_ database at any given site can be created. The first database is always created by the `initdb` command when the data storage area is initialized. (See [Section 18.2](https://www.postgresql.org/docs/13/creating-cluster.html).) This database is called `postgres`. So to create the first “ordinary” database you can connect to `postgres`.

A second database, `template1`, is also created during database cluster initialization. Whenever a new database is created within the cluster, `template1` is essentially cloned. This means that any changes you make in `template1` are propagated to all subsequently created databases. Because of this, avoid creating objects in `template1` unless you want them propagated to every newly created database. More details appear in [Section 22.3](https://www.postgresql.org/docs/13/manage-ag-templatedbs.html).

As a convenience, there is a program you can execute from the shell to create new databases, `createdb`.

```
createdb dbname
```

`createdb` does no magic. It connects to the `postgres` database and issues the `CREATE DATABASE` command, exactly as described above. The [createdb](https://www.postgresql.org/docs/13/app-createdb.html) reference page contains the invocation details. Note that `createdb` without any arguments will create a database with the current user name.

#### Note

[Chapter 20](https://www.postgresql.org/docs/13/client-authentication.html) contains information about how to restrict who can connect to a given database.

Sometimes you want to create a database for someone else, and have them become the owner of the new database, so they can configure and manage it themselves. To achieve that, use one of the following commands:

```
CREATE DATABASE dbname OWNER rolename;
```

from the SQL environment, or:

```
createdb -O rolename dbname
```

from the shell. Only the superuser is allowed to create a database for someone else (that is, for a role you are not a member of).\
