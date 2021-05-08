# JavaScript

JavaScript是一门脚本语言，通常与HTML和Css结合使用，由**ECMAScript**+**BOM&DOM**组成

## ECMAScript

### 加载方式

一般有内部加载和外部加载等形式

内部加载：直接在HTML文件内使用`<script>`标签

外部加载：外部写好.js文件后通过`<script src="path">`的方式导入

可以在一个HTML文件内定义多个`<script>`，JS加载的顺序取决于它在什么位置

### 数据类型

JavaScript中有五种基本数据类型：

1. number：包括整数、小数、NaN
2. string：字符串，单双引号都可
3. boolean：true&false
4. null：对象为空的占位符
5. undefined：未定义。如果一个变量没有初始化值，则会被默认赋予undefined

还有最基本的引用数据类型：对象

### 变量

变量指**一小块存储数据的内存空间**

> Java是**强类型**的语言：当为变量申请内存空间时，会规定好申请的内存空间存放的数据类型，后续变量存放的数据更改，也只能是对应的数据类型
>
> JavaScript是**弱类型**的语言：当为变量申请空间时，申请的内存空间存放任何类型的数据

`var num = 1`→`num =“abc”`在JavaScript中是可行的

输出可以使用`document.write( num +“<br>”)`，用`<br>`可以换行

`typeof`关键字可以知道变量的类型

### Function

创建Function有三种方式：

1. `var fun  = new Function(形式参数列表，方法体)`不常用
2. `function 方法名(形式参数列表){方法体}`
3. `var 方法名 = function(形式参数列表){方法体}`

特点：

1. 形参的类型不用定义
2. Function是一个对象，在后续如果有重复定义Function，**后定义的会覆盖先定义的**
3. 在JS中，方法的调用**只与方法的名称有关，与参数列表无关**
4. 传入的参数实际会存放在一个隐藏的数组`arguments`中

### Array

JS中的数组可以存放任意数据类型，同时可以用下标获取值

`var arr = new Array(元素列表)`或者`var arr = new Array(默认长度)`

或者`var arr = [元素列表]`

可以使用`Array.join()`的方法定义分割，将数组中的元素按照指定的分隔符拼接成字符串

```javascript
var a = [1,"new",true]
document.write(a.join("--")+"<br>")//1--new--true
```

也可以使用`Array.push()`向数组的尾部添加新的元素，返回新的长度

### Date

`var date = new Date()`默认生成美国格式的当前时间

方法：

1. `toLocalString()`返回date对象对应的时间本地字符串格式
2. `getTime()`自1970-1-1起的毫秒时间

### RegExp

正则表达式对象，定义字符串组成规则的对象

正则表达式有如下规则：

* 查找单个字符：\[\]
  * `[a]` `[ab]` `[a-zA-Z0-9_]`
  * `[\d]`表示单个数字字符 `[0-9]`
  * `[\w]`表示单个单词字符 `[a-zA-Z0-9_]`
* 量词符号：
  * `n?`表示n出现0次或1次
  * `n*`表示n出现0次或多次
  * `n+`表示n出现1次或多次
  * `{m,n}`表示m$\leq$数量$\leq$n，m和n都可以缺省，分别标识上下限
* 开始结束符号：
  * `^`表示开始
  * `$`表示结束

正则表达式对象：

1. 创建：
   1. `var reg = new RegExp(“正则表达式”)`
   2. `var reg = /正则表达式/`
2. 方法：`test()`用于检测字符串是否符合规则

```javascript
var reg = new RegExp(/^\w{3,10}$/)
var reg2 = /^\\d{5,16}/
var reg3 = /^\w{3,10}$/
var name = "abcde"
document.write(reg.test(name)+"<br>")//true
document.write(reg3.test(name)+"<br>")//true
document.write(reg2.test(22234)+"<br>")//true
```

### Global

全局对象，Global中封装的方法不需要对象就可以直接调用

方法：

1. `encodeURI()`URI编码
2. `decodeURI()`URI解码
3. `encodeURIComponent()`URI编码
4. `decodeURIComponent()`URI解码
5. `parseInt()`将字符串转为数字，**逐一判断每一个字符是否是数字，直到不是数字为止，然后将前面的数字部分转为number，然后舍弃非数字部分**

   ```javascript
   var str = "146asdwqe";
   var str2 = parseInt(str);
   document.write(str2+1);//147
   ```

6. `isNaN()`判断是否为NaN，在标准情况下`NaN==NaN`会返回**false**，只有这个方法能够判断NaN
7. `eval()`会解析传入的参数**作为JS的代码片段**

URI编码：

中文字符的传输需要转换成对应的字节，UTF-8一个中文3字节，GBK一个中文2字节

`encodeURI(word)`可以将传入的文字根据当前页面的编码转换成URI编码

`decodeURI(encode)`可以将对应的URL编码转换成原本的文字

`encodeURIComponent()`和`decodeURIComponent()`方法相同，只不过可以转化更多的字符，例如`:`,`//`也会被转化成URL编码

## BOM

Browser Object Model 浏览器对象模型

总共由五个对象组成：

1. Window：窗口对象
2. Navigator：浏览器对象
3. Screen：显示器屏幕对象
4. History：历史记录对象
5. Location：地址栏对象

