## 登录逻辑

#### userInfo：

vue-element-admin 中，userInfo 是每次都发送请求来获取的，请求的参数是 token



controller 中，userInfo 是在第一次登录时发送请求获取到后，将 userInfo 保存在 localStorage 中，并且在每次 `router.beforeEach()` 中都读取 localStorage 中的 userInfo 并保存在 store 中



#### token 与登录权限逻辑：

vue-element-admin，在 main.js 中使用 `router.beforeEach()`，守卫中来判断是否有 token，没有 token 就跳转到登录页面。有 token 的话再从 userInfo 中读取用户的权限，然后挂载有权限的路由



controller 中，在 router.js 路由表中使用 `router.beforeEach()`，守卫中先判断 `to` 要进入的页面是否需要登录，如果需要登录则从 localStorage 中读取 token，判断是否有 token 和 token 是否过期，如果没有 token 或已过期则跳转到登录页。

如果有 token，那么判断该用户是否拥有进入该页面的权限。用户拥有的权限保存在 userInfo 中的 permissions 数组中，数组中每一项的 id 代表相应的权限，使用 map() 方法生成只有 id 的数组，然后判断 `to` 的路由的 meta 中需要哪个权限才能访问，看这个数组中是否含有这个权限，若有的话则进入，如果没有，则跳转到未授权的自定义页面

如果是不需要权限的页面就可以直接进入。



controller 这种方式的优点是，简洁明了，但是有个缺点，就是路由的 meta 里面没有 roles 数组，这样子 sidebar 就没法剔除掉无权限进入的页面，所以如果加上 roles 数组那么就可以在 menu 里进行筛选，这样就完美了

vue-element-admin 的缺点是需要根据权限来动态的挂载路由，这样子比较复杂



> 二者的相同点是都是在登录页中获取 token 并将之保存，并且在 axios 的拦截器中将每个请求带上 token



> controller 中的全局前置守卫，每次都会读取 token, userInfo，并且 `store.commit('saveUserInfo', userInfo)`。如果当前 token 已过期，则会立马 `store.commit('saveUserInfo', **null**)` 将用户信息抹去，同时 token 因为过期了，清不清除都一样



## controllerLayout

现在所有的页面都在这一个 Layout 下面

```js
watch: {
    // controllerList 变化时，重新设置 controller
  '$store.state.controllerList' (val) {
    if (this.$route.name === 'Summarys') {
      this.$store.commit('setController', null)
      return
    }
    const { controller: ip } = this.$route.params
    const controller = val.find(i => i.ip === ip)
    if (controller) {
      this.$store.commit('setController', controller)
    }
  },
  // 路由变化时，重新设置 controller
  $route (val) {
    if (val.name === 'Summarys') {
      this.$store.commit('setController', null)
      return
    }
    const { controller: ip } = val.params
    const controller = this.controllerList.find(i => i.ip === ip)
    if (controller) {
      this.$store.commit('setController', controller)
    }
  }
}
```

这里，controllerList 是发送异步请求获取到的，那么当我们页面已经渲染了，这个异步请求可能还没发送完，此时也就还没有 setController。

但是此时，子页面和组件页面可能已经渲染了，所以也已经发送一些其他请求了，但这些请求需要从 vuex 里面取 controller 的值来添加当前的 qhost 和 qport，但是此时又还没有 setController。所以我们需要给 `<router-view>` 一个 `v-if="!!controller"`，这样可以使其在 setController 之后，才去挂载子页面，然后子页面的请求才会发送。

但是这样子在全部资源池下时，原本就没有 controller，子页面也不会挂载。那么就多添加一个条件 `v-if="!!controller || activeIp === 'all'"`

```js
 if (proxy === 'controller') {
   			// 依赖于 controller
        const controller = store.state.controller || {}
        const { api_version, data_version, ip, port } = controller
        Object.assign(axiosOption.headers, {
          'q-host': ip,
          'q-port': port,
          'api-version': api_version,
          'data-version': data_version
        })
      }
```



## 全部资源池摘要页

### 侧边栏逻辑

全部资源池的 layout 与 单个资源池的 layout 是同一个，点击全部资源池时，跳转 `controllers/summarys`，单个资源池点击时跳转 `controller/:controller/summary`

