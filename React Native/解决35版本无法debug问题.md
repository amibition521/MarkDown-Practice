解决35版本无法debug问题

### 解决35版本无法debug问题

1.由于35版本无法debug，原因是debug模式下无法生成bundle.现在不知道为啥没有bundle可用。导致升级40版本解决这个问题。

下面是遇到的问题：

* `com.android.ddmlib.installException:Failed to install all`----通过升级node版本来解决
* `Unable to upload some APKS`--- 原因暂时没有找到，网上说可以降低gradle版本来解决，但是试过之后没有解决。
* `Could not get BatchedBridge, make sure your bundle is packaged correctly`---debug模式下无法连接bundle，可以手动生成，但是没有developer munu.

2.下面是一些收获：

* ```
  //生成bundle和map
  "bundle-android": "react-native bundle --platform android --dev true --entry-file index.android.js --bundle-output android/app/src/main/assets/index.android.bundle --sourcemap-output android/app/src/main/assets/index.android.map --assets-dest android/app/src/main/res/"
  ```

* ```
  //连接服务器，及时生成bundle,需要安装curl.exe，将localhost替换成本机ipconfig
  curl "http://localhost:8081/index.android.bundle?platform=android" -o "android/app/src/main/assets/index.android.bundle"
  ```

* `useLibrary 'org.apache.http.legacy`---引入apache网络请求,对gradle版本有要求。

### 40版本遇到的问题以及解决方案

1.图片的写法与所处的位置

问题：

```
Unable to resolve module image!ic_dev_gray from /Users/nieyuxin/tower/xin/feibao/tabbar.js: Module does not exist in the module map or in these directories:
  /Users/nieyuxin/tower/xin/feibao/node_modules

This might be related to https://github.com/facebook/react-native/issues/4968
To resolve try the following:
  1. Clear watchman watches: `watchman watch-del-all`.
  2. Delete the `node_modules` folder: `rm -rf node_modules && npm install`.
  3. Reset packager cache: `rm -fr $TMPDIR/react-*` or `npm start -- --reset-cache`.
```

这个问题出现原因是图片的引用不对。`source = require('image!ic_dev_gray')`这种方式在0.4版本已经被停用，这种方式有几个弊端。

```
放在Android层
1.不能热更新
2.需要重新编译
3.不能跨平台
4.不方便适配
5.不能用目录
6.不能有同名却不同扩展名的图片
```

解决方案：

​	1.采用`source = require('./imgj/ic_dev_gray')`方式加载图片，

2.导入第三方库导致的依赖问题

Error:`more than one library with package name 'com.facebook.react`

解决方案：

第一种：在android/app/build.gradle中添加

```
configurations.all {
    exclude group: 'com.facebook.react', module: 'react-native'
}
```

第二种方案：在android/app/build.gradle中添加

```
compile(project(':react-native-custom-module')) {
    exclude group: 'com.facebook.react', module: 'react-native'
}
```

