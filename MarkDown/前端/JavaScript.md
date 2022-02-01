# 一、JavaScript 编写位置

> 写在 `<script>` 标签中（推荐）

```html
<script type="text/javascript">
	alert("你好");
</script>
```



> 写在按钮 `onclick` 属性中（不推荐）

```html
<button onclick="alert('讨厌...')" type="button">点我一下</button>
```



> 写在超链接 `href` 属性中（不推荐）

```html
<a href="javascript:alert('讨厌...')">metoo</a>
```



> 还可以写在**外部 JS 文件**中，通过 **script** 标签引入进来（开发中常用方式）

```html
<script src="./js/outer.js" type="text/javascript" charset="utf-8">
    <!--一旦该 script 标签引入了外部 JS 文件，内部 JS 代码就会失效。需要重新创建一个 script 标签-->
</script>
```



# 二、基本语法

**JS 中严格区分大小写** 



> 声明变量：**var  x = 123;** 

关键字：**var** 



## 1、数据类型

* 数据类型就是字面量的值
    * JS 中不像其他语言可以使用具体的类型来声明变量，所有变量都是用 var 来声明
    * 所以只能由字面量所表示的含义来判断变量类型
* 获取变量类型：**typeof  a** 

> **JS** 中有六大数据类型：
> 	**String-字符串、Number-数值、Boolean-布尔值、Null-空值、Undefined-未定义、Object-对象** 



**String**-字符串用引号 ("") 包含，单引号、双引号都行

* var  string="tfc";



## 2、强制类型转化

* 调用 **toString()、String()** 方法将其他值转换为字符串值

```html
<script type="text/javascript">
    var a = 123;
    var str = a.toString();
</script>
```

* 调用 **Number()、parseInt()、parseFloat()** 将其他值转换为数值
    * **parseInt()、parseFloat()** 可以提取数字部分进行转换，只能提取以数字开头的：**123a->123** 

```html
<script type="text/javascript">
    var a = "123";
    var str = Number(a);
</script>
```



## 3、对象

对象的分类：

> 内建对象

* 由ES标准中定义的对象，在任何的ES的实现中都可以使用
* 比如：**Math、String、Number、Boolean、Function、Object....** 

> 宿主对象

* 由 **JS** 的运行环境提供的对象，目前来讲主要指由浏览器提供的对象，比如 **BOM、DOM** 

> 自定义对象

* 由开发人员自己创建的对象



创建对象：**var obj = new Object();** 

添加属性

```javascript
//向obj中添加一个name属性
obj.name = "孙悟空";
//向obj中添加一个gender属性
obj.gender = "男";
//向obj中添加一个age属性
obj.age = 18;
```

删除属性

* 语法：delete 对象.属性名
* **delete obj.name;** 



​	如果要使用特殊的属性名，不能采用 "." 的方式来操作，需要使用另一种方式，读取时也需要采用这种方式。在 [ ] 中可以直接传递一个变量，这样变量值是多少就会读取那个属性

* 语法：对象 ["属性名"] = 属性值



​	JS中的变量都是保存到栈内存中的，基本数据类型的值直接在栈内存中存储，值与值之间是独立存在，修改一个变量不会影响其他的变量

​	对象是保存到堆内存中的，每创建一个新的对象，就会在堆内存中开辟出一个新的空间，而**变量保存的是对象的内存地址（对象的引用）**，如果两个变量保存的是同一个对象引用，当一个通过一个变量修改属性时，另一个也会受到影响

​	当比较两个基本数据类型的值时，就是比较值

​	而比较两个引用数据类型时，它是**比较的是对象的内存地址**，如果两个对象是一摸一样的，但是地址不同，它也会返回 **false** 



​	使用对象字面量，可以在创建对象时，直接指定对象中的属性，对象字面量的属性名可以加引号也可以不加，建议不加，如果要使用一些特殊的名字，则必须加引号

> 语法：{属性名:属性值,属性名:属性值....}

