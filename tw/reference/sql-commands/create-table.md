---
description: 版本：11
---

# CREATE TABLE

CREATE TABLE — 定義一個新的資料表

## 語法

```text
CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } | UNLOGGED ] TABLE [ IF NOT EXISTS ] table_name ( [
  { column_name data_type [ COLLATE collation ] [ column_constraint [ ... ] ]
    | table_constraint
    | LIKE source_table [ like_option ... ] }
    [, ... ]
] )
[ INHERITS ( parent_table [, ... ] ) ]
[ PARTITION BY { RANGE | LIST } ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [, ... ] ) ]
[ WITH ( storage_parameter [= value] [, ... ] ) | WITH OIDS | WITHOUT OIDS ]
[ ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP } ]
[ TABLESPACE tablespace_name ]

CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } | UNLOGGED ] TABLE [ IF NOT EXISTS ] table_name
    OF type_name [ (
  { column_name [ WITH OPTIONS ] [ column_constraint [ ... ] ]
    | table_constraint }
    [, ... ]
) ]
[ PARTITION BY { RANGE | LIST } ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [, ... ] ) ]
[ WITH ( storage_parameter [= value] [, ... ] ) | WITH OIDS | WITHOUT OIDS ]
[ ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP } ]
[ TABLESPACE tablespace_name ]

CREATE [ [ GLOBAL | LOCAL ] { TEMPORARY | TEMP } | UNLOGGED ] TABLE [ IF NOT EXISTS ] table_name
    PARTITION OF parent_table [ (
  { column_name [ WITH OPTIONS ] [ column_constraint [ ... ] ]
    | table_constraint }
    [, ... ]
) ] FOR VALUES partition_bound_spec
[ PARTITION BY { RANGE | LIST } ( { column_name | ( expression ) } [ COLLATE collation ] [ opclass ] [, ... ] ) ]
[ WITH ( storage_parameter [= value] [, ... ] ) | WITH OIDS | WITHOUT OIDS ]
[ ON COMMIT { PRESERVE ROWS | DELETE ROWS | DROP } ]
[ TABLESPACE tablespace_name ]

where column_constraint is:

[ CONSTRAINT constraint_name ]
{ NOT NULL |
  NULL |
  CHECK ( expression ) [ NO INHERIT ] |
  DEFAULT default_expr |
  GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY [ ( sequence_options ) ] |
  UNIQUE index_parameters |
  PRIMARY KEY index_parameters |
  REFERENCES reftable [ ( refcolumn ) ] [ MATCH FULL | MATCH PARTIAL | MATCH SIMPLE ]
    [ ON DELETE action ] [ ON UPDATE action ] }
[ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY DEFERRED | INITIALLY IMMEDIATE ]

and table_constraint is:

[ CONSTRAINT constraint_name ]
{ CHECK ( expression ) [ NO INHERIT ] |
  UNIQUE ( column_name [, ... ] ) index_parameters |
  PRIMARY KEY ( column_name [, ... ] ) index_parameters |
  EXCLUDE [ USING index_method ] ( exclude_element WITH operator [, ... ] ) index_parameters [ WHERE ( predicate ) ] |
  FOREIGN KEY ( column_name [, ... ] ) REFERENCES reftable [ ( refcolumn [, ... ] ) ]
    [ MATCH FULL | MATCH PARTIAL | MATCH SIMPLE ] [ ON DELETE action ] [ ON UPDATE action ] }
[ DEFERRABLE | NOT DEFERRABLE ] [ INITIALLY DEFERRED | INITIALLY IMMEDIATE ]

and like_option is:

{ INCLUDING | EXCLUDING } { DEFAULTS | CONSTRAINTS | IDENTITY | INDEXES | STORAGE | COMMENTS | ALL }

and partition_bound_spec is:

IN ( { numeric_literal | string_literal | NULL } [, ...] ) |
FROM ( { numeric_literal | string_literal | MINVALUE | MAXVALUE } [, ...] )
  TO ( { numeric_literal | string_literal | MINVALUE | MAXVALUE } [, ...] )

index_parameters in UNIQUE, PRIMARY KEY, and EXCLUDE constraints are:

[ WITH ( storage_parameter [= value] [, ... ] ) ]
[ USING INDEX TABLESPACE tablespace_name ]

exclude_element in an EXCLUDE constraint is:

{ column_name | ( expression ) } [ opclass ] [ ASC | DESC ] [ NULLS { FIRST | LAST } ]
```

## 說明

CREATE TABLE 將在目前資料庫中建立一個新的，初始化為空的資料表。該資料表將由發出此指令的使用者擁有。

如果加上了綱要名稱（例如，CREATE TABLE myschema.mytable ...），那麼將在指定的綱要中建立資料表。否則，它將在目前綱要中建立。臨時資料表存在於特殊綱要中，因此在建立臨時資料表時無法使用綱要名稱。資料表的名稱必須與同一綱要中的任何其他資料表、序列、索引、檢視表或外部資料表的名稱不同。

CREATE TABLE 還自動建立一個資料型別，表示與資料表的一個資料列對應的複合型別。因此，資料表不能與同一綱要中的任何現有資料型別具有相同的名稱。

可選擇性加上限制條件子句指定新的資料列或更新資料列必須滿足的限制條件才能使其插入或更新操作成功。限制條件是一個 SQL 物件，它有助於以各種方式定義資料表中的有效值集合。

定義限制條件有兩種方法：資料表限制條件和欄位限制條件。欄位限制條件被定義為欄位定義的一部分。資料表限制條件定義不依賴於特定欄位，它可以包含多個欄位。 每個欄位限制條件也可以寫為資料表限制條件；欄位限制條件只是在其限制僅影響一欄位時使用的語法方便。

為了能夠建立資料表，您必須分別對所有欄位型別或 OF 子句中的型別具有 USAGE 權限。

## 參數

`TEMPORARY` 或 `TEMP`

如果使用此參數，則將資料表建立為臨時資料表。臨時資料表會在連線結束時自動刪除，或者選擇性地在目前交易事務結束時刪除（請參閱下面的 ON COMMIT）。當臨時資料表存在時，目前連線不會顯示具有相同名稱的現有永久資料表，除非它們使用綱要限定的名稱引用。在臨時資料表上建立的任何索引也都自動是臨時的。

