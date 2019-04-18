~~~js
```js

```

//创建electron-vue项目

vue init simulatedgreg/electron-vue electron-print-demo

yarn

yarn run dev

~~~

目录结构

 ![](http://1.z9ls.com/t6/701/1555557576x2890149512.png)

 



 

现在主进程 /src/main/index.js

![](http://1.z9ls.com/t6/701/1555557607x2890149512.png)







~~~js
*import* {

app,

BrowserWindow,

ipcMain

} *from* 'electron'
~~~



引入ipcMain

 

然后

![](http://1.z9ls.com/t6/701/1555557617x2890149512.png)

 

在这个位置插入一下代码

~~~js
*//**在主线程下，通过**ipcMain**对象监听渲染线程传过来的**getPrinterList**事件*

ipcMain.on('getPrinterList', (event) => {

*//**在主线程中获取打印机列表*

const list = mainWindow.webContents.getPrinters();

*//**通过**webContents**发送事件到渲染线程，同时将打印机列表也传过去*

mainWindow.webContents.send('getPrinterList', list);

}); 
~~~

接下来在LandingPage.vue中也就是渲染进程中添加代码

![](http://1.z9ls.com/t6/701/1555557627x2890149512.png)

先引入

~~~js
const ipcRenderer = require("electron").ipcRenderer;

使用ipcRenderer与主进程通信，并获取返回值

ipcRenderer.send("getPrinterList");

*//**监听主线程获取到打印机列表后的回调*

ipcRenderer.once("getPrinterList", (event, data) => {

*//data**就是打印机列表*

console.log(data);

});
~~~

![](http://1.z9ls.com/t6/701/1555558322x2918527158.png)

这是我这里获取到的所有打印设备

 

重点来了

![](http://1.z9ls.com/t6/701/1555558166x2918527158.png)

如果在components中创建的话，会报一个错

![](http://1.z9ls.com/t6/701/1555558152x2918527158.png)

索性直接方法static中，如果你害怕打包后会找不到的话，我在最后会提供一个方法

 

新建一个print.html文件

~~~html
<!DOCTYPE *html*>

<html *lang*="en">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1.0">

<meta http-equiv="X-UA-Compatible" content="ie=edge">

<script src="https://cdn.bootcss.com/vue/2.2.2/vue.min.js"></script>

<title>Document</title>

<style>

*@page* {

margin: 0;

}

*.**a* {

padding-left: 100px;

}

</style>

</head>

<body>

<div id='app'>

<div class="a" v-for="v in arr">{{v}}</div>

</div>

</body>

<script>

*//**引入**ipcRenderer**对象*

const {

ipcRenderer

} = require('electron')

new Vue({

el: "#app",

data: {

arr: []

},

mounted() {

ipcRenderer.on('ping', (e, arr) => { *//**接收响应*

console.log(e)

console.log(arr)

*this*.arr = arr;

ipcRenderer.sendToHost('pong') *//**向**webview**所在页面的进程传达消息*

})

},

methods: {}

})

</script>

</html>
~~~



这里我使用vue的网络地址js,使用基础的js编写也可以

创建完成，回到LandingPage.vue中添加以下代码

注意两个参数

silent  是否静默打印

deviceName  打印机名字

~~~html
<webview *ref*="printWebview" *src*="../../../static/print.html" *nodeintegration*></webview>
~~~

再编写一个print方法

~~~js
send() {

const webview = document.querySelector("webview");

webview.addEventListener("dom-ready", () => {

*//dom-ready---webview**加载完成*

webview.openDevTools(); *//**这个方法可以打开**print.html**的控制台*

var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 11, 12,13,14];

*//**在**send**时将**arr**传递过去*

webview.send("ping", arr); *//**向**webview**嵌套的页面响应事件*

});

webview.addEventListener("ipc-message", event => {

console.log(event.channel); *// Prints "pong"* *在此监听事件中接收**webview**嵌套页面所响应的事件*

*if* (event.channel == "pong") {

console.log("通信成功");

webview.print(

{

*//**是否是静默打印**,true**为静默打印，**false**会弹出打印设置框*

silent: true,

printBackground: true,

*//**打印机的名称，**this.print1**为在**getPrinterList()**方法中获取到的打印机名字。*

*//**注意在**demo**中这是我打印机的设备，在使用本**demo**时，先去**getPrinterList()**中找到你使用的打印机*

deviceName: *this*.print1

},

data => {

*//**这个回调是打印后的回调事件，**data**为**true**就是打印成功，为**false**就打印失败*

console.log("webview success", data);

}

);

}

});

}
~~~

![](http://1.z9ls.com/t6/701/1555558189x2918527158.png)

最后在package.json中修改

~~~json
"win": {

​      "icon": "build/icons/icon.ico",

​      "extraResources": "./static/*.html"

​    },
~~~



打包后，electron-print\build\win-ia32-unpacked\resources中就会存在static

![](http://1.z9ls.com/t6/701/1555558201x2918527158.png)

static中

![](http://1.z9ls.com/t6/701/1555558215x2918527158.png)

 