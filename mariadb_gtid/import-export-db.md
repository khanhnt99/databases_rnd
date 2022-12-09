# Import export database 
## Export database `(thực hiện trên source node)`
- Export database
    + ` mysqldump --user=root --password="password" dbname --single-transaction --add-drop-table | gzip > /op
t/backup_dbname_v1.sql.gz`
- Copy sang node mới 
    + `scp /opt/backup_dbname_v1.sql.gz root@ip:/root/`

## Import database
- Gunzip database file
    + `gunzip backup_cma_v1.sql.gz`
- Import database
    + `mysql -u root -ppassword dbname < backup_dbname_v1.sql`
- Tạo user access vào database
    - `CREATE OR REPLACE USER 'user'@'%' IDENTIFIED BY 'password';`
    - `GRANT ALL PRIVILEGES ON dbname.* TO 'user'@'%';`