### 路由中途跳转到某 controller 时

这样在路由刷新或 controllerList 刷新时，就会设置当前 controller

```js
watch: {
    // controllerList 变化时，重新设置 controller
    '$store.state.controllerList' (val) {
      if (this.$route.name === 'Summarys') {
        this.$store.commit('setController', null)
        return
      }
      const { controller: ip } = this.$route.params
      const controller = val.find(i => i.ip === ip)
      if (controller) {
        this.$store.commit('setController', controller)
      }
    },
    // 路由变化时，重新设置 controller
    $route (val) {
      if (val.name === 'Summarys') {
        this.$store.commit('setController', null)
        return
      }
      const { controller: ip } = val.params
      const controller = this.controllerList.find(i => i.ip === ip)
      if (controller) {
        this.$store.commit('setController', controller)
      }
    }
  },
```

在单个资源池时，刷新页面，则执行上述逻辑，用来 commit 当前的 controller

> 这里只能使用路由的参数来记录当前 `controller`！！！！，因为刷新之后 `store` 也重新刷新了，但是初识时会初始化 `controllerList`，然后根据刷新前路由中的参数，来在 controllerList 中查找上次的 controller

### 路由逻辑

新建一个路由

```js
{
      path: '/controllers',
      name: 'Controllers',
      component: ControllerLayout,
      redirect: {
        name: 'Summarys'
      },
      children: [
        {
          path: 'summarys',
          name: 'Summarys',
          meta: {
            menuName: 'Summary'
          },
          isMenu: true,
          component: Summary
        }
      ]
    },
```

> 注意，这里必须要有子节点，才可以复用 controllerLayout。因为如果直接使用 `/controllers` 作为全部资源池页面，那么只能显示 controllerLayout，没有子页面。但当我们有子页面时，子页面也可以复用 Summary 组件，此时 Summary 里因为没有 controller 则会显示全部资源池的内容。

### 全部资源池摘要页折线图

通过 tab 控制两个 echarts 图表的显示与隐藏

全局页面没有选中的控制节点，而是通过 controllerList 展示

项目需求是在图表外面展示自定义样式的 legend，并且鼠标 hover 上去时在固定的位置展示自定义样式的 tooltip



首先需要通过 controllerList 展示当前已有的控制节点，但是去除某个控制节点时不能同时改变 controllerList 本身，于是创建一个新数组 controllerLegend 来展示，发送请求后，先将新数组初始化成 controllerList 的数组

展示的控制节点是通过 antd 的 checkbox 组件修改样式完成，这样每次点击时都会有一个事件 chooseLegend，事件的参数就是当前选中的数组

先说同个组件中的任务折线图。任务折线图是通过请求获取所有的数据，将数据保存到 taskChartList 数组中，然后每次点击时的 chooseLegend 事件触发，并将新数组 controllerLegend 设置为当前选中的数组，然后再用 indexOf 方法看当前选中的数组的名字有哪些是和 taskChartList 数组中的 item.name 相同，如果相同，则将这个 task 数据 push 到一个 temp 数组，将这个数组作为参数传递到 getOption 方法

getOption 方法返回 echarts 的 option，在这个方法中，将传进来的数组做处理，做一些数据结构的转变，将数据转换成适用于 xAxis.type = time 的形式，然后再一个个 push 到 series 的 data 中来展示。然后 echarts 中感受到了 option 的变化时会重新 setOption，setOption(this.option, true)第二个参数表示是否 merge 上次的 option，若为 false 则是重置数据。通过重置数据的方法来达到点击外部 legend 时更新 echarts。

再说说父子组件关系的算力折线图。这个折线图点击外部 legend 更新数据是通过 controllerLegend 数组的变化传递给子组件，子组件再 watch 这个值触发回调，回调中则重新设置 option 从而更新 echarts 数据

### 资源池分页器

