---
title: 关于css，你应该知道的
date: 2018-04-27 14:24:20
tags:
---
https://github.com/kamranahmedse/developer-roadmap
## css预编译
### sass
#### 语法
1、文件后缀名
.sass:不使用大括号和分号
.scss:和css格式接近，使用大括号和分号
    `
	
	//文件后缀名为sass的语法
	body
  		background: #eee
  		font-size:12px
	p
  		background: #0982c1

	//文件后缀名为scss的语法
	body {
  		background: #eee;
  		font-size:12px;
	}
	p{
  		background: #0982c1;
	} `

2、导入
通过@import来导入，导入的sass文件会被编程合并到一个css文件，但是如果在sass中导入css文件，效果和css的导入样式一样，会保留以@import形式存在。sass文件导入可以忽略后缀名。一般基础的文件命名方法以_开头，如_mixin.scssz。这种文件在导入的时候可以不写下划线，@import "mixin"。(顺便提一句，尽量少用css的@import因为它是在引用它的css文件下载下来以后，才会再去下载，用的多会严重影响浏览器并行加载css文件，影响页面渲染性能)
    `

	//被导入的sass文件a.scss
	//-------------------------------
	body {
  		background: #eee;
	}
	//需要导入样式的sass文件b.scss
	@import "reset.css";
	@import "a";
	p{
  		background: #0982c1;
	}

	//转译出来的b.css样式
	@import "reset.css";
	body {
  		background: #eee;
	}
	p{
  		background: #0982c1;
	}
 `
3、注释
多行注释：/**/
单行注释：// 单行注释不会被编译
4、变量
必须以$开头，后面紧跟变量名，变量名和变量值之间用：隔开，如果值后面加上！default则表示默认值。

普通变量：定义之后可以在全局范围内使用
    `

		//sass style
		//-------------------------------
		$fontSize: 12px;
		body{
    		font-size:$fontSize;
		}

		//css style
		//-------------------------------
		body{
    		font-size:12px;
		}`
默认变量：仅需要在值后面加！default，一般是用来设置默认值，然后根据需求来覆盖的，覆盖的方式只要在默认变量之前重新声明下变量即可。
    `

		//sass style
		//-------------------------------
		$baseLineHeight:        1.5 !default;
		body{
    		line-height: $baseLineHeight;
		}

		//css style
		//-------------------------------
		body{
    		line-height:1.5;
		}

		//sass style
		//-------------------------------
		$baseLineHeight:        2;
		$baseLineHeight:        1.5 !default;
		body{
    		line-height: $baseLineHeight;
		}

		//css style
		//-------------------------------
		body{
    		line-height:2;
		}`
特殊变量：一般我们定义的变量都为属性值，可直接使用，但是如果变量作为属性或在某些特殊情况下等则必须要以#{$variables}形式使用。
    `

		//sass style
		//-------------------------------
		$borderDirection:       top !default;
		$baseFontSize:          12px !default;
		$baseLineHeight:        1.5 !default;

		//应用于class和属性
		.border-#{$borderDirection}{
  			border-#{$borderDirection}:1px solid #ccc;
		}
		//应用于复杂的属性值
		body{
    		font:#{$baseFontSize}/#{$baseLineHeight};
		}

		//css style
		//-------------------------------
		.border-top{
  			border-top:1px solid #ccc;
		}
		body {
  			font: 12px/1.5;
		}`

多值变量：多值变量分为list和map类型，list类似js数组，map类似js对象

list数据可通过空格，逗号或小括号分隔多个值，可用nth($var,$index)取值。关于list数据操作还有很多其他函数如length($list),join($list1,$list2,[$separator]),append($list,$value,[$separator])等
	`

		//定义一维数据
		$px: 5px 10px 20px 30px;

		//二维数据，相当于js中的二维数组
		$px: 5px 10px, 20px 30px;
		$px: (5px 10px) (20px 30px);

		//使用
		//sass style
		//-------------------------------
		$linkColor:         #08c #333 !default;//第一个值为默认值，第二个鼠标滑过值
		a{
  			color:nth($linkColor,1);

  			&:hover{
    			color:nth($linkColor,2);
  			}
		}

		//css style
		//-------------------------------
		a{
  			color:#08c;
		}
		a:hover{
  			color:#333;
		}`

