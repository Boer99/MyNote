
# HTML 简介

全称：HyperText Markup Language（超文本标记语言）。
- 超文本：是一种组织信息的方式，通过超链接将不同空间的文字、图片等各种信息组织在一起，能从当前阅读的内容，跳转到超链接所指向的内容。
- 标记：文本要变成超文本，就需要用到各种标记符号。
- 语言：每一个标记的写法、读音、使用规则，组成了一个标记语言。

从 HTML 1.0 开始发展，期间经历了很多版本，目前 HTML 的最新标准是：HMTL 5，具体发展史如图

![](assets/Pasted%20image%2020240204134556.png)

> WSC：World Wide Web Consortium（万维网联盟），创建于1994年，是目前Web技术领域，最具影响力的技术标准机构。共计发布了200多项技术标准和实施指南，对互联网技术的发展和应用起到了基础性和根本性的支撑作用，官网：https://www.w3.org

# HTML 入门

## 标签

标签 又称 元素，是 HTML 的基本组成单位。
- 标签分为：双标签 与 单标签 （绝大多数都是双标签）
- 标签名不区分大小写，但推荐小写，因为小写更规范。
- 双标签：`<marquee>尚硅谷，让天下没有难学的技术！</marquee>`
- 单标签：`<input>` 或 `<input/>`。`/` 可以省略

HTML 标签属性
- 用于给标签提供 附加信息。
- 可以写在：起始标签 或 单标签中
- 有些特殊的属性，没有属性名，只有属性值 `<input disabled>`
- 注意点：
	- 不同的标签，有不同的属性；也有一些**通用属性**，在任何标签内都能写
	- 属性名、属性值不能乱写，都是 W3C 规定好的。
	- 属性名、属性值，都不区分大小写，但**推荐小写**。
	- 双引号，也可以写成单引号，甚至不写都行，但还是**推荐写双引号**。
	- 标签中不要出现同名属性，否则**后写的会失效**

## HTML 基本结构

- 想要呈现在网页中的内容写在 body 标签中。
- head 标签中的内容不会出现在网页中。
- head 标签中的 title 标签可以指定网页的标题

```html
<html>

	<head>
		<title>网页标题</title>
	</head>
	
	<body>
		......
	</body>

</html>
```

> 在网页中，如何查看某段结构的具体代码？—— 点击鼠标右键，选择“检查”。
> 
> 【检查】 和 【查看网页源代码】的区别：
> - 【查看网页源代码】看到的是：程序员编写的源代码。
> - 【检查】看到的是：经过**浏览器 “处理” 后**的源代码。
> - 备注：日常开发中，【检查】用的最多。
>   
>   比如在 head 标签里写 `<marquee>尚硅谷，让天下没有难学的技术！</marquee>`，检查查看后，发现会被移到 body 标签里
>   
>   浏览器的检查功能是有限的，错得太离谱也没用

## HTML 文档声明

- 作用：告诉浏览器当前网页的版本。
- 写法：
	- 旧写法：要依网页所用的 HTML 版本而定，写法有很多。
		- 具体有哪些写法请参考 ：W3C 官网-文档声明（了解即可，千万别背！）
	- 新写法：一切都变得简单了！**W3C 推荐使用 HTML 5 的写法**。
		- `<!DOCTYPE html>` 或
		- `<!DOCTYPE HTML>` 或
		- `<!doctype html>`
- 注意：文档声明，必须在网页的第一行，且在 html 标签的外侧

## HTML 字符编码

- 编码、解码，会遵循一定的规范 —— 字符集
- 使用原则
	- 存储时，务必采用合适的字符编码 。否则：无法存储，数据会丢失！
	- 存储时采用哪种方式编码 ，读取时就采用哪种方式解码。否则：数据错乱（乱码）！
- 平时编写代码时，统一采用 UTF-8 编码（最稳妥）。
- 为了让浏览器在渲染 html 文件时，不犯错误，可以通过 meta 标签配合 charset 属性指定字符编码。

