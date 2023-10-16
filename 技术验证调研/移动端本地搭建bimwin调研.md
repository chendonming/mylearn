
# 如何在flutter中搭建http服务器

dart 本身具有 `io`包,  可以创建服务
```dart
import 'dart:io';  
import 'dart:async';
```
```dart
var server = await HttpServer.bind(InternetAddress.LOOPBACK_IP_V4, 8080);
```

或者第三方包`get_server`
```dart
import 'package:get_server/get_server.dart' as gs;
void main() {
  gs.runApp(
    gs.GetServerApp(home: gs.FolderWidget('folderName')),
  );
}
```

资料引用:
- https://stackoverflow.com/questions/63426705/how-to-create-http-server-app-in-flutter
- https://medium.com/@naik.rpsn/http-server-running-on-a-mobile-app-with-flutter-1ef1e717dda1
- https://stackoverflow.com/questions/67219756/how-to-run-a-flutter-web-application-offline
- https://pub.dev/packages/get_server


# 纯web端实现 Service Worker API

它采用 JavaScript 文件的形式，控制关联的页面或者网站，拦截并修改访问和资源请求，细粒度地缓存资源。你可以完全控制应用在特定情形（最常见的情形是网络不可用）下的表现。

要求：出于安全考量，Service worker 只能由 HTTPS 承载，毕竟修改网络请求的能力暴露给[中间人攻击](https://developer.mozilla.org/zh-CN/docs/Glossary/MitM)会非常危险，如果允许访问这些强大的 API，此类攻击将会变得很严重。在 Firefox 浏览器的[用户隐私模式](https://support.mozilla.org/zh-CN/kb/private-browsing-use-firefox-without-history)，Service Worker 不可用。

资料引用:
- https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API