

## 名词解释

### 1、pid

​	process id 进程id

### 2、文件描述符

非负整数，**是索引，指向被打开的文件**。每个进程中的文件描述符是一个指针数组，前三位有默认值(0: 输入，1:输出，2:错误流)，从第四位开始指向打开的文件。(https://juejin.cn/post/6878936399681159176)

### 3、套接字socket

定义：表示端口的ip地址+端口号

位置：应用层 —— socket ——tcp/udp（双方通信用socket的outputstream和inputstream），而不必感知tcp/udp以及ip

i/o机制：socket进行通信之前需要把 数据拷贝到缓存区 （输入缓存+输出缓存）。

输出缓存的数据还没发出去时，此时还有数据要发，出现输出缓存不够用以及输入缓存有空余的问题。

### 4、单点登录

后端sso：后端单点登录，即登录一次，多个应用共享登录状态，不依赖浏览器验证身份

前端sso：同上，用浏览器存储令牌，实现共享登录状态



# 💡零碎知识点

### 1. vconsole

**定义**：移动端console的内容在原生上显示，pc页面看不到，用了vconsole以后原声和pc都能显示；但是不能打印dom节点（this.$ref, this.$parent等)，此时采用原本的console.log，注释掉vconsole就行。

**使用**：

```
//  安装
npm install vconsole 

//  使用
import VConsole from 'vconsole'

const vConsole = new VConsole()
console.log(xxx)

// 销毁
vConsole.destroy();
```

### **2. 模版渲染完后触发**

函数写在数据请求回来后的settimeout里 / 点击按钮之后触发

### 3. 父子组件生命周期顺序

#### a. 调用过程

> **foreCreate**->父**created**->父**beforeMount**->子**beforeCreate**->子**created**->子**beforeMount**->子**mounted**->父mounted

#### b. 更新过程

> 父**beforeUpdate**->子**beforeUpdate**->子**updated**->父updated

#### c. 引出问题

1️⃣ 父组件在created中请求数据，请求是异步的，子组件使用props渲染dom时，数据可能还未请求回来，因此用v-if或者watch

v-if (在父组件中设置，dh5-op常用)

```html
<div class="test">
    <children v-if="data1" :data="data1" ></children>
</div>
```

watch版本 (在子组件中设置，用得少)

```js
props:['data1'],
watch:{
    data1:{
      deep:true,
      handler:function(newVal,oldVal) {
        this.$nextTick(() => {
          this.data1 = newVal
          this.showData1(this.data1)   // 重新渲染
        })
      }
    },
}
```

2️⃣ props改变，模版不更新；

传入的props改变，子组件中的props会改变，直接用props中的数据渲染，模版会更新；

props在created和mounted中二次加工后，再渲染在模版中时，props改变，created和mounted不会再次调用，模版不会自动更新 需要用 ➡️ watch看props是否改变，改变修改模版中用到的数据。

### 4、z-index

#### **定义**

- z-index 较大的重叠元素会覆盖较小的元素。
- 当父元素设置了`z-index`属性时，子元素的`z-index`属性只与同级子元素作比较时才有意义。此时子元素与父元素外的所有的外部元素进行堆叠层级顺序对比时，都以父元素的`z-index`属性值为准进行对比，子元素本身的`z-index`属性值无效。
- 当父元素未设置了`z-index`属性，子元素的`z-index`属性与父元素外的所有的外部元素进行堆叠层级顺序对比时，都以元素本身的`z-index`属性值为准进行对比。

#### **创建层级上下文**

- 设置了position为absolute或relative，且z-index不是auto的元素
- 设置了z-index，且不为auto的流动元素
- 设置了opacity，且不为1的元素
- **设置了transform，且不为none的元素**（父级增加transform-style: preserve-3d;）设置了transform，子元素z-index会失效
- 设置了mix-blend-mode值，且不为normal的元素
- 设置了isolation 为 isolate的元素on mobile WebKit and Chrome 22+,
- 设置position为fixed的元素

#### **属性值**

- number
- auto
- inherit
- unset
- initial



#### **适用范围**

position: relative | absolute | fixed ｜sticky的元素（非static），父元素为flex, grid的元素



继承属性也有默认值

unset：如果是继承属性，设为继承值；非继承属性，设为默认值；

initial：强制设为默认值

### 5、transition标签

组件之间过渡，和name一起搭配使用

```
<transition name="sky" :duration='500'>
	<router-view></router-view>
</transition>
```

会有6个样式

```css
.sky-enter-from
.sky-enter-active
.sky-enter-to

.sky-leave-from
.sky-leave-active
.sky-leave-to

// 不执行样式
<transition name="sky" :css='false'>
```

6个生命周期

```
<transition 
	@before-enter='beforeEnter'
	@enter='enter'
	@after-enter='afterEnter'
	
	@before-leave='beforeLeave'
	@leave='leave'
	@after-leave='afterLeave'
>

methods: {
	beforeEnter(el) {   //生命周期函数的第一个参数代表的就是动画元素
		
	},
	enter(el, done) {
		done()      //enter、leave有参数done,调用后不会执行afterEnter、afterLeave
	}
}
```



# 问题合集

1、try catch是处理的请求的哪种类型错误

任意类型的，网络错误、服务器响应错误、请求代码错误，除非后端配的是状态码200单还是请求错误



2、监控对象里面属性的变化

```
watch:{
	'queryData.vehicleType': function (vehType) {},
}
```



3、多个子元素有相同事件，可以把事件绑定在父元素上（冒泡）

特殊：子组件click名称不同也行

```html
<div class="parent">
  <div @click="handleClick"></div>
  <Children @handleChildrenClick="handleClick" />
</div>
```



# nginx

### 问题

为什么可以部署前端代码
cors和代理服务器不一样，为什么要提cors
动态和静态请求都是进行转发到各自服务器上处理，怎么体现出nginx静态处理能力很强

