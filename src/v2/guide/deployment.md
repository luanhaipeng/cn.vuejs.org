---
title: 生产环境部署
type: guide
order: 20
---

## 利用生产状态

开发时，Vue 会提供很多警告来帮你解决常见的错误与陷阱。生产时，这些警告语句却没有用，反而会增加你的载荷量。再次，有些警告检查有小的运行时费用，生产时可以避免的。

### 不用打包工具

如果用 Vue 完整独立版本（直接用 `<script>` 元素引入 Vue），生产时应该用精简版本（`vue.min.js`)。请查看[安装指导](installation.html#独立版本)，附有开发与精简版本。

### 用打包工具

如果用 Webpack 或 Browserify 类似的打包工具时，生产状态会在 Vue 源码中由 `process.env.NODE_ENV` 决定，默认在开发状态。Webpack 与 Browserify 两个打包工具都提供方法来覆盖此变量并使用生产状态，警告语句也会被精简掉。每一个 `vue-cli` 模版有预先配置好的打包工具，但了解怎样配置会更好。

#### Webpack

使用 Webpack 的 [DefinePlugin](http://webpack.github.io/docs/list-of-plugins.html#defineplugin) 来指定生产环境，以便在压缩时可以让 UglifyJS 自动删除代码块内的警告语句。例如配置：

``` js
var webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ]
}
```

#### Browserify

- 运行打包命令，设置 `NODE_ENV` 为 `"production"`。等于告诉 `vueify` 避免引入热重载和开发相关代码。
- 使用一个全局 [envify](https://github.com/hughsk/envify) 转换你的 bundle 文件。这可以精简掉包含在 Vue 源码中所有环境变量条件相关代码块内的警告语句。例如：


``` bash
NODE_ENV=production browserify -g envify -e main.js | uglifyjs -c -m > build.js
```

- 使用 vueify 中包含的 extract-css 插件，提取样式到单独的css文件。

``` bash
NODE_ENV=production browserify -g envify -p [ vueify/plugins/extract-css -o build.css ] -e main.js | uglifyjs -c -m > build.js
```

## 跟踪运行时错误

如果在组件渲染时出现运行错误，错误将会被传递至全局 `Vue.config.errorHandler` 配置函数（如果已设置）。利用这个钩子函数和错误跟踪服务（如 [Sentry](https://sentry.io)，它为 Vue 提供[官方集成](https://sentry.io/for/vue/)），可能是个不错的主意。

## 提取 CSS

使用[单文件组件](./single-file-components.html)时，`<style>` 标签在开发运行过程中会被动态实时注入。在生产环境中，你可能需要从所有组件中提取样式到单独的 CSS 文件中。有关如何实现的详细信息，请查阅 [vue-loader](http://vue-loader.vuejs.org/en/configurations/extract-css.html) 和 [vueify](https://github.com/vuejs/vueify#css-extraction) 相应文档。

`vue-cli` 已经配置好了官方的 `webpack` 模板。
