# 近期项目总结

canvas篇

卡住时间最久的一块，原因是对canvas这块内容处理的比较少，没有经验。

1.canvas中引入图片后，再把canvas导出成一张图片，出现跨域问题

点击查看[MDN对canvas跨域的解释](https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_enabled_image)

根据MDN对canvas的介绍，如果在canvas绘图的时候，图片的地址如果来自于其他域名，会出现安全问题，虽然你依然可以看到canvas中的图片，但这已经是一个被污染的canvas，在使用toDataURL()会出现跨域问题而抛错。

解决办法是选择的是在我们cdn服务器添加我们的域名，同时依次设置img.crossOrigin = 'Anonymous'和img.src = src，这两个设置顺序非常重要。除了调用toDataURL外，调用getImageData和toBlob也会抛错。

2.canvas在引入图片后，在导出发现canvas中一些内容缺失
这个问题如果之前有经验的话，很容易发现是因为图片的onload事件是一个异步事件，只需要把后续代码移入到onloa 事件就可以解决。 是f's

3.canvas文字无法换行

4.canvas转成图片会变模糊，canvas转成图片以后体积太大导致无法上传

5.canvas根据图片大小，进行图片裁剪展示大小

6.在一段文字后又写一段文字时，如果前一段文字发生变化，两端文字