```html
<section style="height: 70px;" class="flex-y page-box">
    <i 
       v-if="controllerList.length > 3"
       class="iconfont icon-ico_arrow_left-disabled page-button"
       :style="{ color: pageNum === 0 ? 'rgba(0, 0, 0, 0.04)' : '#0083ff' }"
       @click="paginate('back')"
       />
    <div class="flex" style="height: 100%; flex: 1;">
      <div
         v-for="item in pageList[pageNum]"
         :key="item.ip"
         style="position: relative; width: 33%; padding: 5px 0;"
         class="flex-column flex-between flex-y">
           <div style="position: absolute; top: 0; height: 100%;" class="flex-column flex-between">
              {{ item.name || item.ip }}
              <div style="z-index: 1; margin-bottom: 3px; color: #666;">
                {{ item.ip }}:{{ item.port }}
              </div>
           </div>
           <div
              style="position: absolute; bottom: 0; width: 98%; max-width: 150px; height: 28px; background: #f9f9fa; border-radius: 12px; color: #666;"
              class="flex-center" />
      </div>
    </div>
    <i
      v-if="controllerList.length > 3"
       class="iconfont icon-ico_arrow_right-selected page-button"
       :style="{ color: pageNum + 1 < pageList.length ? '#0083ff' : 'rgba(0, 0, 0, 0.04)' }"
       @click="paginate('next')"
     />
</section>
```

```js
data () {
  return {
    pageNum: 0, // 资源池分页页数
  }
},
computed: {
  pageList () {
    const list = []
    this.controllerList.forEach((item, index) => {
      const num = Math.floor(index / 3)
      if (list[num]) {
        list[num].push(item)
      } else {
        list[num] = []
        list[num].push(item)
      }
    })
    return list
  }
},
methods: {
  // 资源池分页
  paginate (direction) {
    const len = this.pageList.length
    if (direction === 'next') {
      if (this.pageNum + 1 < len) {
        this.pageNum += 1
      }
    } else {
      if (this.pageNum > 0) {
        this.pageNum -= 1
      }
    }
  },
}
```

新创建一个 pageList 数组，index 即相当于其页码

data 中定义 pageNum

然后每点击一次触发回调函数，在函数中将 pageNum + 1 即可

然后模版中循环 `v-for="item in pageList[pageNum]"`



## Echarts

### Ecahrts 封装

同时支持同步 setOption() 与 异步 setOption 的形式

```js
watch: {
    // 观察option的变化
    option: {
      handler () {
        // 异步形式，传递了 option 后才 setOption
        if (!this.chart) {
          this.initChart()
        }
        this.setOption()
      },
      deep: true // 对象内部属性的监听
    }
  },
mounted () {
    if (this.init) {
      this.initChart()
      if (Object.keys(this.option).length) {
        this.setOption()
      }
    }
  },
```

> 注意：watch 了 option，如果 option 变了，则看是否 init，没有的话则 init，然后再 setOption。
>
> 但是如果在 echart 组件上同时加了 v-if 和 :init="false" 的话，初始时 echart 没有渲染，也没有传 option，然后 option 更新后，echart 的 v-if 才为 true，才开始渲染，这时候只会执行 echart 组件中的 mounted，并不会执行 watch 的 option，那么这时候 init 又是 false，所以 echart 这时也不会初始化，那么就压根不会渲染了。
>
> 此时需要将 v-if 换成 v-show，这样子 option 就会一开始是 {}，异步调用之后获得值然后更新。或者把 :init="false" 去掉，让其一开始就初始化



### 折线图刻度自定义

问题总的来说，就是：

