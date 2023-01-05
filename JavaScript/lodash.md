## pickBy

lodash 的 pickBy 与 Array.prototype.every 差不多

可以直接使用这个方法来删除请求参数中为空的参数



## omit

有些情况 omit 无法删除：

```js
params.forEach(item => {
 item = omit(item, [‘key’])
})
```



omit 方法要使用返回的新对象，因为其不改变原有对象



## get

lodash 的 get 方法的用处：层级较多时，安全取值

https://segmentfault.com/q/1010000037502997