```javascript
var obj2 = {
    name:"猪八戒",
    age:13,
    gender:"男",
    test:{name:"沙僧"}
};
```



## 4、函数

* 语法一

```js
function 函数名 ([形参1,形参2...形参N]) {
	语句...
};  // 分号不能忘
```

```js
function fun2(){
    console.log("这是我的第二个函数~~~");
    alert("哈哈哈哈哈");
    document.write("~~~~(>_<)~~~~");
}
```

* 语法二

```js
var 函数名  = function([形参1,形参2...形参N]){
	语句....
};
```

```js
var fun3 = function(){
    console.log("我是匿名函数中封装的代码");
};
```



* 带参函数

```js
function sum(a,b){
    console.log("a = "+a);
    console.log("b = "+b);
    console.log(a+b);
}

sum(123,456);
```



* 有返回值的函数

```js
function sum(a , b , c){
    var d = a + b + c;
    return d;
}

var  x = sum(1,2,3);
```



* 立即执行函数
    * 函数定义完，立即被调用，这种函数叫做立即执行函数
    * 立即执行函数往往只会执行一次

> 语法

```js
(function(){
    alert("我是一个匿名函数~~~");
})();
```

```js
(function(a,b){
    console.log("a = "+a);
    console.log("b = "+b);
})(123,456);
```



* 对象方法

​     函数也可以称为对象的属性，如果一个函数作为一个对象的属性保存，那么我们称这个函数时这个对象的方法，调用这个函数就说调用对象的方法（method），但是它只是名称上的区别，没有其他的区别

```js
var obj2 = {
    name:"猪八戒",
    age:18,
    sayName:function(){
     console.log(obj2.name);
    }
};
```



* 枚举对象中的属性：使用 for ... in 语句

```js
var obj = {
    name:"孙悟空",
    age:18,
    gender:"男",
    address:"花果山"
};

for(var n in obj){
    console.log("属性名:"+n);
    console.log("属性值:"+obj[n]);
}
```



## 5、作用域

* 全局作用域

​    直接编写在 **script 标签**中的 **JS** 代码，都在全局作用域，全局作用域在页面打开时创建，在页面关闭时销毁，在全局作用域中有一个**全局对象 window，它代表的是一个浏览器的窗口，它由浏览器创建我们可以直接使用** 

​	**在全局作用域中，创建的变量都会作为 window 对象的属性保存，创建的函数都会作为 window 对象的方法保存**，全局作用域中的变量都是全局变量，在页面的任意的部分都可以访问的到

```js
var c = "hello";
console.log(window.c);

function fun(){
    console.log("我是fun函数");
};
window.fun();
```



## 6、this

​	解析器在调用函数每次都会向函数内部传递进一个隐含的参数，这个隐含的参数就是 **this，this** 指向的是一个对象，这个对象我们称为**函数执行的上下文对象** 

根据函数的**调用方式的不同**，**this** 会指向不同的对象

* 以**函数形式**调用时，**this** 永远都是 **window** 
    * **fun()** 
* 以**方法形式**调用时，**this** 就是调用方法的那个对象
    * **obj.sayName();** 





## 7、构造函数

构造函数就是一个普通的函数

* **创建方式**和普通函数没有区别，不同的是**构造函数函数名习惯上首字母大写** 
* 调用方式的不同
    * 普通函数是直接调用，而构造函数需要使用 **new** 关键字来调用

```js
function Person(name , age , gender){
    this.name = name;
    this.age = age;
    this.gender = gender;
    this.sayName = function(){
     alert(this.name);
    };
}

var per = new Person("孙悟空",18,"男");
```



## 8、原型对象

​	我们所创建的每一个函数，解析器都会向函数中添加一个属性 **prototype** 属性，这个属性对应着一个对象，这个对象就是我们所谓的**原型对象** 

​	如果函数作为普通函数调用 **prototype** 没有任何作用，当函数以构造函数的形式调用时，它所创建的对象中都会有一个隐含的属性，指向该构造函数的原型对象，我们可以通过` __proto__`来访问该属性

