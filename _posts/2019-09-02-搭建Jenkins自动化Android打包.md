## Jenkins自动化打包体系构建

每次开发完一个版本，开发人员都要更改配置，编译打包，发版。过程枯燥，等待时间漫长，而且一不留意还会配置更改错误，打错包，因此构建一个自动化打包能节省时间，提高保障。

### 搭建环境

* Mac10.14.6
* 开发工具AndroidStudio

#### 1.adb搭建
打开终端，输入adb，检验是否已经存在。
![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-2.png)

如果没有，那就创建:

* 先shitf+command+.查看用户目录下隐藏文件是否存在 .bash_profile，如果没有，那么终端touch .bash_profile，如果有那么终端sudo chimed -R 777 文件路径    更改权限

* 打开文件输入AndroidSdk中platform-tool路径，格式：export PATH=${PATH}:路径

* 保存后，终端输入

  ```shell
  source .bash_profile
  ```

* 输入adb检验是否配置成功

#### 2.gradle搭建

终端输入

```shell
brew install gradle
```

(等待下载安装，过程可能会比较久)

输入

```shell
gradle -version
```

检验是否配置成功：

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-1.png)

#### 3.jenkins搭建

终端输入

```shell
brew install jenkins
```

如果提示没有brew，那么就先安装brew。

jenkins安装过程比较慢，可能安装开始会遇到该错误：

```shell
jenkins: Java 1.8 is required to install this formula.
Install AdoptOpenJDK 8 with Homebrew Cask:
  brew cask install homebrew/cask-versions/adoptopenjdk8
```

那么按照提示安装openjdk，输入

```shell
brew cask install homebrew/cask-versions/adoptopenjdk8
```

安装完jenkins后，注意出现的话：

```shell
To have launchd start jenkins now and restart at login:
  brew services start jenkins
Or, if you don't want/need a background service you can just run:
  jenkins
```

输入jenkins就可以运行了，然后打开浏览器输入localhost:8080

输入密码，选择插件安装（这里我选择推荐的插件）。

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-3.png)

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-4.png)

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-5.png)

安装完成，设置用户名等信息，然后地址栏输入http://localhost:8080/login?from=%2F登录：

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-6.png)



##### 设置jenkins

登录成功后你可能看到空白页，解决方案步骤：

* 1.访问该地址http://localhost:8080/pluginManager/advanced

* 2.页面下拉到底部，将升级的站点的https改为http

* 3.重启jenkins

  * 重启方法1:localhost:8080/restart

  * 方法二：

    ```shell
    #启动jenkins
    sudo service jenkins start
    #重启
    service jenkins restart
    ```

重启成功后：

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-7.png)



#### 4.jenkins相关环境配置

##### a.配置Android SDk

* 点击Jenkins设置(manage Jenkins) 
*  Configure System（系统设置）
* 下拉找到全局属性，勾选Environment variables，新增键值对，键的名称为：ANDROID_HOME    值的名称为：你电脑上的android SDK的路径
* 保存

#### 5.准备一个github项目

### 实战演练

#### 1.新建一个任务

选择构建一个自由软件风格的项目(Freestyle project)

源码管理中选择Git，输入Git仓库地址，选择对应的分支，构建之前可以根据需求再构建环境中勾选相应的选项。

#### 2.构建完成

构建完成，相关任务在执行状态中不见了，那么进入系统管理——配置中查看主目录，构建的项目在：/Users/xxx/.jenkins/jobs/Android_Test

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-8.png)



点进去发现没有apk文件生成。配置相关的编译构建环境。

#### 3.编译配置设置

点击构建的项目，配置，下拉选择增加构建步骤，选择Invoke Gradle script。

Tasks填写build，RootBuildscript填写app中的build.gradle文件路径，即：

```shell
${workspace}/app
```

${workspace} = .jenkins/workspace

然后保存。开始构建项目。

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-10.png)



#### 4.构建完成

我一次性构建打包，打了21个包，花了1个小时。

打包输出的地址为.jenkins/workspace/项目名/app/build/outputs/apk

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-11.png)



