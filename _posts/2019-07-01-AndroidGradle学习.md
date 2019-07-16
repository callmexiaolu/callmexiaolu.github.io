

## Gradle学习

#### android中build文件配置

```groovy
apply plugin: 'com.android.application'

apply plugin: 'kotlin-android'

apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 28 //编译工程的sdk
    defaultConfig {
        applicationId "com.example.learn" //生成app的包名
        minSdkVersion 15 //app最低支持操作系统版本
        targetSdkVersion 28 //app基于哪个版本开发的
        versionCode 1 //app内部版本号，开发人员看
        versionName "1.0"//app版本名称，用户看。
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"  //用于配置单元测试使用的Runner
    }
    signingConfigs {
        release {
            keyAlias '翼课帮家长' //签名证书中密钥别名
            keyPassword '147258369' //签名证书中密钥的密码
            storeFile file('key/EkwingParent.jks') //签名证书文件
            storePassword '147258369'  //签名证书文件密码
        }
    }

    buildTypes {
        release {
            minifyEnabled true//是否开启代码混淆
            //加载默认混淆配置文件，可同时配置多个Proguard配置文件
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
            //配置Proguard混淆使用的配置文件
            //proguardFile
            shrinkResources false //是否自动清理未使用的资源，默认false
            signingConfig signingConfigs.release //配置签名信息
            zipAlignEnabled true //启用zipAlign优化。能提高系统和应用运行效率，更快读写apk资源，降低内存使用
        }
        debug{
            jniDebuggable true //是否可生成一个可供调试Jni代码的apk
            minifyEnabled false
            multiDexEnabled true //是否启用自动拆分多个Dex的功能。程序中代码太多，超过65535个方法时，拆分为多个Dex的处理。
        }
    }
}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support.constraint:constraint-layout:1.1.3'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.2'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.2'
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.0-M1'
}
```



#### 使用共享库

