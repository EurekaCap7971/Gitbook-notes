# XML

## XML

Extensible Markup Language 可拓展标记语言，标签都是**自定义的**

通常用于**配置文件**，可以通过网络传输

### 基本语法

1. XML文档的后缀名：`.xml`
2. XML文档第一行**必定为文档声明**
3. XML文档中有且仅有一个根标签
4. 属性值必须用单双引号包括起来
5. 标签必须正确关闭，可自闭合
6. 标签区分大小写

### 组成部分

1. 文档声明
   * 格式：`<? xml 属性列表 ?>`
   * 属性列表：
     * `version`：xml版本
     * `encoding`：编码
     * `standalone`：是否独立，标记xml是否依赖其他文件
2. 标签：
   * 标签名字具有特定的规则，同Java变量命名规则
3. 属性
   * 属性值唯一
4. 文本：
   * CDATA区：在CDATA区中的文本会被原样展示
     * 格式：`<![CDATA[文本]]>` 

### 约束

规定XML文档的书写规则，在XML文档中引入约束文件可以对XML文档的标签等内容进行规定

#### DTD

DTD是一种简单的约束技术，分为外部DTD和内部DTD

内部DTD定义在XML文件内部

外部DTD可从本地获取或网络获取：

* 本地：`<!DOCTYPE 根标签名 SYSTEM “DTD文件的位置">`
* 网络：`<!DOCTYPE 根标签名 PUBLIC “DTD文件名" “DTD文件的位置URL">`

缺陷：只能规定标签以及标签类型，但是不能限定标签的内容

#### schema

引入：

1. 填写XML文档的根元素
2. 引入xsi前缀：`xmlns:xsi=“http://www.w3.org./2001/XMLSchemal-instance”`
3. 引入xsd文件命名空间：`xsi:schemaLocation=“http://www.itcast.cn/xml  student.xsd”`
4. 为每一个xsd约束声明一个前缀作为标识：`xmlns=“http://www.itcast.cn/xml”`

```text
<students xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://www.itcast.cn/xml"
    xsi:schemaLocation="http://www.itcast.cn/xml student.xsd">
```

### 解析

操作xml文档，将文档中的数据读取到内存中

解析的两种方式：

* DOM：将标记语言文档XML一次性加载进内存，在内存中形成一颗DOM树
  * 优点：操作方便，读取进内存之后可以对文档进行CRUD的所有操作
  * 缺点：一次性加载进内存，非常消耗内存
* SAX：逐行读取，基于事件驱动
  * 由于是逐行读取，因此会根据读取到的内容，基于事件进行判断
  * 优点：不占内存，读取完之后会释放
  * 缺点：只能读取，不能修改

#### 解析器

JAXP：SUN公司提供的解析器，支持DOM和SAX

DOM4J：一款优秀的解析器

JSOUP：一款Java的HTML解析器，可以直接解析URL地址

PULL：一款Android操作系统内置的解析器，只支持SAX

#### Jsoup

Jsoup工具类可以解析xml或html文档，并返回Document对象

```java
public class JsoupDemo {
    public static void main(String[] args) throws IOException {
        String path = JsoupDemo.class.getClassLoader().getResource("student.xml").getPath();
        System.out.println(path);
        Document doc = Jsoup.parse(new File(path), "utf-8");
        Elements name = doc.getElementsByTag("name");
        System.out.println(name.get(0).text());
    }
}
```

返回的Document对象可以通过`getElementsBy···()`等方法对DOM树中的元素进行操作，获取到的Elements对象也可以获取元素对象及其属性值：

* `getElementsBy···()`根据···获取Elements，是Element元素的集合\(ArrayList\)
* `String attr(String key)`根据标签名获取元素对象集合
* `String text/html()`获取Element的text文本或html文本

**快捷查询方法**

1. 选择器Select：
   * `Elements select(String cssQuery)`，语法参考Selector类
   * 获取Student标签且number属性为001的age子标签

     ```java
     document.select("Student[number='001']>age")
     ```
2. XPath路径语言：
   * 使用XPath需要额外导入Jar包
   * `List<JXNode> jxDocument.selN(xpath)`返回JXNode的集合
   * XPath语法包含有很多种，`//标签名`可以获取所有带有此标签的结点，`//标签/子标签`，可以获取标签下的子标签，`//标签名/子标签[@id]`获取标签下子标签带有id属性的标签，可以判断属性的值