​	原型对象就相当于一个公共的区域，**同一个类的所有实例都可以访问到这个原型对象**，我们可以将对象中共有的内容，统一设置到原型对象中

​	当我们访问对象的一个属性或方法时，它会先在对象自身中寻找，如果有则直接使用，如果没有则会去原型对象中寻找，如果找到则直接使用

​	以后我们创建构造函数时，**可以将这些对象共有的属性和方法，统一添加到构造函数的原型对象中**，这样不用分别为每一个对象添加，也不会影响到全局作用域，就可以使每个对象都具有这些属性和方法了

```js
function MyClass(){

}

//向MyClass的原型中添加属性a
MyClass.prototype.a = 123;

//向MyClass的原型中添加一个方法
MyClass.prototype.sayHello = function(){
    alert("hello");
};

var mc = new MyClass();

var mc2 = new MyClass();

//console.log(MyClass.prototype);
//console.log(mc2.__proto__ == MyClass.prototype);

//向mc中添加a属性
mc.a = "我是mc中的a";

//console.log(mc2.a);

mc.sayHello();
```



## 9、数组

创建数组对象
var arr = new Array();

向数组中添加元素
arr[0] = 10;
arr[1] = 33;
arr[2] = 22;
arr[3] = 44;

获取数组的长度：arr.length



使用字面量创建数组时，可以在创建时就指定数组中的元素

* var arr = [1,2,3,4,5,10];



数组中也可以放数组，如下这种数组我们称为二维数组

* arr = [[1,2,3],[3,4,5],[5,6,7]];



> push()

该方法可以向数组的末尾添加一个或多个元素，并返回数组的新的长度

> pop()

该方法可以删除数组的最后一个元素，并将被删除的元素作为返回值返回

> unshift()

向数组开头添加一个或多个元素，并返回新的数组长度

* **arr.unshift("牛魔王","二郎神");** 

> shift()

可以删除数组的第一个元素，并将被删除的元素作为返回值返回

> slice()

该方法不会改变元素数组，而是将截取到的元素封装到一个新数组中返回

* 参数1：截取开始的位置的索引，**包含**开始索引
* 参数2：截取结束的位置的索引，**不包含**结束索引
* **var result = arr.slice(1,4);** 

> splice()

可以用于删除数组中的指定元素

* 参数1：表示开始位置的索引
* 参数2：表示删除的数量
* 参数3：可以传递一些新的元素，这些元素将会自动插入到开始位置索引前边
* **var result = arr.splice(3,0,"牛魔王","铁扇公主","红孩儿");** 

> concat()

可以连接**两个或多个**数组，并将新的数组返回，该方法不会对原数组产生影响

* **var result = arr.concat(arr2,arr3,"牛魔王","铁扇公主");** 

> join()

该方法可以将数组转换为一个字符串，该方法不会对原数组产生影响，而是将转换后的字符串作为结果返回

* 在 join() 中可以指定一个字符串作为参数，这个字符串将会成为数组中元素的连接符
* 如果不指定连接符，则默认使用 "," 作为连接符
* **result = arr.join("@-@");** 

> reverse()

该方法用来反转数组

* 该方法**会直接修改原数组** 

> sort()

可以用来对数组中的元素进行排序

* 也**会影响原数组**，默认会按照 **Unicode** 编码进行排序
* 即使对于纯数字的数组，也会按照Unicode编码来排序，所以**对数字进排序时，可能会得到错误的结果** 



> forEach()：这个方法只支持 IE8 以上的浏览器

**forEach()** 方法需要一个函数作为参数

* 像这种函数，由我们创建但是不由我们调用的，我们称为**回调函数** 

```js
/*
	浏览器会在回调函数中传递三个参数：
		第一个参数，就是当前正在遍历的元素
		第二个参数，就是当前正在遍历的元素的索引
		第三个参数，就是正在遍历的数组
*/
arr.forEach(function(value , index , obj){
    console.log(value);
});
```



