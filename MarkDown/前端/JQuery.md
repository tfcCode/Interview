> 简介

​		JQuery 是一款 JavaScript 库，用来简化原生 JS 操作

> 版本

* 1.x 兼容 IE 6、7、8
  * 相对其它版本文件较大，官方只做BUG维护，功能不再新增，**最终版本：1.12.4（2016年5月20日）** 
* 2.x  不兼容 IE 6、7、8
  * 相对其它版本文件较小，官方只做BUG维护，功能不再新增，**最终版本：2.2.4（2016年5月20日）** 
* 3.x 不兼容 IE 6、7、8
  * 只支持最新的浏览器，很多老的 JQuery 插件不支持这个版本，相对 1.x 文件较小，提供不包含 Ajax 动画 API 版本



> 简单使用

1、下载 **JQuery** 库

2、引入 **JQuery** 文件

```html
<head>
    <script src="js/jquery.js" type="text/javascript" charset="utf-8"></script>
</head>
```

3、编写 **JQuery** 代码

```html
<script>
     原生 JS
    window.onload = function () {

    }
     JQuery 
    $(document).ready(function () {
        alert("Hello JQuery")
    });
</script>
```



## 1、原生 JS 和 JQuery 的区别

> 原生 JS 和 JQuery 入口函数加载模式不同

* 原生 JS 会等到 DOM 元素加载完毕，并且图片也加载完毕后才执行
* JQuery 会等到 DOM 元素加载完毕，但不会等到图片加载完毕就执行

* 原生 JS 如果写了多个入口函数，后面的会覆盖前面的，但 JQuery 不会，JQuery 会依次执行



## 2、JQuery 入口函数写法

* 方式一

```html
$(document).ready(function () {
    alert("Hello JQuery")
});
```

* 方式二

```html
jQuery(document).ready(function () {
    alert("Hello JQuery")
});
```

* 方式三（推荐）

```html
$(function () {
    alert("Hello JQuery")
});
```

* 方式四

```html
jQuery(function () {
    alert("Hello JQuery")
});
```



## 3、JQuery 的冲突问题

JQuery 中的 $ 符号可能会和其他框架冲突，解决办法：释放 $ 的使用权，改用 JQuery 代替

```html
<script>
     释放 $ 符号使用权
    jQuery.noConflict();
    jQuery(document).ready(function () {
        alert("Hello JQuery")
    });
</script>
```

* 注意
  * 释放操作必须在其他 JQuery 代码之前执行
  * 释放之后就不能再使用 $，改用 **jQuery** 

**==也可以自定义一个符号代替 $==** 

```html
<script>
     释放 $ 符号使用权
    var pt = jQuery.noConflict();
    pt(document).ready(function () {
        alert("Hello JQuery")
    });
</script>
```

## 4、核心函数：$();

### 4.1 参数一：函数

* 作为入口函数

```html
$(function () {
    alert("Hello JQuery")
});
```

### 4.2 参数二：字符串

> 接收一个字符串选择器

* 返回一个 JQuery 对象，里面保存了找到的 DOM 元素

```html
var box1 = $(".box1");
var box2 = $("#box2")
```

> 接收一个代码片段

* 返回一个 JQuery 对象，里面保存了创建好的 DOM 元素

```html
 接收一个字符串代码片段
var p = $("<p>我是段落</p>")
console.log(p);
```

### 4.3 参数三：DOM 元素

* 返回一个包装好的 JQuery 对象

```html
var span = document.getElementsByTagName("span")[0];
var $span = $(span);
console.log($span);
```



## 5、JQuery 对象

**jQuery 对象 是一个伪数组** 

> 伪数组

​		包含 0~length-1 个属性，并且含有 length 这个属性



## 6、静态方法、实例方法

> each 方法

```javascript
var arr = [1, 3, 5, 7, 9];
var obj = {0:1, 1:3, 2:5, 3:7, 4:9, length:5}; 伪数组

原生的forEach方法只能遍历数组, 不能遍历伪数组（第一个参数: 遍历到的元素，第二个参数: 当前遍历到的索引）
arr.forEach(function (value, index) {  可以
	console.log(index, value);
});
obj.forEach(function (value, index) {  不可以
	console.log(index, value);
});

利用jQuery的each静态方法遍历数组，也可以编译伪数组
$.each(arr, function (index, value) {
	console.log(index, value);
});
$.each(obj, function (index, value) {
	console.log(index, value);
});
```

