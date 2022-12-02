# Cấu hình keydb active-replica

- Sửa file config 
    + `replica-read-only no`
- Sửa `requirepass` 
    + `mypassword123`
- Config active replica
    ```
    masterauth mypassword123
    active-replica yes
    multi-master yes
    replicaof 10.0.0.2 6379
    replicaof 10.0.0.3 6379
    ```
- Restart lại keydb
    + `systemctl restart keydb`
- Check 
    + `keydb-cli`
    + `auth mypassword123`
    + `info replication`

__Docs__
- https://docs.keydb.dev/docs/multi-master