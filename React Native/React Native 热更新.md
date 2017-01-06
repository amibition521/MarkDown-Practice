### React Native 热更新

1.相应的bundle命令

```javascript
react-native bundle --platform android --entry-file index.android.js --bundle-output ./bundle/index.android.bundle --assets-dest ./bundle/ --dev false
```

2.图片的替换

图片的来源：

```
//读取res目录下文件
source = require('image!ic_comment')
//在img文件夹下存在ic_comment.png
source = require('./img/ic_comment.png')
```

针对第一种写法，在app升级的时候，无法自动升级；在升级的时候，必须用第二种写法。

source = require('./img/ic_comment.png')，必须确保img文件夹存在，利用bundle 生成相应的drawable-mhdpi文件夹中会出现一个以img_ic_comment命名的png图片，将index.android.bundle和drawable-mhdpi一起打包zip，就可以实现图片的替换。