> holdReady(true) 方法

作用: 暂停入口函数的执行，true 为暂停，false 为执行

```html
<script>
     $.holdReady(true); 作用: 暂停ready执行
    $.holdReady(true);
    $(document).ready(function () {
        alert("ready");
    });
</script>
```



## 7、内容选择器

```html
<script>
    $(function () {
         :empty 作用:找到既没有文本内容也没有子元素的指定元素
        var $div = $("div:empty");
        console.log($div);

          :parent 作用: 找到有文本内容或有子元素的指定元素
        var $div = $("div:parent");
        console.log($div);

         :contains(text) 作用: 找到包含指定文本内容的指定元素
        var $div = $("div:contains('我是div')");
        console.log($div);

         :has(selector) 作用: 找到包含指定子元素的指定元素
        var $div = $("div:has('span')");
        console.log($div);
    });
</script>
```



## 8、属性操作

1、什么是属性节点？

​		`<span name = "it666"></span>`，在编写 HTML 代码时，在 HTML 标签中添加的属性就是属性节点。在浏览器中找到 span 这个 DOM 元素之后, 展开看到的都是属性，**在 attributes 属性中保存的所有内容都是属性节点** 

2、如何操作属性节点?

```html
  赋值：DOM元素.setAttribute("属性名称", "值");
获取值：DOM元素.getAttribute("属性名称");

var span = document.getElementsByTagName("span")[0];
span.setAttribute("name", "lnj");
console.log(span.getAttribute("name"));
```

任何对象都有属性, 但是只有 DOM 对象才有属性节点



### attr 和 removeAttr 方法

> attr(name|pro|key,val|fn) 方法

​		作用是获取或者设置属性节点的值，可以传递一个参数, 也可以传递两个参数，如果传递**一个参数, 代表==获取==属性节点的值**。如果传递**两个参数, 代表==设置==属性节点的值** 

* 注意点:
  * 如果是获取：无论找到多少个元素, 都**只会返回第一个元素指定的属性节点的值** 
    * `console.log($("span").attr("class"));` 
  * 如果是设置：找到多少个元素就会设置多少个元素
    * `$("span").attr("class", "box");` 
  * 如果是设置：**如果设置的属性节点不存在, 那么系统会自动新增** 
    * `$("span").attr("abc", "123");` 

> removeAttr(name) 方法

删除属性节点

* **会删除所有找到元素指定的属性节点** 



### prop 和 removeProp 方法

> prop() 方法

特点和 attr 方法一致

* **eq（0）表示得到第一个元素** 

```html
<span class="span1" name="it666"></span>
<span class="span2" name="lnj"></span>

$("span").eq(0).prop("demo", "it666");
$("span").eq(1).prop("demo", "lnj");
console.log($("span").prop("demo"));
```

> removeProp() 方法

特点和 removeAttr 方法一致：`$("span").removeProp("demo");` 



注意：prop方法不仅能够操作属性, 他还能操作属性节点



### 什么时候用 attr 和 prop？

```html
<input type="checkbox">

console.log($("input").prop("checked"));  true / false
console.log($("input").attr("checked"));  checked / undefined
```

1、prop 方法操作 input 时如果选中返回 true，否则返回 false

2、attr 方法操作 input 时如果选中返回 `checked`，否则返回 `undefined` 

​		官方推荐在操作属性节点时，**具有 true 和 false 两个属性的属性节点，如 checked, selected 或者 disabled 使用 prop()，其他的使用 attr()** 



## 9、操作 CSS 类

**addClass 方法**：添加一个类，如果要添加多个, 多个类名之间用**==空格==**隔开即可

**removeClass 方法**：删除一个类，如果想删除多个, 多个类名之间用**==空格==**隔开即可

**toggleClass 方法**：切换类（有就删除, 没有就添加）

```html
<body>
    <button>添加类</button>
    <button>删除类</button>
    <button>切换类</button>
    <div></div>
</body>

<script>
    $(function () {
        var btns = document.getElementsByTagName("button");
        btns[0].onclick = function () {
            $("div").addClass("class1 class2");
        }
        btns[1].onclick = function () {
            $("div").removeClass("class2 class1");
        }
        btns[2].onclick = function () {
            $("div").toggleClass("class2 class1");
        }
    });
</script>
```



## 10、操作 CSS 样式

