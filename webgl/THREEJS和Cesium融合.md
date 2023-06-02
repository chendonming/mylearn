# 加载Cesium

首先我们需要让Cesium run起来，以下是从官网copy的run起来的最简易代码
```js
import { Viewer } from "cesium";
import "cesium/Build/Cesium/Widgets/widgets.css";

const viewer = new Viewer("cesiumContainer");
```

现在控制台出现如下报错信息：
```
Unable to determine Cesium base URL automatically
```

这是因为Cesium是一个专用于GIS的三维库，并不是three这样的通用库，所以Cesium会有小组件、静态资源等，需要指定地址。

如果是单纯的web网页，直接指定 `window.CESIUM_BASE_URL=""`, 这里的CESIUM_BASE_URL值看你Cesium文件夹所在位置情况填写

如果是`webpack`, 查看cesium官方的webpack指南, https://github.com/CesiumGS/cesium-webpack-example/blob/main/TUTORIAL.md

如果是`vite`, 安装`vite-plugin-cesium`,然后：
```js
import cesium from 'vite-plugin-cesium'; // 引入插件

export default defineConfig({
  plugins: [vue(), cesium()],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
})
```