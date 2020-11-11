# 如何实现跨域

## jsonp

1. 最常见的一种跨域方式，其背后原理就是
    1. 利用了script标签不受同源策略的限制，在页面中动态插入了script
    1. script标签的src属性就是后端api接口的地址
    1. 并且以get的方式将前端回调处理函数名称告诉后端，后端在响应请求时会将回调返还，并且将数据以参数的形式传递回去。
    1. 实例代码
        1. 前端
            ```
            var script = document.createElement('script');
            script.src = 'http://127.0.0.1:2333/jsonpHandler?callback=_callback'
            document.body.appendChild(script);      //插入script标签
            //回调处理函数 _callback
            var _callback = function(obj){
                for(key in obj){
                    console.log('key: ' + key +' value: ' + obj[key]);
                }
            }
            ```
        1. 后端
            ```
            app.get('/jsonpHandler', (req,res) => {
                let callback = req.query.callback;
                let obj = {
                    type : 'jsonp',
                    name : 'weapon-x'
                };
                res.writeHead(200, {"Content-Type": "text/javascript"});
                res.end(callback + '(' + JSON.stringify(obj) + ')');
            })
            ```

## CORS

1. Cross-Origin Resource Sharing(跨域资源共享)是一种允许当前域（origin）的资源（比如html/js/web service）被其他域（origin）的脚本请求访问的机制。
    1.  当使用XMLHttpRequest发送请求时，浏览器如果发现违反了同源策略就会自动加上一个请求头:origin。
    1. 后端在接受到请求后确定响应后会在Response Headers中加入一个属性:Access-Control-Allow-Origin,值就是发起请求的源地址(http://127.0.0.1:8888)。
    1. 浏览器得到响应会进行判断Access-Control-Allow-Origin的值是否和当前的地址相同，只有匹配成功后才进行响应处理。
    1. '*'允许任何域。
    1. 实例代码
        1. 前端
            ```
            var xhr = new XMLHttpRequest();
            xhr.onload = function(data){
                var _data = JSON.parse(data.target.responseText)
                for(key in _data){
                    console.log('key: ' + key +' value: ' + _data[key]);
                }
            };
            xhr.open('POST','http://127.0.0.1:2333/cors',true);
            xhr.setRequestHeader('Content-Type','application/x-www-form-urlencoded');
            xhr.send();
            ```
        1. 后端
            ```
            app.post('/cors',(req,res) => {
                if(req.headers.origin){
                    res.writeHead(200,{
                        "Content-Type": "text/html; charset=UTF-8",
                        "Access-Control-Allow-Origin":'http://127.0.0.1:8888'
                    });
                    let people = {
                        type : 'cors',
                        name : 'weapon-x'
                    }
                    res.end(JSON.stringify(people));
                }
            })
            ```

## 服务器跨域

1. 在前后端分离的项目中可以借助服务器实现跨域，具体做法是：
    1. 前端向本地服务器发送请求
    1. 本地服务器代替前端再向api服务器接口发送请求进行服务器间通信
    1 .本地服务器其实就是个中转站的角色，再将响应的数据返回给前端

## postmessage跨域

1. 在HTML5中新增了postMessage方法，postMessage可以实现跨文档消息传输（Cross Document Messaging）。
1. 该方法可以通过绑定window的message事件来监听发送跨文档消息传输内容。
1. 使用postMessage实现跨域的话原理就类似于jsonp，动态插入iframe标签，再从iframe里面拿回数据。
1. 语法
    ```
    originWindow.postMessage(message, targetOrigin);
    // originWindow：要发起请求的窗口引用 
    // message: 要发送出去的消息，可以是字符串或者对象 
    // targetOrigin: 指定哪些窗口能够收到消息，其值可以是字符串*（表示无限制）或者一个确切的URI。
    // 只有目标窗口的协议、主机地址或端口这三者全部匹配才会发送消息。
    ```

## 修改document.domain跨子域

1. 前提条件：这两个域名必须属于同一个基础域名!而且所用的协议，端口都要一致，否则无法利用document.domain进行跨域，所以只能跨子域。
1. ​ 在根域范围内，允许把domain属性的值设置为它的上一级域。例如，在”aaa.xxx.com”域内，可以把domain设置为 “xxx.com” 但不能设置为 “xxx.org” 或者”com”。
1. 例子
    1. 现在存在两个域名aaa.xxx.com和bbb.xxx.com。在aaa下嵌入bbb的页面，由于其document.name不一致，无法在aaa下操作bbb的js。
    1. 可以在aaa和bbb下通过js将document.name = 'xxx.com';设置一致，来达到互相访问的作用。