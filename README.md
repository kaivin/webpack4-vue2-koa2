# webpack4 + vue2 + koa2

之前已经做过 [webpack4.x][1] ，[vue2-webpack4][2] 两个循序渐进的项目，本项目将在`vue2-webpack4`项目的基础上进一步加进后端服务 `koa` 来完善整个项目


## 实现内容

* koa2 将取代 webpack-dev-server 作为打包服务的部署方案


## 配置步骤

1. [配置koa服务][3] 
2. [优化终端显示(chalk)][4] 

## 命令

1. 安装

```
npm install
```

2. 运行开发环境

```
npm start
```

3. 输出生产环境

```
npm run build
```


[1]:https://github.com/kaivin/webpack4.x "webpack4.x"
[2]:https://github.com/kaivin/vue2-webpack4 "vue2-webpack4"
[3]:https://github.com/kaivin/webpack4-vue2-koa2/blob/master/README/01：配置koa服务.md "配置koa服务"
[4]:https://github.com/kaivin/webpack4-vue2-koa2/blob/master/README/02：优化终端显示(chalk).md "优化终端显示(chalk)"