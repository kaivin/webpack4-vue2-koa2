# 配置 koa 服务

本项目是基于 [vue2-webpack4][1] 开发的，所以这里可需要先克隆该项目到本地，`npm i` 下载所有插件，然后开始本项目的开发

安装 `koa`

```
npm i koa -D
```

先用 `koa` 开启一个服务，监听 `app.config.js` 配置的端口号

新建 `config/koa-server.js` 文件:

```
const Koa = require('koa')

const appConfig = require('./../app.config')

const app = new Koa()
const uri = 'http://' + appConfig.appIp + ':' + appConfig.appPort

const server = app.listen(appConfig.appPort,appConfig.appIp,() => {
    console.log('Example app listening on ' + uri + '\n');
})

process.on('SIGTERM', () => {
    console.log('Stopping dev server')
    server.close(() => {
        process.exit(0)
    })
})
```

服务已经开启，但是还未和 `webpack` 组合起来，使用该服务的话，那么我们之前配置 `webpack` 开发服务就不需要了，这里需要用到两个新的插件， `kao` 下结合 `webpack` 起服务以及热加载的插件:

```
npm i koa-webpack-dev-middleware koa-webpack-hot-middleware -D
```

`middleware` 中间件，`koa` 使用过程中，需要加载很多其他功能，这些搭配的功能，在实例化 `koa` 之后，以及 `koa` 监听端口号之前的操作，都可以叫做中间件操作

**这里需要注意的是`webpack-dev-middleware`和`webpack-dev-middleware`两个插件是针对`express`做的插件，这里不是使用，所以下载的是针对`koa`的这两个插件**

修改 `koa-server.js`:

```
const Koa = require('koa')
const convert = require('koa-convert')

const open = require('opn')
const webpack = require('webpack')
const webpackDevMiddleware = require('koa-webpack-dev-middleware')
const webpackHotMiddleware = require('koa-webpack-hot-middleware')

const appConfig = require('./../app.config')
const config = require('./webpack/webpack.dev.config')
const clientCompiler = webpack(config)

const app = new Koa()
const uri = 'http://' + appConfig.appIp + ':' + appConfig.appPort

const devMiddleware = webpackDevMiddleware(clientCompiler, {
    publicPath: config.output.publicPath,
    headers: { 'Access-Control-Allow-Origin': '*' },
    stats: {
      colors: true,
    },
    noInfo: false,
    watchOptions: {
      aggregateTimeout: 300,
      poll: true
    },
})

// 中间件,一组async函数，generator函数需要convert转换
const middleWares = [
    // webpack开发中间件
    convert(devMiddleware),
    // webpack热替换中间件
    convert(webpackHotMiddleware(clientCompiler)),
]

middleWares.forEach((middleware) => {
    if (!middleware) {
      return
    }
    app.use(middleware)
})
  
console.log('> Starting dev server...')
  
devMiddleware.waitUntilValid(() => {
    console.log('> Listening at ' + uri + '\n')
    open(uri)
})

// 错误处理
app.on('error', (err) => {
    console.error('Server error: \n%s\n%s ', err.stack || '')
})

const server = app.listen(appConfig.appPort,appConfig.appIp,() => {
    console.log('Example app listening on ' + uri + '\n');
})

process.on('SIGTERM', () => {
    console.log('Stopping dev server')
    devMiddleware.close()
    server.close(() => {
        process.exit(0)
    })
})
```

然后删除`config/webpack/webpack.dev.config.js`文件中的本地服务相关代码:

```
config.devServer = {
    port: appConfig.appPort,
    contentBase: path.resolve(__dirname, '../../dist'),
    historyApiFallback: true,
    host: appConfig.appIp,
    overlay:true,
    hot:true,
    inline:true,
    after(){
        open(`http://${this.host}:${this.port}`)
        .then(() => {
            console.log(chalk.cyan(`http://${this.host}:${this.port} 已成功打开`));
        })
        .catch(err => {
            console.log(chalk.red(err));
        });
    }
}

```

并在该文件中新增一下代码：

```
Object.keys(config.entry).forEach(function (name) {
    config.entry[name] = ['webpack-hot-middleware/client?path=/__webpack_hmr&timeout=2000&reload=true'].concat(config.entry[name])
})
```

最后修改`package.json`中的 `start` 命令：

```
"start": "node config/koa-server.js",
```

执行命令 `npm start` 查看效果，一切还是能正常显示的



[1]:https://github.com/kaivin/vue2-webpack4 "vue2-webpack4"