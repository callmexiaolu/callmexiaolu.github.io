##### 报错unable to find valid certification path to requested target

* 可能AndroidStudio设置了代理，导致build.gradle中加载第三方包时无法寻找到相应地址，从而报错。
  解决办法：设置搜索http proxy，设置为没有代理。
* 可能由于网络波动，导致变异无法寻找到第三方包的地址。
  解决办法：设置搜索http proxy，设置代理，根据自己ss进行设置，这样子就![](http://github.com/callmexiaolu/callmexiaolu.github.io/raw/master/img/post-img-androidError.png)OK了

* 也有可能由于buildgradle文件中，设置的引用maven问题，调整一下顺序，或者更改一下引用的仓库，就OK了。