map数据以key和value成对出现
	`

		//定义
		$heading: (h1: 2em, h2: 1.5em, h3: 1.2em);

		//使用
		//sass style
		//-------------------------------
		$headings: (h1: 2em, h2: 1.5em, h3: 1.2em);
		@each $header, $size in $headings {
  			#{$header} {
    			font-size: $size;
  			}
		}

		//css style
		//-------------------------------
		h1 {
  			font-size: 2em;
		}
		h2 {
  			font-size: 1.5em;
		}
		h3 {
  			font-size: 1.2em;
		}
`

全局变量：在变量后面加！global现在用不上
目前的变量机制，在选择器中声明的变量会覆盖外面全局声明的变量（sass没有局部变量）

5、嵌套
选择器嵌套和属性的嵌套

选择器嵌套：我们最常用的方式
	`

		#top_nav{
  			line-height: 40px;
  			text-transform: capitalize;
  			background-color:#333;
  			li{
    			float:left;
  			}
  			a{
    			display: block;
    			padding: 0 10px;
    			color: #fff;

    			&:hover{
      				color:#ddd;
    			}
  			}
		}`

属性嵌套：
    `

		//sass style
		//-------------------------------
		.fakeshadow {
  			border: {
    			style: solid;
    			left: {
      				width: 4px;
      				color: #888;
    			}
    			right: {
      				width: 2px;
      				color: #ccc;
    			}
  			}
		}

		//css style
		//-------------------------------
		.fakeshadow {
  			border-style: solid;
  			border-left-width: 4px;
  			border-left-color: #888;
  			border-right-width: 2px;
  			border-right-color: #ccc;
		}`

6、@at-root

sass3.3.0中新增功能，用来跳出选择器嵌套的。默认所有的嵌套，继承所有上级选择器，但有了这个就可以跳出所有上级选择器。
	`

	//sass style
	//-------------------------------
	//没有跳出
	.parent-1 {
  		color:#f00;
  		.child {
    		width:100px;
  		}
	}

	//单个选择器跳出
	.parent-2 {
  		color:#f00;
  		@at-root .child {
    		width:200px;
  		}
	}

	//多个选择器跳出
	.parent-3 {
  		background:#f00;
  		@at-root {
    		.child1 {
      			width:300px;
    		}
    		.child2 {
      			width:400px;
    		}
  		}
	}

	//css style
	//-------------------------------
	.parent-1 {
  		color: #f00;
	}
	.parent-1 .child {
  		width: 100px;
	}

	.parent-2 {
  		color: #f00;
	}
	.child {
  		width: 200px;
	}

	.parent-3 {
  		background: #f00;
	}
	.child1 {
  		width: 300px;
	}
	.child2 {
  		width: 400px;
	}`

7、混合（mixin）

sass中使用 @mixin声明混合，可以传递参数，参数名以$符号开始，多个参数以逗号分开，也可以给参数设置默认值。声明的 @mixin通过 @include来调用。
	`

	//无参数mixin
	//sass style
	//-------------------------------
	@mixin center-block {
    	margin-left:auto;
    	margin-right:auto;
	}
	.demo{
    	@include center-block;
	}

	//css style
	//-------------------------------
	.demo{
    	margin-left:auto;
    	margin-right:auto;
	}

	//有参数mixin
	//sass style
	//-------------------------------
	@mixin opacity($opacity:50) {
  		opacity: $opacity / 100;
  		filter: alpha(opacity=$opacity);
	}

	//css style
	//-------------------------------
	.opacity{
  		@include opacity; //参数使用默认值
	}
	.opacity-80{
  		@include opacity(80); //传递参数
	}`

8、继承

sass中，选择器继承可以让选择器继承另一个选择器的所有样式，并联合声明。使用选择器的继承，要使用关键词 @extend，后面紧跟需要继承的选择器。

	`

	//sass style
	//-------------------------------
h1{
  border: 4px solid #ff9aa9;
}
.speaker{
  @extend h1;
  border-width: 2px;
}

//css style
//-------------------------------
h1,.speaker{
  border: 4px solid #ff9aa9;
}
.speaker{
  border-width: 2px;
}`

