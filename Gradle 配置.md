# Gradle 配置

```groovy
signingConfigs {
        // local.properties file in the root director
        Properties properties = new Properties()
        properties.load(project.rootProject.file('local.properties').newDataInputStream())
        def keystorePwd = properties.getProperty("password")
        def keystoreAlias = properties.getProperty("alias")
        def keystoreAliasPwd = properties.getProperty("alias_password")
        def storeFilePath = properties.getProperty("path")
        debug {
            storeFile file(storeFilePath)
            storePassword keystorePwd
            keyAlias keystoreAlias
            keyPassword keystoreAliasPwd
        }
        release {
            storeFile file(storeFilePath)
            storePassword keystorePwd
            keyAlias keystoreAlias
            keyPassword keystoreAliasPwd
        }
    }

    buildTypes {
        debug {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.debug
            debuggable true
        }
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            signingConfig signingConfigs.release
            debuggable false
        }
    }

    applicationVariants.all { variant ->
        variant.outputs.each { output ->
            output.outputFileName = "DolphinNews-${defaultConfig.versionName}-${new Date().format("yyyy-MM-dd")}.apk"
        }
    }
```



# signingConfigs多配置，As切换生成指定配置包的方法

**buildTypes{**

​	**debug {**
​            **signingConfig null**

​	**}**

**}**



```
android {
    signingConfigs {
        ay7 {
            storeFile file("../ay7/platform.jks")
            keyAlias "androidplatform"
            storePassword "android"
            keyPassword "android"
        }
        ay2 {
            storeFile file("../ay2/platform.jks")
            keyAlias "platform6125"
            storePassword "android"
            keyPassword "android"
        }
        ay19 {
            storeFile file("../ay19/platform.jks")
            keyAlias "androidplatform"
            storePassword "android"
            keyPassword "android"
        }
        a18 {
            storeFile file("../a18/platform.jks")
            keyAlias="platform6125"
            storePassword="android"
            keyPassword="android"
        }
    }

    namespace 'com.xconnect.cariot'
    compileSdk 36

    defaultConfig {
        applicationId "com.xconnect.cariot"
        minSdk 24
        targetSdk 36
        versionCode 2
        versionName "1.0.0"
        flavorDimensions "car"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"

        ndk {
            //noinspection ChromeOsAbiSupport
            abiFilters 'arm64-v8a', 'armeabi-v7a'
        }
    }
    productFlavors {
        ay7 {
            isDefault = true
            signingConfig signingConfigs.ay7
            dimension "car"
        }
        ay2 {
            signingConfig signingConfigs.ay2
            dimension "car"
        }
        ay19 {
            signingConfig signingConfigs.ay19
            dimension "car"
        }
        a18 {
            signingConfig signingConfigs.a18
            dimension "car"
        }
    }
    buildTypes {
        debug {
            signingConfig null
            minifyEnabled false
            debuggable true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            buildConfigField("Integer", "vehiclePlatform", project.ext.vehicle_platform)
            buildConfigField("String", "vin", project.ext.vin)
        }
        release {
            minifyEnabled true
            debuggable false
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            buildConfigField("Integer", "vehiclePlatform", project.ext.vehicle_platform)
            buildConfigField("String", "vin", project.ext.vin)
        }
    }
```
