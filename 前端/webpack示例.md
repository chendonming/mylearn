# webpack示例

## 启动一个纯的html+js+ts的页面 带有热替换功能

版本号:
```json
{
  "webpack": "5.75.0",
  "webpack-cli": "4.10.0",
  "webpack-dev-server": "4.15.1",
  "html-webpack-plugin": "5.5.3"
}
```

```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: './ui/index.ts',  // 入口文件的路径
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    },
    extensions: ['.js', '.ts']
  },
  output: {
    path: path.resolve(__dirname, 'uioutput'),  // 输出目录的绝对路径
    filename: '[name].js'  // 输出文件的名称
  },
  devServer: {
    static: path.join(__dirname, 'uioutput'),  // 服务的根目录
    port: 3000,  // 服务运行的端口号
    hot: true  // 启用热替换
  },
  devtool: "source-map",
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'  // HTML模板文件的路径
    })
  ],
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: {
          loader: 'ts-loader',
        },
        exclude: /node_modules/,
      },
    ]
  }
};
```

## 使打包后的文件分割

默认情况下的打包如下:
```
- main.js
- main.js.map
```

需要的打包如下:
```
- main.js
- main.js.map
- a.js
- a.js.map
- b.js
- b.js.map
...
```

使用到的思路为 动态导入( [Dynamic Imports](https://webpack.js.org/guides/code-splitting/#dynamic-imports) )

配置文件:
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  mode: 'development',
  entry: './ui/index.ts',  // 入口文件的路径
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    },
    extensions: ['.js', '.ts']
  },
  output: {
    path: path.resolve(__dirname, 'uioutput'),  // 输出目录的绝对路径
    filename: '[name].js'  // 输出文件的名称
  },
  devServer: {
    static: path.join(__dirname, 'uioutput'),  // 服务的根目录
    port: 3000,  // 服务运行的端口号
    hot: true  // 启用热替换
  },
  devtool: "source-map",
  plugins: [
    new HtmlWebpackPlugin({
      template: './public/index.html'  // HTML模板文件的路径
    })
  ],
  module: {
    rules: [
      {
        test: /\.ts$/,
        use: {
          loader: 'ts-loader',
        },
        exclude: /node_modules/,
      },
    ]
  }
};
```

主要的入口文件，改为如下模样:
```js
const app = import('app')
const app1 = import('app1)
```

打包后的大概信息如下:
```
- main.js
- main.js.map
- app.js
- app.js.map
- app1.js
- app1.js.map
```