nginx和proxy配置项都是代理服务器，区别是什么

### 1、简单请求&非简单请求

1）、请求方式为post、get 、head中的一种

2）、请求头信息不超过以下5个：accept, accept-language, content-language, last-event-id, content-type(只包含这三种类型：application/x-www-form-urlencoded、multipart/form-data、text/plain)

跨域用cors处理（请求头加上origin，非简单请求多一次option请求）



### 2、正向&反向代理

正向代理：代理服务器转发客户端的请求，代理服务器和原始服务器通信，最后把结果返回给客户端。**正向代理隐藏了真实客户端**，真实客户端对服务器不可见；

反向代理：代理服务器收到请求后，将请求转发给内网真正的服务器（内网中有很多服务器，处理请求的是其中一台），将代理服务器结果返回给客户端。**反向代理隐藏了真实服务器**，真实服务器对客户端不可见；（跨域常见why？）



sum up: 一般给客户端做代理的都是正向代理，给服务器做代理的就是反向代理



### 3、nginx.conf结构

```nginx
user  nginx;                        # 运行用户，默认即是nginx，可以不进行设置
worker_processes  1;                # Nginx 进程数，一般设置为和 CPU 核数一样
error_log  /var/log/nginx/error.log warn;   # Nginx 的错误日志存放目录
pid        /var/run/nginx.pid;      # Nginx 服务启动时的 pid 存放位置

#events影响nginx服务器和客户端之前的网络连接
events {
    use epoll;     # 使用epoll的I/O模型(如果你不知道Nginx该使用哪种轮询方法，会自动选择一个最适合你操作系统的)
    worker_connections 1024;   # 每个进程允许最大并发数
}

#http 代理、缓存、第三方模块等配置（使用最频繁
http { 

}

#server nginx虚拟主机的相关配置

#upstream 配置后端服务器地址



```

pid：process id，nginx正在执行的进程id





# webpack

npm、nvm和npx区别

- npm：包管理工具，如安装、下载卸载、更新包，下载后放在node_modules中
- npx：包执行工具，执行node_modules中的包
- nvm: 管理 Node.js 版本的工具，切换不同 Node.js 版本。





修改webpack配置，自动运行npx webpack

```
npx webpack --watch
或者
module.exports = {
  watch: true,
};
```







打包后dist的html，和template中的html一样，复制过来，并自动把css和js添加到`</body>`前

```
const HtmlWebpackPlugin() = require('html-webpack-plugin')

module.exports = {
	plugins: [
		new HtmlWebpackPlugin({
			template: './index.html',
			filename:'app.html',
			inject: 'body'
		})
	]
}
```

报错时定位源代码，而不是打包后的代码

```
module.exports = {
	devtool: 'inline-source-map'
}
```



webpack-dev-server

修改代码后，不用刷新浏览器；

server的根目录dist并没有输出物理文件？打包好的文件放在了内存中，提高了开发、编译效率

```js
devServer: {
	static: './dist'    // 当前目录下的dist文件，作为server的根目录
}
```

```js
//  打包
npx webpack   

//  启动一个服务，是在本地内存中开启空间启动为服务器，使用前需安装npm install 	webpack-dev-server（webpack4版本）
npx webpack-dev-server --save   

//  并自动打开网页
npx webpack-dev-server --open  

// 等同于npx webpack-dev-server --save，是webpack5版本，无需安装
npx webpack serve 	
```



### 1️⃣ loader

定义：webpack只能理解js和json文件，css需要loader解析并转化为模块

使用：

```js
module.exports = {
    module: {
       rules: [
        {
          text:/\.(css|less)$/,        // 以.css结尾的文件, less-loader可以解析.css和.less
        	use: ['style-loader', 'css-loader', 'less-loader']      //  打包前用loader解析,顺序是从后往前，less转为css，解析css-loader，再解析style-loader，最后返回一个js
        }
       ]                   
    }
}
```

安装

```
npm install css-loader -D   //能够import css安装在本地目录下的开发环境
npm install style-loader -D  //把css放置到html的header中，<style>....</style>
npm install less-loader less -D  //安装less和less-loader
```

#### 1、抽离css文件：**mini-css-extract-plugin**

作用：webpack5新特性，把css放在单独文件中，引入html <link>

```
npm install mini-css-extract-plugin -D  //webpack5专用，把css放在单独文件中，引入html <link>
```

```js
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

module.exports = {
	plugins: [
		new MiniCssExtractPlugin()
	],
	module: {
		rules: [
			{
				text: /\.(css|less)$/,
				use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader']    //不以<style>的方式引入 
			}
		]
	}
}
```

指定打包后的位置

（原本：dist/main.css  -->  dist/styles/xxx.css)

```
plugins: [
		new MiniCssExtractPlugin({
			filename:'styles/[contenthash].css'   
		})
	],
```

#### 2、压缩css

npm install css-minimizer-webpack-plugin -D

1、引入  

2、mode从development ->  production

3、optimization的minimizer

```js
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin')

module.exports = {
  mode: 'production'
	optimization: {
	  minimizer: [
	  	new CssMinimizerPlugin()
	  ]
	}
}
```

#### 3、加载csv, xml, tsv等数据

csv用逗号做分隔符，tsv用制表符作分隔（Tab, \t）

```
npm install csv-loader xml-loader -D   //csv-loader可加载tsv-loader
```

```js
module: {
	rules: [
		{
			test: /\.(csv|tsv)$/,
			use: 'csv-loader'
		},
		{
			test: /\.xml$/,
			use: 'xml-loader'
		}
	]
}
```

xml文件  ——>  json对象

csv文件   ——>   数组（每行作为一个数组元素）

####  4、自定义json模块(parser)

加载yaml  toml  json5文件

npm install yaml toml json5 -D

