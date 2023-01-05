# 1. resize()

## 1.1 ResizeObserver 方法

window.onsize() 可以侦听整个窗口的 size 变化。而如果要侦听某个 Dom 元素的 size 变化的话，就要使用 `ResizeObserver` 方法



使用场景：

在项目中 引入了 Echarts 进行绘图，在将菜单隐藏或打开的时候，不能触发 window.resize() 方法，导致绘制的图形发生了偏移（也就是此时 Echarts 不会触发 resize() 方法，只有当 window.resize() 变化时才会自动触发），目前需要的步骤就是在让菜单打开或者隐藏的时候，手动触发一次 resize 方法，让 Echarts 图形自适应。



解决方案：

1. 监听 Echarts 外层的 Dom 元素的 size 变化，变化时触发回调函数，在回调函数中执行 Echarts 的 resize() 方法。

> 不过这样子会报一个警告，因为当父组件引用子组件时，父组件的 mounted() 在先，子组件的 mounted() 在后，也就是说如果在 A 组件中引入 Echarts 作为子组件，然后在 A 组件的 mounted() 钩子函数中执行 this.$refs.chart.resize()，这时候子组件 Echarts 还未进行渲染，而通常 initChart() 这个初始化 Echarts 的方法会写在 Echarts 组件中的 mounted() 钩子函数中时，那么就会报这个警告。
>
> 因为这时候还未 initChart() 形成 Echarts 的实例，就去执行 Echarts 的 resize() 方法，所以就会报错：`resize() is not a function`

> 所以我们应该直接监听 Echarts 的 canvas 挂载的那个 DOM 上

2. 在将菜单隐藏或打开的时候执行 resize() 方法

> 这个时候可以封装一个通用的 Echarts 的 resize() 方法，将其绑定在 window.onresize 上，然后当菜单隐藏或打开时，也执行 window.onresize，那么这个时候就会自动执行这个方法



参考网址：

https://juejin.cn/post/6844903749253595143#heading-8

https://developer.mozilla.org/zh-CN/docs/Web/API/ResizeObserver



## 1.2 random

父组件引用 Echarts 组件，传入一个时间作为 random 参数，Echarts 组件中再侦听这个 random，一改变则调用组件中的 resize() 方法。

那么当做某个操作需要 resize() 时，就 new Date().getTime() 来改变当前的时间，然后就会随即 resize()



## 1.3 window.addEventListener

每次在 Echarts 组件的 mounted() 中，先 removeEventListener，再 addEventListener



# 2. 饼状图

## 2.1 位置

饼状图位置不能用 grid.left 等

要用 series 里的 left 和 right 等，还有 center['50%', '50%']



## 2.2 graphic

想在图标区域任何地方添加文字可以使用 title 和 graphic，graphic 可配置性更强



# 3. option

静态的 option 传入时，配置项中的值都不能使用变量，所以我们应该使用一个 getOption 函数来 return 一个 option 配置，这样里面的任何属性就能使用变量了

例如：

```js
getOption () {
      return {
        color: ['#0083FF', '#00CFDE', '#07C270', '#FFCC01', '#FF3B3B', '#6D5DD3'],
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b}: {c} ({d}%)',
          backgroundColor: '#282B31', // 提示框浮层的颜色
          padding: 12 // 提示框浮层内边距
        },
        legend: {
          orient: 'vertical', // 竖直排列
          left: '40%',
          itemWidth: 10,
          itemHeight: 10, // 图例大小
          icon: 'circle', // 图例形状
          height: 108, // 整个图例的高度
          padding: 21, // 内边距
          itemGap: 16, // 图例之间的间隔
          textStyle: {
            height: 22,
            lineHeight: 22,
            fontSize: 14,
            color: 'rgba(0, 0, 0, 0.65)',
            rich: true // 此处开启富文本形式才能指定宽高
          }
        },
        graphic: [
          {
            type: 'text',
            left: '22.5%',
            top: '40%',
            style: {
              text: '总数',
              textAlign: 'center',
              fill: 'rgba(0, 0, 0, 0.45)',
              fontSize: 14
            }
          },
          {
            type: 'text',
            left: '22.5%',
            top: '50%',
            style: {
              text: this.usedDevice,
              textAlign: 'center',
              fill: 'rgba(0, 0, 0, 0.45)',
              fontSize: 14
            }
          }
        ],
        series: [
          {
            name: 'GPU型号',
            type: 'pie',
            radius: ['53%', '70%'], // 内外圆的半径
            avoidLabelOverlap: false,
            center: ['25%', '50%'],
            label: {
              show: false
            },
            labelLine: {
              show: false
            },
            data: [
              { value: 1048, name: '搜索引擎' },
              { value: 735, name: '直接访问' },
              { value: 580, name: '邮件营销' },
              { value: 484, name: '联盟广告' },
              { value: 300, name: '视频广告' }
            ]
          }
        ]
      }
    }
```