# 三、查询

我们可以在事件对应的属性中设置一些js代码，这样当事件被触发时，这些代码将会执行

```js
<button id="btn">我是一个按钮</button>
<script type="text/javascript">
    //获取按钮对象
    var btn = document.getElementById("btn");
	//绑定一个单击事件
	//像这种为单击事件绑定的函数，我们称为单击响应函数
	btn.onclick = function(){
    	alert("你还点~~~");
	};
</script>
```



## 1、文档加载顺序

浏览器在加载一个页面时，是按照自上向下的顺序加载的，读取到一行就运行一行

* 如果将 `script` 标签写到 ` <head>` 标签里面，在代码执行时，页面还没有加载，页面没有加载 **DOM** 对象也没有加载，会导致无法获取到 **DOM** 对象，无法获取到 **DOM** 对象也就无法获取元素



> 解决方案

​		**onload 事件**会在整个页面加载完成之后才触发，**为 window 绑定一个 onload 事件**，该事件对应的响应函数将会在页面加载完成之后执行，这样可以确保我们的代码执行时所有的 **DOM** 对象已经加载完毕了

* 这样可以将所有代码写到该事件中

```js
window.onload = function(){
    //获取id为btn的按钮
    var btn = document.getElementById("btn");
    //为按钮绑定一个单击响应函数
    btn.onclick = function(){
     alert("hello");
    };
};
```



## 2、DOM 查询

* 获取元素对象

> 通过 **document** 对象获取

1、**getElementById()**：通过 **id** 属性获取一个元素节点对象

2、**getElementsByTagName()**：通过标签名获取一组元素节点对象

3、**getElementsByName()**：通过 **name** 属性获取一组元素节点对象





## 3、DOM 增删改

> 在 **ul** 标签下增加一个 **li** 节点

1、创建 **li** 节点

* **var li = document.createElement("li");** 

2、创建文本节点

* **var gzText = document.createTextNode("广州");** 

3、添加

* **li.appendChild(gzText);** 



> 插入一个节点

1、创建一个节点

* **var li = document.createElement("li");** 
* **var gzText = document.createTextNode("广州");** 
* **li.appendChild(gzText);** 

2、插入

 **insertBefore** 可以在指定的子节点前插入新的子节点。**语法：父节点.insertBefore( 新节点 , 旧节点 );** 

* **city.insertBefore(li , bj);** 



> 替换一个节点

```js
//使用"广州"节点替换#bj节点
myClick("btn03",function(){
    //创建一个广州
    var li = document.createElement("li");
    var gzText = document.createTextNode("广州");
    li.appendChild(gzText);

    //获取id为bj的节点
    var bj = document.getElementById("bj");

    //获取city
    var city = document.getElementById("city");

    /*
     * replaceChild()
     * 	- 可以使用指定的子节点替换已有的子节点
     * 	- 语法：父节点.replaceChild(新节点,旧节点);
     */
    city.replaceChild(li , bj);
});
```



> 删除一个节点

```js
//删除#bj节点
myClick("btn04",function(){
    //获取id为bj的节点
    var bj = document.getElementById("bj");
    //获取city
    var city = document.getElementById("city");

    /*
	* removeChild()
	* 	- 可以删除一个子节点
	* 	- 语法一：父节点.removeChild(子节点);
	* 	- 语法二：子节点.parentNode.removeChild(子节点);
	*/
    //city.removeChild(bj);
    bj.parentNode.removeChild(bj);
});
```



> 读取标签中的文本内容

```js
//读取#city内的HTML代码
myClick("btn05",function(){
    //获取city
    var city = document.getElementById("city");
    alert(city.innerHTML);
});

//设置#bj内的HTML代码
myClick("btn06" , function(){
    //获取bj
    var bj = document.getElementById("bj");
    bj.innerHTML = "昌平";
});
```





















































