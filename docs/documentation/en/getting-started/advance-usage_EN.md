# Advanced Use Guide

Here we introduce some of Doris's advanced features.

## Table 1 Structural Change

Schema of the table can be modified using the ALTER TABLE command, including the following modifications:

* Additional columns
* Delete columns
* Modify column type
* Changing column order

Examples are given below.

Schema of Table 1 is as follows:

```
+----------+-------------+------+-------+---------+-------+
| Field    | Type        | Null | Key   | Default | Extra |
+----------+-------------+------+-------+---------+-------+
| siteid   | int(11)     | No   | true  | 10      |       |
| citycode | smallint(6) | No   | true  | N/A     |       |
1242, 1242, 1246, Varchar (32), 1244, 1244, True'124; 124;
S/PV.124; PV.124; Bigint (20); No.124; fals_124; 0_124; Sum_124;
+----------+-------------+------+-------+---------+-------+
```

We added a new column of uv, type BIGINT, aggregation type SUM, default value is 0:

`ALTER TABLE table1 ADD COLUMN uv BIGINT SUM DEFAULT '0' after pv;`

After successful submission, you can view the progress of the job by following commands:

`SHOW ALTER TABLE COLUMN;`

When the job state is FINISHED, the job is completed. The new Schema is in force.

After ALTER TABLE is completed, you can view the latest Schema through `DESC TABLE'.

```
mysql> DESC table1;
+----------+-------------+------+-------+---------+-------+
| Field    | Type        | Null | Key   | Default | Extra |
+----------+-------------+------+-------+---------+-------+
| siteid   | int(11)     | No   | true  | 10      |       |
| citycode | smallint(6) | No   | true  | N/A     |       |
1242, 1242, 1246, Varchar (32), 1244, 1244, True'124; 124;
S/PV.124; PV.124; Bigint (20); No.124; fals_124; 0_124; Sum_124;
UV Bigint (20) False 0 Sum 20 False 0 0 Sum 20 False 20 False False 0 Ultraviolet Ultraviolet Ultraviolet Ultraviolet
+----------+-------------+------+-------+---------+-------+
5 rows in set (0.00 sec)
```

The following command can be used to cancel the job currently being executed:

`CANCEL ALTER TABLE COLUMN FROM table1`

For more help, see `HELP ALTER TABLE'.

## 2 Rollup

Rollup can be understood as a physical index structure of Table. ** Physicalization ** is because its data is physically stored independently, and ** indexing ** means that Rollup can adjust column order to increase the hit rate of prefix index, or reduce key column to increase data aggregation.

Examples are given below.

Schema of Table 1 is as follows:

```
+----------+-------------+------+-------+---------+-------+
| Field    | Type        | Null | Key   | Default | Extra |
+----------+-------------+------+-------+---------+-------+
| siteid   | int(11)     | No   | true  | 10      |       |
| citycode | smallint(6) | No   | true  | N/A     |       |
1242, 1242, 1246, Varchar (32), 1244, 1244, True'124; 124;
S/PV.124; PV.124; Bigint (20); No.124; fals_124; 0_124; Sum_124;
UV Bigint (20) False 0 Sum 20 False 0 0 Sum 20 False 20 False False 0 Ultraviolet Ultraviolet Ultraviolet Ultraviolet
+----------+-------------+------+-------+---------+-------+
```

For table1 detailed data, siteid, citycode and username form a set of keys, which aggregate the PV field. If the business side often has the need to see the total amount of PV in the city, it can build a rollup with only citycode and pv.

`ALTER TABLE table1 ADD ROLLUP rollup_city(citycode, pv);`

After successful submission, you can view the progress of the job by following commands:

`SHOW ALTER TABLE ROLLUP;`

When the job state is FINISHED, the job is completed.

When Rollup is established, you can use `DESC table1 ALL'to view the Rollup information of the table.

```
mysql> desc table1 all;
+-------------+----------+-------------+------+-------+--------+-------+
| IndexName   | Field    | Type        | Null | Key   | Default | Extra |
+-------------+----------+-------------+------+-------+---------+-------+
| table1      | siteid   | int(11)     | No   | true  | 10      |       |
|             | citycode | smallint(6) | No   | true  | N/A     |       |
1.1.2.1.2.1.1.2.2.1.1.1.1.2.1.1.2 124; "Username" - 124; Varchar (32) 124; No.124; True_124; "124";
Regulation 124; 1246; 12420; 12424; 12444; 1244; 1240; 1240; 1244;
UV = 124; UV = 124; Bigint (20) False 0 Sum
|             |          |             |      |       |         |       |
| rollup_city | citycode | smallint(6) | No   | true  | N/A     |       |
Regulation 124; 1246; 12420; 12424; 12444; 1244; 1240; 1240; 1244;
+-------------+----------+-------------+------+-------+---------+-------+
8 rows in set (0.01 sec)
```

The following command can be used to cancel the job currently being executed:

`CANCEL ALTER TABLE ROLLUP FROM table1;`

After Rollup is established, the query does not need to specify Rollup to query. Or specify the original table for query. The program automatically determines whether Rollup should be used. Whether Rollup is hit or not can be viewed by the `EXPLAIN your_sql;'command.

