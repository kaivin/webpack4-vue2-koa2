# 优化终端显示插件

项目早起已经下载过该插件 `chalk`

在我们运行 `npm start` 时，有些输出的关键信息，想要着色突出处理，需要用到这个插件

首先修改 `webapck.dev.config.js` 文件`new FriendlyErrorsPlugin`:

```
new FriendlyErrorsPlugin({
    compilationSuccessInfo: {
        messages: [chalk.green(`You application is running here http://${appConfig.appIp}:${appConfig.appPort}`)],
        // notes: ['额外注释']
    }
}),
```

再修改 `koa-server.js` 文件：

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

const chalk = require('chalk');// 改变命令行中输出日志颜色插件

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
  
console.log(chalk.cyan('> Starting dev server...'))
  
devMiddleware.waitUntilValid(() => {
    console.log(chalk.magenta('> Listening at ' + uri + '\n'))
    open(uri)
})

// 错误处理
app.on('error', (err) => {
    console.error(chalk.bold.red('Server error: \n%s\n%s ', err.stack || ''))
})

const server = app.listen(appConfig.appPort,appConfig.appIp,() => {
    console.log(chalk.green('Example app listening on ' + uri + '\n'));
})

process.on('SIGTERM', () => {
    console.log(chalk.red('Stopping dev server'))
    devMiddleware.close()
    server.close(() => {
        process.exit(0)
    })
})
```

执行命令`npm start` 终端中的一些输出日志都已着色