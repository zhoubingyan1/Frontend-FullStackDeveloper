####Vue如何根据webpack的打包参数，选择加载不同的模块
https://segmentfault.com/a/1190000014571631


```shell

https://github.com/shuidian/vue-multipage

# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build // 打包全部模块到一个资源包下面，每个模块的入口是module.html文件，静态资源都在static目录中，这种方式可以复用重复的资源
npm run build moduleName1,moduleName2,... // 打包指定模块到一个资源包下面,每个模块的入口是module.html文件，静态资源都在static目录中，这种方式可以复用重复的资源
npm run build-all // 打包所有模块，然后每个模块彼此独立，有几个模块，就产生几个静态资源包，这种方式不会复用重复的资源

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

###背景
在实际的开发过程中，单页应用并不能满足所有的场景。传统单页应用所生成的成果物，在单个系统功能拆分和多个系统灵活组装时并不方便。举个例子，我们在A系统中开发了一个实时视频预览模块和一个gis模块，传统方式打包A系统，我们会生成一个静态资源包，这个包，包含了实时视频预览模块和gis模块在内的所有模块。那么，现在有一个新的系统要做，碰巧需要原来A系统中的gis模块，那么我们只能把A系统打包，然后拿过来，通过引入url的方式，只使用其中的一个模块。这样做的问题很明显，我只需要A系统中的一个模块，但是我要引入整个A系统的资源包，这显然不合理。那么我们的需求就是从这里产生的，如果我们在开发系统A时，能够按模块划分生成多份静态资源包，最终的成果物中，会有多个子目录，每个子目录可独立运行，完成一个业务功能。这样的话，我们有任何系统需要我们开发过的任何模块，都可以直接打包指定的模块，灵活组装。

###场景分析
首先，这种方案不是完全适合任何场景的，在使用时，还需要注意鉴别是否适用于当前业务场景。下面分析一下这种方式的优缺点：

优点：
1、可与其他系统灵活组装
2、各个模块相互不受影响，所以不受框架和开发模式的制约
3、不同模块可以分开部署
4、后期维护风险小，可以持续的、稳定的进行维护（万一哪天vue/react/angular被淘汰了，不会受太大影响，每个模块分别迭代就好）

缺点：
1、各个模块有相互独立的资源包，那么如果有相同的资源引用，不能复用
2、模块的组装要依赖iframe，所以要对浏览器安全设置、cookie共享等问题进行单独处理
3、用iframe来包裹组件，组件所能控制到的范围就是其所在的iframe，当涉及到全屏的应用场景时，会比较麻烦
4、不同组件之间的通信比较麻烦

以上只是分析应用场景，下面重点讲解一下如何实现多模块独立打包。

###目标
vue-cli默认打包方式的成果物与我们修改后生成的成果物结构对比如下：

https://image-static.segmentfault.com/297/967/2979676027-5adef1ff6873d_articlex

https://image-static.segmentfault.com/367/231/3672317858-5adef228bdd49_articlex

上图为默认配置下，打包成果物的目录结构，下图为我们修改配置后，打包成果物的目录结构


###修改webpack配置的详细步骤
第一步:增加build/module-conf.js用来处理获取模块目录的

```shell
    var chalk = require('chalk')
    var glob = require('glob')

    // 获取所有的moduleList
    var moduleList = []
    var moduleSrcArray = glob.sync('./src/modules/*')
    for(var x in moduleSrcArray){
    moduleList.push(moduleSrcArray[x].split('/')[3])
    }
    // 检测是否在输入的参数是否在允许的list中
    var checkModule = function () {
    var module = process.env.MODULE_ENV
    // 检查moduleList是否有重复
    var hash = {}
    var repeatList = []
    for(var l = 0;l < moduleList.length; l++){
        if(hash[moduleList[l]]){
        repeatList.push(moduleList[l])
        }
        hash[moduleList[l]] = true
    }
    if(repeatList.length > 0){
        console.log(chalk.red('moduleList 有重复：'))
        console.log(chalk.red(repeatList.toString()))
        return false
    }
    let result = true
    let illegalParam = ''
    for (let moduleToBuild of module.split(',')) {
        if (moduleList.indexOf(moduleToBuild) === -1) {
        result = false
        illegalParam = moduleToBuild
        break
        }
    }
    if(result === false){
        console.log(chalk.red('参数错误，允许的参数为：'))
        console.log(chalk.green(moduleList.toString()))
        console.log(chalk.yellow(`非法参数：${illegalParam}`))
    }
    return result
    }

    // 获取当前要打包的模块列表
    function getModuleToBuild () {
    let moduleToBuild = []
    if (process.env.NODE_ENV === 'production') {
        /* 部署态，构建要打包的模块列表，如果指定了要打包的模块，那么按照指定的模块配置入口
        *  这里有个特性，即使参数未传，那么获取到的undefined也是字符串类型的，不是undefined类型
        * */
        if (process.env.MODULE_ENV !== 'undefined') {
        moduleToBuild = process.env.MODULE_ENV.split(',')
        } else {
        // 如果未指定要打包的模块，那么打包所有模块
        moduleToBuild = moduleList
        }
    } else {
        // 开发态，获取所有的模块列表
        moduleToBuild = moduleList
    }
    return moduleToBuild
    }

    exports.moduleList = moduleList
    exports.checkModule = checkModule
    exports.getModuleToBuild = getModuleToBuild

