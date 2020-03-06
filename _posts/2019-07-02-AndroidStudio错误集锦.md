#### 1. 报错unable to find valid certification path to requested target

* 可能AndroidStudio设置了代理，导致build.gradle中加载第三方包时无法寻找到相应地址，从而报错。
  解决办法：设置搜索http proxy，设置为没有代理。
* 可能由于网络波动，导致变异无法寻找到第三方包的地址。
  解决办法：设置搜索http proxy，设置代理，根据自己ss进行设置，这样子就![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-img-androidError.png)OK了
* 也有可能由于buildgradle文件中，设置的引用maven问题，调整一下顺序，或者更改一下引用的仓库，就OK了。



#### 2. javax.net.ssl.SSLHandshakeException: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found

出现此报错的原因是证书校验出错。在Android低版本(大概5.1以下)是没有https证书校验的，所以一些需要证书校验才能通过的访问链接也能拿到数据，但是高版本就不行了，因为google修复了这个漏洞。出现这个问题的一般都是用了抓包软件或者开启代理等。



#### 3. Cause: buildOutput.apkData/apkInfo must not be null

造成该错误的原因是输出的outPut.json文件冲突，但是我们所打的包是正常的，能够正常使用，可以通过更换输出apk路径来解决该问题。网上所说的clean project 、rebuild project 、重新启动AS、删除build文件都没用。



#### 4.Java ConcurrentModificationException异常原因和解决方法

造成该异常的原因是遍历list，vector的同时进行了修改list，vector，从而引发并发修改异常。

具体原因（掘金中已经有开发者贴出来了）：

当执行remove(Object o)方法后，ArrayList对象的size减一此时size==4, modCount++了，然后Iterator对象中的cursor==5，hasNext发回了true，导致增强for循 环去寻找下一个元素调用next()方法，checkForComodification做校验的时候，发现modCount 已经和Iterator对象中的expectedModCount不一致，说明ArrayList对象已经被修改过， 为了防止错误，抛出异常ConcurrentModificationException。


作者：XING辋链接：https://juejin.im/post/5a992a0d6fb9a028e46e17ef来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

