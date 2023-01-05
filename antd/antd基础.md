# 1. Menu

## 1.1 样式

### 1.1.1 menu 右侧空隙

菜单栏 collapse 状态下的 icon + span 样式

ant-menu 右侧灰色线条及空隙不是 margin 导致，而是 ant-menu-item 的 width 不是 100%

记得看源码



#### 1.1.2 气泡框宽度

气泡框默认样式中有个最小宽度，要将之去除才能自定义气泡框的宽度：

```css
.ant-menu-vertical.ant-menu-sub {
  min-width: 96px;
}
```



### 1.1.3 tooltip

menu 的 index.css 中引入了 tooltip 的样式，若要将折叠后菜单栏默认显示的 tooltip 隐藏掉，那么需要：

```css
.ant-tooltip {
  visibility: hidden;
}
```



### 1.1.4 绝对定位

自定义 menu-item 的样式宽度时，最好不要用 margin 来控制更宽或者更窄，这样会在菜单栏折叠时产生挤压。

最好是将 menu-item 使用绝对定位，这样子也不占空间。

> 多使用 padding 而不是 margin 来处理间距。因为 margin 会造成挤压，padding 是在元素内部，不会造成挤压。
>
> 但是 margin 不会影响元素 content 的宽度（属于外部），padding 会影响自身 content 的宽度（属于内部）。



## 1.2 事件

在刷新后保持菜单选中（在刷新后如果当前选中的菜单是二级菜单则展开当前菜单的父级菜单）

有两个 API：

```
openKeys	当前展开的 SubMenu 菜单项 key 数组
defaultOpenKeys	初始展开的 SubMenu 菜单项 key 数组
```

一个是当前，一个是初始。

**defaultOpenKeys**设置值后是只生效一次的 也就是说初始设置**defaultOpenKeys**的对应属性为空 等 ajax 请求到数据后再去设置，菜单就不会变化了，**所以不能用这个**。动态数据不能使用这个，静态的可以。

只能用 `openKeys` 了，又发现一个小问题，设置**openKeys后手动点击其他没展开的菜单，菜单不会有变化** ，这样就得用到另一个 API：

```
openChange	展开/关闭的回调
```

每次点击可展开/收起的菜单时都会触发这个属性所对应的函数，设置 `openKeys` 后再配合 `openChange` 切换菜单（展开和收起菜单时才会触发 openChange）即可解决问题。

```vue
<template>
	 <a-menu 
     theme="dark" 
     :openKeys="openKeys" // 重点 当前展开的菜单
     mode="inline"
     @openChange="onOpenChange" // 重点 当可以展开的菜单被点击时
     :defaultSelectedKeys="defaultSelectedKeys" // 默认选中的菜单
     style="width: 100%"
   >
  </a-menu>
</template>

<script>
  data () {
    return {
      openKeys: ['Resource'] // 一开始进入界面时就将这个二级菜单展开
    }
  }
  methods: {
    onOpenChange(openKeys) {  // 当菜单被展开时触发此处
      /* 
      经测试传入的变量openKeys是数组 点击已经展开的菜单时传入的是空数组
      点击未展开的菜单时传入的是[当前展开菜单的key,点击的菜单key]
      下面的if判断用openKeys === [] 无效 所以我只能判断数组length是否等于0
      */
      if (openKeys.length !== 0) {  
        this.openKeys = [openKeys[1]] // 点击未展开的菜单时传入的是[当前展开菜单的key,点击的菜单key]，所以永远是取第二个，也就是openKeys[1]
      } else {
        this.openKeys = ['']
      }
    },
  }
</script>
```

参考网址：https://blog.csdn.net/qq_35069272/article/details/104990576



## 1.3 selected-keys

```html
<a-menu
        ref="menu"
        class="controller-menu"
        mode="inline"
        :selected-keys="[activeName]"
        @select="select"
      >
        <template v-for="item in menu">
          <a-menu-item :key="item.name">
            <span class="menu-title">{{ item.meta.title }}</span>
          </a-menu-item>
        </template>
      </a-menu>
```

当前激活的 keys，需使用 selected-keys，不要使用 default-keys，那个只会在刷新的时候生效一次，而 selected-keys 是实时的



# 2. Radio

`a-radio-group` 的 v-model 绑定的变量如果是 Number，则无法绑定，`必须要是 String`。如果写的是静态值 value="1"，则每次选中时 value 也会是字符串 1 而不是数值 1。