占位选择器 %
从sass 3.2.0以后就可以定义占位选择器 %。这种选择器的优势在于：如果不调用则不会有任何多余的css文件，避免了以前在一些基础的文件中预定义了很多基础的样式，然后实际应用中不管是否使用了 @extend去继承相应的样式，都会解析出来所有的样式。占位选择器以 %标识定义，通过 @extend调用。

	`

	//sass style
	//-------------------------------
	%ir{
  		color: transparent;
  		text-shadow: none;
  		background-color: transparent;
  		border: 0;
	}
	%clearfix{
  		@if $lte7 {
    		*zoom: 1;
  		}
  		&:before,
  		&:after {
    		content: "";
    		display: table;
    		font: 0/0 a;
  		}
  		&:after {
    		clear: both;
  		}
	}
	#header{
  		h1{
    		@extend %ir;
    		width:300px;
  		}
	}
	.ir{
  		@extend %ir;
	}

	//css style
	//-------------------------------
	#header h1,
		.ir{
  			color: transparent;
  			text-shadow: none;
  			background-color: transparent;
  			border: 0;
	}
	#header h1{
  		width:300px;
	}`

9、函数
sass定义了很多函数可供使用，当然你也可以自己定义函数，以@fuction开始，实际项目中我们使用最多的应该是颜色函数，而颜色函数中又以lighten减淡和darken加深为最，其调用方法为 lighten($color,$amount)和 darken($color,$amount)，它们的第一个参数都是颜色值，第二个参数都是百分比。

10、运算
sass具有运算的特性，可以对数值型的Value(如：数字、颜色、变量等)进行加减乘除四则运算。请注意运算符前后请留一个空格，不然会出错。
	`

	$baseFontSize:          14px !default;
	$baseLineHeight:        1.5 !default;
	$baseGap:               $baseFontSize * $baseLineHeight !default;
	$halfBaseGap:           $baseGap / 2  !default;
	$samllFontSize:         $baseFontSize - 2px  !default;

	//grid
	$_columns:                     12 !default;      // Total number of columns
	$_column-width:                60px !default;   // Width of a single column
	$_gutter:                      20px !default;     // Width of the gutter
	$_gridsystem-width:            $_columns * ($_column-width + $_gutter); //grid system width`

11、条件判断及循环
@if判断
@if可一个条件单独使用，也可以和 @else结合多条件使用
	`

	//sass style
	//-------------------------------
	$lte7: true;
	$type: monster;
	.ib{
    	display:inline-block;
    	@if $lte7 {
        	*display:inline;
        	*zoom:1;
    	}
	}
	p {
  		@if $type == ocean {
    		color: blue;
  		} @else if $type == matador {
    		color: red;
  		} @else if $type == monster {
    		color: green;
  		} @else {
    		color: black;
  		}
	}

	//css style
	//-------------------------------
	.ib{
    	display:inline-block;
    	*display:inline;
    	*zoom:1;
	}
	p {
  		color: green;
	}`
三目判断
语法为： if($condition, $if_true, $if_false) 。三个参数分别表示：条件，条件为真的值，条件为假的值。
	`

	if(true, 1px, 2px) => 1px
	if(false, 1px, 2px) => 2px`

for循环
for循环有两种形式，分别为： @for $var from <start> through <end>和 @for $var from <start> to <end>。$i表示变量，start表示起始值，end表示结束值，这两个的区别是关键字through表示包括end这个数，而to则不包括end这个数。
	`

	//sass style
	//-------------------------------
	@for $i from 1 through 3 {
  		.item-#{$i} { width: 2em * $i; }
	}

	//css style
	//-------------------------------
	.item-1 {
  		width: 2em;
	}
	.item-2 {
  		width: 4em;
	}
	.item-3 {
  		width: 6em;
	}`

@each循环
语法为： @each $var in <list or map>。其中 $var表示变量，而list和map表示list类型数据和map类型数据。sass 3.3.0新加入了多字段循环和map数据循环。