```js
module: {
  rules: [
    {
      test: /\.yaml$/,
      type: 'json',
      parder: {
        parse: yaml.parse
      }
    },
    {
      test: /\.toml$/,
      type: 'json',
      parder: {
        parse: toml.parse
      }
    },
    {
      test: /\.json5$/,
      type: 'json',
      parder: {
        parse: json5.parse
      }
    }
  ]
}
```



yaml文件：key-value形式，缩进表示父子

```yaml
name: John Doe
address:
  street: 123 Main Street
  city: Springfield
  state: Illinois
email: johndoe@example.com
```

toml文件: key=value, [owner]创建表格

```toml
name=John Doe

[address]
street = "123 Main Street"

[email]
address = "johndoe@example.com"

[phone]
mobile = "555-555-5555"
```

json5文件: 可以使用注释、key可以不用引号，可以使用转义字符 \t \n等

```json5
{
  // 这是一个JSON5示例
  name: 'John Doe',
  age: 30,
  isStudent: false,
  address: {
    street: '123 Main \n\St  reet',
    city: 'Springfield'
  }
}
```



css也可以加载img

xxxclass { background-image: url('./assets/xxx.svg') }

js中: dom.classList.add('xxxclass')



用资源模块**加载font字体**

```
module.exports = {
	mode: 'development',
	module:{
		rules: [
			{
			test: /\.(woff|woff2|eot|ttf|otf)$/,
			type: 'asset/resource'
			}
		]
	}
}
```

js里面正常使用定义好的icon类，dom.classList.add('iconfont')

css里面，@font-face{font-family: 'iconfnt', src:',/assets/iconfont.ttf'}   icon {font-famil: 'iconfont'}

### 2️⃣ babel-loader

把打包后的dist的js文件里面的es6转换为es5

- babel-loader: 允许webpack使用babel
- babel/core:  babel核心库，负责实际的转换
- babel/preset-env: 预设，根据指定的环境（node,浏览器版本）自动确定需要的插件。

安装：npm install babel-loader babel/core  babel/preset-env -D

```js
module :{
	rules: [
		{
			test: /\.js$/,
      use: {
        loader: 'babel-loader',
        options: {
          preset: ['babel/preset-env']
        }
      }
		}
	]
}
```

安装regeneratorRuntime：babel生成的全局辅助函数，用来兼容async/await语法

```
// 安装regeneratorRuntime运行时需要的内容
npm install @babel/runtime -D 
 
// 在需要async/await的时候自动require导入regeneratorRuntime 
npm install @babel/plugin-transform-runtime -D
```

```js
module :{
	rules: [
		{
			test: /\.js$/,
      use: {
        loader: 'babel-loader',
        options: {
          preset: ['babel/preset-env'],
          plugins: [
            [
              '@babel/plugin-transform-runtime'    //增加了对loader的配置
            ]
          ]
        }
      }
		}
	]
}
```

### 3️⃣ 代码分离

#### 1、入口起点

设置多个打包入口，对应多个出口；

缺点：多个入口中共用的包会各自打包到对应的boundle.js中，打包了重复代码，体积变大；

```js
module.exports = {
  entry: {
    index: '.src/index.js',
    another: './src/another-module.js'
  },
  output: {
    filename: '[name].boundle.js'      // [name]是entry的index
  }
}
```

#### 2、防止重复 

设置多个打包入口，对应多个出口，并把公共代码导出到公共的包中；

方法一：dependOn

```js
module.exports = {
  entry: {
    index: {
      import: '.src/index.js',
      dependOn: 'shared'
    },
    another: {
      import: './src/another-module.js',
      dependOn: 'shared'
    },
    shared: 'lodash'                    // 在以上两个入口文件中遇到require(lodash)时，打包到shared的包中
  },
  output: {
    filename: '[name].boundle.js'      // [name]是entry的index
  }
}
```

方法二：splitChunks

```js
module.exports = {
	 entry: {
    index: '.src/index.js',
    another: './src/another-module.js'
  }, 
  output: {
    filename: '[name].boundle.js'      // [name]是entry的index
  },
  optimization: {
    minimizer: [
      new CssMinimizerPlugin()
    ],
    splitChunks: {
      chunks:  'all'        //重复引入/静态资源（lodash、babel-runtime)自动打包到各自单独的包中 lodash一个bundle，babel一个
    }
  }
}
```

####    3、动态导入

- import('./myModule.js')：异步加载模块或资源，模块的路径
- /* webpackChunkName: "myChunk" */ ：注释，打包好的模块名字
- then：module是加载成功的模块，如loadsh

```js
import(/* webpackChunkName: "myChunk" */ './myModule.js')
  .then((module) => {
    // 在这里可以使用加载的模块
  })
  .catch((error) => {
    // 处理加载模块时的错误
  });

```

可以import和splitChunks结合使用，引入的资源相同，只会打包成一个bundle

单独使用import不需要配置多个入口和splitChunks 

##### 动态导入应用

a. 懒加载（按需加载） ：在需要的时候引入，如按钮点击后才进入math模块，不点击永远不引入

b.预获取/预加载：webpack 4.6以上，

预获取：在首页加载完后，网络空闲时加载模块，是在html header里增加了

` <link rel="prefetch" as="script" href="hyttp://localhost:8080/math.bundle.js">`

```js
import(/* webpackChunkName: "myChunk", webpackPrefetch: true */ './myModule.js')
  .then((module) => {
    // 在这里可以使用加载的模块
  })
  .catch((error) => {
    // 处理加载模块时的错误
  });
```

预加载：跟懒加载差不多

```js
import(/* webpackPreload: true */ './myModule.js')
```

### 4️⃣ 缓存

目的：打包好的文件能够被缓存，文件被修改时不走缓存能够获取新文件。

##### **1、打包文件的文件名**

