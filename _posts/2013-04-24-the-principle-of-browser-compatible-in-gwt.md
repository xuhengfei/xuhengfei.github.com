---
layout: blog
title: GWT中对不同浏览器兼容性实现原理
excerpt: 
---

目前大多数的  JS框架都有跨浏览器到功能，他们会根据不同的浏览器来执行不同的代码，从而达到一致的功能。  
GWT也有这种兼容所有浏览器到功能，他是如何实现的呢。  

GWT中每一个模块都有一个gwt.xml配置文件，  
如: com.google.gwt.dom.DOM.gwt.xml这个配置文件  
{% highlight xml %}  
<module>
  <inherits name="com.google.gwt.core.Core"/>
  <inherits name="com.google.gwt.user.UserAgent"/>

  <replace-with class="com.google.gwt.dom.client.DOMImplOpera">
    <when-type-is class="com.google.gwt.dom.client.DOMImpl"/>
    <when-property-is name="user.agent" value="opera"/>
  </replace-with>

  <replace-with class="com.google.gwt.dom.client.DOMImplSafari">
    <when-type-is class="com.google.gwt.dom.client.DOMImpl"/>
    <when-property-is name="user.agent" value="safari"/>
  </replace-with>

  <replace-with class="com.google.gwt.dom.client.DOMImplIE8">
    <when-type-is class="com.google.gwt.dom.client.DOMImpl"/>
    <when-property-is name="user.agent" value="ie8"/>
  </replace-with>

  <replace-with class="com.google.gwt.dom.client.DOMImplIE6">
    <when-type-is class="com.google.gwt.dom.client.DOMImpl"/>
    <when-property-is name="user.agent" value="ie6"/>
  </replace-with>

  <replace-with class="com.google.gwt.dom.client.DOMImplMozilla">
    <when-type-is class="com.google.gwt.dom.client.DOMImpl"/>
    <when-property-is name="user.agent" value="gecko1_8"/>
  </replace-with>

  <replace-with class="com.google.gwt.dom.client.DOMImplMozillaOld">
    <when-type-is class="com.google.gwt.dom.client.DOMImpl"/>
    <when-property-is name="user.agent" value="gecko"/>
  </replace-with>
</module>

{% endhighlight %}

这里有很多replace-with的标签，仔细想一下应该可以猜到他的功能。  
当属性user.agent的值为ie6时，用com.google.gwt.dom.client.DOMImplIE6这个类来实现com.google.gwt.dom.client.DOMImpl这个抽象类。  
也就是说GWT通过判断user.agent来判断用户使用的是什么浏览器，然后用相应的实现类来执行，从而达到不同浏览器到兼容性。  

那么这个user.agent属性是哪来的呢，观察这个xml的开头可以看到此模块继承了<inherits name="com.google.gwt.user.UserAgent"/>  
应该可以在这个UserAgent模块里看出端倪来。  

看一下UserAgent模块的xml文件  
{% highlight xml %}  
<module>

  <!-- Browser-sensitive code should use the 'user.agent' property -->
  <define-property name="user.agent" values="ie6,ie8,gecko,gecko1_8,safari,opera"/>

  <property-provider name="user.agent"><![CDATA[
      var ua = navigator.userAgent.toLowerCase();
      var makeVersion = function(result) {
        return (parseInt(result[1]) * 1000) + parseInt(result[2]);
      };

      if (ua.indexOf("opera") != -1) {
        return "opera";
      } else if (ua.indexOf("webkit") != -1) {
        return "safari";
      } else if (ua.indexOf("msie") != -1) {
        if (document.documentMode >= 8) {
          return "ie8";
        } else {
          var result = /msie ([0-9]+)\.([0-9]+)/.exec(ua);
          if (result && result.length == 3) {
            var v = makeVersion(result);
            if (v >= 6000) {
              return "ie6";
            }
          }
        }
      } else if (ua.indexOf("gecko") != -1) {
        var result = /rv:([0-9]+)\.([0-9]+)/.exec(ua);
        if (result && result.length == 3) {
          if (makeVersion(result) >= 1008)
            return "gecko1_8";
          }
        return "gecko";
      }
      return "unknown";
  ]]></property-provider>

  <!-- Deferred binding to optimize JRE classes based on user agent. -->
  <inherits name="com.google.gwt.emul.EmulationWithUserAgent"/>
</module>
{% endhighlight %}

这里有一个property-provider标签，这就是提供user.agent属性的标签，  
CDATA里面的代码就是js代码，通过这个js代码来判断浏览器类型  
仔细看一下这段js代码：  
{% highlight javascript %}
      var ua = navigator.userAgent.toLowerCase();
      var makeVersion = function(result) {
        return (parseInt(result[1]) * 1000) + parseInt(result[2]);
      };

      if (ua.indexOf("opera") != -1) {
        return "opera";
      } else if (ua.indexOf("webkit") != -1) {
        return "safari";
      } else if (ua.indexOf("msie") != -1) {
        if (document.documentMode >= 8) {
          return "ie8";
        } else {
          var result = /msie ([0-9]+)\.([0-9]+)/.exec(ua);
          if (result && result.length == 3) {
            var v = makeVersion(result);
            if (v >= 6000) {
              return "ie6";
            }
          }
        }
      } else if (ua.indexOf("gecko") != -1) {
        var result = /rv:([0-9]+)\.([0-9]+)/.exec(ua);
        if (result && result.length == 3) {
          if (makeVersion(result) >= 1008)
            return "gecko1_8";
          }
        return "gecko";
      }
      return "unknown";
{% endhighlight %}  

{% highlight javascript %}
navigator.userAgent.toLowerCase();
{% endhighlight %}
这句话就能拿到浏览器的相关信息，并把它们变成小写字母。  

不同浏览器提供的内容不同，分别如下、  
{% highlight html %}
      IE

      Mozilla/4.0 (compatible; MSIE 8.0; Windows NT 6.0)
      Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.2)
      Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
      Mozilla/4.0 (compatible; MSIE 5.0; Windows NT)
      其中，版本号是MSIE之后的数字。

      Firefox

      Mozilla/5.0 (Windows; U; Windows NT 5.2) Gecko/2008070208 Firefox/3.0.1
      Mozilla/5.0 (Windows; U; Windows NT 5.1) Gecko/20070309 Firefox/2.0.0.3
      Mozilla/5.0 (Windows; U; Windows NT 5.1) Gecko/20070803 Firefox/1.5.0.12
      其中，版本号是Firefox之后的数字。

      Opera

      Opera/9.27 (Windows NT 5.2; U; zh-cn)
      Opera/8.0 (Macintosh; PPC Mac OS X; U; en)
      Mozilla/5.0 (Macintosh; PPC Mac OS X; U; en) Opera 8.0
      其中，版本号是靠近Opera的数字。

      Safari

      Mozilla/5.0 (Windows; U; Windows NT 5.2) AppleWebKit/525.13 (KHTML, like Gecko) Version/3.1 Safari/525.13
      Mozilla/5.0 (iPhone; U; CPU like Mac OS X) AppleWebKit/420.1 (KHTML, like Gecko) Version/3.0 Mobile/4A93 Safari/419.3
      其版本号是Version之后的数字。

      Chrome

      Mozilla/5.0 (Windows; U; Windows NT 5.2) AppleWebKit/525.13 (KHTML, like Gecko) Chrome/0.2.149.27 Safari/525.13
      其中，版本号在Chrome只后的数字。
{% endhighlight %}
就是对这个字符串进行判断，来得知浏览器类型，然后设置属性user.agent来告知gwt程序