#内联样式表
通过使用`style-loader`和`css-loader`可以把样式嵌入由webpack打包的js包中。这个方法你可以通过别的模块来模块化你的样式。通过`require("./stylesheet.css")`，可以很简单使用样式。
##安装
通过npm安装加载器。
`npm install style-loader css-loader --save-dev`
##配置
这里是一个配置的例子
```
{
  //...
  module:{
    loaders:[
      {test:/\.css$/,loader:"style-loader!css-loader"}
    ]
  }
}
```
记住，只是难以管理模块的执行顺序，这样设计样式表，顺序就不是很重要了。
###使用CSS
```
// in your modules just require the stylesheet
// This has the side effect that a <style>-tag is added to the DOM.
require("./stylesheet.css");
```

#抽离css包
通过[extract-text-webpack-plugin]()来合并，可以生成输入CSS的文件。
通过代码分割，我们可以使用两个不同的模式：
*创建一个css文件为初始的chunk(详细看 [代码分割](https://github.com/nljshoxbb/webpack-/blob/master/GUIDES/Code%20Splitting.md))和把内联样式加入到chunks中。（推荐）
*创建一个css文件为初始化chunk，同时也包含来自其他chunks的样式。

###插件安装
通过npm安装插件
```
npm install extract-text-webpack-plugin --save
```
###常规操作
为了使用插件你需要标记被移动到指定的css加载器的模块。在编译优化阶段，webpack插件会检查并抽取相应的模块（在第一种模式中，这些仅仅存在于初始化chunk中）。这些模块在编译中被nodejs使用和执行他们内部的代码。之后，这些模块被再一次编译到一个原始的包中和替换一个空的模块。
通过抽离模块功能创建了一个新的包。
###初始化chunk中的样式在输入的时候分离成多个文件。
这个例子展示了多个入口文件，但也是从单一入口点开始的。
```
var ExtractTextPlugin = require("extract-text-webpack-plugin");
module.exports = {
	// 标准入口和输入配置
	entry:{
		posts:"./posts",
		post:"./post",
		about:"./about"
	},
	output:{
		filename:'[name].js',
		chunkFilename:"[id].js"
	},
	module:{
		loaders:[
			//抽离出css文件
			{
				test:/\.css$/,
				loader:ExtractTextPlugin.extract("style-loader","css-loader")
				
			},
			//配置抽离的less文件
			//或者任何一个css预编译语言
			{
				test:/\.less$/,
				loader:ExtractTextPlugin.extract("style-loader","css-loader!less-loader")
			}
			/*你可以使用同样方法使用其他加载器*/
			
		],

	},
	/* 使用插件可以指定输出的文件名 */
	plugins:[
		new ExtractTextPlugin("[name].css")
	]
}
```
你将会得到以下输出文件posts.js posts.css
post.js post.css
about.js about.css
1.js 2.js (don’t contain embedded styles)
* `posts.js` `posts.css`
* `post.js` `post.css`
* `about.js` `about.css`
* `1.js` `2.js`(包含了内联样式)

##所有的样式在输出的时候进行分离
使用第二种模式你只需要把`allChunks`的配置改为`true`:
```
module.exports = {
	//...
	plugins:[
		new ExtractTextPlugin("style.css",{
			allChunks:true
			})
	]
}
你将会得到以下输出文件
* `posts.js` `posts.css`
* `post.js` `post.css`
* `about.js` `about.css`
* `1.js` `2.js`(不包含内联样式)

###把样式放在公用chunk中
你可以通过CommonsChunkPlugin这个插件分离css文件。这样你就可以在公共chunk中使用这些css文件了。
```
module.exports = {
	//...
	plugins:[
		new webpack.optimize.CommonsChunkPlugin("commons","commons.js"),
		new ExtractTextPlugin("[name].css")
	]
}
```
你将会得到以下输出文件
* `commons.js` `commons.css`
* `posts.js` `posts.css`
* `post.js` `post.css`
* `about.js` `about.css`
* `1.js` `2.js` (contain embedded styles)
or with allChunks: true

* `1.js` `2.js` (don’t contain embedded styles)
