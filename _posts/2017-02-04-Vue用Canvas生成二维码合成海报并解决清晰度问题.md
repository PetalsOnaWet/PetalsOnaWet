---
layout:     post
title:     Vue用Canvas生成二维码合成海报并解决清晰度问题
subtitle:   
date:       2019-05-18
author:     PetalsOnaWet
header-img: 
catalog: true
tags:
    - 前端
    - Canvas
    - 微信
---
# 前言

>最近接到一个需求，用文字和图片合成一个海报，用于活动结尾页在微信长按分享，接到需求的第一时间，我就想到用 canvas 来画，但是看到 canvas 繁琐的绘制过程，不由得感到头大，后几经搜索，果然发现已经有人造好了轮子。此篇文章主要记录下实现过程，以及遇到的问题。

# 依赖

```QRCode``` 这个依赖主要是用于移动端将 url 生成二维码，注意名字叫 qrcodejs2 别安装错

```
npm install qrcodejs2 --save 

import QRCode from "qrcodejs2"
```

```html2canvas ```  

这个依赖主要是将当前 HTML 结构以及 css 样式转换为 canvas。比自己用 api 去画方便多了  

```Canvas2Image``` [Github](https://github.com/hongru/canvas2image)  

这个依赖主要是将canvas 转换为图片，实际上，Canvas2Image.js也是基于 canvas.toDataURL 的封装，相比原生的 API 对于转为图片的功能上考虑更为具体(未压缩的包大小为 7.4KB )，适合项目使用。  

# 主要思路

将所有的海报结构都写在一个父级结构中,然后调用 html2canvas 转换为图片，创建 image,通过 css 层级和定位，将 image 置为最顶层，来实现长按分享

# 代码

html 部分  

```
 <!--html 结构,具体 css 不写了-->

 <!-- 海报 html 元素 -->
    <div
      id="posterHtml"
      :style="{backgroundImage: 'url('+posterHtmlBg+')'}"
    >
      <div class="posterHtml">
    
        <div class="posterklass">我是{{name}},邀请您:</div>
        <!-- 二维码 -->

        <div id="qrcodeImg" :if="postcode"></div>

      </div>
    </div>
    
    <!--image 即将要插入的位置，这样的结构处理，配合方法里面的 css 会自动置为最顶层-->
    
    <div id="myCanvas"></div>
    <!--提示用户的文案，其实也可以写在海报 html 结构中，通过 html2canvas 的 ignore 来忽略生成-->
    
    <span class="tip">长按保存该海报，邀请好友来测！</span> 
    
```

js 部分  

```


    
    //生成二维码
    
     createQrcode(text) {
    
      const qrcodeImgEl = document.getElementById("qrcodeImg");
      qrcodeImgEl.innerHTML = "";
      let width = document.documentElement.clientWidth;
      
      //宽度自己定义  我取的设备的宽度 *0.32
      
      width = width * 0.32;

      let qrcode = new QRCode(qrcodeImgEl, {
        width: width,
        height: width,
        colorDark: "#000000",
        colorLight: "#ffffff",
        correctLevel: QRCode.CorrectLevel.H
      });
      
      //text是 要生成的 url
      
      qrcode.makeCode(text);
      
      //生成二维码之后调用生成海报
      
      this.createPoster();
    }
    
    
    //基于 html2canvas.js 可将一个元素渲染为 canvas，只需要简单的调用 html2canvas(element[, options]) 即可,会返回一个包含有 <canvas> 元素的promise：

    //生成海报
    
    createPoster() {
    
      const vm = this;
      const domObj = document.getElementById("posterHtml");

      var width = document.documentElement.clientWidth;
      var height = document.documentElement.clientHeight;
      var canvas = document.createElement("canvas");
      
      //各项参数的意义可参考 html2canvas 文档
      
      var opts = {
       
        canvas: canvas,
        logging: true,
        width: width,
        height: height,
        useCORS: true,
        allowTaint: false,
        logging: false,
        letterRendering: true,

      };
      html2canvas(domObj, opts).then(function(canvas) {
        var context = canvas.getContext("2d");
        var img = Canvas2Image.convertToImage(
          canvas,
          canvas.width,
          canvas.height
        );
        
        img.style.width = canvas.width;

        img.style.height = canvas.height;
        img.style.position = "absolute";
        img.style.top = "0px";
 
        document.getElementById("myCanvas").appendChild(img);
        
        //插入图片后将 html 结构隐藏，避免微信图片上拉 显示图片下 html 结构
        
        vm.$nextTick(() => {
            let e =  document.querySelector("#posterHtml");
             e.style.display = "none";
         })  
    }
    
    //数据示例,随便写了 意思下
    
    data() {
      return {
        name:'', //海报文案
        posterHtmlBg: require('../../assets/images/poster/invite_poster_bg.jpg'), // 背景图
        
      }
    },
    
```

# 问题与解决方案

按理说，这时候的需求应该实现了，但是运行过后，会发现转换成的图片，清晰度感人。肯定不符合需求，所以我搜索了一番，发现以下解决方案。  

第一，背景图不能通过 background 来引用，要单独写个 image 结构  

```

  <div class="main">

    <!-- 海报 html 元素 -->
    
    <div id="posterHtml" >
      <div class="posterHtml">
        <img
          :src="posterHtmlBg"
          style="width:100%;height:100%"
        >
        <div class="posterklass">我是{{name}},邀请您:</div>
        
        <!-- 二维码 -->

        <div id="qrcodeImg" :if="postcode"></div>

      </div>
    </div>
    
    <div id="myCanvas"></div>
    <span class="tip">长按保存该海报，邀请好友来测！</span>
    
  </div>
  
```
canvas 尺寸也会影响到转换质量 上面的代码虽然设置了尺寸，但远远不够，因为浏览器绘制 Canvas 渲染到屏幕中分两个过程：

> 绘制过程：

> webkitBackingStorePixelRatio  
> webkitBackingStorePixelRatio 表示浏览器在绘制 Canvas 到缓存区时的绘制比例，若图片宽高为 200px，webkitBackingStorePixelRatio 为 2，那么 Canvas 绘制这个图片到缓存区时，宽高就变成 400px 

> 渲染过程：devicePixelRatio  

> Canvas 显示到屏幕中还需要渲染过程，渲染过程会根据 devicePixelRatio 参数将缓存区中的 Canvas 进行缩放渲染到屏幕中

> Canvas 绘制会模糊的原因就可以推测：
> 1、devicePixelRatio = device pixel / CSS pixel
> 如果 devicePixelRatio = 2 那么对于 200px * 200px 的图片要绘制到屏幕中，那么对应的屏幕像素(物理像素) 就是 400px * 400px

> 2、在大部分高清屏中，webkitBackingStorePixelRatio = 1 devicePixelRatio = 2
> 将100px * 100px 的图片绘制到屏幕中会经历以下处理：

> webkitBackingStorePixelRatio = 1
> 绘制到缓存区的大小也为：200px * 200px
> devicePixelRatio = 2

> 200px * 200px 的图片对应到屏幕像素为 400px * 400px，devicePixelRatio = 2 浏览器就把缓存区的 200px * 200px 宽高分别放大两倍渲染到屏幕中，所以就导致模糊


so 我们的解决方案就是将 canvas 宽高设置为屏幕的 2 倍

另外 canvas 会默认开启抗锯齿，我们要手动将抗锯齿关闭来实现图片的锐化

[MDN: imageSmoothingEnabled](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/imageSmoothingEnabled)

```
context.mozImageSmoothingEnabled = false;
context.webkitImageSmoothingEnabled = false;
context.msImageSmoothingEnabled = false;
context.imageSmoothingEnabled = false;

```

最终的代码如下  

html 部分  

```
  <div class="main">

    <!-- 海报 html 元素 -->
    <div id="posterHtml" >
      <div class="posterHtml">
        <img
          :src="posterHtmlBg"
          style="width:100%;height:100%"
        >
        <div class="posterklass">我是{{name}},邀请您:</div>
        
        <!-- 二维码 -->

        <div id="qrcodeImg" :if="postcode"></div>

      </div>
    </div>
    <div id="myCanvas"></div>
    <span class="tip">长按保存该海报，邀请好友来测！</span>
  </div>
  
 ```
 
 js 部分
 
 ```
 
 createQrcode: function(text) {
 
      // 生成二维码
      
      const qrcodeImgEl = document.getElementById("qrcodeImg");
      qrcodeImgEl.innerHTML = "";
      let width = document.documentElement.clientWidth;
      width = width * 0.32;

      let qrcode = new QRCode(qrcodeImgEl, {
        width: width,
        height: width,
        colorDark: "#000000",
        colorLight: "#ffffff",
        correctLevel: QRCode.CorrectLevel.H
      });
      qrcode.makeCode(text);
      this.createPoster();
    },

    createPoster() {
    
      // 生成海报
      
      const vm = this;
      const domObj = document.getElementById("posterHtml");

      var width = document.documentElement.clientWidth;
      var height = document.documentElement.clientHeight;
      var canvas = document.createElement("canvas");
      var scale = 2;

      canvas.width = width * scale;
      canvas.height = height * scale;
      canvas.getContext("2d").scale(scale, scale);

      var opts = {
        scale: scale,
        canvas: canvas,
        logging: true,
        width: width,
        height: height,
        useCORS: true,
        allowTaint: false,
        logging: false,
        letterRendering: true,

      };
      html2canvas(domObj, opts).then(function(canvas) {
        var context = canvas.getContext("2d");
        
        // 重要 关闭抗锯齿
        
        context.mozImageSmoothingEnabled = false;
        context.webkitImageSmoothingEnabled = false;
        context.msImageSmoothingEnabled = false;
        context.imageSmoothingEnabled = false;

        var img = Canvas2Image.convertToImage(
          canvas,
          canvas.width,
          canvas.height
        );
         vm.postshow = false;
         vm.postcode = false;
        img.style.width = canvas.width / 2 + "px";

        img.style.height = canvas.height / 2 + "px";
        img.style.position = "absolute";
        img.style.top = "0px";
       
        document.getElementById("myCanvas").appendChild(img);
      
        vm.$nextTick(() => {
            let e =  document.querySelector("#posterHtml");
             e.style.display = "none";
 
         })
      });
      

    }
 ```

到此，需求就完美实现了。如果你有更好的思路请留言告诉我

[参考博客1](http://objcer.com/2017/10/10/High-DPI-Canvas-Render/)
[参考文章2](https://segmentfault.com/a/1190000011478657)
