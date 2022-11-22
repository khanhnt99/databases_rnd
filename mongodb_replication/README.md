# Cấu hình mongodb replication
## Bước 1: Cài đặt mongodb 
- Sửa file `/etc/hosts` để chứa hostname và IP của các node mongodb.
    ```
    10.128.0.2 mongo1
    10.128.0.3 mongo2
    10.128.0.4 mongo3
    ```
- https://www.digitalocean.com/community/tutorials/how-to-install-mongodb-on-ubuntu-20-04
## Bước 2: Cấu hình file `mongod.conf`
- Thực hiện trên cả 3 node
    ```
    net:
    bindIp: 127.0.0.1, mongo1
    ...
    replication:
      replSetName: "rs0"
    ```

    ```
    net:
    bindIp: 127.0.0.1, mongo2
    ...
    replication:
      replSetName: "rs0"
    ```

    ```
    net:
    bindIp: 127.0.0.1, mongo3
    ...
    replication:
      replSetName: "rs0"
    ```
- Restart lại mongodb trên cả 3 node 
    + `systemctl restart mongod`
## Bước 3: Khởi tạo mongodb replicaset cluster
```
rs.initiate(
	{
	  _id: "rs0",
	  members: [
	    { _id: 0, host: "mongo1" },
	    { _id: 1, host: "mongo2" },
	    { _id: 2, host: "mongo3" }
	  ]
	}
)
```
- Tạo database people trên primary node
    + `use people`
    + `db.createCollection('people')`
    + `db.getCollection("people").insert({    "firstName": "test",    "lastName": "tech",    "started": NumberInt("2020")})`

- Kiểm tra đồng bộ dữ liệu trên 2 node secondary
    + `mongo`
    + `rs.slaveOk()`
    + `secondaryOk()`
## Bước 4: Cấu hình enable security authentication
- Tạo user admin trên database admin với role "root".
```
db.createUser(
	{  user: "admin",
     pwd: "password",
     roles: [ { role: "root", db: "admin" } ]
  }
)
```
- Check user đã được tạo 
```
rs0:PRIMARY> use admin
switched to db admin
rs0:PRIMARY> db.getUser('admin');
{
        "_id" : "admin.admin",
        "userId" : UUID("6e759d98-2bb3-450e-8d48-d8c02d37fabc"),
        "user" : "admin",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "root",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```
- Kiểm tra xem user đã được authen hay chưa
    ```
    rs0:PRIMARY> db.auth("admin","password")
    1
    ```
- Thêm 2 dòng vào trong file "/etc/mongod" trên tất cả các node
    ```
    security:
      authorization: enabled
      keyFile: /opt/mongo/mongodb-keyfile
    ```
- Tạo keyfile trên primary node
  + `openssl rand -base64 741 > /opt/mongo/mongodb-keyfile`
  + `chmod 600 /opt/mongo/mongodb-keyfile`
  + `chown mongod:mongod /opt/mongo/mongodb-keyfile`
- Copy keyfile từ `primary node` sang các `secondary node` sau đó chmod và chown
    + `chmod 600 /opt/mongo/mongodb-keyfile`
    + `chown mongod:mongod /opt/mongo/mongodb-keyfile`
- Khởi động lại mongodb trên tất cả các node
    + `systemctl restart mongod`

__Docs__
- https://vietcalls.com/huong-dan-cau-hinh-replica-set-cluster-tren-mongodb/
- https://www.disk91.com/2018/technology/systems/centos-7-install-a-mongo-cluster-with-3-replica-set/