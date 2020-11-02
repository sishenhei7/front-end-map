## 概述

今天同事说，nuxt.js的项目好像没有自动加前缀，我花了很长时间查找原因，最后终于发现，原来是没有加**.browserslistrc文件**。。。记录下来，供以后开发时参考，相信对其他人也有用。

### browserslistrc

>Share target browsers between different front-end tools, like Autoprefixer, Stylelint and babel-preset-env

browserslist的作用如上面所示，是**用来定义需要支持的浏览器范围**的，它在很多前端工具中都有作用，比如Autoprefixer、Stylelint和babel-preset-env。

### autoprefixer

[autoprefixer 官网](https://github.com/postcss/autoprefixer)明确指出，这个插件需要使用browserslistrc来工作，而配置browserslistrc有2种方式，一种是使用**.browserslistrc文件**，另一种是直接**在```package.json```里面配置**。