```html
<body>
	<div></div>
</body>
<script>
    $(function () {
         1.逐个设置
        $("div").css("width", "100px");
        $("div").css("height", "100px");
        $("div").css("background", "red");

         2.链式设置
         注意点: 链式操作如果大于3步, 建议分开
        $("div").css("width", "100px").css("height", "100px").css("background", "blue");

         3.批量设置
        $("div").css({
            width: "100px",
            height: "100px",
            background: "red"
        });

         4.获取CSS样式值
        console.log($("div").css("background"));;
    });
</script>
```



## 11、操作文本

**html() 方法**：和原生JS中的 innerHTML 一模一样

**text() 方法**：和原生JS中的 innerText 一模一样

**val () 方法**：和 input 的 value 属性差不多

```html
<body>
    <button>设置html</button>
    <button>获取html</button>
    <button>设置text</button>
    <button>获取text</button>
    <button>设置value</button>
    <button>获取value</button>
    <div></div>
    <input type="text">
</body>

<script>
    $(function () {
        var btns = document.getElementsByTagName("button");
        btns[0].onclick = function () {
            $("div").html("<p>我是段落<span>我是span</span></p>");
        }
        btns[1].onclick = function () {
            console.log($("div").html());
        }
        btns[2].onclick = function () {
            $("div").text("<p>我是段落<span>我是span</span></p>");
        }
        btns[3].onclick = function () {
            console.log($("div").text());
        }
        btns[4].onclick = function () {
            $("input").val("请输入内容");
        }
        btns[5].onclick = function () {
            console.log($("input").val());
        }
    });
</script>
```



## 12、操作位置和尺寸大小

```html
<body>
    <div class="father">
        <div class="son"></div>
    </div>
    
    <button>获取</button>
    <button>设置</button>
</body>

<script>
    $(function () {
         编写jQuery相关代码
        var btns = document.getElementsByTagName("button");
         监听获取
        btns[0].onclick = function () {
             获取元素的宽度
             console.log($(".father").width());

             offset([coordinates])
             作用: 获取元素距离窗口的偏移量
             console.log($(".son").offset().left);

             position()
             作用: 获取元素距离定位元素的偏移量
            console.log($(".son").position().left);
        }
         监听设置
        btns[1].onclick = function () {
             设置元素的宽度
             $(".father").width("500px")

             设置偏移量
             $(".son").offset({
                 left: 10
             });

             //注意点: position 方法只能获取不能设置
             $(".son").position({
                 left: 10  无效
             });

            $(".son").css({
                left: "10px"  有效
            });
        }
    });
</script>
```

**滚动** 

```html
<body>
    <div class="scroll">我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div我是div</div>
    
    <button>获取</button>
    <button>设置</button>
    <br>
    ....   为了获得网页滚动效果
    <br>
</body>

<script>
    $(function () {
        var btns = document.getElementsByTagName("button");
         监听获取
        btns[0].onclick = function () {
             获取滚动的偏移位
             console.log($(".scroll").scrollTop());

             获取网页滚动的偏移位
             注意点: 为了保证浏览器的兼容, 获取网页滚动的偏移位需要按照如下写法
            console.log($("body").scrollTop()+$("html").scrollTop());
        }
        btns[1].onclick = function () {
             设置滚动的偏移位
            $(".scroll").scrollTop(300);

             设置网页滚动偏移位
             注意点: 为了保证浏览器的兼容, 设置网页滚动偏移位的时候必须按照如下写法
            $("html,body").scrollTop(300);
        }
    });
</script>
```



## 13、事件绑定

> 绑定

```html
<body>
	<button>我是按钮</button>
</body>

<script>
    $(function () {
        // 1.eventName(fn); 编码效率略高，但部分事件 jQuery 没有实现
        $("button").click(function () {
            alert("hello lnj");
        });
        //2.on(eventName, fn); 编码效率略低，但所有 js 事件都可以添加
        $("button").on("click", function () {
            alert("hello click1");
        });
    });
</script>
```

> 解绑

```html
<body>
	<button>我是按钮</button>
</body>

<script>
    $(function () {
        function test1() {
            alert("hello lnj");
        }
        function test2() {
            alert("hello 123");
        }
        // 编写jQuery相关代码
        $("button").click(test1);
        $("button").click(test2);
        
        // off方法如果不传递参数, 会移除所有的事件
        // $("button").off();
        // off方法如果传递一个参数, 会移除所有指定类型的事件
        // $("button").off("click");
        // off方法如果传递两个参数, 会移除所有指定类型的指定事件
        $("button").off("click", test1);
    });
</script>
```



