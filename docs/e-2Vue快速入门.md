# Vue

Vue是一套用于构建用户界面的渐进式框架，最大的不同是Vue被设计成可以自底向上逐层应用。Vue的核心库只关注视图层，易于上手并且便于与第三方库（如：vue-router ： 跳转，vue-resource ： 通讯，vuex ： 管理）或项目整合。

------

​	**HTML + CSS + JS ：Vue只负责视图部分。`给用户看的，刷新后台给的数据`**

网络通讯 ： axios

页面跳转 ：vue-router

状态管理 ： vuex

Vue_UI ：ICE

------

Vue本质是一个对象，根据后端的MVC封层思想划分为MVVM模式，也是3层。核心在于ViewModel层，负责转换Model中的数据对象来让数据变得更容易管理和使用。
> 1. 模版层（页面视图，展示页面）
> 2. 数据层（ViewModel，储存视图数据模型）
> 3. 后台获取数据（ajax）


![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231130161748.png)

## **Vue的生命周期分为4部分*：***

> 1. 创建阶段：
>
>    > - beforeCreate：在实例初始化之后，数据观测（data observer）和事件/侦听器配置之前被调用。
>    > - created：在实例创建完成后被调用。此时已经完成数据观测（data observer），属性和方法的运算，watch/event 事件回调。但是尚未挂载到 DOM，无法访问 DOM 元素。
>
> 2. 挂载阶段：
>
>    > - beforeMount：在挂载开始之前被调用。在该钩子函数执行期间，模板已编译完成，但尚未将模板渲染到页面上。
>    > - mounted：在挂载完成后被调用。此时组件已经被渲染到页面上，可以访问到 DOM 元素。通常在这个阶段进行异步操作、请求数据或与第三方库进行交互。
>
> 3. 更新阶段：
>
>    > - beforeUpdate：在数据更新之前被调用，即在重新渲染之前。可以在该钩子函数中进行状态的更新，但要注意避免无限循环更新。
>    > - updated：在数据更新完成后被调用。该钩子函数调用后，组件已经重新渲染，并且更新的 DOM 已经呈现在页面上。
>
> 4. 销毁阶段：
>
>    > - beforeDestroy：在实例销毁之前被调用。此时组件实例仍然完全可用，可以进行必要的清理工作。
>    > - destroyed：在实例销毁后被调用。该钩子函数调用后，组件实例及其相关的 DOM 已经被完全销毁，事件监听和子组件也都被移除。

------



# 第一个Vue项目 

创建一个Vue对象，内部包含数据、方法、计算属性（methods）、监听（watch）等多个属性，然后挂载在某个HTML节点上，此节点将被Vue所管理。

```html
 <script>
     	// vue2写法：
        // new Vue({
        //     el:"#box",
     	//     data:{......},
     	//     methods:{......},
     	//     ...
        // })
     
     // vue3写法: 函数式
        var obj = { 
            data(){ return {......} },
            methods: {......}，
            ......
        }
        Vue.createApp(obj).mount("#box")
        // 或者 const app = Vue.createApp(Vue组件).mount('#app')
    </script>
```

## Vue最常用基本语法

1. **v-bind  动态绑定属性**：

   > v-bind:属性="....." ：可以绑定任何一个HTML或者Vue属性（存疑），
   > PS：
   >
   > ```HTML
   > <div v-bind:class="iscolor?'red':'blue'">背景颜色</div> <!-- 绑定了class属性，iscolor为Vue中data的一个变量 -->
   > <div id="app">
   >     <span v-bingd:title="message">鼠标悬停显示绑定的title提示信息</span>
   > </div>
   > <script>
   >     data() {
   > 	return {iscolor: true ......}
   > }
   > </script>
   > ```

2. **v-if**

   > v-if="判断条件" ：用于判断
   >
   > PS：
   >
   > ```HTML
   > <h1 v-if="type ==='A'">类型为A时输出A</h1>
   > <h1 v-else-if="type ==='B'">类型为B时输出B</h1>
   > <h1 v-else="type ==='C'">类型为C时输出C</h1>
   > <script>
   >     new Vue = {
   >     	...
   >     data:{ type: "A" ....},
   >     	...
   >     }
   > </script>
   > ```

3. **v-for**

   > v-for="item in list" ：遍历list中的值，返回值为item，可以添加Index表示索引位置。
   >
   > ```Html
   > <li v-for="item,index in List">
   >     {{item.message}} ：{{index}}
   > </li>
   > ```

4. **v-on**

   > v-on:事件="触发函数" ：绑定一个事件，事件列表可以从Jquery中查找。
   > PS：
   >
   > ```html
   > <button v-on:click="function(mes)">点击触发</button>
   > <script>
   > 	methods:{
   >         function(mes){
   >             console.log("点击按钮触发函数了")//mes包含了事件相关内容
   >         }
   >     }
   > </script>
   > ```

5. **v-model**

   > v-model="变量"：双向绑定任意一个值或属性。它在表单<input>、<textarea>、<select>元素上时，创建双向数据绑定会自动选择正确的方式来获取元素。本质为语法糖，负责监听用户输入事件以更新数据，并对一些极端场景进行一些处理。

------

## Vue组件

  vue组件可以理解为自定义标签，也就是可以复用的Vue实例。在创建上，使用Vue实例来创建->Vue.component（name，{template：''，....}）创建一个组件。
```html
<script>
    Vue.compone("name",{
        	props:{key1:value1,
                   key2:value2},
            template:`<div> {{ key1 }}</div> `,
        	methods:{......},
        ......
        		})
</script>
```

## 计算属性

​    计算属性的重点突出在 **属性** 上（属性是名词）， 首先这个属性有 **计算**（不仅仅指属性运算，还包括如去掉首字母等一切操作）的能力，这里的计算就是个函数，也就是说，她是一个能够将计算结果缓存起来的属性（将行为转化成静态的属性），类似缓存。**此计算在内存中，只计算一次，更加快速**

```html
<div id="box">
        <!-- 计算属性 :
            只会调用一次函数，
           专用于计算不常变化的属性把值传入缓存中，节省资源
		   因为是一个属性，所以在使用时没有()
	   -->
       计算属性：{{Cnewdate}}
    <!--  methods:
		   因为是一个方法，所以在使用时需要()
	   -->
        methods：{{Mnewdate()}}
    </div>
    <script>
        // 计算属性（防止模板过重难以维护）,使用领域 computed 以 有返回值的函数 表示
        var obj = {
            computed:{// 计算
                Cnewdate(){	return Date.now(); }
            },
            methods: {//方法
                Mnewdate(){	return 0; }
            }
        }
        vm = Vue.createApp(obj).mount("#box")
    </script>
```

## 插槽slot

类似模版引擎里的开了一个可以修改的口，可以在不改变一个页面其他部分的情况下修改一个小部分的内容。

## axios

~~~html
<script>
    methods: {
                handleFetch(){
                    // get请求的方式
                    axios.get("请求资源路径").then(res=>{
                        ...// res：返回的资源，res.data：获取到的资源放在res的data下，是axios要求的
                        this.datalist = res.data.data.films 
                    }).
                    // post请求方式
                    // json              { "name":'djf', "age" : 18}
                    // x-www-formurlencoded  name=djt&age=100
                    axios.post("路径","name=djt&age=20").then(...)
                    axios.post("路径",{name:'djt',age:20}).then(...)
                },
            },
</script>
~~~