若绑定了 v-model，那么每次 default-value 的值不会生效，只会生效当前使用 v-model 动态绑定的值，所以可以用 v-model 绑定的那个值的默认值作为 radio 的默认选中项



# 3. Breadcrumb

a-breadcrumb 与 transition 的结合使用（某些地方需重定义 a-breadcrumb 的 css 样式）（span 下面放 span 会出现问题）

例如：

```vue
<template>
		<a-breadcrumb v-if="controller" class="layout-breadcrumb">
        <!-- 这里的 tag 要设置成 div，不然就会出现 span 下面嵌套 span 的结构，会出问题 -->
        <transition-group name="breadcrumb" tag="div">
          <a-breadcrumb-item v-for="(item, index) of breadcrumbRoutes" :key="index">
            <span v-if="index === breadcrumbRoutes.length - 1">{{ item.meta.title }}</span>
            <router-link v-else :to="{ name: item.name, params: { controller: $route.params.controller } }">
              {{ item.meta.title }}
            </router-link>
          </a-breadcrumb-item>
        </transition-group>
    </a-breadcrumb>
</template>
```

相应的，需要修改 `antd.less` 中的样式

```less
// 原有样式
.ant-breadcrumb > span:last-child .ant-breadcrumb-separator {
  display: none;
}
.ant-breadcrumb > span:last-child {
  color: rgba(0, 0, 0, 0.65);
}
.ant-breadcrumb > span:last-child a {
  color: rgba(0, 0, 0, 0.65);
}

// 修改后，加上一层 div
.ant-breadcrumb div > span:last-child .ant-breadcrumb-separator {
  display: none;
}
.ant-breadcrumb div > span:last-child {
  color: rgba(0, 0, 0, 0.65);
}
.ant-breadcrumb div > span:last-child a {
  color: rgba(0, 0, 0, 0.65);
}
```



参考 `controller`



# 4. Table

## 4.1 customRender

Table 组件中的 customRender，只能使用以下形式：

```js
{
   title: '节点',
   dataIndex: 'name',
   customRender: (value, row) => {
     return {
       children: <span>{row.name || row.ip}</span>,
       attrs: {}
     }
   }
}
```

而不能使用 es6 的函数省略形式：

```js
{
   title: '节点',
   dataIndex: 'name',
   customRender (value, row) {
     return {
       children: <span>{row.name || row.ip}</span>,
       attrs: {}
     }
   }
}
```

而且，customRender的参数不能使用 es6 的解构，例如 `({ row })`

可能是不能使用 es6 之后的语法吧



关于 customRender 的使用，参考网址：https://blog.csdn.net/qq_42597536/article/details/90256530



> ```js
> {
>    title: '节点',
>    dataIndex: 'name',
>    customRender: (value, row) => {
>      return {
>        children: (
>          <div>	
>          		<span>{row.name || row.ip}</span>
>          </div>
>        ),
>        attrs: {}
>      }
>    }
> }
> ```
>
> 当这种 children 里面返回的 html 比较多时，可以使用括号括起来



customRender 中，有两个属性 children 和 attrs

例如：

```html
<tbody>
  <td></td>
</tbody>
```

children 就是 td 下的 html 解构

attrs 就是 td 标签的 attributes

例如：

```js
customRender: () => {
  children: <div></div>,
  attrs: {
    class: 'td'
  }
}
```

会渲染成：

```html
<tbody>
  <td class="td">
    <div></div>
  </td>
</tbody>
```



## 4.2 key 和 rowKey

如果 dataSource 中的每条数据没有唯一的 key 属性，那么就会报错：

```
Each record in table should have a unique `key` prop, or set `rowKey` to an unique primary key
Each record in dataSource of table should have a unique `key` prop, or set `rowKey` to an unique primary key
错误详情见 https://u.ant.design/table-row-key
```

此时，可以给 dataSou4rce 中的每条数据都加上一个 key 属性，但是 dataSource 是后端传来的没法自行更改，那么就使用自己写的 rowKey：

```html
<a-table
   :columns="columns"
   :data-source="serverList"
   :rowKey="record => record.ip"
   class="mt15"
></a-table>
```

例如我的 dataSource 数据中有个唯一的 ip 值，我就直接把每一行数据的 ip 值设置为 key