## 14、事件冒泡和默行为

> 什么是事件冒泡？

​		当点击子元素时，父元素也会被点击，这种行为叫做事件冒泡

```html
<div class="father">
    <div class="son"></div>
</div>

<script>
    $(function () {
        $(".son").click(function (event) {
            alert("son");
            return false; 			//方式一： 阻止事件冒泡
            event.stopPropagation();//方式二： 阻止事件冒泡
        });
        $(".father").click(function () {
            alert("father");
        });
    });
</script>
```



> 什么是默认行为？

​		比如一个超链接，默认的点击效果是跳转到指定页面，这就是默认行为

```html
<a href="http://www.baidu.com">注册</a>

<script>
    $(function () {
        $("a").click(function (event) {
            alert("弹出注册框");
            return false;            // 方式一： 阻止默认行为
            event.preventDefault();  // 方式二： 阻止默认行为
        });
    });
</script>
```



## 15、自动触发事件

```html
<div class="father">
    <div class="son"></div>
</div>

<script>
    $(function () {
        $(".son").click(function (event) {
            alert("son");
        });

        $(".father").click(function () {
            alert("father");
        });
        /*
          trigger: 如果利用 trigger 自动触发事件,会触发事件冒泡、默认行为
          triggerHandler: 如果利用 triggerHandler 自动触发事件, 不会触发事件冒泡、默认行为
        */
        $(".son").trigger("click");
        $(".son").triggerHandler("click");
    });
</script>
```



## 16、自定义事件

```html
<script>
    $(function () {
        /*
          想要自定义事件, 必须满足两个条件
          1.事件必须是通过on绑定的
          2.事件必须通过 trigger 来触发
        */
        $(".son").on("myClick", function () {
            alert("son");
        });
        $(".son").triggerHandler("myClick");
    });
</script>
```



## 17、事件委托

当**通过 JQuery 动态添加的 HTML 标签不能响应事件**，就会用到事件委托

```html
<body>
    <ul>
        <li>我是第1个li</li>
        <li>我是第2个li</li>
        <li>我是第3个li</li>
    </ul>
    <button>新增一个li</button>
</body>

<script>
    $(function () {
        $("button").click(function () {
            $("ul").append("<li>我是新增的li</li>");
        })
        /*
         以下代码的含义, 让 ul 帮 li 监听 click 事件
         之所以能够监听, 是因为入口函数执行的时候 ul 就已经存在了, 所以能够添加事件
        之所以 this 是 li,是因为我们点击的是 li, 而 li 没有 click 事件, 所以事件冒泡传递给了 ul,ul 响应了事件, 既然事件是从 li 传递过来的,所以 ul 必然指定 this 是谁
            */
        $("ul").delegate("li", "click", function () {
            console.log($(this).html());
        });
    });
</script>
```

> 练习

```html
<body>
    <a href="http://www.baidu.com">点击登录</a>
</body>

<script>
    $(function () {
        $("a").click(function () {

            var $mask = $("<div class=\"mask\">\n" +
                          "        <div class=\"login\">\n" +
                          "            <img src=\"./img/login.png\">\n" +
                          "            <span></span>\n" +
                          "</div>\n" +
                          " </div>\n");
            $("body").append($mask);

            $("body").delegate(".login>span", "click", function () {
                $mask.remove();
            });

            return false;
        });
    });

</script>
```



## 18、鼠标移入移出事件

```html
<body>
    <div class="father">
        <div class="son"></div>
    </div>
</body>

<script>
    $(function () {

        // mouseover / mouseout 事件, 子元素被移入移出也会触发父元素的事件
        $(".father").mouseover(function () {
            console.log("father被移入了");
        });
        $(".father").mouseout(function () {
            console.log("father被移出了");
        });

        //  mouseenter / mouseleave事件, 子元素被移入移出不会触发父元素的事件,推荐使用
        $(".father").mouseenter(function () {
            console.log("father被移入了");
        });
        $(".father").mouseleave(function () {
            console.log("father被移出了");
        });

        // 效果和 mouseenter / mouseleave 一样
        $(".father").hover(function () {
            console.log("father被移入了");
        },function () {
            console.log("father被移出了");
        });

        // 既监听移入，也监听移出
        $(".father").hover(function () {
            console.log("father被移入移出了");
        });
    });
</script>
```



