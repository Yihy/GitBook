---
date : 2017-11-26T13:47:08+02:00
tags : []
title : Jsoup  Html页面处理简单用法
---

#[Jsoup  Html页面处理简单用法]

>Jsoup工具可以把html文本映射为Dom对象，用更简单的操作html。
> 1. 从一个URL，文件或字符串中解析HTML；
    2. 使用DOM或CSS选择器来查找、取出数据；
    3. 可操作HTML元素、属性、文本；

## 加载Html

```java
//中间内容省略
String html="<html>....</html>";
//parse方法可以加载String、File类型的html，url加载、流都可以。
Document parse = Jsoup.parse(html);
//也可以加载html片段
Document parse = Jsoup.parseBodyFragment(html);
```

## 查找Html

Document 、Elements、Element对象都有这些方法
getElementById(String id)
getElementsByTag(String tag)
getElementsByClass(String className)
getElementsByAttribute(String key)
```java
//获取a标签的元素集合
Elements as =parse.getElementsByTag("a");

//查找的方法 同样获得a标签的元素集合
Elements as=parse.select("a");
```
使用类似于CSS或jQuery的语法来查找和操作元素。
``` java
File input = new File("/tmp/input.html");
Document doc = Jsoup.parse(input, "UTF-8", "http://example.com/");

Elements links = doc.select("a[href]"); //带有href属性的a元素
Elements pngs = doc.select("img[src$=.png]");
  //扩展名为.png的图片
Element masthead = doc.select("div.masthead").first();
  //class等于masthead的div标签
Elements resultLinks = doc.select("h3.r > a"); //在h3元素之后的a元素
```

###  select(String) 方法的使用

#### Selector选择器概述
 
 1.  tagname: 通过标签查找元素，比如：a
 2. ns|tag: 通过标签在命名空间查找元素，比如：可以用 fb|name 语法来查找 <fb:name> 元素
 3. \#id: 通过ID查找元素，比如：#logo
 4.  \.class: 通过class名称查找元素，比如：\.masthead
 5.  \[attribute\]: 利用属性查找元素，比如：\[href\]
 6.  \[^attr\]: 利用属性名前缀来查找元素，比如：可以用\[^data-\] 来查找带有HTML5 Dataset属性的元素
 7. \[attr=value\]: 利用属性值来查找元素，比如：\[width=500\]
 8.  \[attr^=value\], \[attr$=value\], \[attr*=value\]:
    利用匹配属性值开头、结尾或包含属性值来查找元素，比如：[href*=/path/]
 9.  \[attr~=regex]: 利用属性值匹配正则表达式来查找元素，比如： img[src~=(?i)\.(png|jpe?g)]
 10.  \*: 这个符号将匹配所有元素

#### Selector选择器组合使用

 1. el#id: 元素+ID，比如： div#logo
 2. el.class: 元素+class，比如： div.masthead
 3. el[attr]: 元素+class，比如： a[href]
 4. 任意组合，比如：a[href].highlight
 5. ancestor child: 查找某个元素下子元素，比如：可以用.body p 查找在"body"元素下的所有 p元素
 6. parent > child: 查找某个父元素下的直接子元素，比如：可以用div.content > p 查找 p 元素，也可以用
 7. body > * 查找body标签下所有直接子元素
 8. siblingA + siblingB: 查找在A元素之前第一个同级元素B，比如：div.head + div
 9. siblingA ~ siblingX: 查找A元素之前的同级X元素，比如：h1 ~ p
 10. el, el, el:多个选择器组合，查找匹配任一选择器的唯一元素，例如：div.masthead, div.logo

#### 伪选择器selectors

 1. :lt(n): 查找哪些元素的同级索引值（它的位置在DOM树中是相对于它的父节点）小于n，比如：td:lt(3) 表示小于三列的元素
 2. :gt(n):查找哪些元素的同级索引值大于n，比如： div p:gt(2)表示哪些div中有包含2个以上的p元素
 3. :eq(n): 查找哪些元素的同级索引值与n相等，比如：form input:eq(1)表示包含一个input标签的Form元素
 4. :has(seletor): 查找匹配选择器包含元素的元素，比如：div:has(p)表示哪些div包含了p元素
 5. :not(selector): 查找与选择器不匹配的元素，比如： div:not(.logo) 表示不包含 class=logo
    元素的所有 div 列表
 6. :contains(text): 查找包含给定文本的元素，搜索不区分大不写，比如： p:contains(jsoup)
 7. :containsOwn(text): 查找直接包含给定文本的元素
 8. :matches(regex): 查找哪些元素的文本匹配指定的正则表达式，比如：div:matches((?i)login)
 9. :matchesOwn(regex): 查找自身包含文本匹配指定正则表达式的元素

*注意：上述伪选择器索引是从0开始的，也就是说第一个元素索引值为0，第二个元素index为1等*

##  编辑内容

只列举几个简单的方法，
``` java
Element element=as.first();
//获得元素文本
element.text();
//获得元素href属性
element.attr("href");
//设置元素href属性
element.attr("href", "http://www.yihy.cc");
//给元素添加新类
element.addClass("link");
//移除元素
element.remove();

//元素也可以查找子标签
element.select("lable");
```

##  消除xss攻击

使用jsoup HTML Cleaner 方法进行清除，但需要指定一个可配置的 Whitelist。

```java
String unsafe =  "<p><a href='http://example.com/' onclick='stealCookies()'>Link</a></p>";
String safe = Jsoup.clean(unsafe, Whitelist.basic());
// now: <p><a href="http://example.com/" rel="nofollow">Link</a></p>
```

> 这个Jar主要用来做爬虫

更多的使用方法请访问
[www.open-open.com/jsoup/](http://www.open-open.com/jsoup/)
[www.jsoup.com/](http://www.jsoup.com/)