## 4.3 expand

### expand 展开事件

```js
expandIcon (props) {
  if (props.record?.NativeDevice) {
    return
  }
  if (props.expanded) {
    return <i class="iconfont icon-icon_embed_arrow_click" onClick={ () => { props.onExpand() } }></i>
  }
  return <i class="iconfont icon-icon_embed_arrow_normal" onClick={ () => { props.onExpand() } }></i>
}
```

必须在这个图标上绑定一个点击事件，并且调用 `props.onExpand()`

> 注意，table 上的 expand 事件并不是点击展开按钮时的绑定事件，使用 expand 并不能将其展开。
>
> expand 只是当其展开时执行的回调



### 使用 css 修改展开 icon

```css
/* 将展开箭头样式重新书写！！ */
:deep(.ant-table-row-expand-icon.ant-table-row-expand-icon-collapsed) {
  width: 8px;
  height: 8px;
  border: 0;
  border-top: 2px solid rgba(0, 0, 0, 0.25);
  border-right: 2px solid rgba(0, 0, 0, 0.25);
  background: transparent;
  border-radius: 0;
  transform: rotate(45deg);
}

:deep(.ant-table-row-expand-icon.ant-table-row-expand-icon-expanded) {
  width: 8px;
  height: 8px;
  border: 0;
  border-top: 2px solid #0083ff;
  border-right: 2px solid #0083ff;
  background: transparent;
  border-radius: 0;
  transform: rotate(135deg);
}
```

这样的好处是：

1. 可以使用 `expandRowByClick` 属性来点击整行都能展开
2. 可以使用 `rowExpandable` 属性来控制某一行不展开

> 如果使用 expandIcon 函数则不能使用以上属性，要自己实现。





## 4.4 width

表头不要折行，表体内容可以折行



## 4.5 pagination

在 table 中使用分页功能时，点击下一页时的事件不是 pagination 中的 change 事件，而是 table 中的 change 事件，要绑定在 table 中，而不能写在 pagination 的对象中

而单独使用 pagination 时则是用 pagination 自己的 change 事件，这两个事件名字都叫 change，但是`参数不一致`，所以需要写两个函数来执行不同的处理！！



注意，pagination 和 table 中的 pagination 的事件不一样

```html
<a-table
         class="mt24"
         :columns="columns"
         :data-source="userList"
         :row-key="record => record.id"
         :pagination="{
                      total: total,
                      current: pageNum,
                      pageSize: pageSize,
                      onShowSizeChange: onPageSizeChange,
                      showSizeChanger: true,
                      showQuickJumper: true,
                      showTotal: total => `共 ${total} 条`
                      }"
         @change="onPageChange"
         />
```

pagination: showSizeChange, change

table 中的 pagination：onShowSizeChange, onChange

## 4.6 customRender

可以添加表格中点击某一行的事件

参考官网文档

https://www.antdv.com/components/table-cn/#customRow-%E7%94%A8%E6%B3%95

## 4.7 column

### sorter

defaultSortOrder: 'ascend'，开启表格的默认排序，'descend'是降序

`sorter: true` 的时候，可以使用服务端的排序，即通过 tabel 的 change 回调事件中获取改变的 sorter，sorter.field 为当前列的 key，sorter.order 即是当前排序的方向，然后在这个回调里，带上排序的参数，再发送刷新列表的请求即可



## 4.8 使表格中间可左右滑动

参考文档

https://www.antdv.com/components/table-cn/#components-table-demo-grouping-table-head



## 4.9 slot

table 的 slot 写法（非 customRender 写法）

不存在 dataIndex 的要换成 key 才能 slot 渲染出来



## 4.10 事件

antd 的 pagesize 变化时会同时触发 onpagechange 和 onpagesizechange 事件



# 5. dropdown

dropdown 在 body 中，但是没在当前的页面中，它是独立出去的，`position: absolute`

所以

```less
<style lang="less">
.controller-layout {
  .ant-dropdown {
    .ant-dropdown-menu .ant-dropdown-menu-item:nth-of-type(1) {
      background: red !important;
    }
  }
}
</style>
```

这样更改是行不通的，因为它不在 `.controller-layout` 下，它是独立出去的



# 6. layout

body 加上 height: 100%;

layout 加上 height: 100%; 且 layout 默认是 flex-column 的

layout-content 加上 height: 100%;

