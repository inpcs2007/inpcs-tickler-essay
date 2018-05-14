# 构建自己的vue前端框架结构

## 规划目录结构

以vue-demo为例讲解基于vue的前后端分离的目录结构

```

└── vue-demo
    ├── build
    │   ├── build.js
    │   ├── check-versions.js
    │   ├── logo.png
    │   ├── utils.js
    │   ├── vue-loader.conf.js
    │   ├── webpack.base.conf.js
    │   ├── webpack.dev.conf.js
    │   └── webpack.prod.conf.js
    ├── config
    │   ├── dev.env.js
    │   ├── index.js
    │   ├── prod.env.js
    │   └── test.env.js
    ├── index.html
    ├── node_modules
    ├── src
    │   ├── api
    │   ├── App.vue
    │   ├── assets
    │   │   └── logo.png
    │   ├── main.js
    │   ├── mock
    │   ├── router
    │   │   └── index.js
    │   ├── styles
    │   ├── utils
    │   ├── vendor
    │   └── views
    │       ├── hello
    │       │   └── HelloWorld.vue
    │       └── login
    │           └── login.vue
    ├── static
    └── test
        ├── e2e
        │   ├── custom-assertions
        │   │   └── elementCount.js
        │   ├── nightwatch.conf.js
        │   ├── runner.js
        │   └── specs
        │       └── test.js
        └── unit
            ├── jest.conf.js
            ├── setup.js
            └── specs
                └── HelloWorld.spec.js
```
###
1.去掉[http://localhost:8080/#/]中的【#】
路由是/#/login这个样子的，这里你的项目可以继续保持这样，但不喜欢也可以改变它。这里我们改变路由router的模式，由默认的hash变为history。

```
Vue.use(Router)

export default new Router({
  mode: 'history',
  routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/login',
      name: Login,
      component: Login
    }
  ]
})
```

错误信息：
element-ui/lib/theme-default/index.css in ./src/main.js
解决方式：
element-ui/lib/theme-chalk/index.css
原因：是找不到包