boundle的内容改变了但文件名没有改变，浏览器会认为打包后的文件没有修改，因此需要把文件名一起打包。

这样浏览器缓存，打包内容变了以后就不会走缓存。

```
output: {
	filename: '[name].[contenthash].js',       // 文件名会随内容的改变而改变
}
```

##### **2、缓存第三方库**

第三方库很少修改，需要被缓存，因此打包后的文件名始终不变。把第三方库打包到vendor chunk（打包后以vendors-node-nodules-...开头）中，利用client/浏览器长效缓存来实现

```js
moduel.exports = {
  optimization: {
    splitChunks: {
    	cacheGroup: {
    	  vendor:{
    	    test: /[\\/]node_modules[\\/]/,   // 拿到node_modules中的文件
    	    name: 'vendors',                  // 所有的第三方库打包到一个vendors.bundle.js里
    	    chunks: 'all'                     // 对所有chunks的第三方库做缓存处理
    	  }
    	}
   }
  }
}
```

3、把js代码打包到一个文件夹里（包含业务代码+第三方）

```
output: {
	filename: 'scripts/[name].[contenthash].js',
	path: path.resolve(_dirname, './dist')
}
```

###  5️⃣ 拆分开发/生产环境

为什么需要拆分：开发环境不需要缓存，生产环境需要公共路径等，不能每次都手动调整mode

#### 1、公共路径

用于指定所有资源的基础路径：dist/html的css和js的引用是相对路径，可以修改为服务器的绝对路径。

```
output: {
	publicPath: 'http://localhost8080/'      // 可以指定为服务器域名
}
```

dist/html的所有路径

```html
<link href="http://localhost8080/styles/xxxxx.css">    // 以前href="styles/xxxxx.css", 相当于dist/styles/xxx.css
```

####  2、环境变量

消除开发、生产环境之间的差异

a) 引入生产环境环境变量

```
npx webpack --env production      // 用户指定的环境
npx webpack --env production --env --goal=local    // 可以添加其他变量 console.log(env) {..., production:true, goal: 'local'}
```

b) module.exports改为函数

```js
module.exports = (env) => {
  console.log(env)               // {WEBPACK_BUNDLE:true , WEBPACK_BUILD:true, production:true}
  return {
    entry:{},
    output:{}
    ...
    mode: env.production ? 'production':'development'
  }
}
```

c) 压缩js：**terser**（本来自动压缩，optimization配置了minimizer后失效，要手动配）

js压缩后，dist会自动产生一个vendor...js.LICENSE.txt；

一般开发环境不会压缩js，terser会失效；

```js
npm install terser-webapck-plugin -D

const TerserPlugin = require('terser-webapck-plugin')

module.exports = (env) => {
   return {
     mode: env.production ? 'production':'development',
     optimization: {
       minimizer: [
         new CssMinimizerPlugin(),
         new TerserPlugin()
       ]
     }
  }
}
```

#### 3、拆分webpack.config

按照production/development拆分为两个配置文件  webpack.config.dev.js和webpack.config.prod.js

a) 修改配置文件，如dev配置的修改

```js
/* 
1、output不需要缓存文件名，修改打包后的文件位置
2、不需要publicPath(js和css加载路径是相对路径，不是服务器路径，如cdn，http://localhost:8080)
3、mode不用做判断，module.exports从函数改为对象
4、devServer和devtool要保留
5、去掉压缩 optimization里的minimizer
6、指定配置文件
*/
output: {
	filename: '[name].[contenthash].js'  -> '[name].js'，
  path: path.resolve(_dirname, './dist')   // 当前文件从./变成了./config，因此改为 '../dist'
  path: path.resolve(_dirname, '../dist')
},
mode: 'development',
devServer: {
  static: './dist'
},
devtool: 'inline-source-map'

npx webpack -c ./config/webpack.config.dev.js   // -c是--config缩写
```

#### 4、运行/打包的别名

start：用于开发环境，开启一个服务器

build：用于生产环境，压缩代码打包文件，准备用于部署

```js
// package.json中
{
	"script": {
    "start": "webpack serve -c ./config/webpack.config.dev.js",    // 开发环境开启服务 npm run start, 别名
    "build": "webpack -c ./config/webpack.config.prod.js"					 // 生产环境打包    npm run build
  }
}
// 关闭性能提示（提示打包体积太大，在webpack.config.prod.js中
module.exports = {
  perfomance: {
    hints: false;
  }
}

```

**npx** webpack serve -c ./config/webpack.config.dev.js和webpack serve -c ./config/webpack.config.dev.js的区别：

有npx，是使用的本地项目的webpack，即项目node_modules中的webpack；

无npx，是使用的全局的webpack，项目中的webpack和全局版本不一致会冲突；

#### 5、提取dev&prod公共配置

在config中配置三个文件：webpack.config.dev.js、 webpack.config.prod.js、  webpack.config.common.js 

```
项目
--node_modules
--config
  --webpack.config.dev.js
  --webpack.config.prod.js
  --webpack.config.common.js 
  --webpack.config.js 
--dist
	--imgs
	--js
		--main.js
	--styles
	app.html
--src
app.js
index.html
package.json
```

公共部分写在common中，dev和prod保留不一样的部分

三个文件合并在一起用**merge**：

a) 安装：npm install web pack-merge -D

b) 在config里新建 webpack.config.js

```js
const { merge } = require("webpack-merge")
const commonConfig = require('./webpack.config.common.js')
const devConfig = require('./webpack.config.dev.js')
const prodfig = require('./webpack.config.prod.js')

module.exports = (env) => {
  switch(true){
    case env.development:
      return merge(commonConfig, devConfig)
    case env.production:
      return merge(commonConfig, prodConfig)
    default
      return new Error('no matching configuration was found')
  }
}
```

c) package.json

```js
{
  "script": {
    "start": "webpack serve -c ./config/webpack.config.common.js --env development",
    "build": "webpack -c ./config/webpack.config.common.js --env production"
  }
}
```

