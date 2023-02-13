

## Axios



### axios 封装

```js
// src/common/axios/index.js

/**
 * 1. 提供一个默认配置的axios实例，同时暴露factory，支持特殊业务生成特殊axios实例
 * 2. 错误请求统一拦截
 * 3. 默认情况下，axios将JavaScript对象序列化为JSON, 更改为默认发送application/x-www-form-urlencode
 * 4. 配置常用请求 get delete post(multipart/form-data) post(application/x-www-form-urlencode) postJson(aplication/json)
 * 5. 默认get请求，参数会多params一层，重写get、delete请求，去掉params
 *    axios.get('/user', params)
 * 6. 处理network error，三种情况，cors，https证书，没有网络，线上只存在https证书问题，所以这里network error一般指证书问题
 */
import axios from 'axios'
import qs from 'qs'
import get from 'lodash/get'
import { error } from '@/common/message'

/**
   * 拦截器中抛出的错误或者未被捕获的错误会被catch
   * 未被捕获的错误统一处理为'Unknown Error'
   * 错误被 catch 后不会再抛出
   */
function _catch (e) {
  return {
    code: 10010,
    msg: 'Unknown Error',
    error: e
  }
}

// 跳转至登录页
function _login () {
  localStorage.removeItem('tokenInfo')
  localStorage.removeItem('userInfo')
  location.href = '/login'
}

/**
   * 所有请求都会进入_then方法
   * 所有拦截的错误，在这里进行提示，并不会返回到业务层，也不作提示
   * 只有正确的 response，会被 resolve，进入到业务层
   * 或当传入了 receiveError = true 时，错误会被 reject 到业务层
   */
function _then (resolve, reject, receiveError = false) {
  return (res) => {
    if (res.code >= 10000) {
      // 所有错误都统一打印到控制台
      console.error(res)

      if (res.code !== 10005) { // 请求取消不提示
        error(typeof res.msg === 'object' ? JSON.stringify(res.msg) : res.msg)
      }
      if (res.code === 10401) {
        return _login()
      }
      if (receiveError) {
        // 当 receiveError = true 时，错误会被 reject 到业务层
        reject(res)
      }
    } else {
      resolve(res)
    }
  }
}

export const CancelToken = axios.CancelToken
const instance = axios.create({
  timeout: 10 * 1000,
  validateStatus: function (status) {
    // 返回 `true` (或者设置为 `null` 或 `undefined`) promise 将被 resolve
    // 否则，promise 将被 reject
    return status >= 200 && status < 300
  }
})

instance.interceptors.request.use((option) => {
  // 处理 post put 请求数据
  const method = option.method.toLowerCase()
  const contentType = option.headers['Content-Type'] || option.headers['content-type']
  // 兼容不同类型的 contentType
  if (['post', 'put'].indexOf(method) > -1 && contentType && contentType.indexOf('x-www-form-urlencoded') > -1) {
    option.data = qs.stringify(option.data)
  }
  // 打个标记，代表该请求为 ajax 异步请求
  // 页面渲染通常是同步的请求
  option.headers['X-Requested-With'] = 'XMLHttpRequest'
  return option
}, function (error) {
  return Promise.reject(error)
})

instance.interceptors.response.use((response) => {
  const { data } = response
  /**
   * 200表示请求成功且有返回值，如果没有返回值，说明解析失败
   */
  return data
}, function (error) {
  // error 是一个对象 { config, request, response, message, code, isAxiosError }
  console.dir(error)
  // 超时请求处理
  if (error.code === 'ECONNABORTED' && error.message.indexOf('timeout') !== -1) {
    return {
      code: 10003,
      msg: 'Timeout'
    }
  }
  if (error.message === 'Network Error') {
    return {
      code: 10004,
      msg: 'Network Error'
    }
  }
  if (axios.isCancel(error)) {
    return {
      code: 10005,
      msg: 'Request Canceled'
    }
  }
  if (error.response && error.response.status === 400) {
    return {
      code: 10400,
      msg: `${get(error, 'response.statusText')}：${get(error, 'response.data.message') || get(error, 'response.data')}`
    }
  }
  if (error.response && error.response.status === 404) {
    return {
      code: 10404,
      msg: 'Not Found'
    }
  }
  if (error.response && error.response.status === 401) {
    return {
      code: 10401,
      msg: 'Not Login'
    }
  }
  if (error.response && error.response.status >= 500) {
    return {
      code: 10500,
      msg: get(error, 'response.data') || 'Server Error'
    }
  }
  
  // 以上错误都不符合时，为未知错误，此时将之抛出
  throw error
})

export default function (options) {
  return new Promise((resolve, reject) => {
    instance(options)
      .catch(_catch) // 将未知错误先 catch，再返回 10010 code，并进入 then 中
      .then(_then(resolve, reject, options.receiveError))
  })
}

```

