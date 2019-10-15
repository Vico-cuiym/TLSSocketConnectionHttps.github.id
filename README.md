# JDK1.6支持TLS1.2协议,并忽略身份验证

* TLS1.2升级成TLS1.1,项目中的区别,就访问时由HTTP变成了HTTPS
* JDK1.6其实不支持TLS1.2,默认由TLS1.1访问请求.JDK1.8默认TLS1.2,比较简单的方案是升级JDK,但是对于老项目,升级JDK无非是
* 挑灯上厕所.在不能升级JDK的情况下,可以使用一下方式

## 本地配置
* 1、引入依赖 <br>
    <pre><code>
        &lt;dependency&gt;
            &lt;groupId&gt;org.bouncycastle&lt;/groupId&gt;
            &lt;artifactId&gt;bcprov-jdk15on&lt;/artifactId&gt;
            &lt;version&gt;1.54</version&gt;
        &lt;/dependency&gt;
    </code></pre>


![Image text](https://github.com/Vico-cuiym/HttpToHttps.github.io/blob/master/imgs/Ctrl%2BR.png)
* 2、进入JDK安装路径 , 以我为例 : C:\Program Files\Java\jdk1.8.0_161\bin;<br>
进入该目录 , 执行如下命令 : `keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keystore "D:\tomcat.keystore"` ;<br>
![Image text](https://github.com/Vico-cuiym/HttpToHttps.github.io/blob/master/imgs/cmd.png)<br>
接着填写一些基本信息<br>
![Image text](https://github.com/Vico-cuiym/HttpToHttps.github.io/blob/master/imgs/keytool.png)<br>
简单介绍下填写的内容<br>
<pre><code>密钥库口令:123456（这个密码非常重要 , 后面会用到）
名字与姓氏:127.0.0.1（以后访问的域名或IP地址，非常重要，证书和域名或IP绑定）
组织单位名称:anything（随便填 , 可以直接按回车）
组织名称:anything（随便填 , 可以直接按回车）
城市:anything（随便填 , 可以直接按回车）
省市自治区:anything（随便填 , 可以直接按回车）
国家地区代码:anything（随便填 , 可以直接按回车）</code></pre><br>
* 3、配置Tomcat服务器
    * 3.1、打开Tomcat中的配置文件 , 以我为例 : E:/WorkingSoftware/apache-tomcat-6.0.29/conf/server.xml
    <pre><code>&lt;!--
    &lt;Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
        maxThreads="150" scheme="https" secure="true"
        clientAuth="false" sslProtocol="TLS"/&gt;
    --&gt;</code></pre>
    * 3.2、去掉注释且修改参数
    <pre><code>
    &lt;Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true"
        maxThreads="150" scheme="https" secure="true"
        clientAuth="false" sslProtocol="TLS" 
        keystoreFile="D:/tomcat.keystore" keystorePass="123456789"/&gt;
    </code></pre>
    >其实就是添加`keystoreFile`和`keystorePass`两个属性 
    >>`keystoreFile`为生成证书地址 , 建议放在启动的Tomcat中
    >>`keystorePass`为密钥库口令
    * 3.3、注释Tomcat路径中conf\server.xml文件中下面一行。
    <code><pre>
    &lt;!--&lt;Listener SSLEngine="on" className="org.apache.catalina.core.AprLifecycleListener"/&gt;--&gt;
    </code></pre>
    * 最后访问链接：https://127.0.0.1:8443/项目名称/ , 就可以看见https的安全认证页面了<br>
    ![Image text](https://github.com/Vico-cuiym/HttpToHttps.github.io/blob/master/imgs/Privatelink.png)<br>


# 🚨以下为参考文档🚨
[TLS 1.0、TLS 1.1、TLS 1.2之间的区别](https://blog.csdn.net/mrpre/article/details/77978293)<br>
[SSL与TLS区别以及介绍](https://blog.csdn.net/adrian169/article/details/9164385)<br>
[keytool常见用法](https://www.cnblogs.com/benio/archive/2010/09/15/1826990.html)<br>
[Tomcat各属性详解](https://blog.csdn.net/weixin_33946605/article/details/92438866)<br>





