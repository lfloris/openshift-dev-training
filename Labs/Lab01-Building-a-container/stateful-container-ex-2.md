# Exercise 2 - Deploying a Stateful Container
​
Creat a new directory to use save the content of the database we're going to populate
​
```
$ mkdir -p /tmp/mysql
```
​
Create a container called `mysql-lab`, mounting the container volume to directory we just created using `-v /tmp/mysql/:/var/lib/mysql`. We also set some other environment variables here using `-e VAR_NAME=VAL`.
​
```
$ sudo docker run --name mysql-lab -e MYSQL_USER=user -e MYSQL_PASSWORD=passw0rd -e MYSQL_DATABASE=persistent -e MYSQL_ROOT_PASSWORD=passw0rd -v /tmp/mysql/:/var/lib/mysql -d mysql:latest
```
​
Enter a terminal session to the container using `docker exec` and log to the mysql server
​
```
$ sudo docker exec -it mysql-lab /bin/bash 
$ mysql -uuser -ppassw0rd -h127.0.0.1
```
​
Modify the `persistent` database and populate it with some sample data, then exit mysql and the container.
​
```
mysql> use persistent;
Database changed

mysql> CREATE TABLE LAB (id int(11) NOT NULL,
       name varchar(255) DEFAULT NULL,
       code varchar(255) DEFAULT NULL,
       PRIMARY KEY (id));
​Query OK, 0 rows affected, 1 warning (0.15 sec)

mysql> show tables;
​+----------------------+
| Tables_in_persistent |
+----------------------+
| LAB                  |
+----------------------+
1 row in set (0.01 sec)

mysql> insert into LAB (id, name, code) values (1,'Test','1');
Query OK, 1 row affected (0.08 sec)

mysql> SELECT * FROM LAB;
+----+------+------+
| id | name | code |
+----+------+------+
|  1 | Test | 1    |
+----+------+------+
1 row in set (0.00 sec)

mysql> exit
Bye
root@c5bc46bcede7:/# exit
```

Stop the `mysql-lab` container, and remove it

```
$ sudo docker stop mysql-lab
mysql-lab
$ sudo docker rm mysql-lab
mysql-lab
```

In the previous lab, we would have seen all of this data removed, since containers are ephemeral by nature. This time, we set the `-v` so all the database data should now exist in `/tmp/mysql`.

```
$ ls /tmp/mysql/
 auto.cnf        ca-key.pem          '#ib_16384_1.dblwr'   ibtmp1               persistant        sys
 binlog.000001   ca.pem               ib_buffer_pool      '#innodb_temp'        private_key.pem   undo_001
 binlog.000002   client-cert.pem      ibdata1              mysql                public_key.pem    undo_002
 binlog.000003   client-key.pem       ib_logfile0          mysql.ibd            server-cert.pem
 binlog.index   '#ib_16384_0.dblwr'   ib_logfile1          performance_schema   server-key.pem
```

Now, if we start up another container, we should see that the database entries still exist.

```
$ sudo docker run --name mysql-lab2 -e MYSQL_USER=user -e MYSQL_PASSWORD=passw0rd -e MYSQL_DATABASE=persistent -e MYSQL_ROOT_PASSWORD=passw0rd -v /tmp/mysql/:/var/lib/mysql -d mysql:latest
$ sudo docker exec -it mysql-lab2 /bin/bash 
$ mysql -uuser -ppassw0rd -h127.0.0.1
mysql> use persistent;
Database changed
mysql> show tables;
​+----------------------+
| Tables_in_persistent |
+----------------------+
| LAB                  |
+----------------------+
1 row in set (0.00 sec)
mysql> SELECT * FROM LAB;
+----+------+------+
| id | name | code |
+----+------+------+
|  1 | Test | 1    |
+----+------+------+
1 row in set (0.00 sec)

mysql> exit
Bye
root@55ba46dc4df7:/# exit
```

Stop the `mysql-lab2` container, and remove it

```
$ sudo docker stop mysql-lab2
mysql-lab2
$ sudo docker rm mysql-lab2
mysql-lab2
```

Remove the data in `/tmp/mysql`

```
$ rm -rf /tmp/mysql
```

Lab complete. Please move on to [Networking Containers](networking-containers-ex-3.md)