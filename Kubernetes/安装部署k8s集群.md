## 部署集群

子节点需要创建 `/etc/kubernetes/admin.conf` 文件，将 master 上的 admin.conf 文件复制到这里，然后执行

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```