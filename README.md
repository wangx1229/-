# 近期项目总结

hook篇

第一步升级项目react， 15.4 -> 16.8 没有别的目的，就是为了再熟悉一下react hook...

hook和之前的写法对比，一个是函数，一个是类。hook使得函数组件具有生命周期和状态，这很方便复用状态逻辑以及逻辑拆分，每个hook专注于一件事情！虽然缺少很小一部分通常难以用到的生命周期，但是hook足以满足大部分需求。

hook中不需要使用this进行绑定，所有方法前也都不在需要this，所以看起来更简洁一些

hook中使用useState来初始化state，修改state时也不是合并，而是重写。当然可以传递一个函数来进行过合并。

hook可以减少一些重复的逻辑代码。比如官方例子中的订阅id这个事件。 我们根据组件接受的id去订阅一个事件。这段订阅代码，我们需要写在componentDidMount以及componentUpdate两个生命周期中，同时还需要判断组件更新时id是否发生改变。我们还需要去取消订阅，放在componentwillunmount中。而hook，只需要放在一个依赖于id的effect中就可以完整上面的订阅以及取消订阅段落及，可以看到代码少了多少。

![compare1](https://github.com/wangx1229/hook-canvas-uploadImage/blob/master/imgs/compare1.png)

hook不仅仅是为了减少代码，更多时候，会使用自定义hook去封装一段状态逻辑，甚至可以多个地方使用。每一个封装的状态逻辑，业务逻辑，只负责这一件事情，别的事情，交给别的hook，这样会有更强的解耦性。和封装组件一样，可以减少代码，是代码思路看起来更加简洁清晰

hook改变了思考方式，原来是命令式的，改变某个值，做某件事情。hook更多时候则是依赖，我们在一开始写入这个hook，一旦依赖发生变化，接下里的事情会自己在hook中进行处理，这样我们就完全不用关心里面具体做了什么，有点类似于函数式编程，只是我们hook中不是纯函数，会有副作用。

canvas篇

卡住时间最久的一块，原因是对canvas这块内容处理的比较少，没有经验。

1.canvas中引入图片后，再把canvas导出成一张图片，出现跨域问题

点击查看[MDN对canvas跨域的解释](https://developer.mozilla.org/zh-CN/docs/Web/HTML/CORS_enabled_image)

根据MDN对canvas的介绍，如果在canvas绘图的时候，图片的地址如果来自于其他域名，会出现安全问题，虽然你依然可以看到canvas中的图片，但这已经是一个被污染的canvas，在使用toDataURL()会出现跨域问题而抛错。

解决办法是选择的是在我们cdn服务器添加我们的域名，同时依次设置img.crossOrigin = 'Anonymous'和img.src = src，这两个设置顺序非常重要。除了调用toDataURL外，调用getImageData和toBlob也会抛错。

2.canvas在引入图片后，在导出发现canvas中一些内容缺失
这个问题如果之前有经验的话，很容易发现是因为图片的onload事件是一个异步事件，只需要把后续代码移入到onload 事件就可以解决。 但是一层层嵌套代码...反正我是忍不了。

封装loadImg方法，返回promise，愉快的使用async函数。

![load](https://github.com/wangx1229/hook-canvas-uploadImage/blob/master/imgs/load.png)

3.canvas转成图片会变模糊，canvas转成图片以后体积太大导致无法上传，以及安卓长按图片保存时偶然崩溃

解决图片模糊的办法是canvas通过属性设置的宽高为通过样式设置宽高的两倍

在安卓中长按崩溃很少出现，安卓开发给出的原因是图片加载时间过长。但是压缩图片质量，或者在图片加载后在打开，均未解决该问题，最后这个bug没有解决，选择了使用jsbridge处理。

4.canvas根据图片大小，进行图片裁剪，并展示图片当中的内容

第一版没有对上传图片做限制，所以只能对上传的图片进行裁剪后展示，否则图片会发生变形。html中，img这个dom元素有object-fit属性可以设置，但是canvas是完完全全的位图，img没有这个属性，只能手动裁剪。

在canvas的图片加载过之后，对比图片的宽高，以小的边为基准进行裁剪。举个例子，当图片宽比高长时，以高为基准，根据需要展示的内容的宽高比，计算截图的图片宽度，然后在依据截取图片的宽高去计算截图片的坐标位置。

5.canvas文字中遇到的问题，一个是在一段文字后又写一段文字时，如果前一段文字发生变化，两端文字可能会重叠或者距离增加， 另一个是文字换行

相同的起始坐标，对文字的定位和对其他内容的定位，发现位置竟然不一样。原因就在于[textBaseline](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/textBaseline)属性。而他又是基于[文本基线](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Drawing_text)的。

[ctx.measureText（）](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/measureText) 他会返回一个[TextMetrics](https://developer.mozilla.org/zh-CN/docs/Web/API/TextMetrics)对象，我们可以使用它的width属性来计算内容的宽度。但是有一点需要注意，就是他会根据当前ctx.font的大小来计算。此外我们还可以通过fontBoundingBoxAscent，fontBoundingBoxDescent两个属性来近似的计算文本的高度。

这样以来，我们就可以获得文本的高度和宽度，再依据第一段文字的x，y初始位置就可以动态的计算第二段文字的初始位置

现在返回我们之前提到的第一个问题，当我们第一段文字发生癌变时，我们就可以计算出第一段啊文字

canvas在fillText时是不会换行的，设置好宽度之后，内容过多，会发现字体重叠。这当然是不满足我们需求的，需要做出改进

初步解决思路。将内容的每个字符串分割组成一个数组，然后去遍历这个数组，每次使用累计值和当前值进行拼接，然后去计算文本的宽度，当大于我们指定的最大宽度时，我们使用累计值进行本行的绘制，然后另起一行，并且将累积值设置为当前值。

![textwrap](https://github.com/wangx1229/hook-canvas-uploadImage/blob/master/imgs/wrapText.png)

上传图片篇

1.手机拍照时，图片发生旋转

处理旋转问题，选择解决方法是使用FileReader读取图片信息，将图片放到canvas中旋转以后再使用canvas导出图片。需要exif.js这个文件库去读取图片的orientation属性。

2.图片上传时，特别是拍照时，图片过大，导致请求失败，请求超时

当canvas转成base64时，图片大小会发生变化。为了压缩图片质量，第一步可以通过设置你自身canvas的大小来控制导出图片的质量，第二步设置toDataURL的参数，来控制图片的质量，第三步就是我们自己在前端计算图片的大小，不允许上传太大的图片。因为有计算大小，所以就会涉及到base64的原理以及位(bit)和字节(byte)之间的转换。

3.图片裁剪，选择react-cropper插件进行裁剪。

项目中涉及到的其他点

canvas内容防止模糊自适应的原理

promise的原理

base64原理

函数式编程的理解

关于组件的认识

&& 和 || 比较符

使用扩展运算符去改变state

使用try catch捕获错误