内部元素 加上 height: 100%;



# 7. DatePicker

## a-range-picker

自定义渲染样式

```html
<a-range-picker
       v-model="searchModel.from"
       format="YYYY-MM-DD HH:mm"
       :placeholder="['', '']"
       @change="onSearch"
       >
       <div style="width: 300px; height: 32px; border: 1px solid rgba(0, 0, 0, 0.15); border-radius: 2px;" class="flex-center">
         <span v-if="!searchModel.from" style="color: rgba(0, 0, 0, 0.25);" class="flex-y">
           请选择任务开始时间的时间范围
         	 <a-icon type="calendar" style="margin-left: 70px;" />
         </span>
         <span v-else>
           {{ `${searchModel.from[0].format('YYYY-MM-DD HH:mm')} ~ ${searchModel.from[1].format('YYYY-MM-DD HH:mm')}` }}
           <i class="iconfont icon-ico_off" style="color: rgba(0, 0, 0, 0.25); cursor: pointer;" @click.stop="clearTime" />
         </span>
       </div>
</a-range-picker>
```

自定义时，allow-clear 可清除功能不生效，需要自己添加 icon 然后写点击事件来清除



# 8. Form

antd 的 form 必须要设置 :**label-col**="{ span: 4 }" :**wrapper-col**="{ span: 14 }" 之后，表单才能横向显示



a-form-model中动态删除表单项引起的prop的问题，使用this.$modal.destroyAll()来解决

https://www.chinacion.cn/article/610.html



antd 的表单布局使用 a-row



# 9. Select

带搜索功能的 a-select 下拉滑动懒加载：

```js
// 监听了 a-select 的 popupScroll 事件

handleScroll (e) {
  const { target } = e
  var total = target.scrollTop + target.offsetHeight
  var scrollHeight = target.scrollHeight
  if (total >= scrollHeight - 1 && total < scrollHeight + 1 && this.pageNum < this.crmSearch.length) {
    this.pageNum += 1 // pageNum 是当前页码，crmSearch.length 是数据总长度
  }
}
```



# 10. Modal

ant modal.confirm 也可以添加 centered 属性来使其居中



iview 的 modal 会直接绑定到 vue.prototype 上面



# 11. Icon

antd 的 icon 可以使用 theme=“filled” 来使其变成实心



# 12. Upload

beforeUpload 钩子函数，return false 时也会将 fileList 上传到本地，此时若想要中断，可以`return Promise.reject(new Error('sth error'))`



# 13. input

## type = number 的问题

```html
<a-input
   class="input"
   style="width: 192px;"
   type="number"
   placeholder="输入数值"
/>
```

如果 `type = number` 那么将会是 html 的原始 Input 框，此时要将其后面的 icon 去除的话，需要以下代码

```css
input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button {
  margin: 0;
  -webkit-appearance: none !important;
}
```

但如果 a-input 里面也加了 suffix

```html
<a-input
   class="input"
   style="width: 192px;"
   type="number"
   placeholder="输入数值"
   suffix="%"
/>
```

那么就需要写成这样：

```css
.ant-input::-webkit-outer-spin-button,
.ant-input::-webkit-inner-spin-button {
  margin: 0;
  -webkit-appearance: none !important;
}
```



# AutoComplete

```vue
<template>
	<div>
    <a-auto-complete
      v-model:value="selectValue"
      :options="options"
      style="width: 100%"
      :placeholder="$t('mixedFilter.placeholder')"
      :filter-option="filterOption"
      :bordered="false"
      :default-active-first-option="defaultActiveFirstOption"
      :get-popup-container="() => $refs.selectRef"
      @select="completeSelect"
      @change="selectChange"
      @keydown.enter="selectEnter"
      @dropdown-visible-change="selectDropdownChange"
      @blur="selectBlur"
    />
  </div>
</template>
<script setup>
const selectValue = ref('')
const showSelect = ref(true)
const selectBlur = (e) => {
  selectValue.value = "";
  showSelect.value = false;
  // 不加nextTick的话会出错
  nextTick(() => {
    showSelect.value = true;
  });
};
</script>
```

如果在 autocomplete 框失去焦点时将输入的内容置空，那么鼠标再次点击 autocomplete 框时，无法弹出下拉选项，此时需要重新渲染一下 autocomplete。一定要加上 nextTick，不然时间过快相当于没有重新渲染。