单个字段list数据循环
	`

	//sass style
	//-------------------------------
	$animal-list: puma, sea-slug, egret, salamander;
	@each $animal in $animal-list {
  		.#{$animal}-icon {
    		background-image: url('/images/#{$animal}.png');
  		}
	}

	//css style
	//-------------------------------
	.puma-icon {
  		background-image: url('/images/puma.png');
	}
	.sea-slug-icon {
  		background-image: url('/images/sea-slug.png');
	}
	.egret-icon {
  		background-image: url('/images/egret.png');
	}
	.salamander-icon {
  		background-image: url('/images/salamander.png');
	}`

## less和sass的区别

1.编译环境不一样
Sass的安装需要Ruby环境，是在服务端处理的，而Less是需要引入less.js来处理Less代码输出css到浏览器，也可以在开发环节使用Less，然后编译成css文件，直接放到项目中，也有 Less.app、SimpleLess、CodeKit.app这样的工具，也有在线编译地址。

2.变量符不一样，Less是@，而Scss是$，而且变量的作用域也不一样。
3.输出设置，Less没有输出设置，Sass提供4中输出选项：nested, compact, compressed 和 expanded。
输出样式的风格可以有四种选择，默认为nested

nested：嵌套缩进的css代码
expanded：展开的多行css代码
compact：简洁格式的css代码
compressed：压缩后的css代码

4.Sass支持条件语句，可以使用if{}else{},for{}循环等等。而Less不支持。
5. 引用外部CSS文件
scss引用的外部文件命名必须以_开头, 如下例所示:其中_test1.scss、_test2.scss、_test3.scss文件分别设置的h1 h2 h3。文件名如果以下划线_开头的话，Sass会认为该文件是一个引用文件，不会将其编译为css文件.
6.Sass和Less的工具库不同

相同点：两者都是CSS预处理器，都具有相同的功能，可以帮助我们快速编译代码，帮助我们更好的维护我们的样式代码或者说维护项目吧。不同点：语法规则不同，当然功能或许略有差别
选择Sass的原因：
1、Sass也是成熟的CSS预处理器之一，而且有一个稳定，强大的团队在维护
2、Sass对于我来说参考的教程多
3、Sass有一些成熟稳定的框架，特别是Compass，新秀还有Foundation之类
4、还有一个原因是国外讨论Sass的同行要多于LESS

### 比较sass、less、stylus
1、变量：就像其他编程语言一样，免于多处修改。

Sass：使用「$」对变量进行声明，变量名和变量值使用冒号进行分割
Less：使用「@」对变量进行声明
Stylus：中声明变量没有任何限定，结尾的分号可有可无，但变量名和变量值之间必须要有『等号』。但需要注意的是，如果用“@”符号来声明变量，Stylus会进行编译，但不会赋值给变量。就是说，Stylus 不要使用『@』声明变量。Stylus 调用变量的方法和Less、Sass完全相同。

2、作用域：有了变量，就必须得有作用域进行管理。就想js一样，它会从局部作用域开始往上查找变量。

Sass：它的方式是三者中最差的，不存在全局变量的概念
Less：它的方式和js比较相似，逐级往上查找变量
Stylus：它的方式和Less比较相似，但是它和Sass一样更倾向于指令式查找


3、嵌套：对于css来说，有嵌套的写法无疑是完美的，更像是父子层级之间明确关系

三者在这处的处理都是一样的，使用「&」表示父元素


## postcss
对于css命名冲突的问题，由来已久，可以说我们前端开发人员，天天在苦思冥想，如何去优雅的解决这些问题。css并未像js一样出现了AMD、CMD和ES6 Module的模块化方案。

那么，回到问题，如何去解决呢？我们的前人也有提出过不同的方案：

