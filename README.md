简书上写了更详细的的文档
https://www.jianshu.com/p/45df1dc37478

项目环境

node 10.15.3

yarn  1.15.2

win10

代码完成时间2019-4-18

废话不多说，先放源码

GitHub

[https://github.com/951477037/electron-print](https://github.com/951477037/electron-print)

```
git clone https://github.com/951477037/electron-print.git
```

```
//安装依赖
yarn
```

```
//运行项目
yarn run dev
```

```
//打包项目
yarn run build
```

目录结构
![目录结构](http://upload-images.jianshu.io/upload_images/15562516-2df5d50d68200ae6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先在主进程 /src/main/index.js

```
//引入ipcMain
import {
  app,
  BrowserWindow,
  ipcMain
} from 'electron'
```

![index.js代码](http://upload-images.jianshu.io/upload_images/15562516-94f9a652f47fd4b3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在createWindow方法里添加以下代码，获取打印机列表

```
  //在主线程下，通过ipcMain对象监听渲染线程传过来的getPrinterList事件
  ipcMain.on('getPrinterList', (event) => {
    //在主线程中获取打印机列表
    const list = mainWindow.webContents.getPrinters();
    //通过webContents发送事件到渲染线程，同时将打印机列表也传过去
    mainWindow.webContents.send('getPrinterList', list);
  });
```

![index.js](http://upload-images.jianshu.io/upload_images/15562516-488a2a3db1183179.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来在LandingPage.vue中也就是渲染进程中添加一下代码

```
const ipcRenderer = require("electron").ipcRenderer;
```

```
//使用ipcRenderer与主进程通信，并获取返回值
ipcRenderer.send("getPrinterList");
//监听主线程获取到打印机列表后的回调
ipcRenderer.once("getPrinterList", (event, data) => {
//data就是打印机列表
console.log(data);
});
```

![LandingPage.vue](http://upload-images.jianshu.io/upload_images/15562516-ab53ac235e403bfc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
输出结果如下
![image](http://upload-images.jianshu.io/upload_images/15562516-032b9165b71ec638.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重点来了！！！
在static中新建一个print.html文件（如果你害怕打包后会找不到的话，我在最后会提供一个方法不知道你看得仔不仔细），如下图所示
![目录结构](http://upload-images.jianshu.io/upload_images/15562516-8ae3b2ba49ae60a9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果不在static中新建的话会报错（具体原因我明没有深入去研究）
![报错](http://upload-images.jianshu.io/upload_images/15562516-4de40faf2737dc9d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <script src="https://cdn.bootcss.com/vue/2.2.2/vue.min.js"></script>
    <title>Document</title>
    <style>
        @page {
            margin: 0;
        }
        .a {
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
    //引入ipcRenderer对象
    const {
        ipcRenderer
    } = require('electron')
    new Vue({
        el: "#app",
        data: {
            arr: []
        },
        mounted() {
            ipcRenderer.on('ping', (e, arr) => { //接收响应
                console.log(e)
                console.log(arr)
                this.arr = arr;
                ipcRenderer.sendToHost('pong') //向webview所在页面的进程传达消息
            })
        },
        methods: {}
    })
</script>

</html>

```

创建完成，回到LandingPage.vue中添加以下代码
注意两个参数

```
silent  是否静默打印
deviceName  打印机名字

```

把deviceName换成你自己的打印机名字

```
<template>
  <div>
    <webview src="../../../static/print.html" nodeintegration></webview>
  </div>
</template>

<script>
const ipcRenderer = require("electron").ipcRenderer;
export default {
  name: "landing-page",
  components: {},
  data() {
    return {
      print0: "",
      print1: ""
    };
  },
  mounted() {
    this.getPrinterList(); //首先先获取
    this. print();
  },
  methods: {
    print() {
      const webview = document.querySelector("webview");
      console.log(webview);
      webview.addEventListener("dom-ready", () => {
        console.log("dom-ready");
        //dom-ready---webview加载完成
        webview.openDevTools(); //这个方法可以打开print.html的控制台
        var arr = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 11, 12, 13, 14];
        //在send时将arr传递过去
        webview.send("ping", arr); //向webview嵌套的页面响应事件
      });
      webview.addEventListener("ipc-message", event => {
        console.log(event.channel); // Prints "pong" 在此监听事件中接收webview嵌套页面所响应的事件
        if (event.channel == "pong") {
          console.log("通信成功");
          webview.print(
            {
              //是否是静默打印,true为静默打印，false会弹出打印设置框
              silent: true,
              printBackground: true,
              //打印机的名称，this.print1为在getPrinterList()方法中获取到的打印机名字。
              //注意在demo中这是我打印机的设备，在使用本demo时，先去getPrinterList()中找到你使用的打印机
              deviceName: this.print1
            },
            data => {
              //这个回调是打印后的回调事件，data为true就是打印成功，为false就打印失败
              console.log("webview success", data);
            }
          );
        }
      });
    },
    getPrinterList() {
      ipcRenderer.send("getPrinterList");
      //监听主线程获取到打印机列表后的回调
      ipcRenderer.once("getPrinterList", (event, data) => {
        //data就是打印机列表
        this.print0 = data[3].name;
        this.print1 = data[5].name;
        console.log(data[3].name);
        console.log(data[5].name);
      });
    }
  }
};
</script>

<style>
</style>


```

运行代码

![打印](http://upload-images.jianshu.io/upload_images/15562516-5fffc25bfa27c616.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


打包的方法！！！
打包前在package.json中修改

```
"win": {

      "icon": "build/icons/icon.ico",

      "extraResources": "./static/*.html"

    },

```

打包后，electron-print\build\win-ia32-unpacked\resources中就会存在static
![打包后目录](http://upload-images.jianshu.io/upload_images/15562516-00af6067ba65f9dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

static中

![打包后目录](http://upload-images.jianshu.io/upload_images/15562516-bd25464efc41abb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果觉得有用请点个赞，转发请注明来源，谢谢
