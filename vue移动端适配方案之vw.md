##### 一.关于视窗viewport

_viewport_：视口，即用以显示网页的那部分区域。通常视口比屏幕要宽要高，用户可以通过滑动来查看视口的其他部分。

以下是针对移动端页面viewport meta的优化：

```json
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">
```

- width：控制 viewport 的大小，可以指定的一个值，如 600，或者特殊的值，如 device-width （设备的宽度）。
- height：和 width 相对应，指定高度。
- initial-scale：初始缩放比例，也即是当页面第一次 load 的时候缩放比例。
- maximum-scale：允许用户缩放到的最大比例。
- minimum-scale：允许用户缩放到的最小比例。
- user-scalable：用户是否可以手动缩放。

这样设置后，用户就不能缩放视口了，而且视口的高宽等于屏幕的高宽。体验完美 smile:

##### 二.vw的优势？

###### 1.与rem比较

我们知道1rem的长度等于当前页面根元素的font-size值。相对于rem，vw更加简单，优雅和低耦合。

使用rem时，思路是让font-size的值跟随html的宽度改变，以达到弹性布局的效果。

其有两个弊端：一是rem与font-size强耦合，二是在html头文件中插入js使得代码并不优雅。

相对的，1vw等于屏幕宽度的1%，是个固定的值呢~低耦合。

使用vw并不需要写额外的js，直接用就行了，简单优雅。

###### 2.与百分比比较

100vw与width:100%有什么区别呢？100%是根据父元素的宽度来计算的，而vw则是根据viewport得到的，是固定的。

需要注意的是，在移动端，滚动条是不占位的。

而在PC端，右侧的滚动条属于viewport的范畴，因此100vw包括了滚动条，但100%不包括。也就是说，在有滚动条的情况下，设置元素宽度为100vw比设置为100%更宽。

所以，如果在PC端设置宽度为100vw，若有纵向滚动条，则会出现横向滚动条。

总结：在	PC端，还是用100%比较好。

###### 3.rem比vw流行的原因

rem兼容性比vw好。ios8和android4.4以上才完全支持。目前已经不存在这些了，直接用vw吧。

##### 三.在vue中如何配置

我用的是vue-cli2.0构建的项目↓

默认的.postcssrc.js文件：

```json
module.exports = { 
  "plugins": { 
    "postcss-import": {}, 
    "postcss-url": {}, 
    "autoprefixer": {} 
  } 
}
```

要完成vw布局的兼容方案，除了默认的三个插件，我们还需要：

[postcss-aspect-ratio-mini](https://github.com/yisibl/postcss-aspect-ratio-mini)

[postcss-px-to-viewport](https://github.com/evrone/postcss-px-to-viewport)

[postcss-write-svg](https://github.com/jonathantneal/postcss-write-svg)

[postcss-cssnext](https://github.com/MoOx/postcss-cssnext)

[cssnano](https://github.com/ben-eb/cssnano)

先安装：

```json
npm i -s postcss-aspect-ratio-mini postcss-px-to-viewport postcss-write-svg postcss-cssnext postcss-viewport-units cssnano
```

在cssnano的配置中，我们会使用到preset : 'advanced'，所以需要另外安装：

```json
npm i -s -d cssnano-preset-advanced
```

然后修改.postcssrc.js文件为：

```json
module.exports = {
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    "postcss-aspect-ratio-mini": {},
    "postcss-write-svg": {
      utf8: false
    },
    "postcss-cssnext": {},
    "postcss-px-to-viewport": {
      viewportWidth: 750, // 视窗的宽度，对应的是我们设计稿的宽度，一般是750
      viewportHeight: 1334, // 视窗的高度，根据750设备的宽度来指定，一般指定1334，也可以不配置
      unitPrecision: 3, // 指定`px`转换为视窗单位值的小数位数（很多时候无法整除）
      viewportUnit: 'vw', // 指定需要转换成的视窗单位，建议使用vw
      selectorBlackList: ['.ignore', '.hairlines'], // 指定不转换为视窗单位的类，可以自定义，可以无限添加,建议定义一至两个通用的类
      minPixelValue: 1, // 小于或等于`1px`不转换为视窗单位，你也可以设置为你想要的值
      mediaQuery: false // // 允许在媒体查询中转换`px`
    },
    "cssnano": {
      preset: "advanced",
      autoprefixer: false,
      "postcss-zindex": false
    }
  }
}
```

最后重启项目，ok。在css中使用px会自动转换为vw了。适配完成。



关于postcss，以后再说吧。。。



<https://www.w3cplus.com/mobile/vw-layout-in-vue.html>





