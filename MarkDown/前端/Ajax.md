# 一、简介

Ajax 可以在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容

> 简单使用

```html
<body>
	<button>发送请求</button>
</body>
<script>
    window.onload = function (ev) {
        var oBtn = document.querySelector("button");
        oBtn.onclick = function () {
            // 1.创建一个异步对象
            var xmlhttp = new XMLHttpRequest();
            // 2.设置请求方式和请求地址
            /*
            	参数顺序：
	                method：请求的类型；GET 或 POST
    	            url：文件在服务器上的位置
        	        async：true（异步）或 false（同步），永远为 true
                */
            xmlhttp.open("GET", "04-ajax-get.php", true);
            // 3.发送请求
            xmlhttp.send();
            // 4.监听状态的变化
            xmlhttp.onreadystatechange = function (ev2) {
                /*
                    0: 请求未初始化
                    1: 服务器连接已建立
                    2: 请求已接收
                    3: 请求处理中
                    4: 请求已完成，且响应已就绪
                    */
                if(xmlhttp.readyState === 4){
                    // 判断是否请求成功
                    if(xmlhttp.status >= 200 && xmlhttp.status < 300 ||
                       xmlhttp.status === 304){
                        // 5.处理返回的结果
                        console.log("接收到服务器返回的数据");
                    }else{
                        console.log("没有接收到服务器返回的数据");
                    }

                }
            }
        }
    }
</script>
```







































































































































