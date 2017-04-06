---
title: webpack踩坑
date: 2017-03-02 17:36:26
tags: 
	- 前端 
	- 打包工具
	- webpack
categories: JavaScript
---

## 运行效率
- 如果项目是多入口配置，在本地开发阶段不需要每次都跑全部的
- 可以获取到运行命令行参数决定，跑哪些页面，加快速度
- process.env.npm_config_argv或者使用yargs这个npm包获取命令行传入的参数

```javascript
var scriptArg = process.env.npm_config_argv && JSON.parse(process.env.npm_config_argv);
var targetDir = scriptArg && scriptArg.original[2] && scriptArg.original[2].substr(1) || '';

var js = glob.sync('./' + (targetDir || '**') + '/*.js').reduce(function (prev, curr) {
    prev[curr.slice(6, -3)] = ['./' + curr.substr(6)];
    return prev;
}, {});
```

```javascript
var argv = require('yargs').argv;
var list = argv._[0] || '*';

var js = glob.sync('./' + list + '/*.js').reduce(function (prev, curr) {
    prev[curr.slice(6, -3)] = [curr];
    return prev;
}, {});
```
- process.env.npm_config_argv：在使用yarn run xxx的时候不能获取到参数，在使用npm run xxx的时候可以获取到
- yargs这个包是都可以获取到的

## 路径问题
- 问题描述：在上线和测试环境下代码发到cdn上，静态资源和页面地址不在一处，需要配置不同的前缀地址，才能引入线上script
- 通过**output -> publicPath** 根据不同的环境变量变更path路径，解决加载问题

```javascript
output: {
	path: path.resolve('./build'),
	filename: '[name].js',
	publicPath: env === 'development'
		? '/' : (
			env === 'production'
				? '//xxx.production.com/'
				: '//xxx.test.com/'
		)
}
```

## htmlPlugin
### 普通配置

```javascript
var htmlFiles = glob.sync('./**/*.html');
var html = htmlFiles.map(function (item) {
    console.log(item.substr(6));
    return new HtmlWebpackPlugin({
        data: {
            build: true
        },
        filename: item.substr(6),
        template: 'ejs-compiled!' + item,
        inject: false,
        minify: {
            removeComments: true,
            collapseWhitespace: true,
            preserveLineBreaks: true,
            collapseInlineTagWhitespace: true,
            collapseBooleanAttributes: true,
            removeRedundantAttributes: true,
            useShortDoctype: true,
            caseSensitive: true,
            minifyJS: true,
            minifyCSS: true,
            quoteCharacter: '"'
        }
    });
});
```

- 把html 连接到webpack的plugin属性上（concat）一个配置只能针对一个文件
- data的内容，可以在template里获取到然后做相应的改动
- filename：输出html的名称，可以带路径
- template: 模板名称及使用什么编译器 ejs-compiled可以使用ejs语法
- inject：是否把所有静态资源自动添加到html里，这里不使用，目前的模板配置是直接引入了入口的文件，后面在回调里添加hash实现，下图是模板文件：

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
  <%- include ../header.html -%>
  <title></title>
  <meta http-equiv="Cache-Control" content="no-cache">
</head>
<body>
  <div id="mount"></div>
  <script src="./xxx.js" defer></script>
</body>
</html>
```

- 可以使用ejs的语法
- minify：压缩模板配置
	- removeComments：删除注释
	- collapseWhitespace：删除多余的空格
	- preserveLineBreaks：保留每个标签tag的换行符
	- collapseInlineTagWhitespace：不保留行内元素两边的空格
	- collapseBooleanAttributes：使input的一些的布尔值的属性合并，不写等于了，例如：disabled
	- removeRedundantAttributes：删除多余的标签，例如没有赋值的style，class等
	- useShortDoctype：替换成较短的html5 doctype
	- caseSensitive：区分大小写的方式处理自定义标签
	- minifyJS：行内scripts标签，或者attribute里的是否压缩
	- minifyCSS：行内style标签，或者attribute里的是否压缩
	- quoteCharacter：对于attribute用什么符号
	
### plugin完成后的一些操作

```javascript
function () {
	this.plugin('done', function (stats) {
		var replaceInFile = function (filePath, toReplace, replacement) {
			var str = fs.readFileSync(filePath, 'utf8');
			var out = str.replace(toReplace, replacement);
			fs.writeFileSync(filePath, out);
		};

		htmlFiles.map(function (item) {
			replaceInFile(path.join(config.build.assetsRoot, item.substr(6)),
				/(src|href)=\"([\/\w\.]+\.(js|css))\"/g,
				function ($0, $1, $2) {
					var file = $2.split('.');
					file.splice(-1, 0, stats.hash);
					file = file.join('.');
					return $1 + '="' + file + '"';
				}
			);
		});
	});
}
```

- plugin执行完后执行替换html的js引用，增加对应的hash值，stats.hash
- 获取到数据流后通过正则匹配，替换
### 参考文档：
http://perfectionkills.com/experimenting-with-html-minifier/#collapse_whitespace
http://perfectionkills.com/experimenting-with-html-minifier/#remove_comments
http://perfectionkills.com/experimenting-with-html-minifier/#collapse_boolean_attributes
http://perfectionkills.com/experimenting-with-html-minifier/#use_short_doctype
https://github.com/kangax/html-minifier#options-quick-reference
https://github.com/jantimon/html-webpack-plugin

## sass-loader
设置import相对路径，不过最后引用的时候用**~**开头，node_modules里的scss引用可以直接相对"~"引用

{% asset_img 3.jpeg %}

https://github.com/webpack-contrib/sass-loader#imports

## resolve

```javascript
resolve: {
	extensions: ['', '.js', '.vue'],
	fallback: [path.join(__dirname, '../node_modules')],
	alias: {
		'vue$': 'vue/dist/vue',
		'scss': path.resolve(__dirname, '../src/modules/scss'),
		'vue-directive': path.resolve(__dirname, '../src/modules/vue-directive'),
		'request': path.resolve(__dirname, '../src/modules/request'),
		'common': path.resolve(__dirname, '../src/modules/common'),
		'vue-component': path.resolve(__dirname, '../src/modules/vue-component'),
		'vue-fragment': path.resolve(__dirname, '../src/modules/vue-fragment')
	}
}
```

- extensions：扩展名
- fallback：
- alias：别名，具体定义一个别名的路径，方便引入modules

{% asset_img 7.jpeg %}

webpack2把root，fallback迁移到了modules的配置里，告诉webpack解析的搜索路径
Explicit is better than implicit
alias清晰的比modules的好，效率？？

## 配置代理
- 背景：本地开发嵌套iframe的页面的时候，两个项目的端口号不统一，导致iframe同源策略不允许互相操作跳转
- 解决：配置到同一端口号的域名上。“devServer.proxy”

```javascript
devServer: {
	host: 'localhost',
	port: 8080,
	historyApiFallback: true,
	inline: true,
	hot: true,
	proxy: {
		'/pages': 'http://localhost:8899'
	},
	stats: {
		children: false,
		chunks: false
	}
}
```


## 注意点
1. 路径是否正确，publicPath，path，如果是相对路径相对哪里，最好配置成path.resolve(__dirname, xxx)的形式
2. 如果用的别人配置好的多文件的配置，检查loader是否有重复的，可能出现多个loader配置重复的问题，等于是模块解析了2次，会报错
3. 拆分模块的情况下，一定要配置publicPath，确定异步加载的模块地址