For more help, see `HELP ALTER TABLE'.

## 2 Query of Data Table

### 2.1 Memory Limitation

To prevent a user's query from consuming too much memory. Queries are controlled in memory. A query task uses no more than 2GB of memory by default on a single BE node.

When users use it, if they find a `Memory limit exceeded'error, they usually exceed the memory limit.

Users should try to optimize their SQL statements when they encounter memory overrun.

If it is found that 2GB memory cannot be satisfied, the memory parameters can be set manually.

Display query memory limits:

```
mysql> SHOW VARIABLES LIKE "%mem_limit%";
+---------------+------------+
1.1.1.1.2.2.1.2.2.2.2.2.1.1 Amended as follows:
+---------------+------------+
*124th; execution *limit *124; 21474848 *124;
+---------------+------------+
1 row in set (0.00 sec)
```

` The unit of exec_mem_limit is byte, and the value of `exec_mem_limit'can be changed by the `SET' command. If changed to 8GB.

`SET exec_mem_limit = 8589934592;`

```
mysql> SHOW VARIABLES LIKE "%mem_limit%";
+---------------+------------+
1.1.1.1.2.2.1.2.2.2.2.2.1.1 Amended as follows:
+---------------+------------+
124th; self -imposed limit 124; 8589934592
+---------------+------------+
1 row in set (0.00 sec)
```

>* The above modification is session level and is only valid within the current connection session. Disconnecting and reconnecting will change back to the default value.
>* If you need to modify the global variable, you can set it as follows: `SET GLOBAL exec_mem_limit = 8589934592;` When the setup is complete, disconnect the session and log in again, and the parameters will take effect permanently.

### 2.2 Query timeout

The current default query time is set to 300 seconds. If a query is not completed within 300 seconds, the query will be cancelled by the Doris system. Users can use this parameter to customize the timeout time of their applications and achieve a blocking mode similar to wait (timeout).

View the current timeout settings:

```
mysql> SHOW VARIABLES LIKE "%query_timeout%";
+---------------+-------+
1.1.1.1.2.2.1.2.2.2.2.2.1.1 Amended as follows:
+---------------+-------+
124QUERY WENT TIMEOUT $124; 300 1244;
+---------------+-------+
1 row in set (0.00 sec)
```

Modify the timeout to 1 minute:

SET query timeout =60;'

>* The current timeout check interval is 5 seconds, so timeouts less than 5 seconds are not very accurate.
>* The above modifications are also session level. Global validity can be modified by `SET GLOBAL'.

35; "3535; 2.3 broadcast / shufle join"

By default, the system implements Join by conditionally filtering small tables, broadcasting them to the nodes where the large tables are located, forming a memory Hash table, and then streaming out the data of the large tables Hash Join. However, if the amount of data filtered by small tables can not be put into memory, Join will not be able to complete at this time. The usual error should be caused by memory overrun first.

If you encounter the above situation, it is recommended to use Shuffle Join, also known as Partitioned Join. That is, small and large tables are Hash according to Join's key, and then distributed Join. This memory consumption is allocated to all computing nodes in the cluster.

Use Broadcast Join (default):

```
mysql> select sum(table1.pv) from table1 join table2 where table1.siteid = 2;
+--------------------+
+ 124; Sum (`Table1'`PV `124);
+--------------------+
|                 10 |
+--------------------+
1 row in set (0.20 sec)
```

Use Broadcast Join (explicitly specified):

```
mysql> select sum(table1.pv) from table1 join [broadcast] table2 where table1.siteid = 2;
+--------------------+
+ 124; Sum (`Table1'`PV `124);
+--------------------+
|                 10 |
+--------------------+
1 row in set (0.20 sec)
```

20351;29992929292Join:

```
mysql> select sum(table1.pv) from table1 join [shuffle] table2 where table1.siteid = 2;
+--------------------+
+ 124; Sum (`Table1'`PV `124);
+--------------------+
|                 10 |
+--------------------+
1 row in set (0.15 sec)
```

### 2.4 Query Retry and High Availability

When multiple FE nodes are deployed, users can deploy load balancing layers on top of multiple FEs to achieve high availability of Doris.

Here are some highly available solutions:

** The first **

I retry and load balancing in application layer code. For example, if a connection is found to be dead, it will automatically retry on other connections. Application-level code retry requires the application to configure multiple Doris front-end node addresses.

** Second **

If you use MySQL JDBC connector to connect Doris, you can use jdbc's automatic retry mechanism:

```
jdbc:mysql:/[host:port],[host:port].../[database][? propertyName1][=propertyValue1][&propertyName2][=propertyValue2]...
```

** The third **

Applications can connect to and deploy MySQL Proxy on the same machine by configuring MySQL Proxy's Failover and Load Balance functions.

`http://dev.mysql.com /doc /refman /5.6 /en /mysql -proxy -using.html `