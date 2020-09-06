# 【项目】VueJs饿了么App开发

## 第一章 项目介绍

学习内容：

* Vue.js框架
* Vue-cli脚手架，搭建基本环境
* vue-router官方管理路由
* vue-resource Ajax通讯
* Webpack 构建工具
* es6+eslint estlint是es6的代码风格检查工具

## 第二章 Vue.js介绍

### 1.MVVM原理

![1](E:\markdown笔记\笔记图片\17\1.png)

![2](E:\markdown笔记\笔记图片\17\2.png)

### 2.Vue.js核心思想

* 数据驱动

  ![2](E:\markdown笔记\笔记图片\17\3.png)

  ![2](E:\markdown笔记\笔记图片\17\4.png)

* 组件化

  ![2](E:\markdown笔记\笔记图片\17\5.png)

## 第三章 Vue-cli开启Vuejs项目

### 1.vue-cli介绍

![6](E:\markdown笔记\笔记图片\17\6.png)

### 2. Vue-cli安装及新建项目

* 安装Vue-Cli：`npm install -g @vue/cli`，安装完成后使用`vue --version`测试。

* 新建项目：`vue init webpack sell`，其中webpack是我们新建项目的模板，sell是项目的名称。下面是新建项目的命令：

  ```
  PS E:\demo\front\sell> vue init webpack sell
  
  ? Project name sell
  ? Project description sell app
  ? Author depers <dev_fengxiao@163.com>
  ? Vue build standalone
  ? Install vue-router? Yes
  ? Use ESLint to lint your code? Yes
  ? Pick an ESLint preset Standard
  ? Set up unit tests No
  ? Setup e2e tests with Nightwatch? No
  ```

### 3.项目文件介绍

```
build			--webpack配置相关文件
config			--webpack配置相关文件
node_modules	--通过npm安装的相关文件
src				--存放项目源码
static			--存放第三方静态资源文件，该目录下的.gitkeep文件表示即使该目录为空也可以提交至git仓				  库；
babelrc			--babel配置文件，presets表示预设插件，plugins中可以配置插件，"comments": false编					 译后代码不生成注释
editorconfig	--编辑器的配置
.eslintignore	--忽略语法检查的目录文件
.eslintrc.js	--eslint的配置文件
.gitignore		--git忽略文件配置
.postcssrc.js	--使用配置文件允许你在由 postcss-loader 处理的普通CSS文件和 *.vue 文件中的 CSS 之				   间共享相同的配置，其中postcss-loader一般用于添加兼容性属性前缀
index.html		--html的入口文件
package-lock.json
package.json	--项目依赖的配置文件
README.md		--项目说明文件
```

### 4.webpack配置解析

入口文件为build/webpacl.dev.conf.js

## 第四章 项目实战准备工作

### 1.需求分析

![7](E:\markdown笔记\笔记图片\17\7.png)

### 2.项目资源准备

项目中的resource目录，有关项目的一些图片。

### 3.图标字体的制作

通过[https://icomoon.io](https://icomoon.io/)网站将svg图片制作成css可以使用的图标字体。

### 4.项目目录的设计

```
common			--放一些公共文件
|--fonts		--放svg的字体文件
|--js	
|--stylus		--放svg的css文件(icon.styl)
commponents		--vue组件
|--heander		
|--|heander.vue
router
App.vue
main.js
```

### 5. mock数据（模拟后台数据）

在build\webpack.dev.conf.js文件中编写如下代码：

```javascript
const appData = require('./../data.json')
const seller = appData.seller
const goods = appData.goods
const ratings = appData.ratings

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


## 第五章 项目实战-页面骨架开发

### 1. 组件拆分（上）

* 配置static\css\reset.css文件

* 编辑index.html文件

  ```
  <meta name="viewport"
          content="width=device-width,initial-scale=1.0,maximum-scale=1.0,minimum-scale=1.0,user-scalable=no">
      <title>sell</title>
      <link rel="stylesheet" type="text/css" href="static/css/reset.css">
  ```

* 配置.eslintrc.js文件

### 2.组件拆分（上）

* 在vscode中创建代码片段：

  * 选择：文件>首选项>用户代码片段

  * 新建全局代码片段

  * 编写模板代码：

    注意：把代码片段写在json里。每个代码段都是在一个代码片段名称下定义的，并且有prefix、body和description。prefix是用来触发代码片段的。使用 $1，$2 等指定光标位置，这些数字指定了光标跳转的顺序，$0表示最终光标位置。

    ```javascript
  "vue-template": {
  		"prefix": "vue",
  		"body": [
  		  "<template>",
  		  "  <div class=\"$1\">",
  		  "",
  		  "  </div>",
  		  "</template>",
  		  "",
  		  "<script type=\"text/ecmascript-6\">",
  		  "export default {",
  		  "  name: '$1',",
  		  "  data() { ",
  		  "    return {",
  		  "",
  		  "    }",
  		  "  }",
  		  " }",
  		  "</script>",
  		  "",
  		  "<style lang=\"stylus\" rel=\"stylesheet/stylus\" scoped>",
  		  "  .$1{",
  		  "",
  		  "  }",
  		  "</style>"
  	
  		],
  		"description": "my vue template"
  	  }
  }
    ```

* export与export default的区别

  >`export` 本质上就是规定模块[js文件]的对外接口[属性或方法]
  >`export default` 则是在 `export` 的基础上，为规定模块提供一个默认的对外接口

  * 1.`export default `用于规定模块的默认对外接口

  * 2.`export default` 在同一个模块中只能出现一次

    在一个文件或模块中，export、import可以有多个，export default仅有一个。

  * 3.通过export方式导出，在导入时要加{ }，export default则不需要
  
    ```
    var name="李四";
    export { name }
    //import { name } from "/.a.js" 
    
  可以写成：
    var name="李四";
  export default name
    //import name from "/.a.js" 这里name不需要大括号
    ```

### 3.组件拆分（下）

* [Vue warn]: Do not use built-in or reserved HTML elements as component id: header

  说明我们使用的组件是html的原生标签，我们应该这样写：

  ```javascript
  export default {
    components: {
      'v-header': header
    }
  }
  ```

* 代码中我们使用vue-loader下的postcss包将不同浏览器中css的相关浏览器配置抹平，方便我们开发。

### 4.



