```
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
