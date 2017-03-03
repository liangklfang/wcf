### 1.该工具的三种打包模式

首先必须说明一下，该工具是基于webpack2的，所以很多配置都是需要遵守webpack2规范的。如果需要安装，直接运行下面的命令就可以了。

```js
npm install -g webpackcc//同时必须注意，我们局部安装的优先级要高于全局安装的
```

#### 1.1 webpack-dev-server模式

这种模式你只要在wcf后添加devServer参数，表明我们的文件应该使用compiler.outputFileSystem=MemoryFileSystem来完成。此时不会在output.path下产生我们的文件，而是直接从内存中获取，结合URL映射的方式。同时该模式会自动在output.path路径下通过html-webpack-plugin产生一个html(内存中不可见,同时需要加上--dev表明是开发模式),并自动加载我们的所有chunk.

```js
 wcf --devServer --dev
 //此时打开localhost:8080就会看到我们使用test/index.html作为template的页面
```

#### 1.2 webpack本身的watch模式

这种模式使用webpack自己的watch方法来完成，监听package.json中entry配置的文件的变化。你需要添加--watch --dev。如下:

```js
wcf --watch --dev
```

该模式除了会监听entry文件的变化。当我们自定义的webpack.config.js(通过--config传入)文件内容变化的时候会自动退出编译，要求用户重启!

#### 1.3 webpack普通模式

此时不会监听文件的变化，只是完成webpack的一次编译然后退出！

```js
wcf --dev
```

### 2.该工具的配置参数

注意，该工具的所有的配置都是基于webpack的，各个参数的意义和webpack是一致的。

#### 2.1 shell参数

--version

表示我们的版本号

-w/--watch

表示是否启动webpack的watch模式，参数值可以是一个数字，默认是200ms。

-h/--hash

表示文件名是否应该包含hash值，注意这里如果传入这个参数那么文件名都会包含chunkhash而不是hash。

--dll <dllWebpackConfigFile>

此时你需要输入一个webpack.dll.js,通过这个文件我们产生一个json文件用于DllReferencePlugin

-m/--manifest <manifest.json>

此时你需要传入一个json文件给DllReferencePlugin，此时我们会在自动添加DllReferencePlugin

--publicPath <publicPath>

表示我们webpack的publicPath参数

--devtool <devtool>

用于指定sourceMap格式，默认是"cheap-source-map"

--stj <filename>

是否在output.path路径下产生stats.json文件，该文件作用可以[参见这里](https://github.com/liangklfangl/commonchunkplugin-source-code)

--dev

是否是开发模式，如果是我们会添加很多开发模式下才会用到的Plugin

--devServer

因为该工具集成了webpack-dev-server的打包模式，可以使用这个参数开启。

--config <customConfigFile>

让使用者自己指定配置文件，配置文件内容会通过[webpack-merge](https://github.com/survivejs/webpack-merge)进行合并。

#### 2.2 webpack默认参数

<cod>entry:</cod>

我们通过在package.json中配置entry字段来表示入口文件，比如：

```json
 "entry": {
    "index": "./test/index.js"
  }
```

这样就表示我们会将test目录下的index.js作为入口文件来打包，你也可以配置多个入口文件，其都会打包到output.path对应的目录下。当然，你也可以通过上面shell配置的config文件来更新或者覆盖入口文件。覆盖模式采用的就是上面说的webpack-merge。


<code>output.path:</code>

我们默认的打包输出路径是process.cwd()/dest，你可以通过我们的--config参数来覆盖。


<code>loaders:</code>

使用url-loader来加载png|jpg|jpeg|gif图片，小于10kb的文件将使用DataUrl方式内联：

```js
{
  test: /\.(png|jpg|jpeg|gif)(\?v=\d+\.\d+\.\d+)?$/i,
  use: {
     loader:require.resolve('url-loader'),
     //If the file is greater than the limit (in bytes) the file-loader is used and all query parameters are passed to it.
     //smaller than 10kb will use dataURL
     options:{
      limit : 10000
     }
  }
 }
```

json文件采用json-loader加载:

```js
{ 
   test: /\.json$/, 
   loader:require.resolve('json-loader')
}
```

html文件采用html-loader来加载：

```js
{ 
    test: /\.html?$/, 
    use:{
      loader: require.resolve('html-loader'),
      options:{
      }
    }
  }
```

sass文件采用如下三个loader顺次加载：

```js
{
    test: /\.scss$/,
    loaders: ["style-loader", "css-loader", "sass-loader"]
}
```

当然，你可以通过[getWebpackDefaultConfig.js](https://github.com/liangklfangl/wcf/blob/master/src/getWebpackDefaultConfig.js)来查看更多的loader信息。其中内置了很多的功能，包括css module,压缩css,集成autoPrefixer,precss，直接import我们的css文件等等。

注意：上面任何内置的参数都是可以通过config(shell配置)文件来替换的！


### 3.内置plugins

为了保证开发环境的效率，在使用wcf的时候建议传入--dev表明是在开发环境中，这时候wcf会安装一些仅仅在开发环境使用的包。

开发环境独有的plugin：

```js
HotModuleReplacementPlugin
HtmlWebpackPlugin
//在output目录下产生一个index.html，但是该文件是在内存中的
```

生产环境独有的plugin：

```js
UglifyJsPlugin
ImageminPlugin
```

共有的包：

```js
CommonsChunkPlugin
ExtractTextPlugin
StatsPlugin//shell参数传入--stj
DllPluginDync//shell传入manifest
```


### 4.集成HMR热加载

在webpack-dev-server模式下，我们是可以开启HMR的。此时你只需要在你shell命令中传入devServer就可以了，当然前提是你的模块本身是支持HMR的。关于HMR的相关内容你可以参考[这篇文章](https://github.com/liangklfangl/webpack-hmr)。注意，我们的devServer是如下的配置:

```js
 devServer:{
      publicPath:'/',
      open :true,
      contentBase:false
    }
```

此时你运行wcf --devServer就会发现会自动打开页面，如果你不需要该功能可以通过自定义配置文件来覆盖它！

当然，在我们wcf中，你只要运行"wcf --devServer"此时打开html页面，你修改test目录下的任何文件就可以看到效果是及时显示的，这就是一个HMR的例子。原因在于我们在package.json中采用的就是配置test/index.js作为入口文件：

```js
 "entry": {
    "index": "./test/index.js"
  }
```

### 5.说明

(1)我们的ExtractTextPlugin采用的是contentHash,而不是chunkHash,原因可以阅读[Webpack中hash与chunkhash的区别，以及js与css的hash指纹解耦方案](http://www.cnblogs.com/ihardcoder/p/5623411.html)