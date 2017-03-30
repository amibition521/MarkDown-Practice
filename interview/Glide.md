### Glide 

1.特性

* 流式接口
* 缓存
  * 三级缓存：内存 硬盘 网络
  * 原图和缩略图
  * RGB_565
  * 缓存策略：
    * `skipMemoryCache(true)`      `diskCacheStrategy(DiskCacheStrategy.NONE)` 
* 平滑的动画
  * crossFade()、dontAnimate()
* 重设大小
  * override centerCrop FitCenter
* Gif  
* `.Priority`   优先级 (并不是严格遵守的)
* 缩略图 `thumbnail` 
* 回调 `SimpleTarget` 和`ViewTarget`用于自定义视图类