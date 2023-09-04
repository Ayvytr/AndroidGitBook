# Apk瘦身

## 为什么apk体积增加？

1. 资源增加，尤其是适配多机型的图片
2. 代码量增加
3. 第三方库增加
5. 多平台so库



## 瘦身方法



### 混淆

1. 开启minifyEnable。它的作用不仅是混淆代码，还有压缩优化的功能，没有引用到的代码不会生成在apk中

2. 慎重选择开源库，尽量避免使用体积大的开源库

3. 开启shrinkResources功能，去除无用的resource文件，它需要配合minifyEnable使用，同样存在反射机制引用的问题，这种情况会被误删

4. defaultConfig中使用resConfigs剔除第三方库或者SDK中的资源

   ```
   defaultConfig {
   
   。。。
   
   resConfigs "zh" //表示只使用中文
   
   resConfigs "xxhdpi" // 表示只是用xxhdpi目录下的资源文件
   
   }
   ```


### so库瘦身，去除不必要平台的so支持

```
defaultConfig {

ndk {

abiFilters "armeabi-v7a","x86"

}

}
```



### 使用采用WebP，.9图，vector，xml等多种措施压缩图片/减少图片占用空间

### AndResGurad压缩资源路径和文件名