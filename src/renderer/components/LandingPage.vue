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
      console.log(111);
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