```html
<head>
	<meta charset="UTF-8"/>
</head>
```

> 常见字符集
> 1. ASCII ：大写字母、小写字母、数字、一些符号，共计128个。
> 2. ISO 8859-1 ：在 ASCII 基础上，扩充了一些希腊字符等，共计是256个。
> 3. GB2312 ：继续扩充，收录了 6763 个常用汉字、682个字符。
> 4. GBK ：收录了的汉字和符号达到 20000+ ，支持繁体中文。
> 5. UTF-8 ：包含世界上所有语言的：所有文字与符号。—— 很常用。

## HTML 设置语言

- 主要作用：
	- 让浏览器显示对应的**翻译提示**。
	- 有利于搜索引擎优化。
- 具体写法：`<html lang="zh-CN">`

> 扩展知识： lang 属性的编写规则。
> 
> 1）第一种写法（ 语言-国家/地区 ），例如：
> - zh-CN ：中文-中国大陆（简体中文）
> - zh-TW ：中文-中国台湾（繁体中文）
> - zh ：中文
> - en-US ：英语-美国
> - en-GB ：英语-英国
> 
> 2）第二种写法（ 语言—具体种类）已不推荐使用，例如：
> - zh-Hans ：中文—简体
> - zh-Hant ：中文—繁体
> 
> W3School 上的说明：《语言代码参考手册》、《国家/地区代码参考手册》
> 
> W3C 官网上的说明：《Language tags in HTML》

## HTML 标准结构

- 配置 VScode 的内置插件 emmet ，可以对生成结构的属性进行定制。
- 在存放代码的文件夹中，存放一个 favicon.ico 图片，可配置网站图标

```html
<!DOCTYPE html>
<html lang="zh-CN">
	<head>
		<meta charset="UTF-8">
		<title>我是一个标题</title>
	</head>
	
	<body>
	
	</body>
</html>
```

# HTML 基础