```

第二步：增加build/build-all.js用来处理循环执行打包命令
```shell
const path = require('path')
const execFileSync = require('child_process').execFileSync;
const moduleList = require('./module-conf').moduleList || []

const buildFile = path.join(__dirname, 'build.js')

for( const module of moduleList){
  console.log('正在编译:',module)
  // 异步执行构建文件，并传入两个参数，module：当前打包模块，separate：当前打包模式（分开打包）
  execFileSync( 'node', [buildFile, module, 'separate'], {})
}
```

第三步：修改build/build.js增加MODULE_ENV参数，用来记录当前打包的模块名称，增加MODE_ENV参数，用来记录当前打包的模式
```shell

'use strict'
require('./check-versions')()
const chalk = require('chalk')

process.env.NODE_ENV = 'production'
// MODULE_ENV用来记录当前打包的模块名称
process.env.MODULE_ENV = process.argv[2]
// MODE_ENV用来记录当前打包的模式，total代表整体打包（静态资源在同一个目录下，可以复用重复的文件），separate代表分开打包（静态资源按模块名称分别独立打包，不能复用重复的文件）
process.env.MODE_ENV = process.argv[3]

// 如果有传参时，对传入的参数进行检测，如果参数非法，那么停止打包操作
const checkModule = require('./module-conf').checkModule
if (process.env.MODULE_ENV !== 'undefined' && !checkModule()) {
  return
}

const path = require('path')
const ora = require('ora')
const rm = require('rimraf')
const webpack = require('webpack')
const config = require('../config')
const webpackConfig = require('./webpack.prod.conf')

const spinner = ora('building for production...')
spinner.start()

rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  webpack(webpackConfig, (err, stats) => {
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false, // If you are using ts-loader, setting this to true will make TypeScript errors show up during build.
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    if (stats.hasErrors()) {
      console.log(chalk.red('  Build failed with errors.\n'))
      process.exit(1)
    }

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})

``` 

第四步：修改config/index.js的配置，修改打包时的出口目录配置、html入口模板的配置以及静态资源路径配置
```shell
'use strict'
// Template version: 1.3.1
// see http://vuejs-templates.github.io/webpack for documentation.

const path = require('path')

const MODULE = process.env.MODULE_ENV || 'undefined'
// 入口模板路径
const htmlTemplate =  `./src/modules/${MODULE}/index.html`

module.exports = {
  dev: {

    // Paths
    assetsSubDirectory: 'static',
    assetsPublicPath: '/',
    proxyTable: {},

    // Various Dev Server settings
    host: 'localhost', // can be overwritten by process.env.HOST
    port: 8086, // can be overwritten by process.env.PORT, if port is in use, a free one will be determined
    autoOpenBrowser: false,
    errorOverlay: true,
    notifyOnErrors: true,
    poll: false, // https://webpack.js.org/configuration/dev-server/#devserver-watchoptions-

    // Use Eslint Loader?
    // If true, your code will be linted during bundling and
    // linting errors and warnings will be shown in the console.
    useEslint: true,
    // If true, eslint errors and warnings will also be shown in the error overlay
    // in the browser.
    showEslintErrorsInOverlay: false,

    /**
     * Source Maps
     */

    // https://webpack.js.org/configuration/devtool/#development
    devtool: 'cheap-module-eval-source-map',

    // If you have problems debugging vue-files in devtools,
    // set this to false - it *may* help
    // https://vue-loader.vuejs.org/en/options.html#cachebusting
    cacheBusting: true,

    cssSourceMap: true
  },

  build: {
    // Template for index.html
    index: path.resolve(__dirname, '../dist', MODULE, 'index.html'),
    // 加入html入口
    htmlTemplate: htmlTemplate,
    // Paths
    // assetsRoot: path.resolve(__dirname, '../dist', MODULE),
    // 这里判断一下打包的模式，如果是分开打包，要把成果物放到以模块命名的文件夹中
    assetsRoot: process.env.MODE_ENV === 'separate' ? path.resolve(__dirname, '../dist', MODULE) : path.resolve(__dirname, '../dist'),
    assetsSubDirectory: 'static',
    // 这里的路径改成相对路径，原来是assetsPublicPath: '/',
    // assetsPublicPath: '/',
    assetsPublicPath: '',

    /**
     * Source Maps
     */

    productionSourceMap: true,
    // https://webpack.js.org/configuration/devtool/#production
    devtool: '#source-map',

    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    productionGzip: false,
    productionGzipExtensions: ['js', 'css'],

    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  }
}
```

第五步：修改webpack.base.conf.js的入口配置，根据传参，动态配置入口文件
```shell
    entry() {
    // 初始化入口配置
    const entry = {}
    // 所有模块的列表
    const moduleToBuild = require('./module-conf').getModuleToBuild() || []
    // 根据传入的待打包目录名称，构建多入口配置
    for (let module of moduleToBuild) {
      entry[module] = `./src/modules/${module}/index.js`
    }
    return entry
  },
