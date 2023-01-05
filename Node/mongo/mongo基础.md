## 创建admin用户

进入mongo容器，admin数据库

```sh
docker exec -it mongo mongo admin
```

创建一个具有所有数据库所有权限的用户

```sh
db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
```

尝试使用上面创建的用户信息进行连接

```sh
use admin
db.auth('admin', '123456')
```



## mongoose使用admin账户连接其他数据库

使用 `authSource` 参数

```sh
mongodb://admin:123456@${mongoUrl}/jira?authSource=admin
```

