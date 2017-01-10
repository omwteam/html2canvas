**html2canvas html截图插件图片放大清晰度解决方案，支持任意放大倍数，解决原插件图片偏移问题**
----------------------------------------------------

Author:youzebin (2016.12.6)
插件下载地址：https://github.com/niklasvh/html2canvas

1.首先引入html2canvas.js html2canvas 0.5.0-beta4 最新版即可

必要步骤1：修改插件的源码： （修改的地方有两处）

#### 1. 代码第 999 行 renderWindow 的方法中 修改判断条件 增加一个options.scale存在的条件：


源码：

```
if (options.type === "view") {
                canvas = crop(renderer.canvas, {width: renderer.canvas.width, height: renderer.canvas.height, top: 0, left: 0, x: 0, y: 0});
            } else if (node === clonedWindow.document.body || node === clonedWindow.document.documentElement || options.canvas != null) {
                canvas = renderer.canvas;
            } else {
                canvas = crop(renderer.canvas, {width:  options.width != null ? options.width : bounds.width, height: options.height != null ? options.height : bounds.height, top: bounds.top, left: bounds.left, x: 0, y: 0});

            }

```
改为：

```
if (options.type === "view") {
                canvas = crop(renderer.canvas, {width: renderer.canvas.width, height: renderer.canvas.height, top: 0, left: 0, x: 0, y: 0});
            } else if (node === clonedWindow.document.body || node === clonedWindow.document.documentElement) {
                canvas = renderer.canvas;
            }else if(options.scale && options.canvas !=null){
                log("放大canvas",options.canvas);
                var scale = options.scale || 1;
                canvas = crop(renderer.canvas, {width: bounds.width * scale, height:bounds.height * scale, top: bounds.top *scale, left: bounds.left *scale, x: 0, y: 0});
            }
            else {
                canvas = crop(renderer.canvas, {width:  options.width != null ? options.width : bounds.width, height: options.height != null ? options.height : bounds.height, top: bounds.top, left: bounds.left, x: 0, y: 0});
            }
```
#### 2. 代码第 943 行 html2canvas 的方法中  修改width,height：


源码：
```
return renderDocument(node.ownerDocument, options, node.ownerDocument.defaultView.innerWidth, node.ownerDocument.defaultView.innerHeight, index).then(function(canvas) {
    if (typeof(options.onrendered) === "function") {
        log("options.onrendered is deprecated, html2canvas returns a Promise containing the canvas");
        options.onrendered(canvas);
    }
    return canvas;
});
```
改为：

```
width = options.width != null ? options.width : node.ownerDocument.defaultView.innerWidth;
height = options.height != null ? options.height : node.ownerDocument.defaultView.innerHeight;
return renderDocument(node.ownerDocument, options, width, height, index).then(function(canvas) {
    if (typeof(options.onrendered) === "function") {
        log("options.onrendered is deprecated, html2canvas returns a Promise containing the canvas");
        options.onrendered(canvas);
    }
    return canvas;
});
```



2.使用方式

```
var shareContent = document.getElementById("shareContent");//需要截图的包裹的（原生的）DOM 对象
var width = shareContent.offsetWidth; //获取dom 宽度
var height = shareContent.offsetHeight; //获取dom 高度
var canvas = document.createElement("canvas"); //创建一个canvas节点
var scale = 2; //定义任意放大倍数 支持小数
canvas.width = width * scale; //定义canvas 宽度 * 缩放
canvas.height = height * scale; //定义canvas高度 *缩放
canvas.getContext("2d").scale(scale,scale); //获取context,设置scale 
var opts = {
    scale:scale, // 添加的scale 参数
    canvas:canvas, //自定义 canvas
    logging: true, //日志开关
    width:width, //dom 原始宽度
    height:height //dom 原始高度
};

html2canvas(shareContent, opts).then(function (canvas) {
    //如果想要生成图片 引入canvas2Image.js 下载地址：
    //https://github.com/hongru/canvas2image/blob/master/canvas2image.js
    var img = Canvas2Image.convertToImage(canvas, canvas.width, canvas.height);
    console.log(img);
});
```

**2017.1.7 优化插件使用的方式，并附上demo （插件的改动还是按照上面的操作流程）**
-------------------------------------------------

（修复插件的使用方式上存在截图不完整的bug）


