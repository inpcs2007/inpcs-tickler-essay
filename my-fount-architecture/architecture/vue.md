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

###