由於 autovacuum 背景程序無法存取，因此無法對臨時資料表進行清理或分析。所以，應透過線上的 SQL 命令執行適當的清理和分析操作。例如，如果要在複雜查詢中使用臨時資料表，在填入資料後的臨時表上執行 ANALYZE 是個不錯的作法。

選擇性地，可以在 TEMPORARY 或 TEMP 之前寫入 GLOBAL 或 LOCAL。目前這在 PostgreSQL 中沒有任何區別，也已經被棄用；請參閱[相容性](create-table.md#xiang-rong-xing)。

`UNLOGGED`

如果指定了這個選項，則將此表建立為無日誌記錄的資料表。寫入無日誌記錄資料表的資料不寫入 WAL（見[第 30 章](https://github.com/pgsql-tw/gitbook-docs/tree/67cc71691219133f37b9a33df9c691a2dd9c2642/tw/server-administration/30.-gao-ke-kao-du-ji-yu-xie-ri-zhi)），這使得它們比普通的資料表快得多。但是，它們就不是完全安全的：在系統崩潰或不正常關閉之後，會自動清除無日誌記錄的資料表。 無日誌記錄的資料表內容也無法複製到備用伺服器。在無日誌記錄資料表上所建的所有索引也沒有日誌記錄。

`IF NOT EXISTS`

Do not throw an error if a relation with the same name already exists. A notice is issued in this case. Note that there is no guarantee that the existing relation is anything like the one that would have been created.

_`table_name`_

The name \(optionally schema-qualified\) of the table to be created.

`OF` _`type_name`_

Creates a _typed table_, which takes its structure from the specified composite type \(name optionally schema-qualified\). A typed table is tied to its type; for example the table will be dropped if the type is dropped \(with `DROP TYPE ... CASCADE`\).

When a typed table is created, then the data types of the columns are determined by the underlying composite type and are not specified by the `CREATE TABLE` command. But the `CREATE TABLE` command can add defaults and constraints to the table and can specify storage parameters.

`PARTITION OF` _`parent_table`_ FOR VALUES _`partition_bound_spec`_

Creates the table as a _partition_ of the specified parent table.

The _`partition_bound_spec`_ must correspond to the partitioning method and partition key of the parent table, and must not overlap with any existing partition of that parent. The form with `IN` is used for list partitioning, while the form with `FROM` and `TO` is used for range partitioning.

Each of the values specified in the _`partition_bound_spec`_ is a literal, `NULL`, `MINVALUE`, or `MAXVALUE`. Each literal value must be either a numeric constant that is coercible to the corresponding partition key column's type, or a string literal that is valid input for that type.

When creating a list partition, `NULL` can be specified to signify that the partition allows the partition key column to be null. However, there cannot be more than one such list partition for a given parent table. `NULL` cannot be specified for range partitions.

When creating a range partition, the lower bound specified with `FROM` is an inclusive bound, whereas the upper bound specified with `TO` is an exclusive bound. That is, the values specified in the `FROM` list are valid values of the corresponding partition key columns for this partition, whereas those in the `TO` list are not. Note that this statement must be understood according to the rules of row-wise comparison \([Section 9.23.5](https://www.postgresql.org/docs/10/static/functions-comparisons.html#ROW-WISE-COMPARISON)\). For example, given `PARTITION BY RANGE (x,y)`, a partition bound `FROM (1, 2) TO (3, 4)` allows `x=1` with any`y>=2`, `x=2` with any non-null `y`, and `x=3` with any `y<4`.

The special values `MINVALUE` and `MAXVALUE` may be used when creating a range partition to indicate that there is no lower or upper bound on the column's value. For example, a partition defined using `FROM (MINVALUE) TO (10)` allows any values less than 10, and a partition defined using `FROM (10) TO (MAXVALUE)` allows any values greater than or equal to 10.

When creating a range partition involving more than one column, it can also make sense to use `MAXVALUE` as part of the lower bound, and `MINVALUE` as part of the upper bound. For example, a partition defined using `FROM (0, MAXVALUE) TO (10, MAXVALUE)` allows any rows where the first partition key column is greater than 0 and less than or equal to 10. Similarly, a partition defined using `FROM ('a', MINVALUE) TO ('b', MINVALUE)` allows any rows where the first partition key column starts with "a".

Note that if `MINVALUE` or `MAXVALUE` is used for one column of a partitioning bound, the same value must be used for all subsequent columns. For example, `(10, MINVALUE, 0)` is not a valid bound; you should write `(10, MINVALUE, MINVALUE)`.

Also note that some element types, such as `timestamp`, have a notion of "infinity", which is just another value that can be stored. This is different from `MINVALUE` and `MAXVALUE`, which are not real values that can be stored, but rather they are ways of saying that the value is unbounded. `MAXVALUE` can be thought of as being greater than any other value, including "infinity" and `MINVALUE` as being less than any other value, including "minus infinity". Thus the range `FROM ('infinity') TO (MAXVALUE)` is not an empty range; it allows precisely one value to be stored — "infinity".

A partition must have the same column names and types as the partitioned table to which it belongs. If the parent is specified `WITH OIDS` then all partitions must have OIDs; the parent's OID column will be inherited by all partitions just like any other column. Modifications to the column names or types of a partitioned table, or the addition or removal of an OID column, will automatically propagate to all partitions. `CHECK` constraints will be inherited automatically by every partition, but an individual partition may specify additional `CHECK` constraints; additional constraints with the same name and condition as in the parent will be merged with the parent constraint. Defaults may be specified separately for each partition.

Rows inserted into a partitioned table will be automatically routed to the correct partition. If no suitable partition exists, an error will occur. Also, if updating a row in a given partition would require it to move to another partition due to new partition key values, an error will occur.

Operations such as TRUNCATE which normally affect a table and all of its inheritance children will cascade to all partitions, but may also be performed on an individual partition. Note that dropping a partition with `DROP TABLE` requires taking an `ACCESS EXCLUSIVE` lock on the parent table.

_`column_name`_

The name of a column to be created in the new table.

_`data_type`_

The data type of the column. This can include array specifiers. For more information on the data types supported by PostgreSQL, refer to [Chapter 8](https://www.postgresql.org/docs/10/static/datatype.html).

`COLLATE` _`collation`_

The `COLLATE` clause assigns a collation to the column \(which must be of a collatable data type\). If not specified, the column data type's default collation is used.

`INHERITS (` _`parent_table`_ \[, ... \] \)

The optional `INHERITS` clause specifies a list of tables from which the new table automatically inherits all columns. Parent tables can be plain tables or foreign tables.

Use of `INHERITS` creates a persistent relationship between the new child table and its parent table\(s\). Schema modifications to the parent\(s\) normally propagate to children as well, and by default the data of the child table is included in scans of the parent\(s\).

If the same column name exists in more than one parent table, an error is reported unless the data types of the columns match in each of the parent tables. If there is no conflict, then the duplicate columns are merged to form a single column in the new table. If the column name list of the new table contains a column name that is also inherited, the data type must likewise match the inherited column\(s\), and the column definitions are merged into one. If the new table explicitly specifies a default value for the column, this default overrides any defaults from inherited declarations of the column. Otherwise, any parents that specify default values for the column must all specify the same default, or an error will be reported.

`CHECK` constraints are merged in essentially the same way as columns: if multiple parent tables and/or the new table definition contain identically-named `CHECK` constraints, these constraints must all have the same check expression, or an error will be reported. Constraints having the same name and expression will be merged into one copy. A constraint marked `NO INHERIT` in a parent will not be considered. Notice that an unnamed `CHECK` constraint in the new table will never be merged, since a unique name will always be chosen for it.

Column `STORAGE` settings are also copied from parent tables.

If a column in the parent table is an identity column, that property is not inherited. A column in the child table can be declared identity column if desired.

`PARTITION BY { RANGE | LIST } ( {` _`column_name`_ \| \( _`expression`_ \) } \[ _`opclass`_ \] \[, ...\] \)

The optional `PARTITION BY` clause specifies a strategy of partitioning the table. The table thus created is called a _partitioned_ table. The parenthesized list of columns or expressions forms the _partition key_ for the table. When using range partitioning, the partition key can include multiple columns or expressions \(up to 32, but this limit can be altered when building PostgreSQL\), but for list partitioning, the partition key must consist of a single column or expression. If no B-tree operator class is specified when creating a partitioned table, the default B-tree operator class for the datatype will be used. If there is none, an error will be reported.

A partitioned table is divided into sub-tables \(called partitions\), which are created using separate `CREATE TABLE` commands. The partitioned table is itself empty. A data row inserted into the table is routed to a partition based on the value of columns or expressions in the partition key. If no existing partition matches the values in the new row, an error will be reported.

Partitioned tables do not support `UNIQUE`, `PRIMARY KEY`, `EXCLUDE`, or `FOREIGN KEY` constraints; however, you can define these constraints on individual partitions.

`LIKE` _`source_table`_ \[ _`like_option`_ ... \]

The `LIKE` clause specifies a table from which the new table automatically copies all column names, their data types, and their not-null constraints.

Unlike `INHERITS`, the new table and original table are completely decoupled after creation is complete. Changes to the original table will not be applied to the new table, and it is not possible to include data of the new table in scans of the original table.

Default expressions for the copied column definitions will be copied only if `INCLUDING DEFAULTS` is specified. The default behavior is to exclude default expressions, resulting in the copied columns in the new table having null defaults. Note that copying defaults that call database-modification functions, such as `nextval`, may create a functional linkage between the original and new tables.

Any identity specifications of copied column definitions will only be copied if `INCLUDING IDENTITY` is specified. A new sequence is created for each identity column of the new table, separate from the sequences associated with the old table.

Not-null constraints are always copied to the new table. `CHECK` constraints will be copied only if `INCLUDING CONSTRAINTS` is specified. No distinction is made between column constraints and table constraints.

Indexes, `PRIMARY KEY`, `UNIQUE`, and `EXCLUDE` constraints on the original table will be created on the new table only if `INCLUDING INDEXES` is specified. Names for the new indexes and constraints are chosen according to the default rules, regardless of how the originals were named. \(This behavior avoids possible duplicate-name failures for the new indexes.\)

`STORAGE` settings for the copied column definitions will be copied only if `INCLUDING STORAGE` is specified. The default behavior is to exclude `STORAGE` settings, resulting in the copied columns in the new table having type-specific default settings. For more on `STORAGE`settings, see [Section 66.2](https://www.postgresql.org/docs/10/static/storage-toast.html).

Comments for the copied columns, constraints, and indexes will be copied only if `INCLUDING COMMENTS` is specified. The default behavior is to exclude comments, resulting in the copied columns and constraints in the new table having no comments.

`INCLUDING ALL` is an abbreviated form of `INCLUDING DEFAULTS INCLUDING IDENTITY INCLUDING CONSTRAINTS INCLUDING INDEXES INCLUDING STORAGE INCLUDING COMMENTS`.

Note that unlike `INHERITS`, columns and constraints copied by `LIKE` are not merged with similarly named columns and constraints. If the same name is specified explicitly or in another `LIKE` clause, an error is signaled.

The `LIKE` clause can also be used to copy column definitions from views, foreign tables, or composite types. Inapplicable options \(e.g., `INCLUDING INDEXES` from a view\) are ignored.

`CONSTRAINT` _`constraint_name`_

An optional name for a column or table constraint. If the constraint is violated, the constraint name is present in error messages, so constraint names like `col must be positive` can be used to communicate helpful constraint information to client applications. \(Double-quotes are needed to specify constraint names that contain spaces.\) If a constraint name is not specified, the system generates a name.

`NOT NULL`

The column is not allowed to contain null values.

`NULL`

The column is allowed to contain null values. This is the default.

This clause is only provided for compatibility with non-standard SQL databases. Its use is discouraged in new applications.

`CHECK (` _`expression`_ \) \[ NO INHERIT \]

The `CHECK` clause specifies an expression producing a Boolean result which new or updated rows must satisfy for an insert or update operation to succeed. Expressions evaluating to TRUE or UNKNOWN succeed. Should any row of an insert or update operation produce a FALSE result, an error exception is raised and the insert or update does not alter the database. A check constraint specified as a column constraint should reference that column's value only, while an expression appearing in a table constraint can reference multiple columns.

Currently, `CHECK` expressions cannot contain subqueries nor refer to variables other than columns of the current row. The system column `tableoid` may be referenced, but not any other system column.

A constraint marked with `NO INHERIT` will not propagate to child tables.

When a table has multiple `CHECK` constraints, they will be tested for each row in alphabetical order by name, after checking `NOT NULL` constraints. \(PostgreSQL versions before 9.5 did not honor any particular firing order for `CHECK` constraints.\)

`DEFAULT` _`default_expr`_

The `DEFAULT` clause assigns a default data value for the column whose column definition it appears within. The value is any variable-free expression \(subqueries and cross-references to other columns in the current table are not allowed\). The data type of the default expression must match the data type of the column.

The default expression will be used in any insert operation that does not specify a value for the column. If there is no default for a column, then the default is null.

`GENERATED { ALWAYS | BY DEFAULT } AS IDENTITY [ (` _`sequence_options`_ \) \]