d) webpack.config.common.js 

```js
output: {
  filename: '[name].boundle.js'     
  path: path.resolve(_dirname, '../dist')    // 修改路径，打包到项目根目录下
}
```



### 6️⃣高级 source-map

开发环境时，默认devtool: ”eval“；

生产环境时，默认不开启 devtool: “none”；（开启后sourcemap文件体积很大，且把打包后的bundle和sourcemap结合可以反编译出源码，有泄漏源码的风险）

**1、devtool： ”eval“**   

控制台报错/console.log后面的行数，显示的是代码中的位置（行数），而不是打包后的行数

设置为eval时，打包后的boundle.js如下

```js
eval("js源代码\n\n//#sourceRUL:项目目录...入口文件js的name.js")   // 源代码用eval包裹，且在结尾处加上注释//#sourceURL...
```

**2、source-map**

- 在dist下新增main.js**.map**文件 

  ```
  --dist
  	--main.js
  	--main.js.map
  ```

- dist/main.js最后一行加 sourceMappingURL

  ```
  //# sourceMappingURL=main.js.map
  ```

**3、hidden-source-map**   --> 报错/console不会对应源码行数

- 新增main.js**.map**文件 
- 最后一行**不加**sourceMappingURL

**4、inline-source-map**

- 不新增main.js.map

- main.js最后一行新增为

  ```
  //# sourceMappingURL=data:application;charset:utf-8;...(main.js里面的内容)
  ```

**5、eval-source-map**

- 不新增main.js.map

- main.js里改为

  ```
  eval("js源代码\n\n//#sourceRUL=[module]\n//#sourceMappingURL=main.js里面的内容")   // 源代码用eval包裹，且在结尾处加上注释//#sourceURL...
  ```

**6、cheap-source-map**

在main.js.map的mapping包含main.js和源代码的行列映射。

- 去掉列信息"mapping":";;;;AAA,4B"  ->"mapping":";;;;AAA"   --> 打包后文件体积更小
- 需要babel转化时，**不能**显示源代码位置（用到loader的地方）

**7、cheap-module-source-map**

- 去掉列信息"mapping":";;;;AAA,4B"  ->"mapping":";;;;AAA"；  
- 需要babel转化时，也是显示源代码位置

### 7️⃣ devServer

**static**

指定server的物理位置，即哪个目录作为服务器的根目录

⚠️ 注意：服务器根目录是dist，项目根目录是../dist，publicPath默认''，是相对路径，此刻是项目根目录，即../dist，因此改为‘/’（服务器根目录绝对路径dist）

```js
module.exports = {
  output: {
    filename: '[name].[contenthash].js'
    publicPath: '/'                              // 改为绝对路径，项目根目录
  },
  devServer: {
		static: path.resolve(_dirname, './dist')
	}
}
```

**compress**

服务器代码进行压缩，减小服务器到用户端的传输文件大小。

体现在浏览器中是，dist下的文件作为服务器的文件传到浏览器，查看network中的main.js，响应头中有content-Encoding: gzip

```js
devServer: {
	compress: true,
}
```

**port、headers、https、host**

设置端口号、响应头、https、同一个局域网上的其他人也能访问该服务器

```
devServer: {
	port: 3000,
	headers: {
		'X-Access-Token': 'abc123'
	},
	https: true,            			// 请求https://localhost:3000，默认自签名证书，浏览器会显示不安全连接
	http2: true,            			// 自带https
	host: '0.0.0.0',        // 控制台输出的url，同一个局域网的都能访问
}
```

**proxy**

跨域代理

```js
// node开启一个服务器端口 9000, 新建server文件
const http = require('http')
const app = http.createServer((req, res) => {
  if(req.url === '/hello'){
    res.send('hello node')
  }
})
app.listen(9000, 'localhost', () => {
  log('开始监听localhost:9000')
})

// 在app.js中请求该服务器
fetch('/api/hello')
.then(response => response.txt())
.then(result => log(result))

// dist的服务端口是3000, 开启代理
module.exports = {
  devServer: {
    port: 3000,
    proxy: {
      '/api':{                                  // 对所有/api开头的请求进行转发
        target: 'http://localhost:9000',
        pathRewrite: {'^/api': ''}              // 前端请求去掉/api
      }
    }
  }
}
```

**historyApiFallback**

单页面应用，router为history时，刷新出现404 -> 跳转到根目录index.html，或者访问某个url报错时，服务器跳转到设置的url

```js
// 根目录
 devServer: {
   historyApiFallback: true
 }
// 设置对应的url
 devServer: {
   historyApiFallback: {
     rewrites: [
       {form: /^\/$/, to: 'views/landing.html'}
     ]
   }
 }
```

### 8️⃣ 模块热替换和热加载

热替换：程序运行中，替换、增删模块时，**不用重新加载页面**；模块变化的部分会被替换，其他部分保持不变，默认为true

热加载：文件更新时，**自动刷新**服务器&页面，默认为true；热替换是热加载的方式之一；

​				不支持HMR的情况仍然有用，但可能会引起页面闪烁和性能问题。

```
devServer: {
	hot: true,              // webpack5自带，4需要安装hot-module-replacement-plugin
	liveReload: true,
}
```

```
// app.js中（dist中是main.js）, 与hot搭配使用
if(module.hot){
	module.hot.accept('/input.js', () => {
			// 当input.js模块发生变化时，的回调函数
	})
}
```

### 9️⃣ eslint

eslint基本配置