### Window对象

Window对象不需要创建，可以直接使用`window.方法名()`或直接`方法名()`来调用

#### 常用方法

1. 与弹出框相关：
   * `alter()`弹出对话框
   * `confirm()`弹出确认窗口，返回值有true和false
   * `prompt(String)`弹出输入窗口，可传入提示字符，返回值为字符串
2. 与窗口开关有关：
   * `open(url)`打开目标地址的新窗口，返回值为目标地址的**window对象**
   * `close()`关闭**调用此方法的窗口**，可以通过先前返回的window对象来关闭新窗口
3. 与定时器相关：
   * `setTimeout(js,time)`传入两个参数，第一个是js代码或毫秒值时间，返回**唯一标识定时器的id**
   * `clearTimeout(id)`根据id停止计时器
   * `setInterval(js,time)`与定时器相同，但是会**循环执行**
   * `cleatInterval(id)`停止循环计时器

#### 属性

可以获取其他的BOM对象：window、Screen、location、history

可以获取DOM对象：document

### Location

包含当前的URL信息

#### 常用方法

* `reload()`重新加载当前页面

#### 属性

`location.herf`更改当前网页的URL信息，可以直接跳转到对应的页面

### History

历史记录对象

#### 方法

`back()`加载history列表中的前一个URL

`forward()`加载history列表中的下一个URL

`go(n)`加载history列表中的某个具体页面，通过传参决定前进/后退n个页面

#### 属性

`length`：返回当前窗口历史列表中的URL数量

## DOM

Document Object Model 文档对象模型，将标记语言文档的各个组成部分封装为对象，可以使用这些对象来对文档进行CRUD

所有的标记语言文档在加载进内存时都会生成一个树形结构，被称为**DOM树**，所有的结点都是文档中的标签

W3C DOM分为三大部分：

* 核心DOM模型：针对任何结构化文档的标准模型
* XML DOM：针对XML文档的标准模型
* HTML DOM：针对HTML文档的标注模型

### 核心DOM

有五个对象：

* `Document`文档对象
* `Element`元素对象
* `Attribute`属性对象
* `Text`文本对象
* `Comment`注释对象
* `Node`结点对象，以上对象都继承自结点对象

#### Document

控制HTML文档的内容，获取页面标签（元素）对象Element：

* `document.getElementById()`通过Id值获取，返回唯一一个Element
* `document.getElementByTagName()`通过标签名获取，返回Elements数组
* `document.getElementByClassName()`根据Class属性值获取，返回Elements数组
* `document.getElementByName()`，根据name属性值获取，返回Elements数组

创建其他DOM对象：

* `createAttribute()`创建属性
* `createComment()`创建注释
* `createElement()`创建Element
* `createTextNode()`创建Text结点

#### Element

操作获取的Element对象：

* 修改属性值：`Element.xxx=xxx`
* 修改标签体内容：
  * 属性：`Element.innerHTML=xxx`

常用方法：设置属性`setAttritube(标签名,内容)`&`removeAttritube(标签名)`

#### Node

节点对象代表文档树中的一个节点，所有DOM对象都可以被认为是一个节点

方法：

* `appendChild()`在当前节点添加一个子节点
* `removeChild()`删除当前节点的子节点
* `replaceChild()`替换当前节点的子节点
* `parentNode()`返回当前节点的父节点

### HTML DOM

### 事件

某些组件被执行了某些操作之后，触发某些代码的执行

事件有两种方法可以设置：

1. 绑定HTML标签属性，指定事件的操作，属性值为JS代码

   ```javascript
   <img src="xxx.jpg" onclick="alter(123)">
   ```

2. 通过JS获取元素对象，用JS代码指定事件属性，设置函数

   ```javascript
   <!DOCTYPE html>
   <html lang="en">
       <head>
           <meta charset="UTF-8">
           <title id="gg">JavaScript Practice</title>
       </head>
       <body>
           <button type="button" id="Click"> Click me!</button>
       </body>
       <script>
           function func(){
               var title = document.getElementById("gg");
               title.innerHTML="test"
           }
           var button = document.getElementById("Click");
           button.onclick=func;//此处调用不要带括号，只需要函数名，否则会直接执行
       </script>
   </html>
   ```

   **常见的事件**

   1. 点击事件
      * `onclick`单击
      * `ondblclick`双击
   2. 焦点事件
      * `onblur`失去焦点，一般用于判断文本框输入完成后是否符合要求
      * `onfocus`获得焦点
   3. 加载事件
      * `onload`一张页面或图片完成加载
   4. 鼠标事件
      * `onmousedown`按下按键
      * `onmouseup`松开按键
      * `onmousemove`鼠标移动
      * `onmouseover`鼠标移到某元素上
      * `onmouseout`鼠标移出某元素
   5. 键盘事件
      * `onkeydown`按下键盘按键
      * `onkeyup`松开键盘那件
      * `onkeypress`按下并松开按键
   6. 选择和改变
      * `onchange`域的内容被改变，可以用于选择条件下改变文本框内容
      * `onselect`文本被选中
   7. 表单事件
      * `onsubmit`确认按钮被点击，通常用于表单检验，返回值为false可以阻止提交表单
      * `onreset`重置按钮被点击