我们在 axios 封装里，报错的处理都自己处理，`只给业务层返回 response.data 中的数据`，这样用户层就不用再使用 response.status 之类的重复性的工作 



后台返回的 response 的结构如下

```json
{
  code: xxx,
  data: {
    ...
  }
}
```

但 axios 包装的 response 返回回来是不一样的结构，后台的 response 会被 axios 包装在 data 中

https://github.com/axios/axios#response-schema



### request 请求封装

```js
// src/common/request/index.js

/* eslint-disable no-unused-vars */
/* eslint-disable camelcase */
import axios from '@/common/axios'
import store from '@/store'
import * as api from './api'

const urlRegExp = /:\w+/g

const request = {}
for (const i in api) {
  // option 为传入的参数对象
  request[i] = function (option = {}) {
    // i: 接口对象名，api[i]: 整个接口对象
    const initOption = api[i]
    const { proxy, method, url } = initOption
    let mock = false
    const axiosOption = {
      method,
      url,
      headers: {}
    }

    /**
     * 替换url上的参数，例如 /api/get/:id
     */
    if (urlRegExp.test(url)) {
      const newUrl = url.replace(urlRegExp, (match) => {
        const variable = match.substr(1) // 例如 ':id' 中的 'id'
        const params = option.params
        if (!params || typeof params !== 'object') {
          return match
        }
        // 取出真实传入的 params 中的值进行替换
        const value = params[variable]
        delete params[variable]
        return value
      })
      if (urlRegExp.test(newUrl)) {
        throw new Error('invalid request url')
      }
      axiosOption.url = newUrl
    }

    /**
     * 处理转发的请求与mock
     */
    // 处理mock
    if ('mock' in initOption) {
      mock = initOption.mock
    } else {
      mock = process.env.NODE_ENV === 'mock'
    }
    if (mock) {
      // 这里默认yapi的mock scope为11，如果有特殊，再做兼容
      axiosOption.url = '/mock' + axiosOption.url
    } else if (proxy) {
      // 需要转发的请求
      if (proxy === 'prom') {
        const prom = store.state.config.prom
        Object.assign(axiosOption.headers, {
          'q-host': prom.host,
          'q-port': prom.port
        })
      } else if (proxy === 'rootService') {
        const rootService = store.state.config.rootService
        Object.assign(axiosOption.headers, {
          'q-host': rootService.host,
          'q-port': rootService.port
        })
      } else if (proxy === 'messageService') {
        const messageService = store.state.config.messageService
        Object.assign(axiosOption.headers, {
          'q-host': messageService.host,
          'q-port': messageService.port
        })
      } else if (proxy === 'controller') {
        const controller = store.state.controller || {}
        const { api_version, data_version, ip, port } = controller
        Object.assign(axiosOption.headers, {
          'q-host': ip,
          'q-port': port,
          'api-version': api_version,
          'data-version': data_version
        })
      }
      if (process.env.NODE_ENV === 'development') {
        axiosOption.url = '/proxy' + axiosOption.url
      }
    }

    // 将 option 中的其余参数放入 axiosOption
    for (const o in option) {
      if (o !== 'headers') {
        axiosOption[o] = option[o]
      } else {
        if (!axiosOption.headers) {
          axiosOption.headers = {}
        }
        Object.assign(axiosOption.headers, option.headers)
      }
    }

    return axios(axiosOption).then(res => {
      if (process.env.NODE_ENV === 'development' && initOption.transResponse && typeof initOption.transResponse === 'function') {
        res = initOption.transResponse(res)
      }
      return res
    })
  }
}
export default request

```



### api 文件

```js
// src/common/request/api/index.js

export * from './server'
export * from './user'
export * from './message'

export const dashboardGpuChart = {
  // mock: true,
  proxy: 'prom',
  url: '/api/v1/query_range',
  method: 'GET',
  title: '首页GPU利用率'
}

export const dashboardTaskChart = {
  proxy: 'rootService',
  url: '/jobs/statistics',
  method: 'GET',
  title: '获取任务统计详情'
}

export const dashBoardMessage = {
  proxy: 'true',
  url: '/messages',
  method: 'GET',
  title: '获取消息列表'
}

export const config = {
  url: '/api/config',
  method: 'GET',
  title: '获取配置信息'
}

export const serviceList = {
  proxy: 'rootService',
  url: '/service/list',
  method: 'GET',
  title: '获取所有控制节点'
}

```

