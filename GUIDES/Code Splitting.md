#Code Splitting
在大型项目中一次性加载所有代码并不是一个高效的做法，特别是一些代码在特定的情况下才使用的时候，没使用到就加载这些代码显然是低性能的做法。Webpack有一个把基础代码分成"chunks"的功能，并且在他们需要的时候才进行加载。另一些代码绑定工具叫它"layers","rollups","fragments".这个特性就叫做“代码分割”。 这是一个可选的功能，你可以定义你需要分割的代码。Webpack会注意相关的依赖，输出的文件和运行时候插入相关"chunk"。一个常见的错误：代码分割不是提取公用的代码到chunk中。它主要的功能是在把代码分割，并在**需要的时候**加载这些chunk。这样可以保证第一次进入页面的时候加载的东西比较少，在需要代码的时候加载相应的代码。
##定义一个分割点
AMD和CommonJS 有各自的方法，这两种都被支持。
###CommonJs:`require.ensure`
```
require.ensure(dependencies,callback)
```
`require.ensure`这个方法中，所有依赖`dependencies`将被同步加载当调用回调函数`callback`的时候。`require`的函数将被作为回调函数的参数传入。
####Example:
```
require.ensure(['module-a','module-b'],function(require){
    var a = require('module-a');
})
```
注意：`require.ensure`仅仅是加载这些模块，并不会执行他们。
注意：可以忽略回调函数。
##ES6 Module
1.xx不支持
##Chunk 内容
所有的依赖被分割成相应的chunk。依赖也可以递归的加入。
如果你的回调函数通过一个函数表达式（或者函数声明）来分割代码，webpack会自动在这个函数表达式中加入所有需要的依赖到chunk中。
##Chunk 优化
如果两个chunk包含相同的模块，他们会合并到一个当中。这会导致这些chunks拥有多个父级。
如果一个模块在所有的chunk的父级中可用，它不会被分割为chunk.
如果一个chunk包含另一个chunk的所有模块，它会被保存。他能完成多个相应的chunk。
##Chunk 加载
###Chunk 入口
一个chunk入口包含运行时加入的模块。如果chunk包含的模块0，webpack运行时会执行。如果没有，webpack会等待chunk包含了模块0然后再执行。（每当chunk中包含有模块0的时候）
###标准的Chunk
一个标准的Chunk不包含在运行的代码当中。它只会包含模块的分支。这样的架构依赖于chunk的加载算法，模块会被包含在jsonp的回调函数中。Chunk也会包含相应的id。
###初始化Chunk(non-entry)
只有一个不同点，这个操作会计算将要初始化加载的时间（例如入口chunk），chunk的类型会在`CommonsChunkPlugin`插件中进行组合，所以初始化在优化chunk中极为重要。
##分割公用的代码
为了把你的app分为2个文件，命名为`app.js`和`vendor.js`,你可以`require`公用文件`vendor.js`。然后通过`CommonsChunkPlugin `插件显示名字在下面。
```
var webpack = require('webpack');
module.exports = {
    entry:{
        app:'./app.js'
        vendor:['jquery','underscore',...],
    },
    ouput:{
       filename:'bundle.js'
    },
    plugins:[
        new webpack.optimize.CommonChunkPlugin(/*chunkName*/"vendor",/*filename=*/"vendor.bundle.js")]
};
```
这会把在`app`chunk中的所有模块移到`vendor`chunk中。`bundle.js`会只包含app中的代码，不会包含任何依赖的模块。这些依赖的模块都被移动到了`vendor.bundle.js`当中。在HTML中你需要先加载`vendor.bundle.js`然后才能加载`bundle.js`.
```
<script src="vendor.bundle.js"></script>
<script src="bundle.js"></script>
```
##引入多个Chunk
配置多个入口文件，这将会返回多个chunk块。The entry chunk contains the runtime and there must only be one runtime on a page (there are exceptions).
##运行多个入口文件
通过`CommonsChunkPlugin `进行[配置](http://example.com/) ，运行时被移动到公共chunk中。当前的入口点在初始化的chunks中。当只有一个初始化chunk被加载后，入口中的多个chunks才会被加载。这也可以在单一页面中运行多个入口点。
例如：
```
var webpack = require("webpack");
module.exports = {
    entry: { a: "./a", b: "./b" },
    output: { filename: "[name].js" },
    plugins: [ new webpack.optimize.CommonsChunkPlugin("init.js") ]
}
```
```
<script src="init.js"></script>
<script src="a.js"></script>
<script src="b.js"></script>
```
###Commons chunk
`CommonsChunkPlugin `插件可以把有多个入口chunk变为一个新的入口chunk(commons chumk)。运行时会也会把它移动到commons chunk中。这意味着旧的入口chunk现在已变成初始化chunk。在[插件列表](http://example.com/)查看所有的配置。

###Named chunks
`require.ensure`函数第三个参数必须是一个String类型。如果两个分割代码函数第三个参数相同，那么他们将会使用相同的chunk。
###`require.include`
`require.include(request)`
`require.include`是webpack中一个特殊的函数，它当前将会增加一个chunk,当是不会执行这个chunk(The statement is removed from the bundle).
```
require.ensure(["./file"], function(require) {
  require("./file2");
});

// 等价于

require.ensure([], function(require) {
  require.include("./file");
  require("./file2");
});
```
模块中包含多个子chunk的时候，使用`require.include`特别有用。一个通过`require.include`进来的父级，会包含一个子chunk的实例将会隐藏的模块。