# 4. legend

## 4.1 formatter

使用 formatter 来自定义图例的显示格式

```js
legend: {
          orient: 'vertical', // 竖直排列
          left: '40%',
          itemWidth: 10,
          itemHeight: 10, // 图例大小
          icon: 'circle', // 图例形状
          height: 108, // 整个图例的高度
          padding: 21, // 内边距
          itemGap: 16, // 图例之间的间隔
          textStyle: {
            height: 22,
            lineHeight: 22,
            fontSize: 14,
            color: 'rgba(0, 0, 0, 0.65)',
            rich: true // 此处开启富文本形式才能指定宽高
          },
          formatter: name => { // 图例的 formatter，使其能显示为 name：value 的形式
            const data = seriesList.filter(row => row.name === name)
            return `${name}：${data[0].value}`
          }
        },
```

参考网址：https://github.com/apache/echarts/issues/4125



# 5. tooltip

## 5.1 formatter

参考网址：https://blog.csdn.net/shuoSmallWhite/article/details/80106791



# 6. 配置项

## yAxis

### yAxis.minInterval

https://echarts.apache.org/zh/option.html#yAxis.minInterval

y 轴数值的最小间隔，可防止出现小数



## xAxis

### xAxis.maxInterval

type 为 `time` 时，设置横坐标的最大时间间隔

### xAxis.splitNumber

将 x 轴划分成多少段，但有时设置了 maxInterval 或 interval 后再去设置 splitNumber 则不会生效

### xAxis.axisLabel.formatter

可以用于自定义展示的格式，也可以筛选横坐标的点数，让其间隔多少点才展示一个点

### xAxis.axisLabel.showMinLabel xAxis.axisLabel.showMaxLabel

去除首尾的横坐标！

### type

type: 'time' 时，如果整个坐标轴里面只有一个点，那么会自动帮你补齐点数，type: 'category' 时就不会



## legend

### legend.selectedMode

控制是否可以通过点击图例改变系列的显示状态。默认开启图例选择，可以设成 `false` 关闭，使 legend 不可点击

https://echarts.apache.org/zh/option.html#legend.selectedMode





# 7. 注意项

- ECharts 的 legend.data 要与 series.name 一致，这样 legend 才能够显示出来



## width问题

echarts宽度或高度设置成100%，结果变成了100px

这种情况一般都是echarts所在的div一开始是`display:none`，一般出现在以下几种情况：

1. echats放在了tab中

2. 所在div用了`v-show`，这种情况要不换成`v-if`，要不就设置初始值是`true`。换成v-if可行，但在频繁显示或不显示的时候不适用v-if，所以在试用v-show且初始为false的时候，可以使用`resize()`

   ```js
   watch：{
   	show(v){
   		// 在show为true，也就是显示的时候，调用resize 解决100px的问题
   		if(v){
   		  this.$nextTick(() => this.$refs.chart.instance.resize())
   		}
   	}
   }
   ```



参考： https://blog.csdn.net/hyeeee/article/details/108199973?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-1.no_search_link&spm=1001.2101.3001.4242.2



## vue3.0 中使用echarts图 tooltip 不显示的问题

**原因：echarts 实例赋值给 ref 响应式 Proxy对象，导致tooltip不显示**

**解决办法：要用普通变量来接收 echarts 实例**

```js
// before
const myChart = ref() // 定义了一个全局的响应式变量，用于接收echarts实例

onMounted(() => {
  myChart.value = echarts.init(myLine.value as HTMLElement)
  myChart.value.showLoading()
  myChart.value.setOption(option.value, true)
  
  watch(loading, () => {
    if (loading.value) {
		myChart.value.showLoading()
	} else {
		myChart.value.hideLoading()
	}
  })
})

// now
// 定义普通变量来接收实例
let myChart:any

onMounted(() => {
  myChart= echarts.init(myLine.value as HTMLElement)
  myChart.showLoading()
  myChart.setOption(option.value, true)
  
  watch(loading, () => {
    if (loading.value) {
		myChart.showLoading()
	} else {
		myChart.hideLoading()
	}
  })
})
```



参考：

https://blog.csdn.net/anjiongyi/article/details/124255820?spm=1001.2101.3001.6661.1&utm_medium=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-124255820-blog-122115316.pc_relevant_sortByStrongTime&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-1-124255820-blog-122115316.pc_relevant_sortByStrongTime&utm_relevant_index=1



https://blog.csdn.net/weixin_45304198/article/details/122115316