## 19、动画效果

### 显示隐藏动画

```html
<body>
    <button>显示</button>
    <button>隐藏</button>
    <button>切换</button>
    <div></div>
</body>

<script>
    $(function () {
        $("button").eq(0).click(function () {
            // $("div").css("display", "block");
            // 注意: 这里的时间是毫秒
            $("div").show(1000, function () {
                // 作用: 动画执行完毕之后调用
                alert("显示动画执行完毕");
            });
        });
        $("button").eq(1).click(function () {
            // $("div").css("display", "none");
            $("div").hide(1000, function () {
                alert("隐藏动画执行完毕");
            });
        });
        $("button").eq(2).click(function () {
            $("div").toggle(1000, function () {
                alert("切换动画执行完毕");
            });
        });
    });
</script>
```



> 练习：监听网页滚动事件

```html
<script>
    $(function () {
        // 监听网页的滚动
        $(window).scroll(function () {
            // 获取网页滚动偏移位
            var offset = $("html,body").scrollTop();
            if (offset >= 300) {
                $("img").show(1000);
            } else {
                $("img").hide(1000);
            }
        });
    });
</script>
```



### 展开收起动画

```html
<body>
    <button>展开</button>
    <button>收起</button>
    <button>切换</button>
    <div></div>
</body>

<script>
    $(function () {
        $("button").eq(0).click(function () {
            $("div").slideDown(1000, function () { // 回调函数在执行完毕之后调用
                alert("展开完毕");
            });
        });
        $("button").eq(1).click(function () {
            $("div").slideUp(1000, function () {
                alert("收起完毕");
            });
        });
        $("button").eq(2).click(function () {
            $("div").slideToggle(1000, function () {
                alert("收起完毕");
            });
        });
    });
</script>
```



### 淡入淡出动画

```html
<body>
    <button>淡入</button>
    <button>淡出</button>
    <button>切换</button>
    <button>淡入到</button>
    <div></div>
</body>

<script>
    $(function () {
        $("button").eq(0).click(function () {
            $("div").fadeIn(1000, function () {
                alert("淡入完毕");
            });
        });
        $("button").eq(1).click(function () {
            $("div").fadeOut(1000, function () {
                alert("淡出完毕");
            });
        });
        $("button").eq(2).click(function () {
            $("div").fadeToggle(1000, function () {
                alert("切换完毕");
            });
        });
        $("button").eq(3).click(function () {
            $("div").fadeTo(1000, 0.2, function () {
                alert("淡入完毕");
            })
        });
    });
</script>
```



### 自定义动画

> animate（）方法

```html
<body>
	<button>操作属性</button>
    <button>累加属性</button>
    <button>关键字</button>
    <div class="one"></div>
    <div class="two"></div>
</body>
<script>
    $(function () {
        $("button").eq(0).click(function () {
            /*
                $(".one").animate({
                    width: 500
                }, 1000, function () {
                    alert("自定义动画执行完毕");
                });
                */
            $(".one").animate({
                marginLeft: 500
            }, 5000, function () {
                // alert("自定义动画执行完毕");
            });
            /*
                第一个参数: 接收一个对象, 可以在对象中修改属性
                第二个参数: 指定动画时长
                第三个参数: 指定动画节奏, 默认就是swing
                第四个参数: 动画执行完毕之后的回调函数
                */
            $(".two").animate({
                marginLeft: 500                            // 直接操作属性
            }, 5000, "linear", function () {
                // alert("自定义动画执行完毕");
            });
        })
        $("button").eq(1).click(function () {
            $(".one").animate({
                width: "+=100"							  // 属性累加
            }, 1000, function () {
                alert("自定义动画执行完毕");
            });
        });
        $("button").eq(2).click(function () {
            $(".one").animate({
                // width: "hide"
                width: "toggle"							  // 操作关键字
            }, 1000, function () {
                alert("自定义动画执行完毕");
            });
        })
    });
</script>
```



> delay（）、stop（）方法

