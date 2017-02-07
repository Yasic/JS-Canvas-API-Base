# JS-Canvas-API-Base

## ASCII图

我们在终端执行各种命令的时候经常会看到一些终端里显示出来的"图片"，远看仿佛一张图，近看则是一个个的 ASCII码，它们
大致长这样子

![](https://diycode.b0.upaiyun.com/photo/2017/e41cbf778644445c6ec8bd946d2a4b69.jpg)

而今天我们要做的则是用JS把一张给定的图片转换成这种用ASCII字符组成的“ASCII图”
先看看最终效果，假设我们给定的图片是这样子的，

![](https://yasicyu.com/static/Images/yasiclogo_3.jpg)

这是代码处理后的结果，用了 __I'mYasic__ 这8个字符来表示,还是可以分辨出大致的轮廓的。

![](https://diycode.b0.upaiyun.com/photo/2017/f4d9cc9ce30a2fbf6d95ee27651f63a2.jpg)

## 单级图

而另一种图则是单极图，也就是黑白图片，还是刚刚那张图片，输出如下

![](https://diycode.b0.upaiyun.com/photo/2017/64ddc3e77a3eeb180771a8478fd31530.png)

## 基础知识

这两种图都是比较简单的，只需要以下知识即可

* HTML5中的Canvas
* 像素的RGB值
* JS中的Canvas相关API

# 制作ASCII图

一般来说，在计算机当中，我们看到的大多数图片都是由一个个像素点构成的，每一个像素点则由 RGBA 构成，在 css 中我们时常用的  __rgba(255, 255, 255, 255, 0)__就是一组RGBA值， 也即是RGB三原色和Alpha透明度。当然一张图片不是仅仅包含所有像素点数据的，还包括一些描述信息，也称为图片的 profile，这一部分小则几KB,多则几百KB，是图片压缩中经常被处理的部分。

那么对于图片中每一个像素点来说，只要我们改变了其相应的RGBA的值，最终的图片也就变了样子。而修改哪些像素点、修改成什么样的RGBA，则决定着最终的图片风格，这也是许多滤镜采用的机制。

基于以上的理论知识，我们的ASCII图制作思路也就有了。ASCII图其实就是将一张图中的一个像素点，通过计算其RGBA的值，划分成给定的几个量化值，在这里由于我们用的 __I'mYasic__ 这8个字符来表示，所以要分成8组值，每一组用一个ASCII字符来表示，最终就能组成一幅完整的ASCII图片。接下来就是具体的代码实现。

## 获取图片的像素信息

通过 Canvas API 中的 __getImageData()__ 方法我们可以获得一个对象，这个对象的属性里包含一个一维数组 data，这个一维数组每4个元素为一组，代表了一个 canvas 中指定范围的全部像素信息，并且依次是 RED，GREEN，BLUE，ALPHA。因此我们可以先把图片放进 canvas 中，再调用这个方法拿到像素。

不过我很疑惑为什么 data 是一个一维数组，通常处理的图片都是二维图片，如果用二维数组来表示像素信息，代码读取和处理会方便很多，也更容易理解。甚至可以用一个三维数组，专门用一个维度来放置RGBA信息。

获取图片像素信息的代码如下所示

```JavaScript
var canvasContext = canvas.getContext("2d");
canvasContext.drawImage(sourceImg, 0, 0);
var imgData = canvasContext.getImageData(0 , 0, sourceImg.width, sourceImg.height);
var imgDataArray = imgData.data;
```

那么对于某一个像素点的RGBA值就可以这样获取

```JavaScript
var r = imgDataArray[lineIndex];
var g = imgDataArray[lineIndex + 1];
var b = imgDataArray[lineIndex + 2];
var a = imgDataArray[lineIndex + 3];
```

其中 lineIndex 是遍历每一个像素点的基准变量。

## 图片灰度化

灰度化，也就是获取像素点的灰度值。由于每一个像素点包含着RGBA四种信息，而我们需要将所有像素点的RGBA值分成8组，因此需要统一一下RGBA的值，最终得到一个值Y，而相应的像素点的RGBA值满足 __Y = R = G = B__ ，在这里我们不考虑透明度 Alpha。由于RGB的值相等像素点颜色是介于白色与黑色之间的灰色，所以这一过程也称为灰度化。

灰度化算法有很多种，我们在这里采取最简单的方式，即

> Y = (R + G + B) * 1/3

相应代码如下

```JavaScript
function rgb2gray(r, g, b) {
    return r * 0.333 + g * 0.333 + b * 0.333;
}
```

## 灰度图量化

灰度化以后的图片大致长这样子，可以看到色彩已经都变成灰色了。

![](https://diycode.b0.upaiyun.com/photo/2017/17b87672b571ee111b5c75718caf9e13.jpg)

那么接下来就是关键的“量化”过程。也就是说，我们要让把这些不同灰度的值分成8组，并且每一组都赋予一个ASCII字符作为标示，当然选取的ASCII字符也要有一定规律，简单来说就是颜色由深到浅相应的字符由繁到简。而量化过程就是将0-255范围等分成8个区间，依次判断灰度值在哪一个区间内，代码如下。

由于图像像素数目巨大，为了效率，判决时可以采取“二分判决”法提高判决速度。

```JavaScript
function gray2asc(gray) {
    /*ASCII--I'mYasic*/
    /*32 64 96 128 160 192 224 256*/
    gray = 255 - gray;
    if (gray < 128){
        if (gray < 64){
            if (gray < 32){
                return '\''
            }
            else {
                return 'c'
            }
        }
        else {
            if (gray < 96){
                return 'i'
            }
            else {
                return 's'
            }
        }
    }
    else {
        if (gray < 192){
            if (gray < 160){
                return 'I'
            }
            else {
                return 'm'
            }
        }
        else {
            if (gray < 224){
                return 'a'
            }
            else {
                return 'Y'
            }
        }
    }
}
```

## 遍历与显示

上面大概讲解完了对于一个像素点变换为ASCII码的过程，接下来就是遍历和显示了。

### 遍历

遍历全部像素点并变换为ASCII码基本是不可能的，因为图片稍微大一些计算量就增长很多，所以我们折中一下，对于像素阵列的行与列都进行等间隔采样，最终展示出来的图片分辨率会随着采样间隔减小而增强。另外要注意 data 数组是一维数组，并且每4个元素为一组RGBA数据。相应代码如下

```JavaScript
    var result = "";
    var lineIndex = 0;
    for (var lineHeight = 0; lineHeight < sourceImg.height; lineHeight += 12){
        var lineASC = "";
        for (var lineFlag = 0; lineFlag < sourceImg.width; lineFlag += 5){
            lineIndex = (lineHeight * sourceImg.width + lineFlag) * 4;
            var r = imgDataArray[lineIndex];
            var g = imgDataArray[lineIndex + 1];
            var b = imgDataArray[lineIndex + 2];
            lineASC += gray2asc(rgb2gray(r, g, b));
        }
        lineASC += '\n';
        result += lineASC;
    }
```

### 显示

最终获得的 __result__ 字符串就是需要展示的ASCII码。但是必须注意，如果直接展示到页面上会因为每一个字符的字符宽度不一样而导致ASCII图“失真”，这里我们可以采用 Monospace 字体来确保字符宽度一致。

## 制作单极图

其实看完上面部分，就应该知道单极图非常好实现，同样需要获取像素信息并灰度化，只是量化时直接量化为 rgb(0, 0, 0) 和 rgb(255, 255, 255) 两种颜色就可以。

```JavaScript
    var canvasContext = targetCanvas.getContext("2d");
    canvasContext.drawImage(sourceImg, 0, 0);
    var imgData = canvasContext.getImageData(0 , 0, sourceImg.width, sourceImg.height);
    var imgDataArray = imgData.data;
    for (var index = 0; index <= sourceImg.width * sourceImg.height * 4; index += 4){
        var red = imgDataArray[index];
        var green = imgDataArray[index + 1];
        var blue = imgDataArray[index + 2];
        var gray = rgb2gray(red, green, blue);
        if (gray < 128){
            imgData.data[index] = 0;
            imgData.data[index + 1] = 0;
            imgData.data[index + 2] = 0;
        }
        else {
            imgData.data[index] = 255;
            imgData.data[index + 1] = 255;
            imgData.data[index + 2] = 255;
        }
    }
    canvasContext.putImageData(imgData, 0, 0);
```

别忘了最后要用 __putImageData__ 方法将修改后的像素信息放回 canvas 中进行显示。

# 结尾

这一篇博客主要讲了利用JS中的 Canvas API 进行一些简单的像素化操作，但其实还有很多地方可以继续改进。比如一般单极图出来后很多地方会有噪点，也就是一些碍眼的白点和黑点，可以通过一些方式“去掉噪点”，就留在以后写吧！

[ASCII图代码地址](https://github.com/Yasic/JS-Canvas-API-Base/blob/master/Img-ASC.js)

[单极图代码地址](https://github.com/Yasic/JS-Canvas-API-Base/blob/master/MonoImg.js)