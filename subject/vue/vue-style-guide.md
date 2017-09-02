# Vuejs组件风格指南

这篇Wiki是木犀MFE的Vue组件风格指南。首先大家可以看一下这个[Vue.js Component Style Guide](https://github.com/pablohpsilva/vuejs-component-style-guide)，里面的几点。除了这篇文章中讲到的比较普世的规范之外，我们自己在实践中也总结了很多的最佳实践。下面就按Vue.js Component Style Guide的套路来讲一讲这些规范。

## Table of Contents

* [使用单文件格式定义组件](#使用单文件格式定义组件)
* [组件命名](#组件命名)
* [适中的组件粒度](#适中的组件粒度)
* [数据驱动，避免直接操作视图](#数据驱动，避免直接操作视图)


## 使用单文件格式定义组件

我们使用单文件.vue格式定义组件。将组件的逻辑、模板和样式都定义在同一个文件中。然后使用Webpack和vue-loader来构建。

### Why?

+ 有一种模式叫collocate，就是把关系密切的资源放在同一个地方，利于资源直接的协作沟通。比如便利店可能离居民区很近（一个不是很恰当的比喻）。在传统的Web前端开发中，我们常常会说到文件目录结构的规范，有一种规范就主张把一个组件的CSS、HTML和JavaScript放在同一个文件夹下，这样就**比较便于维护**。Vuejs则更进一步，借助现代前端的构建工具，把一个组件各个部分都放到了一个文件里。
+ Vuejs官方文档里陈述的单文件组件的优点有：支持Vue模板语法高亮；支持组件CSS的模块化；支持即插即用的构建流水线；支持组件的模块内定义，而不是全局定义。
+ 传统软件工程中提倡的**高内聚，低耦合**。组件和组件之间的拆分是低耦合，那么组件各个部分之间的单文件定义就是高内聚了。

### How?

使用[vue-loader](https://vue-loader.vuejs.org/en/)。这里Vuejs官网中对于Single File Components（单文件组件）的[介绍](https://vuejs.org/v2/guide/single-file-components.html#Introduction)。在团队中，我们使用[ninja](https://github.com/Muxi-Studio/ninja)来初始化项目目录结构，因此你不用太关心项目构建工具的配置。

## 规范组件命名

这条规范参考了*Vue.js Component Style Guide*中[关于这个话题的讨论](https://github.com/pablohpsilva/vuejs-component-style-guide#vue-component-names)。规范的组件命名要求我们在Vuejs组件的命名中采用一个通用的规范：

+ 要求命名遵循[Custom Element格式](https://www.w3.org/TR/custom-elements/#valid-custom-element-name)，简短的说就是用`-`分隔的小写字母，并且不能使用保留名（比如`font-face`）。
+ 组件命名有命名空间，比如`app-feed`。
+ 组件命名要有意义，简短（二到三个词），可以被轻松的读出来。

### Why?

+ Vuejs中很多特性都是源自Web Component规范，因此我们采纳了Custom Elements中组件用中划线分隔的命名方式。
+ 对组件名加上命名空间有利于我们快速定位组件之间的父子关系。
+ 有意义的组件名方便开发者协作时的沟通和对代码的理解。

### How?

```html
<app-feed> //顶层组件
<feed-item> //feed组件的子组件
<feed-add-btn> 

```
## 适中的组件粒度

在组件化的开发中，有一个很关键的话题就是，什么样的功能模块是应该被拆分成组件的。我们提倡在必要的时候，创建组件。而不是一味的拆分。

### Why?

+ 如果组件内逻辑太多，那会过于臃肿，难以重用和维护。如果组件拆分的太小，也会增加应用的复杂度，并且难以快速定位功能模块代码的位置。

### How?

我们提倡的拆分方法是：

+ 首先将最明显的**通用UI组件**（Modal、表格、菜单、按钮、输入框等等UI库中常见的组件）拆出。这些组件一般不带有特殊的业务逻辑。判断的逻辑就是这个UI组件是否可以跨应用使用，如果可以，那么这个UI组件就属于通用的UI组件。
+ 然后可以根据原型图来拆分页面的**业务逻辑单元**，将一根业务逻辑单元作为一个组件。比如一个微博页面，发微博的逻辑是一个组件，微博的feed流是一个组件，推荐关注是一个组件。
+ 在比较细粒度的组件上，可以先不拆分，在后期觉得单个组件体积过大时，再进行拆分。

## 数据驱动，避免直接操作视图

Vuejs的核心理念就是数据驱动的视图。我们在ViewModel中主要做的是对数据的修改，Vuejs会自动侦测数据的变化，并且更新视图层。

所以我们要求：

+ 禁止在ViewModel中进行DOM操作（包括对DOM节点的操作以及DOM事件的监听等等）。
+ 使用自定义指令封装DOM操作。
+ 如果实在需要在ViewModel中操作DOM，请在群里讨论。

### Why?

+ Vuejs的思想就是使用数据驱动的MVVM模式，用**声明式（declarative）**的写法进行开发。我们只需要声明组件的状态（也就是data），组件就的UI就会自动更新以反映组件的最新状态。而直接对DOM进行操作是一种**命令式**的写法，这样和Vuejs的思想是冲突的。
+ **声明式**的编程范式是目前GUI开发中比较流行的一种模式，借助这种范式我们可以从命令式的思维方式中解脱出来，把UI看成是状态的映射，从而简化UI开发的复杂度。所以我们提倡**声明式**的编程范式。
+ Vuejs的渲染流程是，数据变化，驱动UI变化，如果直接修改UI，就破坏了这个流程。会造成一些Unexpected Behaviour。

### How?

在Vuejs组件中，有时候我们需要监听一些DOM事件，或者想对DOM节点进行直接的操作，获取一些DOM层面的数据。我们可以使用自定义指令进行DOM操作，可以看Vue的官方文档[Custom Directives](https://vuejs.org/v2/guide/custom-directive.html)，以及[使用Vue Directive封装DOM操作](https://elegenthus.github.io/post/VueDirectivesTest/)这篇博客。

## 抽离浏览器API操作

在组件的生命周期函数或者方法中，有时候需要进行网络请求，或者使用BOM API（比如读取localStorage）。所以我们需要将这些操作分拆、封装到对应的模块里。

### Why?

+ Separation of Concern。在ViewModel只对数据进行操作。隐藏一些与数据驱动无关的实现细节。

### How?

*cookie.js*

```
function setCookie(cname, cvalue, exdays) {
    var d = new Date();
    d.setTime(d.getTime() + (exdays * 24 * 60 * 60 * 1000));
    var expires = "expires=" + d.toUTCString() + ";path = /";
    document.cookie = cname + "=" + cvalue + "; " + expires;
}
//获取cookie
function getCookie(cname) {
    var name = cname + "=";
    var ca = document.cookie.split(';');
    for (var i = 0; i < ca.length; i++) {
        var c = ca[i];
        while (c.charAt(0) == ' ') c = c.substring(1);
        if (c.indexOf(name) != -1) return c.substring(name.length, c.length);
    }
    return "";
}

const Cookie = {
    setCookie: setCookie,
    getCookie: getCookie
}

export default Cookie

```

*app.vue*

```
<script>
import Cookie from '../cookie.js'
export default{
	data (){
		return {
         foo: "bar"
		}
	},
  	methods:{
		handler() {
			let token = Cookie.getCookie("token");
		}
  	}
}
</script>

```

## 组件通信规范

### Why?

### How?

## 合理使用data和props

对组件的数据进行分类，其中组件自身的内部状态是data，从父组件传递而来的数据则是props。对于props，要进行类型的校验，并且限制props的类型为原始类型。

### Why?

### How?

## 使用组合进行代码复用

### Why?

### How?

## 在列表中加入key

### Why?

### How?

## 分离静态HTML