```

第六步：修改webpack.dev.conf.js的配置,增加多入口时webpackHtmlPlugin插件的配置，增加静态资源服务器的配置
```shell
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const path = require('path')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const portfinder = require('portfinder')

const HOST = process.env.HOST
const PORT = process.env.PORT && Number(process.env.PORT)
const moduleList = require('./module-conf').moduleList || []
// 组装多个（有几个module就有几个htmlWebpackPlugin）htmlWebpackPlugin，然后追加到配置中
const htmlWebpackPlugins = []
for (let module of moduleList) {
  htmlWebpackPlugins.push(new HtmlWebpackPlugin({
    filename: `${module}/index.html`,
    template: `./src/modules/${module}/index.html`,
    inject: true,
    chunks: [module]
  }))
}

const devWebpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
  },
  // cheap-module-eval-source-map is faster for development
  devtool: config.dev.devtool,

  // these devServer options should be customized in /config/index.js
  devServer: {
    clientLogLevel: 'warning',
    historyApiFallback: {
      rewrites: [
        { from: /.*/, to: path.posix.join(config.dev.assetsPublicPath, 'index.html') },
      ],
    },
    hot: true,
    contentBase: false, // since we use CopyWebpackPlugin.
    compress: true,
    host: HOST || config.dev.host,
    port: PORT || config.dev.port,
    open: config.dev.autoOpenBrowser,
    overlay: config.dev.errorOverlay
      ? { warnings: false, errors: true }
      : false,
    publicPath: config.dev.assetsPublicPath,
    proxy: config.dev.proxyTable,
    quiet: true, // necessary for FriendlyErrorsPlugin
    watchOptions: {
      poll: config.dev.poll,
    },
    setup(app) {
      // 写个小路由，打开浏览器的时候可以选一个开发路径
      let html = `<html><head><title>调试页面</title>`
      html += `<style>body{margin: 0}.module-menu{float: left;width: 200px;height: 100%;padding: 0 8px;border-right: 1px solid #00ffff;box-sizing: border-box}.module-container{float: left;width: calc(100% - 200px);height: 100%}.module-container iframe{width: 100%;height: 100%}</style>`
      html += `</head><body><div class="module-menu">`
      for(let module of moduleList){
        html += `<a href="/${module}/index.html" target="container">${module.toString()}</a><br>`
      }
      html += `</div>`
      html += `<div class="module-container"><iframe src="/${moduleList[0]}/index.html" name="container" frameborder="0"></iframe></div>`
      html += `</body></html>`
      // let sentHref = ''
      // for(var module in moduleList){
      //   sentHref += '<a href="/'+ moduleList[module] +'/index.html">点我调试模块：'+ moduleList[module].toString() +'</a> <br>'
      // }
      app.get('/moduleList', (req, res, next) => {
        res.send(html)
      })
      // 访问根路径时重定向到moduleList
      app.get('/', (req, res, next) => {
        res.redirect('/moduleList')
      })
    }
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env')
    }),
    new webpack.HotModuleReplacementPlugin(),
    new webpack.NamedModulesPlugin(), // HMR shows correct file names in console on update.
    new webpack.NoEmitOnErrorsPlugin(),
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.dev.assetsSubDirectory,
        ignore: ['.*']
      }
    ]),
    // https://github.com/ampedandwired/html-webpack-plugin
    // new HtmlWebpackPlugin({
    //   filename: 'a/index.html',
    //   template: './src/modules/a/index.html',
    //   inject: true,
    //   chunks: ['a']
    // }),
  ].concat(htmlWebpackPlugins)
})

