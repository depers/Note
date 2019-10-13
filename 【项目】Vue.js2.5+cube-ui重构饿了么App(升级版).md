# Vue.js2.5+cube-ui重构饿了么App(升级版)

## 第一章 课程导学

![1](E:\markdown笔记\笔记图片\18\1.png)

![2](E:\markdown笔记\笔记图片\18\2.png)

![3](E:\markdown笔记\笔记图片\18\3.png)

![4](E:\markdown笔记\笔记图片\18\4.png)

![5](E:\markdown笔记\笔记图片\18\5.png)

![6](E:\markdown笔记\笔记图片\18\6.png)

## 第二章 项目准备工作

### 1.Vue-cli3.0介绍

* 官网：<https://cli.vuejs.org/zh/>

* 创建项目：

  * 输入命令：`vue create vue-sell-cube`

  * 参照下图进行设置：

    ![7](E:\markdown笔记\笔记图片\18\7.png)

  * 然后输入：`cd vue-sell-cube`、`npm run serve`运行项目

### 2.目录介绍&cube-ui安装

* 目录介绍

  ```
  node_modules		--通过npm安装的相关文件
  public				--入口文件
  src					--源码文件
  .browserslistrc		--浏览器css前缀配置
  .editorconfig		--编辑器的配置
  .eslintrc.js		--eslint配置
  .gitignore			--git忽略配置
  babel.config.js		--babel预设配置
  package-lock.json	--锁定安装时的包的版本号
  package.json		--npm依赖包文件
  postcss.config.js
  README.md
  ```

* cube-ui安装

  ![8](E:\markdown笔记\笔记图片\18\8.png)

### 3.api 接口 mock

* 在项目根目录中添加data.json文件

* 在vue.config.js中添加路由配置信息：

  ```js
  devServer: {
      // 这里的app其实就相当于express()
      // mock数据
      before(app) {
        app.get('/api/seller', function (req, res) {
            res.json({
              errno: 0,
              data: seller
            })
          }),
          app.get('/api/goods', function (req, res) {
            res.json({
              errno: 0,
              data: goods
            })
          }),
          app.get('/api/ratings', function (req, res) {
            res.json({
              errno: 0,
              data: ratings
            })
          })
      }
    }
  ```


## 第三章 头部组件开发（二期）

### 1.目录结构 & header 组件

* @import '~common/stylus/mixin';`包含的知识点

  * `@import`：**@import** [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)[@规则](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule)，用于从其他样式表导入样式规则。

  * `~`：这个路径别名的设置参见vue.config.js

    ```javascript
    const path = require('path')
    
    function resolve(dir) {
      return path.join(__dirname, dir)
    }
    
    chainWebpack(config) {
        config.resolve.alias
          .set('components', resolve('src/components'))
          .set('common', resolve('src/common'))
      }
    ```

* 该小节还介绍了src\components\support-ico\support-ico.vue，src\components\v-header\v-header.vue。

### 2.axios 封装 & 数据获取

* 安装axios：`npm install axios --save`
* 编写src\api\helpers.js
  * then
  * catch



