## Vue2.x Table 组件从0到1。

### API设计
 
Table组件是个功能比较复杂的组件，所以我们在尝试写一个Vue-table组件之前，参考了一些常用UI组件库中Table组件的设计。首先看的是同样基于Vue的Element组件库，其中的Table组件用法如下：

```
<template>
	<el-table
	  :data="tableData"
	  style="width: 100%">
	  <el-table-column
	    prop="name"
	    label="姓名"
	    width="180">
	  </el-table-column>
	  <el-table-column
	    prop="address"
	    label="地址">
	  </el-table-column>
	</el-table>
</template>

<script>
	export default {
	  data() {
	    return {
	      tableData: [{
	        name: '王小虎',
	        address: '上海市普陀区金沙江路 1518 弄'
	      }, ...]
	    }
	  }
	}
</script>
```

我们可以把表格的信息分为两部分，一部分是表格的数据，另一部分是表格的排列信息。那么在Element UI中，表格数据需要定义在data中，再通过props传到table组件里。表格排列信息则定义在`el-table-column`子组件中。

接下来，再看另一个UI组件库ant design，其Table组件用法如下:

```
const dataSource = [{
  key: '1',
  name: '胡彦斌',
  age: 32,
  address: '西湖区湖底公园1号'
}, ...];

const columns = [{
  title: '姓名',
  dataIndex: 'name',
  key: 'name',
}, {
  title: '年龄',
  dataIndex: 'age',
  key: 'age',
}, {
  title: '住址',
  dataIndex: 'address',
  key: 'address',
}];

<Table dataSource={dataSource} columns={columns} />
```

与Element UI不同，ant design表格列的信息通过props传入组件。

将两个UI库中Table组件相比之后，我们觉得Element UI让用户把表格列信息写在模板中的设计更加直观清晰。所以，最后决定参考Element UI的设计。

----

### 初步设计

用法：

```
<template>
	<m-table :data="tableData">
	    <m-table-col
	    prop="name"
	    label="姓名"
	    width="10%">
	    </m-table-col>
	    <m-table-col
	    prop="address"
	    label="地址"
	    width="40%">
	    </m-table-col>
	</m-table>
</template>

<script>
export default {
  data() {
    return {
      tableData: [{
        name: '王小虎',
        address: '上海市普陀区金沙江路 1518 弄'
      }, ...]
    }
  }
}
</script>
```

这里的`<m-table-col>`组件只作为接入组件，并不具有展示功能。它的主要工作是在其初始化的时候将列的信息，如`prop`、`label`、`width`保存到一个对象中，将该对象`$emit`到父组件`m-table`上。`m-table`组件再将所有列组件的信息保存到`columns`数组中。

我们再来看一下渲染表头和表格主体的展示组件，其关系如下：

```
<m-table>
    <m-table-head :columns="columns"></m-table-head>
    <m-table-body :data="data" :columns="columns"></m-table-body>
</m-table>
```

`<m-table-head>`组件负责渲染表头，`m-table-body`组件负责渲染表格主体。这两个组件所作的处理就是循环生成`<th>`、`<td>`等表格元素，组合成一个表格。至此，一个可以展示数据的Table组件就算完成了。

但是，表格一般需要有一定的操作功能，比较常用的一个就是选中删除某一行。然而，在完成这一步的时候我们遇到了一些问题。

----

### 自定义column

假设现在要定义一列删除按钮，要如何让用户自如地定义按钮组件，最终在表格内部正常渲染，且点击具体的按钮可以获得对应row的信息？

一开始我们设想用`slot`获得子Vnode节点，它们记录了用户自定义组件声明时的内嵌内容，将它们保存在`columns`数组中，再在每一行重复渲染。初步设计如下:

```
<m-table-col
	label="操作"
	prop="active"
	width="10%">
	<m-button></m-button>
</m-table-col>
```

然而VNode是唯一的，而且即使将VNode深复制，也不可能动态改变VNode，比如把每一行的id传进该VNode。

最后的解决方法是`$scopedSlots`。scopedSlots被编译后，返回一个函数，该函数可以动态传入scope并生成VNode，问题便迎刃而解了。

#### Links:

[Table组件中slot内容的跨级传递](http://zxc0328.github.io/2017/09/19/table-component-slot-passing/)
