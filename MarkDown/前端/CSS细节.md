# 内外边距

```css
table 
    border-collapse: collapse; /* 合并内外边距*/
    border-spacing: 0px;     /* 指定边框之间的距离*/
}
```



# 颜色渐变

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <style>
            span {
                background: linear-gradient(to right, red, blue);
                /*
                背景延伸到哪一步：（详情：https://developer.mozilla.org/zh-CN/docs/Web/CSS/background-clip）
                */
                background-clip: text;
                -webkit-background-clip: text;
                color: transparent;
            }
        </style>
    </head>
    <body>
        <span>字体渐变颜色</span>
    </body>
</html>
```



# 弹出div层，简易，周围变灰

原理：用一个 div 遮住全屏，背景为灰色，然后再布局

```html
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv='Content-Type' content='text/html; charset=UTF-8'>
    <title>div+css实现弹出层</title>
    <style>
        #popBox {
            position: absolute;
            left: 40%;
            top: 20%;

            display: none;
            width: 300px;
            height: 200px;
            z-index: 11;
            background: #B8F764;
        }
        #popLayer {
            position: absolute;
            left: 0;
            top: 0;
            display: none;
            z-index: 10;
            background: #DCDBDC;
            opacity: 0.8;
            /* filter: alpha(opacity=80); */
        }
    </style>
    <script src="./js/jquery.js"></script>

    <script type="text/javascript">
        function popBox() {
            var hei = $(window).height()
            var wid = $(window).width()

            $("#popLayer").css({
                height: hei,
                width: wid,
                display: "block"
            })
            $("#popBox").css({
                display: "block"
            })
        }
        function closeBox() {
            $("#popBox,#popLayer").css({
                display: "none"
            })
        }
    </script>
</head>
<body>
    <input type="button" name="popBox" value="弹出框" onclick="popBox()" />
    <div id="popLayer">
        背景层
    </div>
    <div id="popBox">
        <div><a href="javascript:void(0)" onclick="closeBox()">关闭</a></div>
        <div>弹出框</div>
    </div>
</body>
</html>
```



# 获取整个页面的高度、宽度

```javascript
// JQuery 方式
var hei = $(window).height()
var wid = $(window).width()
```



# 获取 Session

```javascript
// 添加数据
$.session.set('key', 'value')
// 删除数据
$.session.remove('key');
// 获取数据
$.session.get('key');
// 清除数据
$.session.clear();
```



# box-sizing

​	在 [CSS 盒子模型](https://developer.mozilla.org/en-US/docs/CSS/Box_model) 的默认定义里，你对一个元素所设置的 [`width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/width) 与 [`height`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/height) 只会应用到这个元素的内容区。如果这个元素有任何的 [`border`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border) 或 [`padding`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/padding) ，**绘制到屏幕上时的盒子宽度和高度会加上设置的边框和内边距值**。这意味着当你调整一个元素的宽度和高度时需要时刻注意到这个元素的边框和内边距



box-sizing 属性可以被用来调整这些表现:

- `content-box` 是默认值。如果你设置一个元素的宽为 100px，那么这个元素的内容区会有 100px 宽，并且任何边框和内边距的宽度都会被增加到最后绘制出来的元素宽度中
- `border-box` 告诉浏览器：**你想要设置的边框和内边距的值是包含在 width 内的**。也就是说，如果你将一个元素的width设为100px，那么这100px会包含它的border和padding，内容区的实际宽度是width减去(border + padding)的值。大多数情况下，这使得我们更容易地设定一个元素的宽高



# 多余文本用 ... 显示

```css
// 多余文本显示省略号 ...
display: -webkit-box;
overflow: hidden;
-webkit-box-orient: vertical;
-webkit-line-clamp: 2; /* 这里是超出几行省略 */
```



# 固定控件的位置

```css
.markdown {
  position: fixed;   // 固定控件位置
  bottom: 5px;       // 固定的具体位置
}
```

