![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-12.png)



### 多渠道不同配置打包以及自行选择配置打包

在项目配置中手动输入构建步骤（即在Task中），例如：

```groovy
clean   //编译前清除项目build的文件
assemble${PRODUCT_FLAVOR_BUILD} //生成相关渠道的apk
```

这些都是gradle的构建任务。

![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-9-2-13.png)



* make gradlew executable：是指gradlew命令可执行

* wrapper location： 是指gradlew文件的位置（一般是根project）

* Task：gradlew执行的任务，图片中即是：./gradlew assembleekwing_tencentRelease，意思为生成一个在build文件中ProductFlavor中的ekwing_tencent的release包，也就是其中一个渠道包。
* root build script：指你要构建项目的build.gradle文件，一般是app目录下的build文件



### 构建名称更改与展示上传蒲公英二维码

安装使用build name setter and descript扩展插件，可以更改构建名字为版本号，更加清晰明了，安装成功后重启jenkins。进入项目配置中，在构建环境中可以看到Set Build Name选项，里面进行设置构建名字与描述。

#### 更改构建名字



#### 构建完成上传到蒲公英

##### 1.命令上传

构建中，增加构建步骤，选择Execute shell，输入以下命令：

```shell
curl -F "file=@/tmp/example.apk" -F "uKey=" -F "_api_key=" https://qiniu-storage.pgyer.com/apiv1/app/upload 
```

##### 2.插件上传

安装插件Upload to pgyer，成功后重启jenkins，项目配置增加构建后步骤选择Upload to pgyer，填写相关信息，上传。

![](https://static.pgyer.com/image/view/admin_images/04966eb745590be33e3c8176b4766855)



![](https://static.pgyer.com/image/view/admin_images/4103592163bae6d1118995914ad5ef21)



参数说明：

|          **参数**           |                           **说明**                           |
| :-------------------------: | :----------------------------------------------------------: |
|         pgyer uKey          |                   蒲公英的 `uKey` （必填）                   |
|        pgyer api_key        |                 蒲公英的 `api_key` （必填）                  |
|           scandir           |                  ipa/apk 所在目录 （必填）                   |
|        file widcard         |                  上传文件的通配符 （必填）                   |
|    installType(optional)    | 应用安装方式，值为(1,2,3)。1：公开，2：密码安装，3：邀请安装。默认为1公开(选填) |
|     password(optional)      |                   设置App安装密码（选填）                    |
| updateDescription(optional) |                     版本更新描述（选填）                     |
|    qrcodePath(optional)     | 如果你需要下载蒲公英返回的二维码，那么这里填写二维码的存储路径，如果你不需要下载，那么你不需要在这里填写任何内容（选填） |
|    envVarsPath(optional)    | 如果你想存储蒲公英返回的上传信息，那么这里填写保存信息的文件路径，如果你不需要保存，那么你不需要在这里填写任何内容（选填） |

注意：
qrcodePath与envVarPath是存储二维码和应用信息的文件路径地址，而不是一个文件夹的地址。例如可以这样填写这两个参数：

```shell
qrcodePath: /Users/James/IOS_Integration/${BUILD_TYPE}/${BUILD_TIME}/qrcode.png
envVarPath: /Users/James/IOS_Integration/${BUILD_TYPE}/${BUILD_TIME}/envVars.txt**
```

##### 3.添加下载二维码图片

构建完成后会输出相应的log：

![](https://static.pgyer.com/image/view/admin_images/27684f4b5e2ca166e992e4c057092d92)

然后在jenkins中使用蒲公英的参数：

![](https://static.pgyer.com/image/view/admin_images/44bc8a82d93e25da1372b306d75d0608)

这样子就显示二维码了。



**注：123内容文字以及图片参考自蒲公英官网，地址为：

[https://www.pgyer.com/doc/view/jenkins](https://www.pgyer.com/doc/view/jenkins)

[https://www.pgyer.com/doc/view/jenkins_plugin](https://www.pgyer.com/doc/view/jenkins_plugin)



