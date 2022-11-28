- Tạo mới user 
```
use admin;
db.createUser(
  {  user: "username",
     pwd: "password",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```


```
mongo --username username --password --authenticationDatabase admin --host host --port 27017
```

- Thêm role cho user 
```
db.grantRolesToUser('ktarget', [{ role: 'readWrite', db: 'ktarget' }])
```

- Kết nối tới database 
```
mongo "mongodb://user:password@10.15.13.111/db_name?authSource=admin"
```

- Bỏ role khỏi user 
```
db.revokeRolesFromUser("admin",[ {role: "readWrite", db:"db_name"} ])
```
