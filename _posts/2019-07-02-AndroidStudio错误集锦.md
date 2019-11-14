#### 1. 报错unable to find valid certification path to requested target

* 可能AndroidStudio设置了代理，导致build.gradle中加载第三方包时无法寻找到相应地址，从而报错。
  解决办法：设置搜索http proxy，设置为没有代理。
* 可能由于网络波动，导致变异无法寻找到第三方包的地址。
  解决办法：设置搜索http proxy，设置代理，根据自己ss进行设置，这样子就![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-img-androidError.png)OK了
* 也有可能由于buildgradle文件中，设置的引用maven问题，调整一下顺序，或者更改一下引用的仓库，就OK了。



#### 2. javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found

出现此报错的原因是证书校验出错。在Android低版本(大概5.1以下)是没有https证书校验的，所以一些需要证书校验才能通过的访问链接也能拿到数据，但是高版本就不行了，因为google修复了这个漏洞。出现这个问题的一般都是用了抓包软件或者开启代理等。



#### 3. Cause: buildOutput.apkData/apkInfo must not be null

打包路径不要选择module，选择project。例如/user/xx/project     这样子的路径，而不是/user/xx/project/app这样子的路径。网上所说的clean project 、rebuild project 、重新启动AS、删除build文件都没用。