开发者文档
- W3C官网： www.w3c.org
- W3School： www.w3school.com.cn
- MDN： [MDN Web Docs (mozilla.org)](https://developer.mozilla.org/zh-CN/) —— 火狐团队，平时用的最多。

## 排版标签

- `<h1>` ~ `<h6>`
	- 语义：标题
	- `<h1>` 最好写一个，  能适当多写。
	- `<h1>`~`<h6>` 不能互相嵌套（不强制）
- `<p>`
	- 语义：段落
	- 很特殊！它里面不能有： `<h1>`~`<h6>` 、`<p>` 、 `<div>`
- `<div>`
	- 语义：没有任何含义，用于整体布局（生活中的包装袋）。

> h1 里面写 h2，能运行，但是浏览器优化以后会把 h2 从 h1 里面拿出来

## 语义化标签（重要）

- 概念：用特定的标签，去表达特定的含义。
- 原则：**标签的默认效果不重要（后期可以通过 CSS 随便控制效果），语义最重要！**
- 举例：对于 h1 标签，效果是文字很大（不重要），语义是网页主要内容（很重要）。
- 优势：
	- 代码结构清晰可读性强。
	- 有利于 SEO（搜索引擎优化）、爬虫、机器人。
	- 方便设备解析（如屏幕阅读器、盲人阅读器等）。

## 块级元素 与 行内元素

- 块级元素：独占一行
	- **排版标签都是块级元素**
- 行内元素：不独占一行
- 使用原则：
	- **块级元素 中能写 行内元素 和 块级元素**
		- 简单记：块级元素中几乎什么都能写
	- **行内元素 中能写 行内元素**
	- 特殊的规则：
		- `<h1>`~`<h6>` 不能互相嵌套
		- `<p>` 中不要写块级元素

> 备注：marquee 元素设计的初衷是：让文字有动画效果，但如今我们可以通过 CSS 来实现了，而且还可以实现的更加炫酷，所以 marquee 标签已经：过时了（废弃了），不推荐使用。我们只是在开篇的时候，用他做了一个引子而已，在后续的学习过程中，这些已经废弃的标签，我们直接跳过。

## 文本标签

- 用于包裹：词汇、短语等。
- 通常写在排版标签里面。
- 排版标签更宏观（大段的文字），文本标签更微观（词汇、短语）

| 常用的文本标签名 | 标签语义 | 单 / 双 标签 |
| ---- | ---- | ---- |
| em | 要着重阅读的内容 | 双 |
| strong | 十分重要的内容（语气比em要强） | 双 |
| span | 没有语义，用于包裹短语的通用容器 | 双 |

不常用的文本标签

![](assets/Pasted%20image%2020240205125709.png)

- 这些不常用的文本标签，编码时不用过于纠结（酌情而定，不用也没毛病）。
- blockquote 与 address 是块级元素，其他的文本标签，都是行内元素。
- 有些语义感不强的标签，我们很少使用，例如：small、b、u、q、blockquote
- HTML标签太多了！记住那些：重要的、**语义感强**的标签即可；截止目前，有这些：h1~h6 、 p 、 div 、 em 、 strong 、 span

## 图片标签

### 基本使用

`<img>`
- 标签语义：图片
- 常用属性：
	- `src` ：图片路径（又称：图片地址）—— 图片的具体位置
	- `alt` ：图片描述
	- `width` ：图片宽度，单位是像素，例如： 200px 或 200
	- `height` ：图片高度， 单位是像素，例如： 200px 或 200
- 单标签

使用：
- 像素（ px ）是一种单位，CSS 中会详细讲解。
- 尽量不同时修改图片的宽和高，可能会造成比例失调。
	- **只修改其中一个，另一个会自动调节**
- 暂且认为 img 是“**行内元素**”
	- CSS 有一个新的元素分类，目前只知道：块、行内。
- alt 属性的作用：
	- 搜索引擎通过 alt 属性，得知图片的内容。—— 最主要的作用。
	- 当图片无法展示时候，有些浏览器会呈现 alt 属性的值。
	- 盲人阅读器会朗读 alt 属性的值。

### 路径的分类

- 相对路径：以**当前位置**作为参考点，去建立路径。
- 绝对路径：以**根位置**作为参考点，去建立路径。
	- 本地绝对路径： `E:/a/b/c/奥特曼.jpg` 。（很少使用）
	- 网络绝对路径： http://www.atguigu.com/images/index_new/logo.png 
	- 注意点：
		- 一旦更换设备，本地绝对路径 处理起来比较麻烦，所以很少使用。
		- 使用网络绝对路径，确实方便，但要注意：若服务器开启了**防盗链**，会造成图片引入失败。

### 常见图片格式

见课件

#todo 重要的：base64

## 超链接

`<a>`
- 语义：超链接
- 主要作用：从当前页面进行跳转。
	- 跳转到指定**页面**
	- 跳转到指定**文件**（也可触发下载）
	- 跳转到**锚点**位置
	- 唤起指定应用
- 常用属性：
	- `[href]` ： 指定要跳转到的具体目标。
	- `[target]` ： 控制跳转时如何打开页面，常用值如下:
		- `_self` ：在本窗口打开。
		- `_blank` ：在新窗口打开。
	- `[id]` ： 元素的唯一 标识，可用于**设置锚点**。
	- `[name]` ： 元素的名字，写在 a 标签中，也能设置锚点。
- 注意点：虽然 `<a>` 是**行内元素**，但它可以**包裹除它自身外的任何元素**！

---
1）跳转到页面

```html
<!-- 跳转其他网页 -->
<a href="https://www.jd.com/" target="_blank">去京东</a>
<!-- 跳转本地网页 -->
<a href="./10_HTML排版标签.html" target="_self">去看排版标签</a>
```

2）跳转到文件
- 若浏览器**无法打开文件，则会引导用户下载**
- `[download]` 强制触发下载，
	- 属性值：下载文件的名称。

```html
<!-- 浏览器能直接打开的文件 -->
<a href="./resource/自拍.jpg">看自拍</a>
<a href="./resource/小电影.mp4">看小电影</a>
<a href="./resource/小姐姐.gif">看小姐姐</a>
<a href="./resource/如何一夜暴富.pdf">点我一夜暴富</a>

<!-- 浏览器不能打开的文件，会自动触发下载 -->
<a href="./resource/内部资源.zip">内部资源</a>

<!-- 强制触发下载 -->
<a href="./resource/小电影.mp4" download="电影片段.mp4">下载电影</a>
```

3）跳转到锚点

锚点是网页中的一个标记点。
- 具有 `[href]` 的 `<a>` 是 超链接
- 具有 `[name]` 的 `<a>` 是 **锚点**

> name 和 id 都是区分大小写的，且 id 最好别是数字开头

第一步：设置锚点

```html
<!-- 第一种方式：a标签配合name属性 -->
<a name="test1"></a>

<!-- 第二种方式：其他标签配合id属性 -->
<h2 id="test2">我是一个位置</h2>
```

> HTML5 更推荐第二种方式

第二步：跳转锚点
-  `[href]` 中的 `#` 相当于**对锚点和路径的区分**

```html
<body>
  <br><br><br><br><br><br><br><br><br><br>
  <a name="test1">我是一个锚点test1</a>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <h2 id="test2">我是一个位置test2</h2>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>
  <br><br><br><br><br><br><br><br><br><br>

  <a href="#test1">跳转到test1锚点</a>
  <a href="#test2">跳转到test2位置</a>

  <a href="#">回到本页面顶部</a>

  <a href="demo2.html#test1">跳到其他页面的锚点</a>

  <a href="">刷新本页面</a>

  <!-- 
    执行一段js,如果还不知道执行什么，可以留空，
    语法：javascript:<js代码>; 
  -->
  <a href="javascript:alert(1);">点我弹窗</a>

</body>
```

4）唤起指定应用

通过 a 标签，可以唤起设备应用程序。

```html
<!-- 唤起设备拨号 -->
<a href="tel:10010">电话联系</a>
<!-- 唤起设备发送邮件 -->
<a href="mailto:10010@qq.com">邮件联系</a>
<!-- 唤起设备发送短信 -->
<a href="sms:10086">短信联系</a>
```

## 列表

有序列表：有顺序或侧重顺序的列表。
```html
<h2>要把大象放冰箱总共分几步</h2>
<ol>
	<li>把冰箱门打开</li>
	<li>把大象放进去</li>
	<li>把冰箱门关上</li>
</ol>
```

无序列表：无顺序或不侧重顺序的列表。
```html
<h2>我想去的几个城市</h2>
<ul>
	<li>成都</li>
	<li>上海</li>
	<li>西安</li>
	<li>武汉</li>
</ul>
```

自定义列表
1. 概念：所谓自定义列表，就是一个包含术语名称以及术语描述的列表。
2. 一个 dl 就是一个自定义列表，一个 dt 就是一个术语名称，一个 dd 就是术语描述（可以有多个）

```html
<h2>如何高效的学习？</h2>
<dl>
	<dt>做好笔记</dt>
	<dd>笔记是我们以后复习的一个抓手</dd>
	<dd>笔记可以是电子版，也可以是纸质版</dd>
	<dt>多加练习</dt>
	<dd>只有敲出来的代码，才是自己的</dd>
	<dt>别怕出错</dt>
	<dd>错很正常，改正后并记住，就是经验</dd>
</dl>
```

列表嵌套
- li 标签最好写在 ul 或 ol 中，不要单独使用。

```html
  <h2>我想去的几个城市</h2>
  <ul>
    <li>成都</li>
    <li>
      <span>上海</span>
      <ul>
        <li>外滩</li>
        <li>杜莎夫人蜡像馆</li>
        <li>
          <a href="https://www.opg.cn/">东方明珠</a>
        </li>
        <li>迪士尼乐园</li>
      </ul>
    </li>
    <li>西安</li>
    <li>武汉</li>
  </ul>
```

## 表格

### 基本结构

一个完整的表格由：表格标题、表格头部、表格主体、表格脚注，四部分组成。

![](assets/Pasted%20image%2020240227012436.png)

表格涉及到的标签：
- `<table>` ：表格
	- width ：设置表格宽度。
		- 默认情况下，每列的宽度，得看**这一列单元格最长的那个文字**。
	- height ：设置表格**最小高度**，表格最终高度可能比设置的值大。
	- border ：设置表格边框宽度。
		- 可以控制表格边框（0就是全没边框，1有边框），但值的大小，并不控制单元格边框的宽度（只控制表格最外圈）
	- cellspacing ： 设置单元格间距
- `<caption>` ：表格标题
- `<thead>` ：表格头部
	- height ：设置表格头部高度。
	- align ： 设置单元格的**水平**对齐方式，可选值如下：left, center, right
	- valign ：设置单元格的**垂直**对齐方式，可选值如下：top, middle, bottom
- `<tbody>` ：表格主体
	- 常用属性与 `<thead>` 相同
- `<tfoot>` ：表格注脚
	- 常用属性与 `<thead>` 相同
- `<tr>` ：每一行（row）
	- 常用属性与 `<thead>` 相同
- `<th>`, `<td>` ：每一个单元格
	- 表格头部中用 `<th>`（head） ，表格主体、表格脚注中用： `<td>`（data）
	- width ：设置单元格的宽度，同列所有单元格全都受影响
		- 给某个 `<th>` 或 `<td>` 设置了宽度之后，他们所在的那一列的宽度就确定了
	- height ：设置单元格的高度，同行所有单元格全都受影响
		- 给某个 `<th>` 或 `<td>` 设置了高度之后，他们所在的那一行的高度就确定了。
	- align ：设置单元格的水平对齐方式
	- valign ：设置单元格的垂直对齐方式
	- **rowspan** ：指定要跨的行数
	- **colspan** ：指定要跨的列数

> 属性可以不记，CSS 都可以控制

![](assets/Pasted%20image%2020240227012623.png)

```html
<body>
  <table border="5">
    <!-- 表格标题 -->
    <caption>学生信息</caption>
    <!-- 表格头部 -->
    <thead>
      <tr>
        <th>姓名</th>
        <th>性别</th>
        <th>年龄</th>
        <th>民族</th>
        <th>政治面貌</th>
      </tr>
    </thead>
    <!-- 表格主体 -->
    <tbody>
      <tr>
        <td>张三</td>
        <td>男</td>
        <td>18</td>
        <td>汉族</td>
        <td>团员</td>
      </tr>
      <tr>
        <td>李四</td>
        <td>女</td>
        <td>20</td>
        <td>满族</td>
        <td>群众</td>
      </tr>
      <tr>
        <td>王五</td>
        <td>男</td>
        <td>20</td>
        <td>回族</td>
        <td>党员</td>
      </tr>
      <tr>
        <td>赵六</td>
        <td>女</td>
        <td>21</td>
        <td>壮族</td>
        <td>团员</td>
      </tr>
    </tbody>
    <!-- 表格脚注 -->
    <tfoot>
      <tr>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>共计：4人</td>
      </tr>
    </tfoot>
  </table>
</body>
```

## 常用标签补充

- `</br>` 
	- 语义：换行
	- **不要用来增加文本之间的行间隔**，应使用 `<p>` 元素，或后面即将学到的 CSS margin 属性。
- `</hr>` 
	- 语义：分隔
	- 如果不想要语义，只是想画一条水平线，那么应当使用 CSS 完成
- `<pre>` 
	- 语义：按原文显示（一般用于在页面中嵌入大段代码）

## 表单

`<form>` 语义：表单
- `[action]` ：用于指定表单的**提交地址**（需要与后端人员沟通后确定）
- `[target]` ：用于控制表单**提交后**，**如何打开页面**，常用值如下：
	- `_self` ：在本窗口打开。
	- `_blank`：在新窗口打开。
- `[method]` ：用于控制表单的**提交方式**（暂时只需了解，在后面Ajax 的课程中，会详细讲解）
- `</input>` 语义：输入框
	- `[name]` ：用于指定**提交数据的名字**（需要与后端人员沟通后确定）
	- `[type]` ：设置输入框的类型
		- `text`：普通文本
			- `[value]` 输入框的默认输入值
		- `password`：密码
			- `[value]` 输入框的默认输入值
		- `radio`：单选框
			- 想要单选效果，**多个 radio 的 name 属性值要保持一致**
			- `[value]` 提交的数据值
				- **要写，不然没意义**
			- `[checked]` ：让该单选按钮默认选中。
		- `checkbox`：复选框
			- `[value]` 提交的数据值
				- **要写，不然没意义**
			- `[checked]` ：让该单选按钮默认选中。
		- `hidden`：用户不可见的一个输入区域，
			- `[value]` 提交的数据值
			- 作用： 提交表单的时候，携带一些固定的数据。
		- 实现按钮：
			- 属性值：
				- `submit`：提交
				- `reset`：重置
				- `button`：普通按钮
			- `[value]` 指定按钮文字
			- **一般不要 `[name]` 属性**
	- `[maxlength]` ：输入框最大可输入长度
- `<button>` 语义：按钮
	- `[type]`
		- 属性值：
			- `submit`：提交（**默认**）
			- `reset`：重置
			- `button`：普通按钮
	- **一般不要 `[name]` 属性
- `<textarea>` 语义：文本域
	- `[name]` ：用于指定**提交数据的名字**（需要与后端人员沟通后确定）
	- `[rows]`：指定默认显示的行数，会影响文本域的高度
	- `[cols]`：指定默认显示的列数，会影响文本域的宽度
- `<select>` 语义：下拉框
	- `[name]` ：用于指定**提交数据的名字**（需要与后端人员沟通后确定）
	- `<option>` 语义：选项
		- `[value]` 提交的数据值
			- 建议设置。如果没有设置，提交的数据是 `<option>` 中间的文字
		- `[selected]` 默认选中

`[disabled]`：禁用表单控件
- `<input>` 、 `<textarea>` 、 `<button>` 、 `<select>` 、 `<option>` 都可以设置

`<label>` 语义：可与表单控件相关联
- 关联之后点击标签文字，与之对应的表单控件就会**获取焦点。**
- 两种与 `<label>` 关联方式：
	1. 让 `<label>` 的 `[for]` 等于表单控件的 `[id]` 
	2. 把表单控件套在 `<label>` 的里面
- 一般不和“按钮”关联，没必要

`<fieldset>` 语义：表单边框
- 可以为表单控件**分组**
- `<legend>` 标签是分组的标题

表单中提交、重置、普通按钮都有 `</input>` 和 `<button>` 两种实现

```html
<input type="submit" value="点我提交表单">
<button>点我提交表单</button>

<input type="reset" value="点我重置">
<button type="reset">点我重置</button>

<input type="button" value="普通按钮">
<button type="button">普通按钮</button>
```

完整案例

```html
<form target="_blank">
    <fieldset>
      <legend>主要信息</legend>
      <label for="zhanghu">账户：</label>
      <input id="zhanghu" type="text" name="wd" value="zhangsan">
      <br>

      <label>
        密码：<input type="password" name="pwd" value="123">
      </label>
      <br>

      性别：
      <label><input type="radio" name="gender" value="male">男</label>
      <label><input type="radio" name="gender" value="female" checked>女</label>
    </fieldset>

    <fieldset>
      <legend>其他信息</legend>
      兴趣：
      <input disabled type="checkbox" name="hobby" value="smoke">抽烟
      <input type="checkbox" name="hobby" value="drink" checked>喝酒
      <input type="checkbox" name="hobby" value="perm" checked>烫头
      <br>

      其他：<textarea name="other" cols="23" rows="10">请补充信息</textarea>
      <br>

      籍贯：
      <select name="from">
        <option value="黑">黑龙江</option>
        <option value="辽">辽宁</option>
        <option value="吉">吉林</option>
        <option value="粤" selected>广东</option>
      </select>

      <input type="hidden" name="tag" value="100">
    </fieldset>

    <button type="button">检测账户是否被注册</button>
    <button>去京东搜索</button>
    <button type="reset">重置</button>
</form>
```

![](assets/Pasted%20image%2020240228004752.png)

## 框架标签

`<iframe>`  语义：框架（在网页中嵌入其他文件）
- `[name]` ：框架名字，**可以与 `[target]` 属性配合**
- `[width]` ： 框架的宽
- `[height]` ： 框架的高度
- `[frameborder]` ：**是否**显示边框，值：0 或者 1

实际应用
- 在网页中嵌入广告
- 与“超链接”或“表单”的 `[target]` 配合，展示不同的内容

```html
<iframe src="https://www.bilibili.com" height="400" width="900" frameborder="1">
</iframe><br>

<!-- 与超链接的target属性配合使用 -->
<a href="https://www.taobao.com" target="container">点我看新闻</a>

<!-- 与表单的target属性配合使用 -->
<form action="https://so.toutiao.com/search" target="container">
<input type="text" name="keyword">
<button>搜索</button>
</form>

<iframe name="container" height="400" width="900" frameborder="1"></iframe>
```

## 字符实体

在 HTML 中我们可以用一种**特殊的形式的内容，来表示某个符号**，这种特殊形式的内容，就是 “HTML 实体”。比如小于号 `<` 用于定义 HTML 标签的开始。如果我们希望浏览器正确地显示这些字符，我们必须在 HTML 源码中插入字符实体。

字符实体由三部分组成：
- 一个 `&` 
- （一个实体名称） 或者 （一个 `#` 和一个 实体编号）
- 最后加上一个分号 `;` 

![](assets/Pasted%20image%2020240228123708.png)

## HTML 全局属性

> 有很多，其他见 MDN

- `[id]` 给标签指定唯一标识
	- 注意： 
		- id 是**不能重复**的
		- 不能在以下 HTML 元素中使用 `<head>`、`<html>`、`<style>`、`<title>`、`<meta>`、`<script>`
	- 作用：
		- 让 `<label>` 标签与表单控件相关联；
		- 与 CSS 、 JavaScript 配合使用，

- `[class]` 给标签指定类名，
	- 随后通过 CSS 就可以给标签设置样式

- `[style]` 给标签设置 CSS 样式

- `[dir]` 内容的方向，
	- 值: `ltr` 、 `rtl`
	- 注意：不能在以下 HTML 元素中使用 `<head>`、`<html>`、`<style>`、`<title>`、`<meta>`、`<script>`

- `[title]` 给标签设置一个**文字提示**
	- 一般超链接和图片用得比较多

- `[lang]` 给**标签**指定语言
	- 具体规范和可选值请参考 [HTML 设置语言](#HTML%20设置语言)
	- 注意：不能在以下 HTML 元素中使用 `<head>`、`<html>`、`<style>`、`<title>`、`<meta>`、`<script>`

## meta 元信息

> 基本信息

配置字符编码

针对 IE 浏览器的兼容性配置。

针对移动端的配置（移动端课程中会详细讲解）

配置网页关键字

配置网页描述信息

针对搜索引擎爬虫配置：

配置网页作者：

配置网页生成工具

配置定义网页版权信息：

配置网页自动刷新

# CSS


