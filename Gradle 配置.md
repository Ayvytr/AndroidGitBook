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

