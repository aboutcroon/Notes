## 待办

- v-auth 以及其他权限控制
- 滑动到底部懒加载
- 各种 utils



## 问题

### 页面初始化请求与查询请求发送两次会有冲突的问题

从全部资源池节点卡片跳转至服务节点页时，会发送一个带有查询条件的请求，而服务节点页 mounted 中会发送默认查询列表请求，这两个请求如果默认查询列表请求在带有查询条件的请求之后返回，则会导致查询时返回了整个列表，所以此时需要去掉默认的查询列表请求



在 pageSize 改变时，也是这个问题。如果此时带有查询条件，而 pageSize 改变时又是默认的查询，就也会有冲突



antd 的 pageSize 变化时，不仅会触发 onPageSizeChange，还会触发 onPageChange，这里面的两个请求也会冲突，所以统一都使用 onPageChange就好，然后在这里面通过 `if (pagination.pageSize !== this.pageSize)` 来判断是否是 pageSize 发生变化



### k8s yaml 文件的环境变量问题

k8s的 yaml 文件里只能传字符串类型，所以 true 和 false 用 ‘1’ 和 ‘0’ 来表示，node 这边再将字符串转成数字

正常的 yaml 文件和 sh 文件里是可以传其他类型的



+0 => 数字0

+1 => 数字1

+’0’ => 数字0

+’1’ => 数字1

+false => 数字0

+true => 数字1

+’false’ => NaN

+’true’ => NaN



修改后

```js
password: {
  update: process.env.PASSWORD_REGULAR_UPDATE ? +process.env.PASSWORD_REGULAR_UPDATE : 0, // 是否定期更新密码，默认不开启
  interval: process.env.PASSWORD_UPDATE_INTERVAL ? +process.env.PASSWORD_UPDATE_INTERVAL : 90, // 定期检测时间，默认90天
  strength: process.env.PASSWORD_STRENGTH_CHECK ? +process.env.PASSWORD_STRENGTH_CHECK : 0 // 是否开启密码强度校验
}
```



### 哪些数据存储在 vuex

与业务相关的就不要存放在 vuex 中，例如某个页面的列表数据

