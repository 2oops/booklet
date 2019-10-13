# Base

1. 本质上，webpack是一个现代的JavaScript应用程序的静态模块打包器（module bundler)，打包时webpack会递归地构建一个依赖关系图，其中包含应用程序需要的每一个模块，然后会将这些模块打包成一个或多个bundle。

   1. 入口`entry`，可指定一个或多个，webpack会找出直接和间接依赖的模块和库

      - `entry`接受字符串，字符串数组和对象语法。对象语法的可扩展性最佳，单页应用和多页应用都可。
      - 使用`CommonsChunkPlugin`为每个页面间的应用程序共享代码创建bundle，由于入口起点增多，多页应用能复用入口起点之间的大量代码/模块，从而收益很大。

   2. `output`更多参数配置：<https://www.webpackjs.com/configuration/output/>

      - 多个入口起点
      - cdn和资源hash

   3. `loader`能让webpack处理非JavaScript文件，因为它本身只能处理js文件。`test`和`use`是说在`import()/require()`语句中被解析为某种文件路径时，使用`loader`转换一下。以下有三种使用loader的方式：

      - 配置[webpack.config.js]

      - 内联：import语句中显式指定loader

        `import Styles from 'style-loader!css-loader?modules!./styles.css';`

      - CLI: shell命令中指定

        `webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'

      loader运行在`Node.js`中，能够执行任何可能的操作；

      可在`package.json`文件中定义一个loader字段；

      可同步和异步；

      loader模块需要导出一个函数。

   4. `plugins`插件执行的是比`loader`范围更广的任务，可以用来处理各种各样的任务。

      1. webpack插件是一个具有`apply`属性的JavaScript对象，`apply`属性会被`webpack compiler`调用，并且该对象可在整个编译生命周期访问。

         ```javascript
         const pluginName = 'ConsoleOnBuildWebpackPlugin'
         class ConsoleOnBuildWebpackPlugin {
             apply(compiler) {
                 complier.hooks.run.tap(pluginName, compilation => {
                     console.log('webpack开始构建');
                 })
             }
         }
         ```

      2. 

   5. `mode`

   6. ```javascript
      const path = require("path");
      const HtmlWebpackPlugin = require('html-webpack-plugin');
      
      module.exports = {
          entry: {
              app: './src/app.js',
              vendor: './src/vendor.js'
          },
          output: {
      		publicPath: '/',
               path: path.resolve(__dirname, 'dist'),//必填
               filename: 'bundle.js'//必填
          },
          module: {
              rules: [
                  {
                      test: /\.vue$/,
                      loader: 'vue-loader',
                      options: {
                          extractCSS: true
                      }
                  },
                  {
                      test: /\.less$/,
                      loader: ['style-loader', 'css-loader', 'less-loader']//注意顺序
                  }
              ]
          },
          plugins: [
              new HtmlWebpackPlugin({template: './src/index.html'})
          ],
          mode: 'production'
      };
      ```

2. webpack配置是标准的`Node.js CommonJs`模块，允许任何技术栈使用webpack。

3. 模块解析：模块会在`resolve.modules`中指定的所有目录内搜索，可使用`resolve.alias`配置选项创建一个别名。

   解析后，解析器将检查路径是否指向文件或目录，如果指向一个文件：

   - 如果有扩展名，则将文件直接打包
   - 否则，将使用`[resolve.extensions]`选项作为文件扩展名去解析。

   如果指向一个文件夹

   - 文件夹中包含`package.json`，则顺序查找`resolve.mainFileds`配置选项中的指定字段，并且 `package.json` 中的第一个这样的字段确定文件路径。
   - `package.json`文件不存在或者该文件的`main`字段没有返回一个有效路径，则顺序查找`resolve.mainFiles`配置选项中的文件名，看是否能在`import/require`目录下匹配到一个存在的文件名。
   - `resolve.extensions`

4. `manifest和runtime`的处理跟缓存有关

   `runtime`包含在模块交互时，连接模块所需的加载和解析逻辑，以及浏览器中已加载模块的连接和懒加载模块的执行逻辑。

   `manifest`是一个数据集合，保留了编译器开始执行、解析和映射应用程序时所有模块的详细要点。

5. 构建目标`target: 'node'`

   ```javascript
   const path = require('path')
   const serverConfig = {
       target: 'node',
       output: {
           path: path.resolve(__dirname, 'dist'),
           filename: 'lib.node.js'
       }
   }
   const clientConfig = {
       target: 'web'// 默认，可省略
       output: {
            path: path.resolve(__dirname, 'dist'),
            filename: 'lib.js'
       }
   }
   module.exports = [serverConfig, clientConfig]
   ```

6. 模块热替换是指在应用程序执行过程中替换、添加或删除模块而无需重新加载整个页面。`devServer: {hot: true}`

   程序代码 =>HMR runtime检查更新 => HMR runtime异步更新 => 通知应用程序代码 => 要求HMR runtime应用更新 => HMR runtime同步更新

***

#### 真实项目配置

参考[webpack：从入门到真实项目配置](<https://juejin.im/post/59bb37fa6fb9a00a554f89d2>)

1. 浏览器不支持`module.exports`

2. `npm i --save-dev @babel/core babel-loader`

3. `babel-preset-env`的作用是告诉babel使用哪种编码规则进行文件处理

   ```javascript
   "babel": {
       "presets": ['@babel/env']
   }
   ```

     `npm i --save-dev @babel/preset-env`，[报错参考](<https://github.com/webpack/webpack/issues/6568>)

4. `url-loader`和`file-loader`两个库处理图片等

5. `css-loader` 和 `style-loader `库。前者可以让 CSS 文件也支持 import，并且会解析 CSS 文件，后者可以将解析出来的 CSS 通过标签的形式插入到 HTML 中，所以后面依赖前者。

6. 使用 `extract-text-webpack-plugin` 插件将 CSS 文件打包为一个单独文件

   `npm i -D extract-text-webpack-plugin@next`

7. 分离代码

8. 抽取共同代码,使用 webpack 自带的插件 `CommonsChunkPlugin`,`webpack4+`删除了该插件，新增了优化后的`SplitChunksPlugin`，[报错参考](<https://blog.csdn.net/yusirxiaer/article/details/82917144>)

9. `clean-webpack-plugin`删除不需要的文件，报错参考[' CleanWebpackPlugin is not a constructor'](<https://www.jianshu.com/p/0e99366ce796>)

10. 每次新增 JS 文件我们都需要手动在 HTML 中新增标签，现在我们可以通过一个插件来自动完成这个功能，即是`html-webpack-plugin`，[github](<https://github.com/johnagan/clean-webpack-plugin#options-and-defaults-optional>)

11. 按需加载代码,`router`

12. 自动刷新,安装`webpack-dev-server`

13. build生产环境代码，添加插件`optimize-css-assets-webpack-plugin`，`UglifyJsPlugin`，[配置参考](<https://juejin.im/post/59bb37fa6fb9a00a554f89d2>)

14. 优化点：

  `vendor` 这种常用的库我们一般可以使用 CDN 的方式外链进来。