module.exports = new Promise((resolve, reject) => {
  portfinder.basePort = process.env.PORT || config.dev.port
  portfinder.getPort((err, port) => {
    if (err) {
      reject(err)
    } else {
      // publish the new Port, necessary for e2e tests
      process.env.PORT = port
      // add port to devServer config
      devWebpackConfig.devServer.port = port

      // Add FriendlyErrorsPlugin
      devWebpackConfig.plugins.push(new FriendlyErrorsPlugin({
        compilationSuccessInfo: {
          messages: [`Your application is running here: http://${devWebpackConfig.devServer.host}:${port}`],
        },
        onErrors: config.dev.notifyOnErrors
          ? utils.createNotifierCallback()
          : undefined
      }))

      resolve(devWebpackConfig)
    }
  })
})

```
第七步：修改webpack.prod.conf.js的配置,增加对不同打包模式的处理。
```shell
'use strict'
const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

const env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : require('../config/prod.env')
// 获取所有模块列表
const moduleToBuild = require('./module-conf').getModuleToBuild() || []
// 组装多个（有几个module就有几个htmlWebpackPlugin）htmlWebpackPlugin，然后追加到配置中
const htmlWebpackPlugins = []
// 判断一下是否为分开打包模式
if (process.env.MODE_ENV === 'separate') {
  // 分开打包时是通过重复运行指定模块打包命令实现的，所以每次都是单个html文件，只要配置一个htmlPlugin
  htmlWebpackPlugins.push(new HtmlWebpackPlugin({
    filename: process.env.NODE_ENV === 'testing'
      ? 'index.html'
      : config.build.index,
    // template: 'index.html',
    template: config.build.htmlTemplate,
    inject: true,
    minify: {
      removeComments: true,
      collapseWhitespace: true,
      removeAttributeQuotes: true
      // more options:
      // https://github.com/kangax/html-minifier#options-quick-reference
    },
    // necessary to consistently work with multiple chunks via CommonsChunkPlugin
    chunksSortMode: 'dependency'
  }))
} else {
  // 一起打包时是通过多入口实现的，所以要配置多个htmlPlugin
  for (let module of moduleToBuild) {
    htmlWebpackPlugins.push(new HtmlWebpackPlugin({
      filename: `${module}.html`,
      template: `./src/modules/${module}/index.html`,
      inject: true,
      // 这里要指定把哪些chunks追加到html中，默认会把所有入口的chunks追加到html中，这样是不行的
      chunks: ['vendor', 'manifest', module],
      // filename: process.env.NODE_ENV === 'testing'
      //   ? 'index.html'
      //   : config.build.index,
      // template: 'index.html',
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }))
  }
}

// 获取当前打包的目录名称
const webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true,
      usePostCSS: true
    })
  },
  devtool: config.build.productionSourceMap ? config.build.devtool : false,
  output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap,
      parallel: true
    }),
    // extract css into its own file
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      // Setting the following option to `false` will not extract CSS from codesplit chunks.
      // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
      // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`,
      // increasing file size: https://github.com/vuejs-templates/webpack/issues/1110
      allChunks: true,
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    /*
    new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV === 'testing'
        ? 'index.html'
        : config.build.index,
      // template: 'index.html',
      template: config.build.htmlTemplate,
      inject: true,
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
    */
    // keep module.id stable when vendor modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // enable scope hoisting
    new webpack.optimize.ModuleConcatenationPlugin(),
    // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      minChunks: Infinity
    }),
    // This instance extracts shared chunks from code splitted chunks and bundles them
    // in a separate chunk, similar to the vendor chunk
    // see: https://webpack.js.org/plugins/commons-chunk-plugin/#extra-async-commons-chunk
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'vendor-async',
      children: true,
      minChunks: 3
    }),

    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ].concat(htmlWebpackPlugins)
})

if (config.build.productionGzip) {
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```

第八步：修改package.json，增加npm run build-all指令
```shell
"scripts": {
    "dev": "webpack-dev-server --inline --progress --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "unit": "jest --config test/unit/jest.conf.js --coverage",
    "test": "npm run unit",
    "lint": "eslint --ext .js,.vue src test/unit",
    "build": "node build/build.js",
    "build-all": "node build/build-all.js"
  },
```

####构建指令
构建指令
npm run build // 打包全部模块到一个资源包下面，每个模块的入口是module.html文件，静态资源都在static目录中，这种方式可以复用重复的资源
npm run build moduleName1,moduleName2,... // 打包指定模块到一个资源包下面,每个模块的入口是module.html文件，静态资源都在static目录中，这种方式可以复用重复的资源
npm run build-all // 打包所有模块，然后每个模块彼此独立，有几个模块，就产生几个静态资源包，这种方式不会复用重复的资源

本文是在参考了以下文章的内容之后写的，在原有的基础上做了一些扩展，支持多入口模式
参考资料：https://segmentfault.com/a/11...

###注意事项
