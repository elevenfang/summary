 最近在工作项目中使用wasce（websphere application community edition, websphere 社区版，3.0.0 version）运行webApp时出现了NoClassDefFoundException， 该class为httpcore.jar中的Args，因为代码中使用HttpClientBuilder.create().build() 创建HttpClient实例，而创建过程会调用到Args。
       遇到这样的情况，首先第一反应是查看webApp对应的WEB-INF/lib查看是否含有该class的对应的dependency，一看确实有的，而且在本地也通过jetty运行起来了，所以不是应用本身的问题了。
       那么可能是服务器影响到了应用。该webApp是在wasce下运行，而wasce本身也依赖于jar，如果出现同一package下的同一个class，是不是优先使用wasce本身的class，从而导致webApp在runtime找不到该class。
     为了验证这个猜测， 首先要去找到有这样的jar， 在repository目录下grep “httpcore” ，找到了httpcore-4.0.1 jar， webApp使用的是4.3.2 version。接着download下来4.0.1 version的source code，没有Args类，初步确定上面的猜测是对的。冲动的我首先想到了直接去改wasce里面的jar（这种想法很不推荐），内容替换成4.3.2 version的，但是jar name和path保持不变，但这样直接导致了这个wasce运行不能起来，就算恢复回去，整个wasce依旧不能运行（猜测可能这种方式的修改直接打乱了wasce的内部，而且不可恢复）。
     转而投向webApp本身，不使用4.3.2的httpcore， 使用httpcore-4.1.0.jar重新实现那一部分代码，但没有去尝试， 因为同事想到另外一个办法，我们查看了Args类的源码， 发现只不过是用来验证某些条件的，不满足就throw exception。
（1）在webApp中的package加同一个package下相同Args。【结果一样的问题，因为该package下所有的class都使用了4.1.0 version 中的】
（2）继承调用Args的类SSLConnectionSocketFactory并重写部分方法， 从而跳过Args的使用。【依旧找不到Agrs， 因为后来发现代码的其他地方也调用了Args，也需要重写，但是对方是个final class， 不能被继承】
   后来去wasce的官网文档 https://publib.boudle.ibm.com/wasce/V3.0.0/en/war.html （需要翻墙），找到了可以override the parent class 的方法，runtime中就会使用webApp中的class。
  具体方法： 在geronimo-web.xml 中添加如下:
<environment>  
    <hidden-classes>
         <filter>org.apache.http</filter>
    </hidden-classes>
</environment>
另外， 还有其他标签可以使用以达到各种不同的效果如<no-overridable-classes>（load from parent loader），<inverse-classloading>.（load from app before parent loader， 谨慎用，因为所有jar都会从app 中load，最好用hidden class实现）。