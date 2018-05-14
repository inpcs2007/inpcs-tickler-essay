# 构建vue基础环境

### 安装vue和element-ui

> 安装vue环境

```
npm install vue

npm install --global vue-cli
```

> 安装vue环境
```
npm install element-ui --save
```

### 通过vue的脚手架安装vue样例工程

```
incps@incpshome:~$ vue init webpack vue-demo

? Project name vue-demo
? Project description A Vue.js project
? Author inpcs <408946001@qq.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Set up unit tests Yes
? Pick a test runner jest
? Setup e2e tests with Nightwatch? Yes
? Should we run `npm install` for you after the project has been created? (recommended) npm

   vue-cli · Generated "vue-demo".


# Installing project dependencies ...
# ========================

npm WARN deprecated socks@1.1.10: If using 2.x branch, please upgrade to at least 2.1.6 to avoid a serious bug with socket data flow and an import issue introduced in 2.1.0

> chromedriver@2.38.3 install /home/incps/vue-demo/node_modules/chromedriver
> node install.js

Downloading https://chromedriver.storage.googleapis.com/2.38/chromedriver_linux64.zip
Saving to /tmp/chromedriver/chromedriver_linux64.zip
Received 781K...
Received 1568K...
Received 2352K...
Received 3136K...
Received 3684K total.
Extracting zip contents
Copying to target path /home/incps/vue-demo/node_modules/chromedriver/lib/chromedriver
Fixing file permissions
Done. ChromeDriver binary available at /home/incps/vue-demo/node_modules/chromedriver/lib/chromedriver/chromedriver

> uglifyjs-webpack-plugin@0.4.6 postinstall /home/incps/vue-demo/node_modules/webpack/node_modules/uglifyjs-webpack-plugin
> node lib/post_install.js

npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.3 (node_modules/fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.3: wanted {"os":"darwin","arch":"any"} (current: {"os":"linux","arch":"x64"})

added 1558 packages in 116.651s


Running eslint --fix to comply with chosen preset rules...
# ========================


> vue-demo@1.0.0 lint /home/incps/vue-demo
> eslint --ext .js,.vue src test/unit test/e2e/specs "--fix"


# Project initialization finished!
# ========================

To get started:

  cd vue-demo
  npm run dev
  
Documentation can be found at https://vuejs-templates.github.io/webpack
```

开始第一个vue之旅吧！