[echarts 怎样使标签间隔而刻度不间隔](https://segmentfault.com/q/1010000010461683?utm_source=sf-similar-question#)



xAxis 为 `time` 时，需要根据时间间隔设置相应的横坐标

需求：

当时间间隔为 1 天，横坐标刻度间隔为 6 小时一个刻度，并且鼠标 hover 时可以根据完全显示返回的点数

当时间间隔为 7 天，横坐标刻度间隔为 1 天一个刻度，并且鼠标 hover 时可以根据完全显示返回的点数

当时间间隔为 30 天，横坐标刻度间隔为 5 天一个刻度，并且鼠标 hover 时可以根据完全显示返回的点数



实际情况是，1 天时能满足，不过 7 天时，1 天一个刻度，鼠标 hover 时能显示的最小粒度只能是某天，而不会显示具体几点几分

30 天时也无法满足



所以通过 xAxis.maxInterval 定义横坐标最大的时间间隔

```js
maxInterval = 3600 * 24 * 1000 // 横坐标最大间隔为 1 天
maxInterval = 3600 * 12 * 1000 // 横坐标最大间隔为 12 小时
```

经发现，`maxInterval = 3600 * 13 * 1000` 或 `maxInterval = 3600 * 15 * 1000` 等数据会被 echarts 直接四舍五入到与 `maxInterval = 3600 * 24 * 1000` 同等效果，所以最大只能取间隔为 12 小时来使之能展示具体时间了，但是这样子横坐标的点会太密集，所以同时要剔除许多点

```js
getOption (series = [], legendData = []) {
  let maxInterval
  let formatter
  let bottom
  if (this.timeSpace >= 1440 * 3) {
    // 刻度为 3 或 7 或 10 天时，使刻度间隔能显示每天的时间
    maxInterval = 3600 * 12 * 1000
  }
  if (this.timeSpace === 1440 * 7) {
    bottom = 25
    // 刻度为 7 天时，刻度较多，筛选显示，使其间隔为 2 个点，也就是 1 天
    formatter = (value, index) => {
      if (index % 2 === 1 || index === 0) {
        return `${moment(value).format('HH:mm')}\n${moment(value).format('MM-DD')}`
      } else {
        return ''
      }
    }
  }
  if (this.timeSpace === 1440 * 30) {
    bottom = 25
    // 刻度为 30 天时，刻度太多，筛选显示，使其间隔为 10 个点，也就是 5 天
    formatter = (value, index) => {
      // 因为间隔为 12 小时，所以每天有 2 个点，所以要使得间隔为 5，则要 % 10 而不是 % 5
      if (index % 10 === 0 || index === 0) {
        return `${moment(value).format('HH:mm')}\n${moment(value).format('MM-DD')}`
      } else {
        return ''
      }
    }
  }
  return {
    grid: {
      top: '30',
      left: '8',
      right: '20',
      bottom: bottom || 0,
      containLabel: true
    },
    xAxis: {
      type: 'time',
      boundaryGap: false,
      splitLine: {
        show: false
      },
      axisLine: {
        lineStyle: {
          color: 'transparent'
        }
      },
      axisLabel: {
        color: '#666',
        formatter: formatter || null
      },
      maxInterval: maxInterval || null
    },
  }
```

> 注意两点：
>
> - axisLabel.formatter 中 return `${moment(value).format('HH:mm')}\n${moment(value).format('MM-DD')}` 时，不能通过 `<br>` 只能通过 `\n`，因为这里的 formatter 返回的是字符串，其他 formatter 返回 DOM 结构时就可以用 `<br>`
> - grid.bottom，当使用了 axisLabel.formatter 时，横坐标会被遮挡，此时需要设置 grid.bottom





## 消息系统

socket

消息类型分为告警和通知

已读未读请求 

消息管理界面 由管理员控制哪些用户可以收到哪些类型的消息

### 点击某条消息跳转到对应页

跳转前需要 `setController`，不然当前 controller 为 null 或者当前 controller 不是你要跳转的那个时，就无法实现

```js
// 添加 controller 的 ip 后跳转
const name = item.body.extendedInfo.controller
const controller = this.controllerList.find(i => i.name === name)
if (controller) {
  // 设置 controller
  this.$store.commit('setController', controller)
  if (this.$route.name !== routerName) {
    this.$router.push({ name: routerName, params: { controller: controller.ip } })
  }
}
```

然后全部资源池下拉框那里，再 `watch` 一下 `$store.state.controller` 来进行下拉框的选中

或者直接通过 computed 里面的 activeIp，由于其依赖 this.controller，所以当 controller 被 set 时，它就直接变化。然后下拉框的 `:selected-keys="[activeIp]`，也会自动选中相应的 controller.name

```js
computed: {
  activeIp () {
    return this.controller?.ip || 'all'
  }
}
```

### 筛选消息为本周（自然周）的逻辑

```js
computed: {
  weekList () {
    return this.list.filter(item => {
      // 已读并且小于 7 天
      if (item.read && distance <= 7) {
         // 相差 7 天内，time 一定大于 creatTime，若 Day 反而小一点，则一定是下一周了
         // 若 Day === 0，则是周日，同时又相差 7 天内，则一定是本周
         if (time.getDay() >= createTime.getDay() || time.getDay() === 0) {
           return true
         } else {
           return false
         }
      } else {
        return false
      }
    })
  }
}
```

### 点击某条消息置为已读，但列表不刷新

现需要点击消息列表中的某条未读消息，小红点消失，但不重新发起刷新列表请求（这样子的话，当前 scroll 已经滑动到下面了，所以再次发起请求会一次性发 pageNum = 1, 2, 3 .. 直到当前页码数的消息请求），只在弹窗消失时刷新列表

```js
// 单条消息点击已读
async read (item) {
  if (!item.read) {
    const res = await request.readMessage({
      params: {
        id: item.id
      },
      data: {
        userId: this.userInfo?.id
      }
    })
    if (res.code === 200) {
      item.read = true
      // 这里则不重新刷新列表了，而是只把 massageList 中的当前消息的 read 置为 true
      // this.pageNum = 1
      // this.getMessageList(true)
      this.getMessageUnreadCount()
    }
  }
  // 跳转
  this.skip(item)
}
```





## 个人中心页

显示该用户的活动记录

## 用户管理页

展示各类用户所拥有的权限，以数组方式

### 删除用户

分页时，删除第 2 页最后一个用户时，刷新列表的 pageNum 要变为 1，如果此时再请求 pageNum 为 2 的数据，会没有数据可显示，但页码会变到第一页，所以有问题

```js
async remove (row) {
  await confirm('确定要删除吗？')
  const res = await request.delUser({
    params: {
      userId: row.id
    }
  })
  if (res.code === 0) {
    // 删除第 2 页最后一个用户时，pageNum 由 2 变为 1
    if ((this.totalUser - 1) % this.pageSize === 0) {
      this.pageNum -= 1
    }
    this.reFresh()
  } else {
    this.$Tip.error(res.msg)
  }
}
```



## 激活页

激活页详情的进度条，根据数据的百分比改变进度，并且在上面显示弹框文字

## 侧边栏

##### 刷新后仍然激活当前的 menu-item，（每次路由切换时都会触发！！！）

```js
computed : {
  activeName () {
    if (this.$root.$route) {
      return this.$root.$route.meta.menuName || this.$root.$route.name
    } else {
      return ''
    }
  }
}
```

这里使用了自己在路由中定义的 `meta.menuName`，这样可以在`服务节点页`查看详情时，也就是查看`服务节点详情页`时，将`服务节点详情页`的 menuName 也设置成和`服务节点页`一样的 menuName，这样子在`服务节点页`点击查看详情页时，虽然会跳转到`不是服务节点的子路由`，但是当前激活的 menu-item 仍然会是一样，仍然会是 `ServerNode`

##### 刷新后，默认打开的二级菜单

```js
computed: {
  openNames () {
    let openNames = null
    let len = this.$root.$route.matched.length - 1
    // 一个循环，遍历所有匹配到的菜单中看是否有 openNames，openNames 一般写在需要打开的二级菜单的父级元素
    while (len > -1 && !openNames) {
      openNames = get(this.$root.$route.matched[len], ['meta', 'openNames'])
      len--
    }
    // 手动指定路径与匹配路径
    return openNames || this.$root.$route.matched.slice(1, -1).map(i => i.name)
  }
}
```

要打开的二级菜单路由中使用自定义的 `meta.openNames`

openNames 一般写在`需要打开的二级菜单的父级元素`，在这里是写在 controller 中

```js
{
    path: '/controller/:controller',
    name: 'Controller',
    component: ControllerLayout,
    meta: {
      openNames: ['Resource']
    },
    redirect: {
      name: 'Summary'
    },
    children: controller.map((item) => {
      return {
        ...item,
        meta: {
          ...item.meta,
          requiresAuth: true
        }
      }
    })
  },
```



## request

### proxy

不需要转发的请求就是直接发到 server，因为页面也是 server 渲染出来的

需要转发的请求就是在 server 的请求中加上 q_host 和 q_port 然后转发到 controller 服务器上

### mock

在请求对象中添加 `mock: true`，则可以在 mock 文件夹中将其拦截，然后单独返回自定义的假数据