以下我总结了一些注意事项，在代码中注释了，仅供参考。
付：完整使用的demo ,如下：

    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport"
      content="width=device-width,initial-scale=1,minimum-scale=1,maximum-scale=1,user-scalable=no">
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">
    <title>html2Canvas demo</title>
    <script>
    document.documentElement.style.fontSize = window.screen.width / 7.5 + 'px';
    </script>
    <style>
    body,
    html,
    div,
    p,
    ul,
    li,
    a,
    img,
    span,
    button,
    header,
    footer,
    section {
    padding: 0;
    margin: 0;
    }
    
    *, :before, :after {
    -webkit-tap-highlight-color: transparent;
    -webkit-user-select: none;
    outline: none;
    box-sizing: border-box;
    -webkit-box-sizing: border-box;
    }
    
    ::-webkit-scrollbar {
    width: 0;
    opacity: 0;
    }
    
    button{
    font-family: simsun,"microsoft yahei", arial, "Helvetica Neue", Helvetica, STHeiTi, sans-serif;
    }
    body {
    font-family: "microsoft yahei", arial, "Helvetica Neue", Helvetica, STHeiTi, sans-serif;
    color: #000;
    background-color: #f5f5f5;
    -webkit-overflow-scrolling: touch;
    }
    .share-container {
    padding-top: 0.72rem;
    width: 2.35rem;
    margin: 0 auto;
    }
    
    .share-content {
    padding-top: 0.72rem;
    height:3rem;
    background-color: blue;
    border-radius: 5px;
    width: 100%;
    }
    .text{
    font-size: 0.36rem;
    color: #f2f2f2;
    }
    .btn-share {
    width: 64%;
    height: 0.89rem;
    background-color: #3baaff;
    border-radius: 0.89rem;
    border: 1px solid #3baaff;
    color: white;
    font-size: 0.36rem;
    margin: 0.75rem 0 0.67rem;
    }
    .btn-share:active{
    background-color: #1b96c8;
    }
    </style>
    </head>
    <body>
    <section class="main-container">
    <header class="share-container" id="shareContainer">
    <div class="share-content" id="shareContent">
      <div class="text">
      <p>文字，图片等内容</p>
      </div>
    </div>
    </header>
    <footer class="footer-center">
    <button class="btn-share" id="btnShare">截&nbsp;图</button>
    </footer>
    </section>
    
    <script src="static/js/html2canvas.js"></script>
    <script>
    
    //定义查找元素方法
    function $(selector) {
    return document.querySelector(selector);
    }
    var main = {
    init:function(){
    main.setListener();
    },
    //设置监听事件
    setListener:function(){
    var btnShare = document.getElementById("btnShare");
    btnShare.onclick = function(){
    main.html2Canvas();
    }
    },
    //获取像素密度
    getPixelRatio:function(context){
    var backingStore = context.backingStorePixelRatio ||
    context.webkitBackingStorePixelRatio ||
    context.mozBackingStorePixelRatio ||
    context.msBackingStorePixelRatio ||
    context.oBackingStorePixelRatio ||
    context.backingStorePixelRatio || 1;
    return (window.devicePixelRatio || 1) / backingStore;
    },
    //绘制dom 元素，生成截图canvas
    html2Canvas: function () {
    var shareContent = $("#shareContent");// 需要绘制的部分的 (原生）dom 对象 ，注意容器的宽度不要使用百分比，使用固定宽度，避免缩放问题
    var width = shareContent.offsetWidth;  // 获取(原生）dom 宽度
    var height = shareContent.offsetHeight; // 获取(原生）dom 高
    var offsetTop = shareContent.offsetTop;  //元素距离顶部的偏移量
    
    var canvas = document.createElement('canvas');  //创建canvas 对象
    var context = canvas.getContext('2d');
    var scaleBy = main.getPixelRatio(context);  //获取像素密度的方法 (也可以采用自定义缩放比例)
    canvas.width = width * scaleBy;   //这里 由于绘制的dom 为固定宽度，居中，所以没有偏移
    canvas.height = (height + offsetTop) * scaleBy;  // 注意高度问题，由于顶部有个距离所以要加上顶部的距离，解决图像高度偏移问题
    context.scale(scaleBy, scaleBy);
    
    var opts = {
    allowTaint:true,//允许加载跨域的图片
    tainttest:true, //检测每张图片都已经加载完成
    scale:scaleBy, // 添加的scale 参数
    canvas:canvas, //自定义 canvas
    logging: true, //日志开关，发布的时候记得改成false
    width:width, //dom 原始宽度
    height:height //dom 原始高度
    };
    html2canvas(shareContent, opts).then(function (canvas) {
       console.log("html2canvas");
    var body = document.getElementsByTagName("body");
    body[0].appendChild(canvas);
    });
    }
    };
    
    //最后运行代码
    main.init();
    
    </script>
    </body>
    </html>

## 运行上面的demo 前有以下 **注意点**：

1. 前面的内容没看过,没下载过html2canvas.js 没按照插件改过说明操作的先改好再说
2. 注意元素的样式的使用：
   外层元素width 不能使用百分比 ,避免导致图片与文字间缩放比例问题
    错误使用方式如  
     .container {
           width:50%;
           margin: 0 auto;
     } 

 需要改成如：

     .container {
           width:300px;
           margin: 0 auto;
     } 

有疑问请在下方留言 或加我qq:302829055 谢谢