This clause creates the column as an _identity column_. It will have an implicit sequence attached to it and the column in new rows will automatically have values from the sequence assigned to it.

The clauses `ALWAYS` and `BY DEFAULT` determine how the sequence value is given precedence over a user-specified value in an `INSERT` statement. If `ALWAYS` is specified, a user-specified value is only accepted if the `INSERT` statement specifies `OVERRIDING SYSTEM VALUE`. If `BY DEFAULT` is specified, then the user-specified value takes precedence. See [INSERT](https://www.postgresql.org/docs/10/static/sql-insert.html) for details. \(In the `COPY` command, user-specified values are always used regardless of this setting.\)

The optional _`sequence_options`_ clause can be used to override the options of the sequence. See [CREATE SEQUENCE](https://www.postgresql.org/docs/10/static/sql-createsequence.html) for details.

`UNIQUE` \(column constraint\)  
`UNIQUE (` _`column_name`_ \[, ... \] \) \(table constraint\)

The `UNIQUE` constraint specifies that a group of one or more columns of a table can contain only unique values. The behavior of the unique table constraint is the same as that for column constraints, with the additional capability to span multiple columns.

For the purpose of a unique constraint, null values are not considered equal.

Each unique table constraint must name a set of columns that is different from the set of columns named by any other unique or primary key constraint defined for the table. \(Otherwise it would just be the same constraint listed twice.\)

`PRIMARY KEY` \(column constraint\)  
`PRIMARY KEY (` _`column_name`_ \[, ... \] \) \(table constraint\)