```js
module.exports = {
  env: {
    browser: true,               // 
    es2021: true,
  },
  extends: 'eslint:recommended',      // 继承现有的配置（安装时指定的配置）
  parserOptions: {                   // eslint解析器选项，以何种方式解析代码
    ecmaVersion: 2021,							//  ECMAScript版本
    sourceType: 'module', 					//  模块模式解析（有export import），或script解析（无导入导出
  },
  rules: {
    // 自定义规则或内置规则，可以在这里添加
    ”no-console“： “error”，    //使用console,错误是error
    “indent": ["error", 2],    // 缩进2空格，错误error
    “quotes”: ["error", "single"]   //单引号引用，错误error
  },
};
```



```js
// 安装
npm i eslint -D

// 创建eslint配置文件 .eslintrc.json(有很多配置)
npx eslint --init 

// 检查src文件 或 vscode安装eslint插件，标红提示你
npx eslint ./src

// 忽略某个规则
rules: {
	"no-console": 0,
}
```

结合webpack使用

```js
module.exports = {
  devServer: {
    client: {
      overlay: false       // 报错时，页面上不出现蒙层，只有控制台报错
    }
  },
  module: {
	rules: [
		{
			test: /\.(js|jsx)$/,
			exclude: /node-modules/,
			use: ['babel-loader', 'eslint-loader']     //先检查，后编译
		}
	]
}
}
```

#### **git hooks与husky**

不在开发时检查，在提交代码时检验代码规范，用husky检验。

husky是用git hooks实现的，是git的钩子函数。

git hooks

创建.gitignore，**/node_modules   忽略node_modules的提交

```js
// 转到git目录中
cd .git  

// 查看git文件的隐藏文件
ls -la

// 进入到git的hooks文件中
cd hooks
ls -la       // 有很多.example结尾的文件，是执行git命令前后自动执行的命令
// pre-commit.sample是commit之前执行的命令
// cat pre-commit.sample，提示需要创建新的文件来执行自定义commit前的命令

touch pre-commit
ls -la // 多了pre.commit文件
vim  -la (i输入)
echo pre-commit (esc :wq)
chmod +x ./pre-commit  //给pre-commit加可执行权限，git在commit前会执行

// 回到项目根目录
cd ../
cd ../ 
修改一些文件代码后
git add .
git commit -m "修改了pre-commit"
// 控制台打印pre-commit，执行了pre-commit 
 
// 在pre-commit中执行eslint检查
cd .git
cd hooks
vim pre-commit (i进入编辑)
npx eslint ./src (esc :wq)

// 提交前会检查src的eslint，相当于控制台执行npx eslint ./src
cd ..
修改代码
git add .
git commit -m "xxx" (执行eslint)
```

检查eslint的代码放在.git中不行，每个人的都不一样，且无法提交

配置放在项目目录下 .mygithooks（.开头，隐藏文件）的pre-commit下

```
// 自定义git配置目录
// Git 将会在项目根目录下的 .mygithooks 目录中寻找 Git 所有钩子脚本，而不再使用默认的 .git/hooks 目录。
git config core.hooksPath .mygithooks     （相当于在config中添加了一句hooksPath = .mygithooks）

// 查看git配置，是否添加成功
cd .git
cat config    （config里是配置文件的路径，里面有hooksPath = .mygithooks）

// 给.mygithooks添加执行权限
cd ..  (回到项目根目录)
chmod +x .mygithooks/pre-commit   
```

#### **husky使用**

1. Install 

```
npm install husky -D
```

2. Git hooks生效

```
npx husky install
```

3. package.json添加

```
"scripts": {
    "prepare": "husky install"      // 命令执行前安装husky
}
```

4. 项目根目录下创建 .husky/pre-commit文件

```
npx husky add .husky/pre-commit
或者手动创建
// pre-commit文件中写
npx eslint ./src

// commit时会执行husky，若报错无权限，添加权限
cd ..   // 回到项目根目录
chmod +x .husky/pre-commit    
```

### 🔟 模块与解析

![webpack模块化](/Users/didi/Desktop/note/img/webpack模块化.jpg)

webpack模块：能够在webpack中成功导入的模块（任何形式，不限于js,css）

<u>除了webpack本身支持以上的模块化（原生），还可以通过loader处理后支持的模块（less、json、ts等）</u>

![webpack解析原理](/Users/didi/Desktop/note/img/webpack解析原理.png)

1、模块解析原理：把各种文件通过 **内置解析方法+loader** 得到模块化的文件；

