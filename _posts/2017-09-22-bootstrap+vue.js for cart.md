---
layout: post
title:  "bootstrap+vue.js实现简单的购物车功能"
color:  green
width:   6
height:  1
date:   2017-09-22 23:03:49 +0800
categories: jekyll update
---
因为此前做过一段时间Django的后端开发，前端方面的知识一直非常欠缺，总觉得前端的知识太繁杂，无从下手。最近工作比较闲，就研究了一阵子，从最基础的html、js开始看，这个星期学习了一下vue.js，觉得真是棒极了。这篇是使用bootstrap+vue.js实现的一个简单的购物车功能。大部分跟着[https://segmentfault.com/a/1190000010801357](http://note.youdao.com/)做的，html的部分采用了bootstrap来写。
## 准备工作
为什么选择vue.js？瞎选的，就像瞎选了Django学一样，不过我猜想用到最后框架无非是大同小异吧。
首先先来个简单的。实现胶囊式导航栏显示切换。


```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>try VUE</title>
	<link rel="stylesheet" href="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/css/bootstrap.min.css">
	<script src="http://cdn.static.runoob.com/libs/jquery/2.1.1/jquery.min.js"></script>
	<script src="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/js/bootstrap.min.js"></script>
  <script src="https://cdn.bootcss.com/vue/2.2.2/vue.min.js"></script>
</head>
<body>
  <div id="container">
    <ul class="nav nav-pills" id="test">
    	<li @click="curId=0" :class="{'active':curId===0}"><a href="#">html</a></li>
    	<li @click="curId=1" :class="{'active':curId===1}"><a href="#">css</a></li>
    	<li @click="curId=2" :class="{'active':curId===2}"><a href="#">javascript</a></li>
    	<li @click="curId=3" :class="{'active':curId===3}"><a href="#">vue</a></li>
    </ul>
    <div v-show="curId===0">
        html<br/>
    </div>
    <div v-show="curId===1">
        css
    </div>
    <div v-show="curId===2">
        javascript
    </div>
    <div v-show="curId===3">
        vue
    </div>
  </div>
</body>
<script>

new Vue({
    el: '#container',
    data: {
        curId: 0
    },
    computed: {},
    methods: {},
    mounted: function () {
    }
})
</script>
</html>

```
1. 引入bootstrap和vue，这里用的都是cdn的方式。
2. html中这一段：

```
<ul class="nav nav-pills" id="test">
    	<li @click="curId=0" :class="{'active':curId===0}"><a href="#">html</a></li>
    	<li @click="curId=1" :class="{'active':curId===1}"><a href="#">css</a></li>
    	<li @click="curId=2" :class="{'active':curId===2}"><a href="#">javascript</a></li>
    	<li @click="curId=3" :class="{'active':curId===3}"><a href="#">vue</a></li>
</ul>
<div v-show="curId===0">
    html<br/>
</div>
<div v-show="curId===1">
    css
</div>
<div v-show="curId===2">
    javascript
</div>
<div v-show="curId===3">
    vue
</div>
```
- class="nav nav-pills"就是bootstrap中的胶囊式导航栏；
- @click="curId=0"，@click是v-on:click的缩写形式，这句话的作用呢就是click到这个元素的时候把curId设为0；
- :class是v-bind:class的缩写形式，这句话就是说当curId==0的时候给这个元素加一个class:active（导航的这个元素被选中的效果）。
- v-show="curId===0"，v-show根据表达式之真假值，切换元素的 display CSS 属性，当我们click到某个元素时，curId的值发生变化，同时绑定class="active"，div中相应id的段落display。

```
<script>

new Vue({
    el: '#container',
    data: {
        curId: 0
    },
    computed: {},
    methods: {},
    mounted: function () {
    }
})
</script>
```
js代码中el表明作用的范围，data我的理解就是传到这个范围内的DOM里的数据。
## 购物车功能实现
首先先放出代码试运行一下，基本的功能包括：
1. 全选（手动选上全部checkbox时，全选框也被勾上）
2. 实时显示所有被选商品数和总价
3. 删除某条记录
4. 删除所有记录  

可以想一下这些功能大概需要怎么样去记录。比如全选应该需要维持一个变量记录是不是所有的商品被选上，也应该有个方法遍历所有记录全部选中；比如删除某条记录应该有个方法接受记录的id并进行记录删除。放上代码：

```
<!DOCTYPE html>
<html>
<head>
	<meta charset="utf-8">
	<title>购物车</title>
	<link rel="stylesheet" href="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/css/bootstrap.min.css">
	<script src="http://cdn.static.runoob.com/libs/jquery/2.1.1/jquery.min.js"></script>
	<script src="http://cdn.static.runoob.com/libs/bootstrap/3.3.7/js/bootstrap.min.js"></script>
  <script src="https://cdn.bootcss.com/vue/2.2.2/vue.min.js"></script>
</head>
<body>
	<div class="container">
  <div id="shopping-cart">
    <h4 class="cart-title">购物清单</h4>
     <table class="table table-hover">
        <thead>
           <tr>
              <th>
								<div class="checkbox">
								    <label>
								      <input type="checkbox" @click="selectAll(isSelectAll)" v-bind:checked="isSelectAll"> 全选
								    </label>
								  </div>
              </th>
              <th>商品</th>
              <th>数量</th>
              <th>单价</th>
              <th>金额（元）</th>
              <th>操作</th>
           </tr>
        </thead>
        <tbody>
           <tr v-for="(item,index) in productList">
              <td>
								<label>
                  <input type="checkbox" @click="item.select=!item.select;console.log(item.select)" v-bind:checked="item.select">
                </label>
              </td>
              <td>{{item.name}}</td>
              <td>
                <div class="product-num" style="border: 1px solid #e3e3e3;
                display: inline-block;
                text-align: center;">
                    <a href="javascript:;" @click="item.num>0?item.num--:0">-</a>
                    <input type="text" class="num-input" v-model="item.num" style="width: 42px;
                      height: 28px;
                      line-height: 28px;
                      border: none;
                      text-align: center;">
                    <a href="javascript:;" @click="item.num++">+</a>
                </div>
              </td>
              <td>{{item.price}}</td>
              <td>{{item.price*item.num}}</td>
              <td><a href="javascript:;" @click="deleteProduct(index)">删除</a></td>
           </tr>
        </tbody>
     </table>
     <div class="col-sm-2">
         <span class="glyphicon glyphicon-trash"></span>
				 <label><a href="javascript:;" @click="deleteSelectedProduct">删除所选商品</a></label>
     </div>
     <div class="col-sm-2">
         <span class="glyphicon glyphicon-shopping-cart"></span> 继续购物
     </div>
     <div class="col-sm-4">

     </div>
     <div class="col-sm-2">
         <span>{{getTotal.totalNum}}</span>件商品总计（不含运费）：￥<span>{{getTotal.totalPrice}}</span>
     </div>
     <div class="col-sm-2">
         <button type="button" class="btn btn-primary">去结算</button>
     </div>
  </div>
</div>
</body>
<script>
new Vue({
    el:'#shopping-cart',
    data:{
        productList:[
            {
                'name': 'iphone8',//产品名称
                'num': 3,//数量
                'price': 4999//单价
            },
						{
                'name': '小米6',//产品名称
                'num': 3,//数量
                'price': 2999//单价
            },
						{
                'name': 'samsung',//产品名称
                'num': 1,//数量
                'price': 4550//单价
            }
        ],
    },
    computed: {
			isSelectAll:function(){
			    return this.productList.every(function (val) { return val.select});
			},
			getTotal:function(){
				var list = this.productList.filter(function (item) {return item.select});
				var totalPrice = 0;
				var totalNum = 0;
				for(var i=0,len=list.length;i<len;i++){
            totalNum += list[i].num;
            totalPrice += list[i].num*list[i].price;
        }
				return {totalNum:totalNum, totalPrice:totalPrice};
			}
		},
    methods:{
			selectAll:function(_isSelect){
				for (var i = 0, len = this.productList.length; i < len; i++) {
                    this.productList[i].select = !_isSelect;
                }
			},
			deleteProduct:function(index){
				this.productList.splice(index,1);
			},
			deleteSelectedProduct:function(){
				this.productList=this.productList.filter(function (item) {return !item.select});
			}
    },
    mounted:function () {
            var _this=this;
            //为productList添加select（是否选中）字段，初始值为true
            this.productList.map(function (item) {
                _this.$set(item, 'select', false);
            })
        }
})
</script>
</html>

```
-
```
<input type="checkbox" @click="selectAll(isSelectAll)" v-bind:checked="isSelectAll">
```
 看一下全选的实现方法，和一开始的设计想法一致，click绑定一个方法，checked属性也绑定一个变量。值得说一下这里的isSelectAll是一个计算属性，计算属性是缓存的，基于它们的依赖进行缓存，只有在它的相关依赖发生改变时才会重新求值。也就是说只有在你改变了某一个记录的选中状态后才会重新计算isSelectAll，在此之前访问会立刻返回上一次的计算结果。
-
```
<tr v-for="(item,index) in productList">
```
，v-for基于源数据多次渲染元素或模板块，这里带索引是为了后面删除某条记录时方便传入id；
-
```
<a href="javascript:;" @click="item.num>0?item.num--:0">-</a>
```
 注意这里的细节，数量是不能减成负数的；
-  input由于绑定了v-model="item.num"，vue其实在其中帮我们做了很多的事情，比如总金额这里
```
<td>{{item.price*item.num}}</td>
```
，由于也是item.num渲染的，所以不需要我们做什么，自动计算单件商品的总金额功能已经实现了。
-  还有一点数据方面需要注意的，因为后端传来的数据肯定不会包含是不是被选中的，这是一个前端数据，但是也需要跟随每一条记录，所以使用了生命周期钩子mounted，关于这个，官方文档是这样写的：el 被新创建的 vm.$el 替换，并挂载到实例上去之后调用该钩子，其实我不是特别明白，大概理解就是mounted之后就做的事情吧。。。

---
初步学习了一下vue.js，觉得甚是神奇。
