+++
date = "2017-07-09T10:11:21+08:00"
title = "像雨果一样写作 2"
showonlyimage = false
image = "/img/blog/blog-using-hugo-2/canvas.png"
draft = false
weight = 21
tags = [ "Blogging", "Hugo", "Image", "Video" ]
categories = [ "Writing" ]
series = [ "Blog like a Pro" ]
+++

最要紧的是准备和插入那些漂亮的图片。
<!--more-->

## 插入图片

人是视觉动物。

“有图有真相”——比如新闻网页中一般都含有相关照片或视频截图。在网站发展初期，只要是不涉及国家领导人的新闻，其网页的边栏、底栏都会有一些清凉图片(有时配合耸人听闻的标题打包显示)，感觉多数新闻聚合类网站都会这招，这些小视频和缩略图对网站点击量也该有不小的贡献。

“一图胜千言”——图片在解释某些复杂事物的时候优势非常明显。

<details>
  <summary>HipHop 握手</summary>
  <a href="https://zh.wikipedia.org/wiki/%E8%B7%AF%E4%BA%BA%E8%B6%85%E8%83%BD100"><img alt="how hiphop shake hands"  src="/img/blog/blog-using-hugo-2/shake-hand.gif"
  class="img-responsive" ></a>
</details>

Markdown 中直接使用 html 插入动图片:
{{< highlight html "style=autumn,linenos=inline">}}
<a href="https://zh.wikipedia.org/wiki/路人超能100">
  <img src="gifname" class="img-responsive" >
</a>
{{< /highlight >}}
进阶教程：[方法1](http://www.ebadf.net/2016/10/19/centering-images-in-hugo/),
[方法2](https://www.adamwills.io/blog/responsive-images-hugo/)

### 生成 GIF 动图

1. 动图的原始视频来源是微博，先提交URL给 [weibomiaopai](https://weibomiaopai.com/download-video-parser.php)，将mp4下载到本地  
2. 在将文件上传至 [ezgif](https://ezgif.com/)，按照网页操作提示就生成最终 GIF 文件
3. 图像处理的另外一个大杀器是 GIMP，比如将 vid 转化为 gif，就可以通过安装插件完全在本地完成
    <details>
      <summary>油管上 GIMP GAP 教学视频</summary>
        {{< youtube -sZYGwfcJ3A >}}
    </details>

## 插入视频

现状就是大多数国内的网站都不支持原生的 html5 中的 video  ，大部分都是还都是用 flash。

- 油管直接用 Hugo 的 shortcodes ;
- 国内视频参见文末的参考文档，我仅仅测试了一个腾讯视频，其他各位使用后还是需要测试一下；

<details>
  <summary>举个腾讯视频的例子</summary>
  <div class="embed-responsive embed-responsive-4by3">
    <iframe class="embed-responsive-item"
    frameborder="0" width="640" height="498" src="https://v.qq.com/iframe/player.html?vid=k0353qun5ey&tiny=0&auto=0" allowfullscreen></iframe>
  </div>
</details>

{{< highlight html "style=autumn,linenos=inline">}}
<div class="embed-responsive embed-responsive-4by3">
  <iframe
    class="embed-responsive-item"
    frameborder="0"
    width="640" height="498"
    src="https://v.qq.com/iframe/player.html?vid=k0353qun5ey&tiny=0&auto=0"
    allowfullscreen>
  </iframe>
</div>
{{< /highlight >}}

## 版式校对

无论 Hugo 、 Jekyll 或其他任何静态站点工具无一例外都提供 serve 命令来在本机启动一个轻量级的 Web 服务器，这样你就能在提交前，很方便地预览一下最终展示的版式效果。当你对内容、配置，主题等做微调的时候，被修改的内容也是即刻在本地服务器中生效( 不用重启 )，非常方便。Huge 的内容都在 content 子目录，Jekyll 则设定了草稿和正式发布两个目录。

值得一提我才知道通过 Chrome DevTools 可以模拟各种不同尺寸、不同分辨率的移动设备( iPhone X, Nexus Y, iPad and Laptop Z )。你也可以改写 GPS 位置和加速器的参数，甚至模拟不同网络条件( wifi，2G，3G，4G，offline )以测试终端响应。[文档](https://developers.google.com/web/tools/chrome-devtools/device-mode/)

缩略语解释
> - GIMP：Adobe PhotoShop 的开源替代物

参考文档

> - 本系列的所有封面图都来自[AWWWARDS TEAM](https://www.awwwards.com/freebie-realistic-vector-pack-of-desktop-elements.html)
> - [Behance](https://www.behance.net) Adobe旗下的设计师交流网站
> - [Dribbble](https://dribbble.com/) 平面和产品界面设计稿展示网站
> - Harttle. [2017-2-12] [国内主要视频网站的嵌入方式](http://harttle.com/2017/02/12/embeding-video-sites.html)