2、每一个webpack打包都会创建一个**compiler对象**，包含打包状态：打包前、打包中、打包成功/失败；compiler对象内置的**解析器resolvers**(resolvers是基于enhanced-resolve包实现的）会解析所有模块的解析；

3、**enhanced-resolve**：解析路径+实现tree shaking

4、webpack可以解析三种路径

- 绝对路径： '/开头'，如'/src/...'   从项目根目录开始
- 相对路径：''./开头'，如'./src/...'   相对当前文件
- 模块路径：在node_modules中查找，如 import _ from 'lodash'

5、 别名

```js
const path = require('path')
module.exports = {
	resolve: {
		alias: {
			'@': path.resolve(__dirname, './src')     // 相对路径
		} 
	}
}
```

6、相同文件名，扩展名优先级

```js
// 有math.json, math.js, math.vue三个文件
// 默认是js json jsx ts
// 路径只写到文件夹，自动引入文件夹下的index文件
import math from '@/components/math'   // 没写拓展名，引入优先级json js vue

resolve: {
		extension: ['.json', '.js', '.vue']
}
```

7、通过cdn引入包，以此减小打包后的体积

手动

```js
// index.html中手动引入
<script src="https://..."></script>

// webpack.config.js中
export.modules = {
  externals: {
    // key 是模块名，value 是window全局变量名
    // key和js中引入的名字对应
    jquery: 'jQuery'   或'$'
  }
}

// main.js中
import $ from 'jquery'
```

自动

```js
// webpack.config.js中
export.modules = {
  externalsType: 'script',      		// 在html中以什么方式引入
  externals: {
    jquery: [
      'https://...',								// 连接cdn，不用在html中引入了
      'jQuery'   										// 或'$'
    ]
  }
}
```





# vue-i18n

准备工作

```js
准备两个翻译文件
// zh.js
export default {
  main:{
    message:"消息",
    display:"展示"
  }
};


// en.js
export default {
  main:{
    message:"message",
    display:"display"
  }
}
```

```js
实例化
import zh from "./zn"
import en from "./en"
import Vue from 'vue'
import VueI18n from 'vue-i18n'
 
Vue.use(VueI18n)

const i18n = new VueI18n({
  locale: "zh",   // 初始化中文
  messages: {
    "zh":zh,
    "en":en
  }
}); 
```

```js
npm install vue-i18n

new Vue({
  el: '#app',
  router,
  store,
  i18n,   // 注入，不能缺少
  render: h => h(App)
})
```

使用

```html
<p>{{ $t("main.message") }}</p>   ->    消息
```

修改语言

```js
this.$i18n.locale = 'en'   ->   自动更新为英文
```

![image-20231027114446989](/Users/didi/Library/Application Support/typora-user-images/image-20231027114446989.png)





# git

## 概念

集中式版本控制：版本记录放在中央服务器上，上传下载极慢，？工作必须联网，中央服务器坏了无法工作

分布式版本控制：版本记录放在每个电脑上，一个服务器用于交换各个电脑上的修改，工作不必联网？



仓库 = 文件 + 修改记录（修改一次，产生一个版本）

git管理的文件只能是文本文件，包括txt、代码，二进制文件音频、视频和word文档无法知道其具体改变

git init后的文件夹多了.git文件



## 版本回退

git status：在哪个分支 + 修改了哪些文件但还未git add

git diff + 文件名：当前文件（还未commit)和上一次commit比做了哪些修改；

git diff HEAD -- readme.txt：工作区的readme和版本库HEAD指向的版本的区别

git log：查看修改的历史记录 commit id + 作者 + 时间 + commit注释

如有三次提交 wrote a readme file、add distributed、append GPL

```
$ git log
commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:06:15 2018 +0800

    append GPL

commit e475afc93c209a690c39c13a46716e8fa000c366
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:03:36 2018 +0800

    add distributed

commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 20:59:18 2018 +0800

    wrote a readme file
```

git log --pretty==online : 只有commit id + 注释

```
$ git log --pretty=oneline
1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
e475afc93c209a690c39c13a46716e8fa000c366 add distributed
eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file
```

HEAD：`HEAD`表示当前版本，也就是最新的提交`1094adb...`，上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，往上100个版本写成`HEAD~100`。

<img src="/Users/didi/Desktop/note/img/gitHEAD.jpg" alt="gitHEAD" style="zoom:50%;" />

git reset：回退到某个版本

​	git log：查看commit历史，会被git reset --hard删除历史

```
// 回退到上一个commit，hard会清清除最近一次提交记录
git reset --hard HEAD^ 
git log   // 三次提交剩两次 wrote a readme file、add distributed

// 回到某个版本
git reset --hard 1094ab (commit id)
回到append GPL这个版本，且输入的id只要前几位就可以，git会自动去找
```

git reflog：查看命令历史。查看每一次命令（commit reset之类的，不包括查看类命令git log）

显示版本号 + commit注释/具体操作

```
$ git reflog
e475afc HEAD@{1}: reset: moving to HEAD^
1094adb (HEAD -> master) HEAD@{2}: commit: append GPL
e475afc HEAD@{3}: commit: add distributed
eaadf4e HEAD@{4}: commit (initial): wrote a readme file
```

git revert



## 工作区、暂存区

工作区：目录

版本库repository：.git文件夹。版本库里有暂存区、master、HEAD等(master为自动创建，HEAD指向master）。

多次修改后git add，修改记录存到暂存区；

git commit多次修改提交到当前分支，缓存区会清空。

<img src="/Users/didi/Desktop/note/img/git暂存区.jpg" alt="git暂存区" style="zoom:50%;" />

git reset HEAD readme.txt：取消并清空暂存区所有修改

git checkout -- 文件名readme.txt：取消工作区所有修改

**删除版本库里的文件**：

```
手动删除文件
git rm test.txt
git commit -m "remove test.txt"
```

**一键还原**到版本库的文件：用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”

```
git checkout -- test.txt
```



一台电脑对应一个ssh，github可以添加多个ssh，实现多台电脑都可以添加代码到远程；

github仓库都是公开可读不可改的，除非交钱变为私有。



**远端空仓库上添加项目**：

添加ssh后，连接远程仓库

如果不是空仓库，直接把连接远程仓库就行，不用push

如果已经连接了另一个仓库，可以断开后再连接这个仓库

```
git remote add origin git@github.com:michaelliao/learngit.git
```

推送+关联master分支-u

```
git push -u origin master
```

**删除远程库并断开链接**

```
git remote rm origin
```

clone时，ssh速度最快，https速度最慢且每次推送都必须输入口令



一个本地仓库，关联两个远程库

1、删除远程连接（`git remote -v `可以查看远程库有几个)

```
git remote rm origin
```

2、连接仓库1

```
git remote add github git@github.com:michaelliao/learngit.git
```

3、连接仓库2

```
git remote add gitee git@gitee.com:michaelliao/learngit.git
```

连接了两个

```
git remote -v
gitee	git@gitee.com:liaoxuefeng/learngit.git (fetch)
gitee	git@gitee.com:liaoxuefeng/learngit.git (push)
github	git@github.com:michaelliao/learngit.git (fetch)
github	git@github.com:michaelliao/learngit.git (push)
```

push

```
git push github master
git push gitee master
```

<img src="/Users/didi/Desktop/note/img/git同时连接两个远程库.jpg" alt="git同时连接两个远程库" style="zoom:50%;" />

## 分支

