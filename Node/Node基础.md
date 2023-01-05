## nodemon

实现 nodemon 热更新

https://segmentfault.com/a/1190000021048135

```
nodemon  -e  ts,tsx  --exec ts-node  ./index.ts
```

这里表示监听 `ts, tsx` 文件热更新，如果改成 `ts, js` 的话就可以监听 .js 文件了



## 报错

### Node.js 抛异常ECONNRESET 退出

> 参考：
>
> https://blog.cuicc.com/blog/2017/03/26/nodejs-ECONNRESET/
>
> https://stackoverflow.com/questions/17245881/how-do-i-debug-error-econnreset-in-node-js

根本原因是断开链接了，但是直接原因其实是 node.js 中有一个地方的错误未捕获，所以代码执行中断了，只需要 catch 了这个错误就可以。



### Error: connect EADDRNOTAVAIL

> 参考
>
> https://stackoverflow.com/questions/17588237/error-connect-eaddrnotavail-while-processing-big-async-loop

报错原因是有太多的异步请求了，比如 Promise.all(arr)，这个 arr 请求数组中有