Object-Oriented CSS
BEM
SMACSS
方案可以说是层出不穷，不乏有团队内部的解决方案。但是大多数都是一个共同点——为选择器增加前缀。写出来的class很长很复杂。
现在的网页开发，讲究的是组件化的思想，因此，急需要可行的css Module方式来完成网页组件的开发。自从2015年开始，国外就流行了CSS-in-JS(典型的代表，react的styled-components)，还有一种就是CSS Module。
对于css，大家都知道，它是一门描述类语言，并不存在动态性。那么，要如何去形成module呢。我们可以先来看一个react使用postcss的例子：
	`

	//example.css

	.article {
    	font-size: 14px;
	}
	.title {
    	font-size: 20px;
	}`
之后，将这些命名打乱
	`

	.zxcvb{
    	font-size: 14px;
	}
	.zxcva{
    	font-size: 20px;
	}`
将之命名对应的内容，放入到json文件中去
	`

	{
    	"article": "zxcvb",
    	"title": "zxcva"
	}`
之后，在js文件中运用
	`

	import style from 'style.json';

	class Example extends Component{
	    render() {
        	return (
            	<div classname={style.article}>
                	<div classname={style.title}></div>
            	</div>
        	)
    	}
	}`
这样子，就描绘出了一副css module的原型。当然，我们不可能每次都需要手动去写这些东西。我们需要自动化的插件帮助我们完成这一个过程。之后，我们应该先来了解一下postCSS。
PostCSS是什么？或许，你会认为它是预处理器、或者后处理器等等。其实，它什么都不是。它可以理解为一种插件系统。
你可以在使用预处理器的情况下使用它，也可以在原生的css中使用它。它都是支持的，并且它具备着一个庞大的生态系统，例如你可能常用的Autoprefixer，就是PostCSS的一个非常受欢迎的插件，被Google, Shopify, Twitter, Bootstrap和CodePen等公司广泛使用。
接下来，我们来看一下PostCSS的配置：

这里我们使用webpack+postcss+postcss-loader+cssnext+postcss-import的组合。
	`

	yarn add --dev webpack extract-text-webpack-plugin css-loader file-loader postcss postcss-loader postcss-cssnext postcss-import`
然后，我们配置一下webpack.config.js：
	`

	const webpack = require('webpack');
	const path = require('path');
	const ExtractTextPlugin = require('extract-text-webpack-plugin');
	module.exports = {
  		context: path.resolve(__dirname, 'src'),
  		entry: {
    		app: './app.js';
  		},
  		module: {
    		loaders: [
      			{
        			test: /\.css$/,
        			use: ExtractTextPlugin.extract({
          				use: [
            				{
              					loader: 'css-loader',
              					options: { importLoaders: 1 },
            				},
            				'postcss-loader',
          				],
        			}),
      			},
    		],
  		},
  		output: {
    		path: path.resolve(__dirname, 'dist/assets'),
  		},
  		plugins: [
    		new ExtractTextPlugin('[name].bundle.css'),
  		],
	};`
然后在根目录下配置postcss.config.js
	`

	module.exports = {
  		plugins: {
    		'postcss-import': {},
    		'postcss-cssnext': {
      			browsers: ['last 2 versions', '> 5%'],
    		},
  		},
	};`
之后，就可以在开发中使用cssnext的特性了
	`

	/* Shared */
	@import "shared/colors.css";
	@import "shared/typography.css";
	/* Components */
	@import "components/Article.css";


	/* shared/colors.css */
	:root {
  		--color-black: rgb(0,0,0);
  		--color-blue: #32c7ff;
	}

	/* shared/typography.css */
	:root {
  		--font-text: "FF DIN", sans-serif;
  		--font-weight: 300;
  		--line-height: 1.5;
	}

	/* components/Article.css */
	.article {
  		font-size: 14px;
  		& a {
    		color: var(--color-blue);
  		}
  		& p {
    		color: var(--color-black);
    		font-family: var(--font-text);
    		font-weight: var(--font-weight);
    		line-height: var(--line-height);
  		}
  		@media (width > 600px) {
    		max-width: 30em;
  		}
	}`

个人感觉css module是比较好的处理css打包的解决方案，给我们开发带来很大的便利，结合sass等css预处理，高效的完成开发工作。

另外关于css结构可以关注一下下面几个关键词

### BEM、OOCSS、SMACSS、SUITCSS、Atomic