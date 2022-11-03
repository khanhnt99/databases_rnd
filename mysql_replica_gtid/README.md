# Cấu hình mysql replication GITD 
## 1. Công cụ sử dụng
- `Percona XtraBackup 8.0` -> ứng với version mysql 8.0

## 2. Các bước thực hiện
### 2.1.Trên master node
- Cài đặt `Percona XtraBackup 8.0`
    + `wget https://downloads.percona.com/downloads/Percona-XtraBackup-8.0/Percona-XtraBackup-8.0.29-22/binary/debian/focal/x86_64/percona-xtrabackup-80_8.0.29-22-1.focal_amd64.deb`
    + `dpkg -i percona-xtrabackup-80_8.0.29-22-1.focal_amd64.deb`
- Thực hiện backup trên master node
    + `xtrabackup --backup --user=root --password=password --port=3306 --target-dir=/data/backups_date/ --socket=/var/run/mysqld/mysqld.sock --no-server-version-check` -> `xtrabackup: completed OK!`
    + Check file `xtrabackup_binlog_info`
- Prepare bản backup 
    + `xtrabackup --prepare --user=root --password=password --port=3306 --target-dir=/data/backups_3_11/ --socket=/var/run/mysqld/mysqld.sock --no-server-version-check`
- Copy bản backup tới replica server
    + `rsync -avpP -e ssh /data/backups_date root@ip_slave:/root/backup_from_master`
- Tạo User replica 
    + `CREATE USER 'repl'@'ip_slave' IDENTIFIED BY 'password_slave';`
    + `GRANT REPLICATION SLAVE ON *.*  TO 'repl'@'ip_slave';`

### 2.2. Trên replica node
- Stop mysql trên replica node
    + `systemctl stop mysql`
- Move datadir cũ sang 1 nơi khác
    + `mkdir -p /backup/db-bkp/mysql/mysql-data-bakup`
    + `mv /var/lib/mysql /backup/db-bkp/mysql/mysql-data-bakup`
- Move dữ liệu từ được copy từ bên master đến folder `/var/lib/mysql` bên slave
    + `mv /root/backup_from_master /var/lib/mysql`
- Chuyển permisstion của folder `/var/lib/mysql` mới được copy
    + `chown -R mysql:mysql /var/lib/mysql`
- Start lại mysql trên replica node
    + `systemctl restart mysql`
- Cấu hình và restart replication
    ```
    RESET MASTER;
    SET GLOBAL gtid_purged='<gtid_string_found_in_xtrabackup_binlog_info>';
    CHANGE REPLICATION SOURCE TO
             SOURCE_HOST="ip_master_node",
             SOURCE_USER="repl",
             SOURCE_PASSWORD="password",
             SOURCE_AUTO_POSITION = 1;
    ```
- Check replication status
    + `SHOW REPLICA STATUS\G`

__Docs__
- https://docs.percona.com/percona-xtrabackup/8.0/howtos/recipes_ibkx_gtid.html
- https://minervadb.com/index.php/setup-mysql-slave-replication-with-percona-xtrabackup/