每次修改串成一条线 = master，master指向最新的修改，HEAD指向当前分支

git**创建、切换、合并、删除分支**都很快

1、只有master分支时（三次修改）：

<img src="/Users/didi/Desktop/note/img/git只有master.jpg" alt="git只有master" style="zoom:50%;" />

2、新建分支dev，当前分支为dev，HEAD指向dev

```
// 相当于git branch dev + git checkout dev,创建并切换
git checkout -b dev 

// ⚠️撤销该文件的所有工作区修改
git checkout -- <file>

// 还可以用switch
git switch -c dev
git switch dev   直接切换
```

<img src="/Users/didi/Desktop/note/img/git新建分支.jpg" alt="git新建分支" style="zoom:50%;" />

3、在新分支上提交修改

<img src="/Users/didi/Desktop/note/img/git在新分支commit.jpg" alt="git在新分支commit" style="zoom:50%;" />

4、合并分支（只有一个分支修改了，是快速合并`Fast forward`）

merge两个分支会产生新的commit，表明该次合并操作

```
// 合并指定分支和当前分支HEAD
git merge dev
```

<img src="/Users/didi/Desktop/note/img/git合并分支.jpg" alt="git合并分支" style="zoom:50%;" />

合并时两个分支有冲突

<img src="/Users/didi/Desktop/note/img/git分支冲突.jpg" alt="git分支冲突" style="zoom:50%;" />

解决冲突后合并，假如保留master的改变

<img src="/Users/didi/Desktop/note/img/git解决冲突后合并分支.jpg" alt="git解决冲突后合并分支" style="zoom:50%;" />

强制非快速合并（快速合并后，删除dev分支，看不出来master时合并过的，非快速合并会在合并时生成一次commit）

```
// --no-ff 是非快速合并，-m是commit注释
git merge --no-ff -m "merge with no-ff" dev
```

<img src="/Users/didi/Desktop/note/img/git非快速合并.jpg" alt="git非快速合并" style="zoom:50%;" />

5、删除分支

```
git branch -d dev
```

<img src="/Users/didi/Desktop/note/img/git删除分支.jpg" alt="git删除分支" style="zoom:50%;" />

当分支未与主分支merge过，删除分支后分支的修改都会丢失，因此删不掉，可以强制删除分支

```
git branch -D dev
```



总结：

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`



### **bug分支**

场景：未commit当前分支修改，无法切换其他分支

1. branch01工作区、暂存区未保存的改动存档 `git stash`

   查看有哪些存档 `git stash list`

   ```
   $ git stash list
   stash@{0}: WIP on dev: f52c633 add merge
   ```

2. 新建bug分支并提交修改、合并分支后

   ```
   git add .
   git commit -m "xxx"   // 得到一个commit id  4c805e2
   	[bugbranch 4c805e2] fix bug 101
   git checkout master   // 一般在父分支进行合并
   git merge  --no-ff -m "merged bug fix" bugbranch
   ```

3. 切回branch01，读档 

```
1、读档同时删除档案记录 
git stash pop
// 查看档案，没有任何记录
git stash list

2、读档同时保留档案记录
git stash apply
// 恢复指定的stash
git stash apply stash@{0}
```

4. branch01的bug也需要修复`git cherry-pick <commit id>`

   ```
   // 把修复的动作，即commit bug放到branch01上， 
   git cherry-pick 4c805e2
   ```



**创建本地分支**（远端有，本地无）

```
// 此时git branch不会显示本地无，远端有的分支
git checkout -b dev origin/dev
```

**git pull失败**，提示`no tracking information`

没有说明远程哪个分支要pull到本地哪个分支

```
git branch --set-upstream-to=origin/dev dev
```

git pull = git fetch + git merge

git pull -r = git pull -rebase = git fetch + git rebase

**git rebase**

- 有分支master、dev，master、dev各自commit了一些修改后(dev commit1, master commit2, dev commit3)，在当前分支master执行git rebase，master的所有修改commit都会根据修改时间转移/穿插进dev分支，master分支的修改会被删除，相当于合并成了一个分支，合并后的commit顺序是提交commit时间顺序 (dev commit1, master commit2, dev commit3)
- 如果是merge，顺序是以分支为单位，在master merge以后（dev commit1, dev commit3, master commit2）
- rebase好处是把commit历史变成直线



## Tag

**定义**

每添加一个新功能就在版本库中打一个tag标签，标签也是版本库的一个快照，标签不能移动。

Git有commit，为什么还要引入tag？

tag就是一个让人容易记住的有意义的名字，它跟某个commit绑在一起。（commit id: 6a5819e...; tag: v1.2)



**本地创建**

当前分支最近一次commit打标签

```
//  标签名为v1.0
git tag v1.0
```

为指定commit打标签

```
git tag v0.9 f52c633   // commit id
```

带有说明的标签，用`-a`指定标签名，`-m`指定说明文字

```
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```

查看所有tag，tag按照数字顺序排序

```
$ git tag
v0.9
v1.0
```

⚠️标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。

**本地删除**

```
$ git tag -d v0.1
```

**标签push到远端**

```
// 某个标签
$ git push origin v1.0
// 所有标签
$ git push origin --tags
```

**删除远端标签**

```
// 先本地删
$ git tag -d v0.9
// 删远端
$ git push origin :refs/tags/v0.9
```

冒号`:`前面是本地的分支名/标签/对象，`:`后是远端分支名/tag/对象，把空推送给远端，从而高效地删除远端的tag



## GitHub

Fork: 在自己的账号下克隆仓库

在自己账号下clone项目，才能push`git clone git@github.com:soniatang/bootstrap.git`

可以推送pull request给官方仓库来贡献代码





参考资料

https://www.liaoxuefeng.com/wiki/896043488029600/900003767775424

https://blog.csdn.net/michaelshare/article/details/79108233（rebase和merge区别）





