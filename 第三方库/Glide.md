# Glide

## 坑

1. 默认情况下，固定链接图片加载后，加载的是缓存的图片，不是网络最新图片，需要设置缓存策略为不使用本地缓存

2. 图片拉伸问题

   ```
   解决方案：（在我手机上没发现这问题，解决方案不确定是否有效）
   1、取消使用place holder：
   Glide.with(context).load(resId). into(imageView);
   2、使用place holder加上dontAnimate()：
   Glide.with(context).load(resId).placeholder(defaultId).dontAnimate().into(imageView);
   3、使用asBitmap加载：
   Glide.with(context).load(imageUrl).asBitmap().placeholder(defaultId).into(imageView);
   ```

3. Glide加载框架生命周期问题：可能Activity中加载图片需要用applicationContext

4. 可以在application中预先初始化：Glide.get(this)

