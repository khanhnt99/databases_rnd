# Cấu hình Mariadb master_slave GTID 
## 1. Thực hiện trên master node
- `mysql_secure_installation`
- Tạo Replication User
    + `CREATE USER 'repl'@'slave_ip' IDENTIFIED BY 'password';`
    + `GRANT REPLICATION SLAVE ON *.*  TO 'repl'@'slave_ip';`
- Backup the Database
    ```
    mariabackup --backup \
       --target-dir=/var/mariadb/backup/ \
       --user=root --password=mypassword
    ```

- Prepare Database
    ```
    mariabackup --prepare \
       --target-dir=/var/mariadb/backup/
    ```

- Copy Backup database đến slave node
    + Thực hiện trên slave node: `mkdir -p /var/mariadb/backup`
    + `scp -r /var/mariadb/backup/ root@slave_ip:/var/mariadb/backup`
## 2. Thực hiện trên slave node
- Stop mariadb
    + `systemctl stop mysql`
- mv `/var/lib/mysql /opt/mysql_bak`
- Restore backup vào trong thư mục `/var/lib/mysql`
    ```
    mariabackup --copy-back \
    --target-dir=/var/mariadb/backup/
    ```
- `chown -R mysql:mysql /var/lib/mysql/`
- Start mariadb
    + `systemctl start mysql`
#### GTID
```
cat xtrabackup_binlog_info
mariadb-bin.000096 568 0-1-2
```

```
SET GLOBAL gtid_slave_pos = "0-1-2";
CHANGE MASTER TO 
   MASTER_HOST="master_ip", 
   MASTER_PORT=3306, 
   MASTER_USER="repl",  
   MASTER_PASSWORD="password", 
   MASTER_USE_GTID=slave_pos;
START SLAVE;
```
- `show slave status\G`
- Phase sau: Sử dụng MariadbMaxscale để LB các node database
    + https://mariadb.com/kb/en/maxscale/

__Docs__
- https://mariadb.com/kb/en/setting-up-a-replica-with-mariabackup/
- https://sqlconjuror.com/mariadb-setup-gtid-replication-using-mariabackup/