```html
<body>
    <button>开始动画</button>
    <button>停止动画</button>
    <div class="one" style="display: none"></div>
    <div class="two"></div>
</body>
<script>
    $(function () {
        $("button").eq(0).click(function () {
            // 在jQuery的{}中可以同时修改多个属性, 多个属性的动画也会同时执行
            // delay 方法的作用就是用于告诉系统延迟时长
            $(".one").animate({
                width: 500
                // height: 500
            }, 1000).delay(2000).animate({
                height: 500
            }, 1000);
        });
        $("button").eq(1).click(function () {
            // 立即停止当前动画, 继续执行后续的动画
            // $("div").stop();
            // $("div").stop(false);
            // $("div").stop(false, false);

            // 立即停止当前和后续所有的动画
            // $("div").stop(true);
            // $("div").stop(true, false);

            // 立即完成当前的, 继续执行后续动画
            // $("div").stop(false, true);

            // 立即完成当前的, 并且停止后续所有的
            $("div").stop(true, true);
        });
    });
</script>
```



## 20、添加节点

```html
<body>
    <button>添加节点</button>
    <ul>
        <li>我是第1个li</li>
        <li>我是第2个li</li>
        <li>我是第3个li</li>
    </ul>
    <div>我是div</div>
</body>

<script>
    $(function () {
        /*
            内部插入
            append(content|fn)
            appendTo(content)
            会将元素添加到指定元素内部的最后

            prepend(content|fn)
            prependTo(content)
            会将元素添加到指定元素内部的最前面

            外部插入
            after(content|fn)
            insertAfter(content)
            会将元素添加到指定元素外部的后面

            before(content|fn)
            insertBefore(content)
            会将元素添加到指定元素外部的前面
            */
        $("button").click(function () {
            // 1.创建一个节点
            var $li = $("<li>新增的li</li>");
            // 2.添加节点
            $("ul").append($li);
            $("ul").prepend($li);

            // $li.appendTo("ul");
            // $li.prependTo("ul");
            
            // $("ul").after($li);
            // $("ul").before($li);
            // $li.insertAfter("ul");
        });
    });
</script>
```



## 21、删除节点

> remove（）、detach（）方法

​		都是删除指定元素，**但利用 remove 删除之后再重新添加，原有的事件无法响应，利用 detach 删除之后再重新添加，原有事件可以响应** 

> empty（）

​		删除指定元素的内容和子元素, **指定元素自身不会被删除** 

```html
<body>
    <button>删除节点</button>
    <ul>
        <li class="item">我是第1个li</li>
        <li>我是第2个li</li>
        <li class="item">我是第3个li</li>
        <li>我是第4个li</li>
        <li class="item">我是第5个li</li>
    </ul>
    <div>我是div
        <p>我是段落</p>
    </div>
</body>

<script>
    $(function () {
        $("button").click(function () {
            // $("div").remove();
            // $("div").empty();
            // $("li").remove(".item");
            var $div = $("div").detach();
            // console.log($div);
            $("body").append($div);
        });
        $("div").click(function () {
            alert("div被点击了");
        });
    });
</script>
```



## 22、替换节点

```html
<body>
    <button>替换节点</button>
    <h1>我是标题1</h1>
    <h1>我是标题1</h1>
    <p>我是段落</p>
</body>

<script>
    $(function () {
        /*
            替换
            replaceWith(content|fn)
            replaceAll(selector)
            替换所有匹配的元素为指定的元素
            */
        $("button").click(function () {
            // 1.新建一个元素
            var $h6 = $("<h6>我是标题6</h6>");
            // 2.替换元素
            // $("h1").replaceWith($h6);
            $h6.replaceAll("h1");
        });
    });
</script>
```



## 23、复制节点

> clone（[Even]，[deepEven]）方法

​		如果传入 false 就是浅复制, 如果传入 true 就是深复制，**浅复制只会复制元素, 不会复制元素的事件，深复制会复制元素, 而且还会复制元素的==事件==** 

```html
<body>
    <button>浅复制节点</button>
    <button>深复制节点</button>
    <ul>
        <li>我是第1个li</li>
        <li>我是第2个li</li>
        <li>我是第3个li</li>
        <li>我是第4个li</li>
        <li>我是第5个li</li>
    </ul>
</body>

<script>
    $(function () {
        $("button").eq(0).click(function () {
            // 1.浅复制一个元素
            var $li = $("li:first").clone(false);
            // 2.将复制的元素添加到ul中
            $("ul").append($li);
        });
        $("button").eq(1).click(function () {
            // 1.深复制一个元素
            var $li = $("li:first").clone(true);
            // 2.将复制的元素添加到ul中
            $("ul").append($li);
        });

        $("li").click(function () {
            alert($(this).html());
        });
    });
</script>
```