The `PRIMARY KEY` constraint specifies that a column or columns of a table can contain only unique \(non-duplicate\), nonnull values. Only one primary key can be specified for a table, whether as a column constraint or a table constraint.

The primary key constraint should name a set of columns that is different from the set of columns named by any unique constraint defined for the same table. \(Otherwise, the unique constraint is redundant and will be discarded.\)

`PRIMARY KEY` enforces the same data constraints as a combination of `UNIQUE` and `NOT NULL`, but identifying a set of columns as the primary key also provides metadata about the design of the schema, since a primary key implies that other tables can rely on this set of columns as a unique identifier for rows.

`EXCLUDE [ USING` _`index_method`_ \] \( _`exclude_element`_ WITH _`operator`_ \[, ... \] \) _`index_parameters`_ \[ WHERE \( _`predicate`_ \) \]

The `EXCLUDE` clause defines an exclusion constraint, which guarantees that if any two rows are compared on the specified column\(s\) or expression\(s\) using the specified operator\(s\), not all of these comparisons will return `TRUE`. If all of the specified operators test for equality, this is equivalent to a `UNIQUE` constraint, although an ordinary unique constraint will be faster. However, exclusion constraints can specify constraints that are more general than simple equality. For example, you can specify a constraint that no two rows in the table contain overlapping circles \(see [Section 8.8](https://www.postgresql.org/docs/10/static/datatype-geometric.html)\) by using the `&&` operator.

Exclusion constraints are implemented using an index, so each specified operator must be associated with an appropriate operator class \(see [Section 11.9](https://www.postgresql.org/docs/10/static/indexes-opclass.html)\) for the index access method _`index_method`_. The operators are required to be commutative. Each \_`exclude_element`\_can optionally specify an operator class and/or ordering options; these are described fully under [CREATE INDEX](https://www.postgresql.org/docs/10/static/sql-createindex.html).

The access method must support `amgettuple` \(see [Chapter 60](https://www.postgresql.org/docs/10/static/indexam.html)\); at present this means GIN cannot be used. Although it's allowed, there is little point in using B-tree or hash indexes with an exclusion constraint, because this does nothing that an ordinary unique constraint doesn't do better. So in practice the access method will always be GiST or SP-GiST.

The _`predicate`_ allows you to specify an exclusion constraint on a subset of the table; internally this creates a partial index. Note that parentheses are required around the predicate.

`REFERENCES` _`reftable`_ \[ \( _`refcolumn`_ \) \] \[ MATCH _`matchtype`_ \] \[ ON DELETE _`action`_ \] \[ ON UPDATE _`action`_ \] \(column constraint\)  
`FOREIGN KEY (` _`column_name`_ \[, ... \] \) REFERENCES _`reftable`_ \[ \( _`refcolumn`_ \[, ... \] \) \] \[ MATCH _`matchtype`_ \] \[ ON DELETE _`action`_ \] \[ ON UPDATE _`action`_ \] \(table constraint\)

These clauses specify a foreign key constraint, which requires that a group of one or more columns of the new table must only contain values that match values in the referenced column\(s\) of some row of the referenced table. If the _`refcolumn`_ list is omitted, the primary key of the _`reftable`_ is used. The referenced columns must be the columns of a non-deferrable unique or primary key constraint in the referenced table. The user must have `REFERENCES` permission on the referenced table \(either the whole table, or the specific referenced columns\). Note that foreign key constraints cannot be defined between temporary tables and permanent tables.

A value inserted into the referencing column\(s\) is matched against the values of the referenced table and referenced columns using the given match type. There are three match types: `MATCH FULL`, `MATCH PARTIAL`, and `MATCH SIMPLE` \(which is the default\). `MATCH FULL` will not allow one column of a multicolumn foreign key to be null unless all foreign key columns are null; if they are all null, the row is not required to have a match in the referenced table. `MATCH SIMPLE` allows any of the foreign key columns to be null; if any of them are null, the row is not required to have a match in the referenced table. `MATCH PARTIAL` is not yet implemented. \(Of course, `NOT NULL` constraints can be applied to the referencing column\(s\) to prevent these cases from arising.\)

In addition, when the data in the referenced columns is changed, certain actions are performed on the data in this table's columns. The `ON DELETE` clause specifies the action to perform when a referenced row in the referenced table is being deleted. Likewise, the `ON UPDATE` clause specifies the action to perform when a referenced column in the referenced table is being updated to a new value. If the row is updated, but the referenced column is not actually changed, no action is done. Referential actions other than the `NO ACTION`check cannot be deferred, even if the constraint is declared deferrable. There are the following possible actions for each clause:

`NO ACTION`

Produce an error indicating that the deletion or update would create a foreign key constraint violation. If the constraint is deferred, this error will be produced at constraint check time if there still exist any referencing rows. This is the default action.

`RESTRICT`

Produce an error indicating that the deletion or update would create a foreign key constraint violation. This is the same as `NO ACTION` except that the check is not deferrable.

`CASCADE`

Delete any rows referencing the deleted row, or update the values of the referencing column\(s\) to the new values of the referenced columns, respectively.

`SET NULL`

Set the referencing column\(s\) to null.

`SET DEFAULT`

Set the referencing column\(s\) to their default values. \(There must be a row in the referenced table matching the default values, if they are not null, or the operation will fail.\)

If the referenced column\(s\) are changed frequently, it might be wise to add an index to the referencing column\(s\) so that referential actions associated with the foreign key constraint can be performed more efficiently.

`DEFERRABLE`  
`NOT DEFERRABLE`

This controls whether the constraint can be deferred. A constraint that is not deferrable will be checked immediately after every command. Checking of constraints that are deferrable can be postponed until the end of the transaction \(using the [SET CONSTRAINTS](https://www.postgresql.org/docs/10/static/sql-set-constraints.html)command\). `NOT DEFERRABLE` is the default. Currently, only `UNIQUE`, `PRIMARY KEY`, `EXCLUDE`, and `REFERENCES` \(foreign key\) constraints accept this clause. `NOT NULL` and `CHECK` constraints are not deferrable. Note that deferrable constraints cannot be used as conflict arbitrators in an `INSERT` statement that includes an `ON CONFLICT DO UPDATE` clause.

`INITIALLY IMMEDIATE`  
`INITIALLY DEFERRED`

If a constraint is deferrable, this clause specifies the default time to check the constraint. If the constraint is `INITIALLY IMMEDIATE`, it is checked after each statement. This is the default. If the constraint is `INITIALLY DEFERRED`, it is checked only at the end of the transaction. The constraint check time can be altered with the [SET CONSTRAINTS](https://www.postgresql.org/docs/10/static/sql-set-constraints.html) command.

`WITH (` _`storage_parameter`_ \[= _`value`_\] \[, ... \] \)

This clause specifies optional storage parameters for a table or index; see [Storage Parameters](https://www.postgresql.org/docs/10/static/sql-createtable.html#SQL-CREATETABLE-STORAGE-PARAMETERS) for more information. The `WITH` clause for a table can also include `OIDS=TRUE` \(or just `OIDS`\) to specify that rows of the new table should have OIDs \(object identifiers\) assigned to them, or `OIDS=FALSE` to specify that the rows should not have OIDs. If `OIDS` is not specified, the default setting depends upon the [default\_with\_oids](https://www.postgresql.org/docs/10/static/runtime-config-compatible.html#GUC-DEFAULT-WITH-OIDS) configuration parameter. \(If the new table inherits from any tables that have OIDs, then `OIDS=TRUE` is forced even if the command says `OIDS=FALSE`.\)

If `OIDS=FALSE` is specified or implied, the new table does not store OIDs and no OID will be assigned for a row inserted into it. This is generally considered worthwhile, since it will reduce OID consumption and thereby postpone the wraparound of the 32-bit OID counter. Once the counter wraps around, OIDs can no longer be assumed to be unique, which makes them considerably less useful. In addition, excluding OIDs from a table reduces the space required to store the table on disk by 4 bytes per row \(on most machines\), slightly improving performance.

To remove OIDs from a table after it has been created, use [ALTER TABLE](https://www.postgresql.org/docs/10/static/sql-altertable.html).

`WITH OIDS`  
`WITHOUT OIDS`

These are obsolescent syntaxes equivalent to `WITH (OIDS)` and `WITH (OIDS=FALSE)`, respectively. If you wish to give both an `OIDS` setting and storage parameters, you must use the `WITH ( ... )` syntax; see above.

`ON COMMIT`

The behavior of temporary tables at the end of a transaction block can be controlled using `ON COMMIT`. The three options are:

`PRESERVE ROWS`

No special action is taken at the ends of transactions. This is the default behavior.

`DELETE ROWS`

All rows in the temporary table will be deleted at the end of each transaction block. Essentially, an automatic [TRUNCATE](https://www.postgresql.org/docs/10/static/sql-truncate.html) is done at each commit.

`DROP`

The temporary table will be dropped at the end of the current transaction block.

`TABLESPACE` _`tablespace_name`_

tablespace\_name 是要在其中建立新資料表的資料表空間名稱。如果未指定，則會使用 [default\_tablespace](../../server-administration/server-configuration/19.11.-yong-hu-duan-lian-xian-yu-she-can-shu.md#default_tablespace-string)，如果此資料表是臨時資料表，則為使用 [temp\_tablespaces](../../server-administration/server-configuration/19.11.-yong-hu-duan-lian-xian-yu-she-can-shu.md#temp_tablespaces-string)。

`USING INDEX TABLESPACE` _`tablespace_name`_

此子句允許選擇與其建立的 UNIQUE，PRIMARY KEY 或 EXCLUDE 限制條件約束關連索引的資料表空間。如果未指定，則使用 [default\_tablespace](../../server-administration/server-configuration/19.11.-yong-hu-duan-lian-xian-yu-she-can-shu.md#default_tablespace-string)，如果此表是臨時資料表，則為 [temp\_tablespaces](../../server-administration/server-configuration/19.11.-yong-hu-duan-lian-xian-yu-she-can-shu.md#temp_tablespaces-string)。

### Storage Parameters

The `WITH` clause can specify _storage parameters_ for tables, and for indexes associated with a `UNIQUE`, `PRIMARY KEY`, or `EXCLUDE` constraint. Storage parameters for indexes are documented in [CREATE INDEX](https://www.postgresql.org/docs/10/static/sql-createindex.html). The storage parameters currently available for tables are listed below. For many of these parameters, as shown, there is an additional parameter with the same name prefixed with `toast.`, which controls the behavior of the table's secondary TOAST table, if any \(see [Section 66.2](https://www.postgresql.org/docs/10/static/storage-toast.html) for more information about TOAST\). If a table parameter value is set and the equivalent `toast.` parameter is not, the TOAST table will use the table's parameter value. Specifying these parameters for partitioned tables is not supported, but you may specify them for individual leaf partitions.

`fillfactor` \(`integer`\)

The fillfactor for a table is a percentage between 10 and 100. 100 \(complete packing\) is the default. When a smaller fillfactor is specified, `INSERT` operations pack table pages only to the indicated percentage; the remaining space on each page is reserved for updating rows on that page. This gives `UPDATE` a chance to place the updated copy of a row on the same page as the original, which is more efficient than placing it on a different page. For a table whose entries are never updated, complete packing is the best choice, but in heavily updated tables smaller fillfactors are appropriate. This parameter cannot be set for TOAST tables.

`parallel_workers` \(`integer`\)

This sets the number of workers that should be used to assist a parallel scan of this table. If not set, the system will determine a value based on the relation size. The actual number of workers chosen by the planner may be less, for example due to the setting of [max\_worker\_processes](https://www.postgresql.org/docs/10/static/runtime-config-resource.html#GUC-MAX-WORKER-PROCESSES).

`autovacuum_enabled`, `toast.autovacuum_enabled` \(`boolean`\)

Enables or disables the autovacuum daemon for a particular table. If true, the autovacuum daemon will perform automatic `VACUUM` and/or `ANALYZE` operations on this table following the rules discussed in [Section 24.1.6](https://www.postgresql.org/docs/10/static/routine-vacuuming.html#AUTOVACUUM). If false, this table will not be autovacuumed, except to prevent transaction ID wraparound. See [Section 24.1.5](https://www.postgresql.org/docs/10/static/routine-vacuuming.html#VACUUM-FOR-WRAPAROUND) for more about wraparound prevention. Note that the autovacuum daemon does not run at all \(except to prevent transaction ID wraparound\) if the [autovacuum](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM) parameter is false; setting individual tables' storage parameters does not override that. Therefore there is seldom much point in explicitly setting this storage parameter to `true`, only to `false`.

`autovacuum_vacuum_threshold`, `toast.autovacuum_vacuum_threshold` \(`integer`\)

Per-table value for [autovacuum\_vacuum\_threshold](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-THRESHOLD) parameter.

`autovacuum_vacuum_scale_factor`, `toast.autovacuum_vacuum_scale_factor` \(`float4`\)

Per-table value for [autovacuum\_vacuum\_scale\_factor](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-SCALE-FACTOR) parameter.

`autovacuum_analyze_threshold` \(`integer`\)

Per-table value for [autovacuum\_analyze\_threshold](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-ANALYZE-THRESHOLD) parameter.

`autovacuum_analyze_scale_factor` \(`float4`\)

Per-table value for [autovacuum\_analyze\_scale\_factor](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-ANALYZE-SCALE-FACTOR) parameter.

`autovacuum_vacuum_cost_delay`, `toast.autovacuum_vacuum_cost_delay` \(`integer`\)

Per-table value for [autovacuum\_vacuum\_cost\_delay](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-COST-DELAY) parameter.

`autovacuum_vacuum_cost_limit`, `toast.autovacuum_vacuum_cost_limit` \(`integer`\)

Per-table value for [autovacuum\_vacuum\_cost\_limit](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-VACUUM-COST-LIMIT) parameter.

`autovacuum_freeze_min_age`, `toast.autovacuum_freeze_min_age` \(`integer`\)

Per-table value for [vacuum\_freeze\_min\_age](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-VACUUM-FREEZE-MIN-AGE) parameter. Note that autovacuum will ignore per-table `autovacuum_freeze_min_age` parameters that are larger than half the system-wide [autovacuum\_freeze\_max\_age](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-FREEZE-MAX-AGE) setting.

`autovacuum_freeze_max_age`, `toast.autovacuum_freeze_max_age` \(`integer`\)

Per-table value for [autovacuum\_freeze\_max\_age](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-FREEZE-MAX-AGE) parameter. Note that autovacuum will ignore per-table `autovacuum_freeze_max_age` parameters that are larger than the system-wide setting \(it can only be set smaller\).

`autovacuum_freeze_table_age`, `toast.autovacuum_freeze_table_age` \(`integer`\)

Per-table value for [vacuum\_freeze\_table\_age](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-VACUUM-FREEZE-TABLE-AGE) parameter.

`autovacuum_multixact_freeze_min_age`, `toast.autovacuum_multixact_freeze_min_age` \(`integer`\)

Per-table value for [vacuum\_multixact\_freeze\_min\_age](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-VACUUM-MULTIXACT-FREEZE-MIN-AGE) parameter. Note that autovacuum will ignore per-table `autovacuum_multixact_freeze_min_age` parameters that are larger than half the system-wide [autovacuum\_multixact\_freeze\_max\_age](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-MULTIXACT-FREEZE-MAX-AGE) setting.

`autovacuum_multixact_freeze_max_age`, `toast.autovacuum_multixact_freeze_max_age` \(`integer`\)

Per-table value for [autovacuum\_multixact\_freeze\_max\_age](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-AUTOVACUUM-MULTIXACT-FREEZE-MAX-AGE) parameter. Note that autovacuum will ignore per-table `autovacuum_multixact_freeze_max_age` parameters that are larger than the system-wide setting \(it can only be set smaller\).

`autovacuum_multixact_freeze_table_age`, `toast.autovacuum_multixact_freeze_table_age` \(`integer`\)

Per-table value for [vacuum\_multixact\_freeze\_table\_age](https://www.postgresql.org/docs/10/static/runtime-config-client.html#GUC-VACUUM-MULTIXACT-FREEZE-TABLE-AGE) parameter.

`log_autovacuum_min_duration`, `toast.log_autovacuum_min_duration` \(`integer`\)

Per-table value for [log\_autovacuum\_min\_duration](https://www.postgresql.org/docs/10/static/runtime-config-autovacuum.html#GUC-LOG-AUTOVACUUM-MIN-DURATION) parameter.

`user_catalog_table` \(`boolean`\)

Declare the table as an additional catalog table for purposes of logical replication. See [Section 48.6.2](https://www.postgresql.org/docs/10/static/logicaldecoding-output-plugin.html#LOGICALDECODING-CAPABILITIES) for details. This parameter cannot be set for TOAST tables.

## Notes

Using OIDs in new applications is not recommended: where possible, using an identity column or other sequence generator as the table's primary key is preferred. However, if your application does make use of OIDs to identify specific rows of a table, it is recommended to create a unique constraint on the `oid` column of that table, to ensure that OIDs in the table will indeed uniquely identify rows even after counter wraparound. Avoid assuming that OIDs are unique across tables; if you need a database-wide unique identifier, use the combination of `tableoid` and row OID for the purpose.

### Tip

The use of `OIDS=FALSE` is not recommended for tables with no primary key, since without either an OID or a unique data key, it is difficult to identify specific rows.

PostgreSQL automatically creates an index for each unique constraint and primary key constraint to enforce uniqueness. Thus, it is not necessary to create an index explicitly for primary key columns. \(See [CREATE INDEX](https://www.postgresql.org/docs/10/static/sql-createindex.html) for more information.\)

Unique constraints and primary keys are not inherited in the current implementation. This makes the combination of inheritance and unique constraints rather dysfunctional.

A table cannot have more than 1600 columns. \(In practice, the effective limit is usually lower because of tuple-length constraints.\)

## 範例

建立資料表 flims 和資料表 distributors：

```text
CREATE TABLE films (
    code        char(5) CONSTRAINT firstkey PRIMARY KEY,
    title       varchar(40) NOT NULL,
    did         integer NOT NULL,
    date_prod   date,
    kind        varchar(10),
    len         interval hour to minute
);

CREATE TABLE distributors (
     did    integer PRIMARY KEY GENERATED BY DEFAULT AS IDENTITY,
     name   varchar(40) NOT NULL CHECK (name <> '')
);
```

建立一個包含二維陣列的資料表：

```text
CREATE TABLE array_int (
    vector  int[][]
);
```

為資料表 films 定義唯一性的資料表限制條件。可以在資料表的一個欄位或多個欄位上定義唯一性的資料表限制條件：

```text
CREATE TABLE films (
    code        char(5),
    title       varchar(40),
    did         integer,
    date_prod   date,
    kind        varchar(10),
    len         interval hour to minute,
    CONSTRAINT production UNIQUE(date_prod)
);
```

定義檢查欄位的限制條件：

```text
CREATE TABLE distributors (
    did     integer CHECK (did > 100),
    name    varchar(40)
);
```

定義 CHECK 資料表限制條件：

```text
CREATE TABLE distributors (
    did     integer,
    name    varchar(40),
    CONSTRAINT con1 CHECK (did > 100 AND name <> '')
);
```

為資料表 films 定義主鍵的資料表限制條件：

```text
CREATE TABLE films (
    code        char(5),
    title       varchar(40),
    did         integer,
    date_prod   date,
    kind        varchar(10),
    len         interval hour to minute,
    CONSTRAINT code_title PRIMARY KEY(code,title)
);
```

為資料表 distributors 定義主鍵限制條件。以下兩個範例是等效的，第一個使用資料表限制條件語法，第二個是欄位限制條件語法：

```text
CREATE TABLE distributors (
    did     integer,
    name    varchar(40),
    PRIMARY KEY(did)
);

CREATE TABLE distributors (
    did     integer PRIMARY KEY,
    name    varchar(40)
);
```

為欄位名稱指定文字常數預設值，透過以序列物件的下一個值來安排要産生的欄位預設值，並使預設值 modtime 成為插入資料列的時間：

```text
CREATE TABLE distributors (
    name      varchar(40) DEFAULT 'Luso Films',
    did       integer DEFAULT nextval('distributors_serial'),
    modtime   timestamp DEFAULT current_timestamp
);
```

在資料表 distributors 上定義兩個 NOT NULL 欄位限制條件，其中一個明確地設定了名稱：

```text
CREATE TABLE distributors (
    did     integer CONSTRAINT no_null NOT NULL,
    name    varchar(40) NOT NULL
);
```

為 name 欄位定義唯一性限制條件：

```text
CREATE TABLE distributors (
    did     integer,
    name    varchar(40) UNIQUE
);
```

同樣，但指定為資料表限制條件：

```text
CREATE TABLE distributors (
    did     integer,
    name    varchar(40),
    UNIQUE(name)
);
```

建立相同的資料表，為資料表及其唯一性索引指定 70％ 填充因子：

```text
CREATE TABLE distributors (
    did     integer,
    name    varchar(40),
    UNIQUE(name) WITH (fillfactor=70)
)
WITH (fillfactor=70);
```

使用排除限制條件建立資料表 circles，以防止任何兩個 circle 重疊：

```text
CREATE TABLE circles (
    c circle,
    EXCLUDE USING gist (c WITH &&)
);
```

在資料表空間 diskvol1 中建立資料表 cinemas：

```text
CREATE TABLE cinemas (
        id serial,
        name text,
        location text
) TABLESPACE diskvol1;
```

建立複合型別和該型別的資料表：

```text
CREATE TYPE employee_type AS (name text, salary numeric);

CREATE TABLE employees OF employee_type (
    PRIMARY KEY (name),
    salary WITH OPTIONS DEFAULT 1000
);
```

建立區間型的分割資料表：

```text
CREATE TABLE measurement (
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (logdate);
```

在分割主鍵中建立一個包含多個欄位的區間分割資料表：

```text
CREATE TABLE measurement_year_month (
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (EXTRACT(YEAR FROM logdate), EXTRACT(MONTH FROM logdate));
```

建立列表型分割資料表：

```text
CREATE TABLE cities (
    city_id      bigserial not null,
    name         text not null,
    population   bigint
) PARTITION BY LIST (left(lower(name), 1));
```

建立區間型分割資料表的分割區：

```text
CREATE TABLE measurement_y2016m07
    PARTITION OF measurement (
    unitsales DEFAULT 0
) FOR VALUES FROM ('2016-07-01') TO ('2016-08-01');
```

在分割主鍵中建立具有多個欄位的區間型分割資料表的幾個分割區：

```text
CREATE TABLE measurement_ym_older
    PARTITION OF measurement_year_month
    FOR VALUES FROM (MINVALUE, MINVALUE) TO (2016, 11);

CREATE TABLE measurement_ym_y2016m11
    PARTITION OF measurement_year_month
    FOR VALUES FROM (2016, 11) TO (2016, 12);

CREATE TABLE measurement_ym_y2016m12
    PARTITION OF measurement_year_month
    FOR VALUES FROM (2016, 12) TO (2017, 01);

CREATE TABLE measurement_ym_y2017m01
    PARTITION OF measurement_year_month
    FOR VALUES FROM (2017, 01) TO (2017, 02);
```

建立列表分割資料表的分割區：

```text
CREATE TABLE cities_ab
    PARTITION OF cities (
    CONSTRAINT city_id_nonzero CHECK (city_id != 0)
) FOR VALUES IN ('a', 'b');
```

建立列表型分割資料表的分割區，該資料表本身進一步進行分區，然後向其加上分割區：

```text
CREATE TABLE cities_ab
    PARTITION OF cities (
    CONSTRAINT city_id_nonzero CHECK (city_id != 0)
) FOR VALUES IN ('a', 'b') PARTITION BY RANGE (population);

CREATE TABLE cities_ab_10000_to_100000
    PARTITION OF cities_ab FOR VALUES FROM (10000) TO (100000);
```

## 相容性

CREATE TABLE 命令基本上符合 SQL 標準，而下面列出了一些例外情況。

### Temporary Tables

儘管 CREATE TEMPORARY TABLE 的語法類似於 SQL 標準的語法，但效果卻不盡相同。在標準中，臨時資料表只定義一次，並在每個需要它們的連線中自動存在（以空內容開始）。 而 PostgreSQL 則要求每個連線為要使用的每個臨時資料表發出自己的 CREATE TEMPORARY TABLE 命令。這使得不同的連線可以為不同的目的使用相同的臨時資料表名稱，而標準的方法限制了給定臨時資料表名稱的所有物件具有相同的資料表結構。

標準對臨時資料表行為的定義大部份都被忽略。PostgreSQL 在這一點上的行為類似於其他幾個 SQL 資料庫。

SQL 標準還區分全域和區域的臨時資料表，其中區域的臨時資料表為每個連線中的每個 SQL 區塊都有一組單獨的內容，儘管它的定義仍然在連線之間共享。由於 PostgreSQL 不支援 SQL 區塊，因此這種區別與 PostgreSQL 無關。

為了相容性，PostgreSQL 將在臨時資料表宣告中接受 GLOBAL 和 LOCAL 關鍵字，但它們目前沒有任何效果。並不鼓勵使用這些關鍵字，因為 PostgreSQL 的未來版本可能採用更符合標準的方式來解譯。

臨時資料表的 ON COMMIT 子句也類似於 SQL 標準，但有一些差異。 如果省略 ON COMMIT 子句，則 SQL 指定預設行為為 ON COMMIT DELETE ROWS。但是，PostgreSQL 中的預設行為是 ON COMMIT PRESERVE ROWS。SQL 中不存在 ON COMMIT DROP 語法。

### Non-deferred Uniqueness Constraints

When a `UNIQUE` or `PRIMARY KEY` constraint is not deferrable, PostgreSQL checks for uniqueness immediately whenever a row is inserted or modified. The SQL standard says that uniqueness should be enforced only at the end of the statement; this makes a difference when, for example, a single command updates multiple key values. To obtain standard-compliant behavior, declare the constraint as `DEFERRABLE` but not deferred \(i.e., `INITIALLY IMMEDIATE`\). Be aware that this can be significantly slower than immediate uniqueness checking.

### Column Check Constraints

The SQL standard says that `CHECK` column constraints can only refer to the column they apply to; only `CHECK` table constraints can refer to multiple columns. PostgreSQL does not enforce this restriction; it treats column and table check constraints alike.

### `EXCLUDE` Constraint

The `EXCLUDE` constraint type is a PostgreSQL extension.

### `NULL` “Constraint”

The `NULL` “constraint” \(actually a non-constraint\) is a PostgreSQL extension to the SQL standard that is included for compatibility with some other database systems \(and for symmetry with the `NOT NULL` constraint\). Since it is the default for any column, its presence is simply noise.

### Inheritance

Multiple inheritance via the `INHERITS` clause is a PostgreSQL language extension. SQL:1999 and later define single inheritance using a different syntax and different semantics. SQL:1999-style inheritance is not yet supported by PostgreSQL.

### Zero-column Tables

PostgreSQL allows a table of no columns to be created \(for example, `CREATE TABLE foo();`\). This is an extension from the SQL standard, which does not allow zero-column tables. Zero-column tables are not in themselves very useful, but disallowing them creates odd special cases for `ALTER TABLE DROP COLUMN`, so it seems cleaner to ignore this spec restriction.

### Multiple Identity Columns

PostgreSQL allows a table to have more than one identity column. The standard specifies that a table can have at most one identity column. This is relaxed mainly to give more flexibility for doing schema changes or migrations. Note that the `INSERT` command supports only one override clause that applies to the entire statement, so having multiple identity columns with different behaviors is not well supported.

### `LIKE` Clause

While a `LIKE` clause exists in the SQL standard, many of the options that PostgreSQL accepts for it are not in the standard, and some of the standard's options are not implemented by PostgreSQL.

### `WITH` Clause

The `WITH` clause is a PostgreSQL extension; neither storage parameters nor OIDs are in the standard.

### Tablespaces

The PostgreSQL concept of tablespaces is not part of the standard. Hence, the clauses `TABLESPACE` and `USING INDEX TABLESPACE` are extensions.

### Typed Tables

Typed tables implement a subset of the SQL standard. According to the standard, a typed table has columns corresponding to the underlying composite type as well as one other column that is the “self-referencing column”. PostgreSQL does not support these self-referencing columns explicitly, but the same effect can be had using the OID feature.

### `PARTITION BY` Clause

The `PARTITION BY` clause is a PostgreSQL extension.

## 參閱

[ALTER TABLE](alter-table.md), [DROP TABLE](drop-table.md), [CREATE TABLE AS](create-table-as.md), [CREATE TABLESPACE](create-tablespace.md), [CREATE TYPE](create-type.md)

