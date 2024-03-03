
# ---------- HTML 简介

全称：HyperText Markup Language（超文本标记语言）。
- 超文本：是一种组织信息的方式，通过超链接将不同空间的文字、图片等各种信息组织在一起，能从当前阅读的内容，跳转到超链接所指向的内容。
- 标记：文本要变成超文本，就需要用到各种标记符号。
- 语言：每一个标记的写法、读音、使用规则，组成了一个标记语言。

从 HTML 1.0 开始发展，期间经历了很多版本，目前 HTML 的最新标准是：HMTL 5，具体发展史如图

![](assets/Pasted%20image%2020240204134556.png)

> WSC：World Wide Web Consortium（万维网联盟），创建于1994年，是目前Web技术领域，最具影响力的技术标准机构。共计发布了200多项技术标准和实施指南，对互联网技术的发展和应用起到了基础性和根本性的支撑作用，官网：https://www.w3.org

# ---------- HTML 入门

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

# ---------- HTML 基础

> 开发者文档
> - W3C官网： www.w3c.org
> - W3School： www.w3school.com.cn
> - MDN： [MDN Web Docs (mozilla.org)](https://developer.mozilla.org/zh-CN/) —— 火狐团队，平时用的最多。

## 排版标签

- `<h1>` ~ `<h6>`
	- 语义：标题
	- `<h1>` 最好写一个，  能适当多写。
	- `<h1>`~`<h6>` 不能互相嵌套（不强制）
- `<p>`
	- 语义：段落
	- 很特殊！它里面**不能有**： `<h1>`~`<h6>`、`<p>`、`<div>`
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
- 通常写在**排版标签里面**。
- 排版标签更宏观（大段的文字），文本标签更微观（词汇、短语）

| 常用的文本标签名 | 标签语义             | 单 / 双 标签 |
| -------- | ---------------- | -------- |
| em       | 要着重阅读的内容         | 双        |
| strong   | 十分重要的内容（语气比em要强） | 双        |
| span     | 没有语义，用于包裹短语的通用容器 | 双        |


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

```html
<meta charset="utf-8">
```

针对 IE 浏览器的兼容性配置。

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

针对移动端的配置（移动端课程中会详细讲解）

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

配置网页关键字
```html
<meta name="keywords" content="8-12个以英文逗号隔开的单词/词语">
```

配置网页描述信息
```html
<meta name="description" content="80字以内的一段话，与网站内容相关">
```

针对搜索引擎爬虫配置：
```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

![](assets/Pasted%20image%2020240228161216.png)

配置网页作者：
```html
<meta name="author" content="tony">
```

配置网页生成工具
```html
<meta name="generator" content="Visual Studio Code">
```

配置定义网页版权信息：
```html
<meta name="copyright" content="2023-2027©版权所有">
```

配置网页自动刷新
```html
<meta http-equiv="refresh" content="10;url=http://www.baidu.com">
```

# ---------- CSS 基础

## 简介

![](assets/Pasted%20image%2020240228162035.png)

CSS 的全称为：*层叠样式表 ( Cascading Style Sheets )* 。

CSS 也是一种**标记语言**，用于给 HTML 结构设置样式，例如：文字大小、颜色、元素宽高等等。

简单理解： CSS 可以美化 HTML , 让 HTML 更漂亮。

核心思想： HTML 搭建结构， CSS 添加样式，实现了：**结构与样式的分离**

## 编写位置

### 行内样式

> 又称：内联样式

- 位置：写在标签的 `[style]` 中
- 特点：**只能控制当前标签的样式**，对其他标签无效。
- 语法：
	-  `[style]` 的值要符合 CSS 语法规范，是 `名:值;` 的形式。

```html
<h1 style="color:red;font-size:60px;">欢迎来到尚硅谷学习</h1>
```

- 适用场景：只有**对当前元素添加简单样式**时，才偶尔使用。
- 缺点：
	- 书写繁琐、样式**不能复用**
	- 并且没有体现出：结构与样式分离 的思想，

### 内部样式

- 位置：写在 html 页面内部的 `<style>` 标签中。
	- `<style>` 理论上可以放在 HTML 文档的任何地方，但**一般都放在 `<head>` 中**
- 语法：

```html
<style>
	h1 {
		color: red;
		font-size: 40px;
	}
</style>
```

- 优点：
	- **样式可以复用**
	- 代码结构清晰
- 缺点：
	- 并没有实现：结构与样式完全分离
	- **多个 HTML 页面无法复用样式**

### 外部样式

- 位置：写在单独的 `.css` 文件中，随后在 HTML 文件中通过 `<link>` **引入使用**
- 语法：
	- `<link>` 要写在 `<head>` 中
		- `[href]` ：引入的文档**来自于哪里**
		- `[rel]` ：( relation ：关系）说明“引入的文档”与“当前文档”之间的**关系**
- 优势：
	- 样式可以复用、
	- 结构清晰、
	- 可触发浏览器的缓存机制，提高访问速度 ，
	- 实现了结构与样式的**完全分离**。
- 实际开发中，几乎都使用外部样式，这是**最推荐的使用方式！**

`.css` 文件的书写

```css
h1{
	color: red;
	font-size: 40px;
}
```

HTML 文件中引入 `.css` 文件

```html

<link rel="stylesheet" href="./xxx.css">
```

### 优先级

优先级规则：行内样式 > 内部样式 = 外部样式
- 行内样式无关顺序，就是最高
- 内部样式、外部样式，优先级和编写顺序有关，**后面的 会覆盖 前面的**
- 同一个样式表中，优先级也和编写顺序有关， **后面的 会覆盖 前面的**

```css
h1{
  color: blue;
}
```

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    h1 {
      color: red;
    }
  </style>
  <link rel="stylesheet" href="./color.css">
</head>

<body>
  <!-- 外部样式覆盖内部样式，蓝色 -->
  <h1>hello</h1>
</body>
```

## 语法规范

CSS 语法规范由两部分构成：
- *选择器*：**找到**要添加样式的**元素**。
- *声明块*：设置具体的样式（声明块是由一个或多个声明组成的），
	- 声明的格式为： `属性名:属性值;`
	- 最好写上：
		- 最后一个声明后的 `;`
		- 选择器与声明块之间，属性名与属性值之间，均有一个**空格**

![](assets/Pasted%20image%2020240228182528.png)

CSS 中的注释：`/* */`

## 代码风格

展开风格 —— 开发时推荐，便于维护和调试。

```css
h1 {
	color: red;
	font-size: 40px;
}
```

紧凑风格 —— 项目上线时推荐，可减小文件体积。

```css
h1{color:red;font-size:40px;}
```

> 项目上线时，我们会通过工具将【展开风格】的代码，变成【紧凑风格】，这样可以减小文件体积，节约网络流量，同时也能让用户打开网页时速度更快

# ---------- CSS 选择器

## 基本选择器

### 通配选择器

作用：可以选中**所有的 HTML 元素**

```css
* {
	属性名: 属性值;
}
```

> 清除样式时，会对我们有很大帮助

### 元素选择器

作用：为页面中 **某种元素** **统一**设置样式

```css
标签名 {
	属性名: 属性值;
}

/* 选中所有h1元素 */
h1 {
	color: orange;
	font-size: 40px;
}
/* 选中所有p元素 */
p {
	color: blue;
	font-size: 60px;
}
```

### 类选择器

作用：根据**元素的 `[class]` 值**，来选中**某些**元素。

```css
.类名 {
	属性名: 属性值;
}

/* 选中所有class值为speak的元素 */
.speak {
	color: red;
}
/* 选中所有class值为answer的元素 */
.answer {
	color: blue;
}
```

- `[class]` 值的标准：
	- 不要使用纯数字、不要使用中文、尽量使用英文与数字的组合，
	- 若由多个单词组成，使用 `-` 做连接，例如：`left-menu` ，
	- 且命名要有意义，做到 “见名知意”。
- 一个元素不能写多个 `[class]`
- 一个元素的 `[class]` 能写多个值，要用**空格**隔开

```html
<h1 class="speak big">你好啊</h1>
```

### ID 选择器

作用：根据**元素的 `[id]` 值**，来精准的选中**某个**元素。

```css
#id值 {
	属性名: 属性值;
}

/* 选中id值为earthy的那个元素 */
#earthy {
	color: red;
	font-size: 60px;
}
```

- `[id]` 值：
	- 尽量由字母、数字、 `_` 、`-` 组成，
	- 最好以字母开头、
	- 不要包含空格、
	- 区分大小写
- 一个元素**只能拥有一个 `[id]`**，多个元素的 `[id]` 值不能相同。
- 一个元素可以**同时拥有 `[id]` 和 `[class]`** 

## 复合选择器

复合选择器建立在基本选择器之上，由多个基础选择器，通过**不同的方式组合**而成。

### 交集选择器

作用：选中**同时符合**多个条件的元素

```css
选择器1选择器2选择器3...选择器n {

}

/* 选中：类名为beauty的p元素，为此种写法用的非常多！！！！ */
p.beauty {
	color: blue;
}

/* 选中：类名包含rich和beauty的元素 */
.rich.beauty {
	color: green;
}
```

注意：
- 有标签名，**标签名必须写在前面。**
- **“id 选择器”、“通配选择器”**，实际应用中**几乎不用** —— 因为没有意义
	- id已经唯一了，不需要交集了
- 交集选择器中**不可能出现两个元素选择器**
	- 例如：一个元素，不可能即是 p 元素又是 span 元素
- 用的最多：**“元素选择器”配合“类名选择器”**，
	- 例如： `p.beauty`

### 并集选择器

> 又称：分组选择器。

作用：选中**多个选择器**对应的元素

```css
选择器1, 选择器2, 选择器3, ... 选择器n {

}

/* 选中id为peiqi，或类名为rich，或类名为beauty的元素 */
#peiqi,
.rich,
.beauty {
	font-size: 40px;
	background-color: skyblue;
	width: 200px;
}
```

注意：
- 一般竖着写（多种复合选择器组合的时候方便区分）
- **任何形式的选择器**，都可以作为并集选择器的一部分 。
- 并集选择器，通常用于**集体声明**，可以缩小样式表体积。

### 结构选择器

#### HTML 元素间的关系

- 父元素：直接包裹某个元素的元素，就是该元素的父元素。
- 子元素：被父元素直接包含的元素（简记：儿子元素）。
- 祖先元素：父亲的父亲......，一直往外找，都是祖先
	- 父元素，也算是祖先元素的一种
- 后代元素：儿子的儿子......，一直往里找，都是后代
	- 子元素，也算是后代元素的一种。
- 兄弟元素：具有相同父元素的元素，互为兄弟元素

#### 后代选择器

作用：选中指定元素中，符合要求的**后代元素**

注意：
- 结构一定要符合之前讲的 HTML 嵌套要求
	- 例如：不能 p 中写 `<h1>` ~ `<h6>` 

```css
/* 
空格隔开
先写祖先，再写后代
*/
选择器1 选择器2 选择器3 ...... 选择器n {

}

/* 选中ul中所有li中的a */
ul li a {
	color: orange;
}

/* 选中类名为subject元素中的所有类名为front-end的li */
.subject li.front-end {
	color: blue;
}
```

#### 子代选择器

作用：选中指定元素中，符合要求的**子元素**

```css
选择器1 > 选择器2 > 选择器3 > ...... 选择器n {

}

/* div中的子代a元素 */
div>a {
	color: red;
}

/* 类名为persons的元素中的子代a元素 */
.persons>a{
	color: red;
}
```

区别后代和子代！
```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div>a {
      color: red;
      font-size: 30px;
    }
  </style>
</head>

<body>
  <div>
    <a href="#">儿子</a>
    <p>
      <a href="#">孙子</a>
    </p>
  </div>
</body>
```

#### 兄弟选择器

1）相邻兄弟选择器
- 作用：选中指定元素后，符合条件的**紧挨着的下一个兄弟**元素。

```css
选择器1+选择器2 {

}

/* 选中div后相邻的兄弟p元素 */
div+p {
	color:red;
}
```

2）通用兄弟选择器：
- 作用：选中指定元素后，符合条件的**下面的所有**兄弟元素。

```css
选择器1~选择器2 {

}

/* 选中div后的所有的兄弟p元素 */
div~p {
	color:red;
}
```

---
```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    /* li+li也行 */
    li~li {
      color: blue;
    }
  </style>
</head>

<body>
  <ul>
    <li>black</li>
    <li>blue</li>
    <li>blue</li>
    <li>blue</li>
  </ul>
</body>
```

### 属性选择器

作用：选中**属性值符合一定要求**的元素

语法：
- `[属性名]` 选中**具有**某个属性的元素。
- `[属性名="值"]` 选中包含某个属性，且属性值**等于**指定值的元素。
- `[属性名^="值"]` 选中包含某个属性，且属性值以指定的值**开头**的元素。
- `[属性名$="值"]` 选中包含某个属性，且属性值以指定的值**结尾**的元素。
- `[属性名*=“值”]` 选择包含某个属性，属性值**包含**指定值的元素。

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    [title]{
      color: red;
    }
    [title="python"]{
      color: blue;
    }
    [title^="s"]{
      color: green;
    }
    [title$="t"]{
      color: pink;
    }
    [title*="script"]{
      color: yellow;
    }
  </style>
</head>

<body>
  <div title="java">red</div>
  <div title="python">blue</div>
  <div title="spring">green</div>
  <div title="springboot">pink</div>
  <div title="javascript">yellow</div>
</body>
```

### 伪类选择器

作用：选中**特殊状态**的元素。

#### 动态伪类

常用：
- `:link` “超链接”**未被访问**的状态。
- `:visited` “超链接”**访问过**的状态。
- `:hover` “鼠标”**悬停**在元素上的状态。
- `:active` “元素”**激活**的状态。

- `:focus` 获取焦点的元素。
	- 表单类元素才能使用

前四个遵循 **LVHA** 的顺序！

> 激活：按下鼠标不松开
> 
> 用户 点击元素、触摸元素、或通过键盘的 “ tab ” 键等方式，选择元素时，就是获得焦点。

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    a:link {
      color: blue;
    }

    a:visited {
      color: pink;
    }

    a:hover {
      color: orange;
    }

    a:active {
      color: green;
    }

    input:focus,
    select:focus {
      background-color: green;
    }
  </style>
</head>

<body>
  <a href="#1">hello</a>
  <a href="#2">hello</a>
  <a href="#3">hello</a>

  <input type="text">
  <select>
    <option>1</option>
    <option>2</option>
  </select>
</body>
```

#### 结构伪类

常用：
- `:first-child` 所有“兄弟元素”中的**第一个**元素
- `:last-child` 所有“兄弟元素”中的**最后一个**元素
- `:nth-child(n)` 所有“兄弟元素”中的**第 n 个**元素

- `:first-of-type` 所有“**同类型**兄弟元素”中的**第一个**元素
- `:last-of-type` 所有“**同类型**兄弟元素”中的**最后一个**元素
- `:nth-of-type(n)` 所有“**同类型**兄弟元素”中的 **第 n 个** 元素

n的值：
- `0 或 不写` ：什么都选不中 —— 几乎不用。
- `n` ：选中所有子元素 —— 几乎不用。
- `1 ~ 正无穷的整数` ：选中对应序号的子元素。
- `2n 或 even` ：选中序号为偶数的子元素。
- `2n+1 或 odd` ：选中序号为奇数的子元素。
- `-n+i` ：选中的是前 i 个。

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    /* 结构1，2 */
    /* div的儿子p元素，且是第一个儿子！（不是第一个p儿子） */
    div.c12>p:first-child {
      color: blue;
    }

    /* 结构3 */
    /* div的后代p元素，且p是父亲元素的第一个儿子 */
    div.c3 p:first-child {
      color: red;
    }

    /* 结构4 */
    /* p元素，且是父亲元素的第一个儿子 */
    p:first-child {
      color: pink;
    }
  </style>
</head>

<body>
  <!-- red：父级body的第一个儿子 -->
  <p>结构4</p>
  <div>
    <!-- red：父级div的第一个儿子 -->
    <p>结构4</p>
    <marquee>
      <!-- red：父级marquee的第一个儿子 -->
      <p>结构4</p>
      <p>结构4</p>
    </marquee>
    <marquee>
      <span>结构4</span>
      <p>结构4</p>
    </marquee>
    <p>结构4</p>
  </div>
  <hr>

  <div class="c12">
    <!-- red -->
    <p>结构1，2</p>
    <p>结构1，2</p>
  </div>
  <hr>
  <div>
    <!-- black：div的第一个儿子，但不是p -->
    <span>结构1，2</span>
    <!-- black：div的第二个儿子 -->
    <p>结构1，2</p>
  </div>
  <hr>

  <div class="c3">
    <!-- red：父级div的第一个儿子 -->
    <p>结构3</p>
    <marquee>
      <!-- red：父级marquee的第一个儿子 -->
      <p>结构3</p>
      <p>结构3</p>
    </marquee>
    <marquee>
      <span>结构3</span>
      <p>结构3</p>
    </marquee>
    <p>结构3</p>
  </div>
</body>
```

了解：
- `:nth-last-child(n)` 所有兄弟元素中的倒数第 n 个
- `:nth-last-of-type(n)` 所有同类型兄弟元素中的 倒数第n个 
- `:only-child` 选择没有兄弟的元素（独生子女）
- `:only-of-type` 选择没有同类型兄弟的元素
- `:root` 根元素
- `:empty` 内容为空元素（空格也算内容）

#### 否定伪类

`:not(选择器)`：排除满足括号中条件的元素。

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    /* :not(属性选择器) */
    div>p:not([title^=a]){
      color: red;
    }
  </style>
</head>

<body>
  <div>
    <p>hello</p>
    <p title="a">hello</p>
    <p title="aa">hello</p>
  </div>
</body>
```

#### UI伪类

- `:checked` 被**选中**的“复选框”或“单选按钮”
- `:enable` 没有 `[disabled]` 的“表单”元素
- `:disabled` 有 `[disabled]` 的“表单”元素

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    input:checked{
      width: 100px;
      height: 100px;
    }
    input:disabled{
      background-color: gray;
    }
    input:enabled{
      background-color: green;
    }
  </style>
</head>

<body>
  <div>
    <input type="checkbox">
    <input type="radio" name="a">
    <input type="radio" name="a">

    <input type="text">
    <input type="text" disabled>
  </div>
</body>
```

#### 目标伪类（了解）

`:target` 选中**锚点**指向的元素

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div{
      height: 300px;
      background-color: gray;
    }
    div:target{
      background-color: green;
    }
  </style>
</head>

<body>
  <a href="#1">1</a>
  <a href="#2">2</a>

  <div id="1"></div>
  <br>
  <div id="2"></div>
</body>
```

#### 语言伪类（了解）

`:lang()` 根据指定的语言选择元素（本质是看 `lang` 属性的值）

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div:lang(en){
      color:red;
    }
    :lang(zh-CN){
      color: blue;
    }
  </style>
</head>

<body>
  <div>hello</div>
  <p lang="zh-CN">你好</p>
  <span>你好呀</span>
</body>

</html>
```

### 伪元素选择器

> 选中的不是元素

作用：选中**元素**中的一些**特殊位置**

常用：
- `::first-letter` 选中元素中的**第一个文字**
- `::first-line` 选中元素中的**第一行文字**
- `::selection` 选中被**鼠标选中**的内容
- `::placeholder` 选中**输入框的提示文字**
- `::before` 在元素**最开始**的位置，创建一个子元素（必须用 `[content]` 指定内容）
- `::after` 在元素**最后**的位置，创建一个子元素（必须用 `[content]` 指定内容）

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div::first-line {
      background-color: yellow;
    }
    p::before{
      content: "￥";
    }
    p::after{
      content: ".00";
    }
  </style>
</head>

<body>
  <div>
    老天爷如果擦亮双眼仔细观看，还会看到未来的宗教精神导师马丁·路德正在威顿堡大学慷慨激昂地鼓吹他自己的宗教思想。如果他专心于中国，则会看到广西柳州的农民起义被血腥镇压，看到山东曹州的农民正在掀起抗暴的烽火，还能看到已上任三年的皇帝朱厚照（明武宗）正在紫禁城里不眠不休地纵欲。
  </div>

  <p>199</p>
  <p>299</p>
  <p>399</p>
</body>
```

## 选择器的优先级

通过不同的选择器，选中相同的元素 ，并且为**相同的样式名设置不同的值**时，就发生了**样式的冲突**。到底应用哪个样式，此时就需要看优先级了。

> 样式不一样就没有冲突了

基本选择器：**行内样式 > ID 选择器 > 类选择器 > 元素选择器 > 通配选择器**

> 不是同类型的选择器，就不能“后来者居上”
> 
> 越精准优先级越高
> 
> 元素在食物链的底层

复合选择器：
- 计算方式：每个选择器，都可计算出一组**权重**，格式为： `(a,b,c)`
	- a : **ID** 选择器的个数。
	- b : **类、伪类、属性** 选择器的个数。
	- c : **元素、伪元素** 选择器的个数。
		- `ul>li` `(0,0,2)`
		- `div ul>li p a span` `(0,0,6)`
		- `#atguigu .slogan` `(1,1,0)`
		- `#atguigu .slogan a` `(1,1,1)`
		- `#atguigu .slogan a:hover` `(1,2,1)`
- 比较规则：按照从左到右的顺序，依次比较大小，当前位胜出后，后面的不再对比
	- `(1,0,0) > (0,2,2)`
	- `(1,1,0) > (1,0,3)`
	- `(1,1,3) > (1,1,2)`
- 特殊规则：
	- **行内样式**权重大于所有选择器
	- `!important` 的权重，大于行内样式，大于所有选择器，**权重最高！**

> 并集选择器的每一个部分是分开算的！

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div {
      color: red !important;
    }

    .aa {
      color: blue;
    }

    #aa {
      color: green;
    }
  </style>
</head>

<body>
  <div id="aa" class="aa" style="color: gray;">
    hello
  </div>
</body>
```

# ---------- CSS 三大特性

## 层叠性

概念：如果发生了**样式冲突**，那就会根据一定的规则（选择器优先级），进行样式的层叠（**覆盖**）

> 样式冲突：元素的同一个样式名，被设置了不同的值

## 继承性

概念：元素会自动拥有其父元素、或其**祖先元素**上所设置的**某些样式**

规则：优先继承**离得近**的。

常见的可继承属性：`text-??`，` font-??`， `line-??`、`color` ......

## 优先级

`!important` > 行内样式 > ID 选择器 > 类选择器 > 元素选择器 > `*` > **继承的样式**

见[选择器的优先级](#选择器的优先级)

# ---------- CSS 常用属性

## 像素

概念：我们的电脑屏幕是，是由一个一个“小点”组成的，每个“小点”，就是一个像素（px）

规律：像素点**越小**，呈现的内容就**越清晰、越细腻**

![](assets/Pasted%20image%2020240301020410.png)

> 注意点：如果电脑设置中开启了缩放，那么就会**影响一些工具的测量结果**，但这无所谓，因
> 为我们工作中都是参考详细的设计稿，去给元素设置宽高。

## 颜色的表示

### 颜色名

编写方式：直接使用颜色对应的英文单词，编写比较简单，例如：
1. 红色：red
2. 绿色：green
3. 蓝色：blue
4. 紫色：purple
5. 橙色：orange
6. 灰色：gray

> 颜色名这种方式，表达的颜色比较单一，所以用的并不多。

### rgb 或 rgba

编写方式：使用 红、黄、蓝 这三种光的三原色进行组合。
- 若三种颜色值相同，呈现的是灰色，值越大，灰色越浅
- `rgb(0, 0, 0)` 是黑色， `rgb(255, 255,255)` 是白色
- 对于 rbga 来说，前三位的 **rgb 形式要保持一致**，要么都是 0~255 的数字，要么都是百分比 

```css
/* 使用 0~255 之间的数字表示一种颜色 */
color: rgb(255, 0, 0);/* 红色 */
color: rgb(0, 255, 0);/* 绿色 */
color: rgb(0, 0, 255);/* 蓝色 */
color: rgb(0, 0, 0);/* 黑色 */
color: rgb(255, 255, 255);/* 白色 */

/* 混合出任意一种颜色 */
color:rgb(138, 43, 226) /* 紫罗兰色 */
color:rgba(255, 0, 0, 0.5);/* 半透明的红色 */

/* 也可以使用百分比表示一种颜色（用的少） */
color: rgb(100%, 0%, 0%);/* 红色 */
color: rgba(100%, 0%, 0%,50%);/* 半透明的红色 */
```

### HEX 或 HEXA

HEX 的原理同与 rgb 一样，依然是通过：红、绿、蓝色 进行组合，只不过要用 6 位（分成 3 组） 来表达

格式为：`#rrggbb`
- 每一位数字的取值范围是： `0 ~ f` ，即：`（ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, a, b, c, d, e, f ）`
- 所以每一种光的最小值是： `00` ，最大值是： `ff`

```css
color: #ff0000;/* 红色 */
color: #00ff00;/* 绿色 */
color: #0000ff;/* 蓝色 */
color: #000000;/* 黑色 */
color: #ffffff;/* 白色 */

/* 如果每种颜色的两位都是相同的，就可以简写*/
color: #ff9988;/* 可简为：#f98 */

/* 但要注意前三位简写了，那么透明度就也要简写 */
color: #ff998866;/* 可简为：#f986 */
```

> 注意点： IE 浏览器不支持 HEXA ，但支持 HEX 

### HSL 或 HSLA

HSL 是通过：色相、饱和度、亮度，来表示一个颜色的，格式为： `hsl(色相,饱和度,亮度)`
- *色相*：取值范围是 `0~360` 度，具体度数对应的颜色如下图：
- *饱和度*：取值范围是 `0%~100%`（向色相中对应颜色中添加灰色，`0%` 全灰，`100%` 没有灰）
- *亮度*：取值范围是 `0%~100%`（`0%` 亮度没了，所以就是黑色。`100%` 亮度太强，所以就是白色了）

HSLA 其实就是在 HSL 的基础上，添加了**透明度**

![](assets/Pasted%20image%2020240301131631.png)

## 字体属性

### font-size

功能：控制字体大小

语法：

```css
div {
	font-size: 40px;
}
```

- Chrome 浏览器支持的最小文字为 12px ，默认的文字大小为 16px ，并且 **0px 会自动消失**
- **不同浏览器默认的字体大小可能不一致**，所以**最好给一个明确的值，不要用默认大小**
	- 通常可以给 **`<body>` 设置 `[font-size]`**，这样 body 中的其他元素就都可以继承了

### font-family

功能：控制字体类型（字体族）

注意点：
- 使用字体的**英文名**字兼容性会更好
- 如果字体名**包含空格**，必须使用**引号包裹**起来。
- 可以设置多个字体，按照从左到右的顺序逐个查找，找到就用，没有找到就使用后面的，
	- 且通常在最后写上 `serif` （衬线字体）或 `sans-serif` （非衬线字体）
- **windows** 系统中，**默认**的字体就是**微软雅黑**

```css
div {
	font-family: "STCaiyun","Microsoft YaHei",sans-serif
}
```

> - 衬线字体：有边角装饰的字体，例如 宋体
> - 非衬线字体：没有有边角装饰的字体，例如 微软雅黑

### font-style

功能：控制字体是否为斜体

常用值：
- `normal` ：正常（默认值）
- `italic` ：斜体（使用字体自带的斜体效果）
- `oblique` ：斜体（强制倾斜产生的斜体效果）

语法：

```css
div {
	font-style: italic;
}
```

> `<em>` 包裹的字体也能倾斜，语义更重要，样式交给 CSS 就好

### font-weight

功能：控制字体的粗细

常用值：
- 关键词：
	- `lighter` ：细
	- `normal` ： 正常
	- `bold` ：粗
	- `bolder` ：很粗 （多数字体不支持）
- 数值：`100~1000` 且无单位，数值越大，字体越粗 （或一样粗，具体得看字体设计时的精确程度）
	- `100~300` 等同于 lighter ，
	- ` 400~500` 等同于 normal ， 
	- `>=600` 等同于 bold 

```css
div {
	font-weight: bold;
}

div {
	font-weight: 600;
}
```

### font 复合写法

功能：可以把上述字体样式合并成一个属性

编写规则：
- 必须有：字体大小、字体族
- 最后一位：字体族
- 倒数第二位：字体大小

**推荐**复合写法
- 不绝对，比如：只设置字体大小，直接用 `font-size` 属性

```css
div {
	font: bold italic 50px "STCaiyun", "STHupo", sans-serif;
}
```

## 文本属性

> - 字体属性：`font` 开头的属性
> - 文本属性：`text` 开头的属性
>   
>   不绝对，可以这么计

### 文本颜色

> [颜色的表示](#颜色的表示)

`[color]` 控制文字的颜色

可选值：
- 颜色名
- `rgb` 或 `rgba`
- `HEX` 或 `HEXA` （十六进制）
- `HSL` 或 `HSLA`

开发中常用的是： `rgb/rgba` 或 `HEX/HEXA` （十六进制）

```css
div {
	color: rgb(112,45,78);
}
```

### 文本间距

- 字母间距：`letter-spacing`
- 单词间距：`word-spacing` （通过**空格**识别词）

属性值为 `px` ，正值让间距增大，负值让间距缩小

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    .aa1 {
      letter-spacing: 10px;
    }

    .aa2 {
      word-spacing: 10px;
    }
  </style>
</head>

<body>
  <div>You got a dream,you gotta protect it.尚硅谷1</div>
  <div class="aa1">You got a dream,you gotta protect it.尚硅谷1</div>
  <div class="aa2">You got a dream,you gotta protect it.尚硅谷1</div>
</body>
```

### 文本修饰

`[text-decoration]` 控制文本的各种装饰线。
- 可选值：
	- `none` ： 无装饰线（常用）
	- `underline` ：下划线（常用）
	- `overline` ： 上划线
	- `line-through` ： 删除线
- 可**搭配**如下值使用：
	- `dotted` ：虚线
	- `wavy` ：波浪线
	- 也可以指定颜色

> 不同于 `font` ，搭配没有顺序要求
> 
> `dotted` 和 `wavy` 不能一块写

```css
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    .aa1 {
      text-decoration: underline dotted wavy red;
    }
  </style>
</head>

<body>
  <div>You got a dream,you gotta protect it.尚硅谷1</div>
  <div class="aa1">You got a dream,you gotta protect it.尚硅谷1</div>
</body>
```

### 文本缩进

`[text-indent]` 控制文本首字母的缩进
- 属性值： css 中的长度单位，例如： `px`

```css
div {
	text-indent:40px;
}
```

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div{
      font-size: 60px;
      /* 首行缩进两个字 */
      text-indent: 120px;
    }
  </style>
</head>

<body>
  <div>在 Java 中，volatile 关键字可以保证变量的可见性，如果我们将变量声明为 volatile ，这就指示 JVM，这个变量是共享且不稳定的，每次使用它都到主存中进行读取。</div>
</body>
```

灵活的文本缩进！

```css
.test {
	font-size: 20px;
	/* 当前元素font-size的两倍 2*20px */
	text-indent: 2em;
	background-color: yellowgreen;
}
```


### 文本对齐_水平

`[text-align]` 控制文本的水平对齐方式
- 常用值：
	- `left` ：左对齐（默认值）
	- `right` ：右对齐
	- `center` ：居中对齐

```css
div {
	text-align: center;
}
```

````html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div {
      font-size: 60px;
      background-color: orange;
      text-align: center;
    }
  </style>
</head>

<body>
  <div>hello</div>
</body>
````

### 细说 font-size

- 由于字体设计原因，**文字最终呈现的大小，并不一定与 `[font-size]` 一致**，可能大，也可能小
- 通常情况下，文字**相对字体设计框**，并不是垂直居中的，**通常都靠下一些**

![](assets/Pasted%20image%2020240302121540.png)

![](assets/Pasted%20image%2020240302121140.png)

![](assets/Pasted%20image%2020240302121259.png)

> font-size 设置的是字体框高度，宽度自适应调节
> 
> 通过 `x` 看基线

### 行高

`[line-height]` 控制一行文字的高度
- 可选值：
	- `normal` ：由浏览器根据文字大小决定的一个默认值。
		- 即没设置 `[line-height]`，**默认 `line-height：normal`**
	- `px` 
	- 数字：参考自身 font-size 的**倍数**（**很常用**）
	- 百分比：参考自身 font-size 的**百分比**
- 注意：
	- 多行文字，行和行间没有缝隙的
	- line-height **过小**：文字产生重叠
		- 最小值是 0 ，不能为负数。
	- line-height **可以继承**
		- 且为了能更好的呈现文字，最好**写数值**。
	- line-height 和 height 关系：
		- 设置 height：高度就是 height
		- 不设置 height：会根据 **line-height 计算高度**
- 应用场景：
	- 对于多行文字：控制行与行之间的距离。
	- 对于单行文字：让 **height 等于 line-height** ，可以实现文字**垂直居中**

> 备注：由于**字体设计原因**，靠上述办法实现的居中，并**不是绝对的垂直居中**，但如果一行中都是文字，不会太影响观感。

```css
div {
	line-height: 60px;
	line-height: 1.5;
	line-height: 150%;
}
```

```html
<head>
  <meta charset="UTF-8">
  <title>Document</title>
  <style>
    div {
      font-size: 40px;
      background-color: orange;
      line-height: 1.5;
    }
    span{
      color: red;
      font-size: 200px;
      line-height: 1.5;
    }
  </style>
</head>

<body>
  <!-- div的高度：40*1.5 + 40*1.5 + 200*1.5 -->
  <div>
    xxx由于字体设计原因，靠上述办法实现的居中，并不是绝对的垂直居中，但x
    <span>x如果</span>
    一行中都是文字，不会太影响观感。
  </div>
</body>
```

### 文本对齐_垂直方向

- 顶部：无需任何属性，在垂直方向上，默认就是顶部对齐。
- 居中：对于**单行文字**，让 `height = line-height` 即可。
- 底部：对于单行文字，目前一个临时的方式： `line-height` = `( height × 2 ) - font-size - x`
	- `x` 是根据字体族，动态决定的一个值

> 多行文字垂直居中、垂直方向上的底部对齐，“定位”是更好的解决方案

### vertical-align

- 属性名：`vertical-align`
- 作用：指定 **行内元素** 或 **表格单元格元素** 的 **垂直对齐方式**
- 常用值：
	- baseline （**默认值**）：使**元素的基线**与**父元素的基线**对齐。
	- top ：使元素的**顶部**与其**所在行**的顶部对齐。
	- middle ：使元素的**中部**与父元素的基线加上父元素字母 x 的一半对齐。
	- bottom ：使元素的**底部**与其**所在行**的底部对齐。
- 注意：
	- vertical-align **不能控制块元素**

```html
<head>
    <meta charset="UTF-8">
    <title>10_vertical-align</title>
    <style>
        div {
            font-size: 100px;
            height: 300px;
            background-color: skyblue;
        }

        span {
            font-size: 40px;
            background-color: orange;
            vertical-align: middle;
        }

        img {
            height: 30px;
            vertical-align: top;
        }

        .san {
            vertical-align: bottom;
        }
    </style>
</head>

<body>
    <!-- span元素 -->
    <div>
        atguigu尚硅谷x<span>x前端</span>
    </div>
    <hr>

    <!-- 图片元素 -->
    <div>
        atguigu尚硅谷x<img src="../images/我的自拍.jpg">
    </div>
    <hr>

    <!-- 表格单元格元素 -->
    <table border="1" cellspacing="0">
        <caption>人员信息</caption>
        <thead>
            <tr>
                <th>姓名</th>
                <th>年龄</th>
                <th>性别</th>
            </tr>
        </thead>
        <tbody>
            <tr height="200">
                <td class="san">张三</td>
                <td>18</td>
                <td>男</td>
            </tr>
            <tr>
                <td>李四</td>
                <td>20</td>
                <td>女</td>
            </tr>
        </tbody>
    </table>
</body>
```

> #Boer 为啥是文本属性？

## 列表属性

- list-style-type 
	- 功能：设置列表**符号**
	- 常用值：
		- none ：不显示前面的标识（很常用！）
		- square ：实心方块
		- disc ：圆形
		- decimal ：数字
		- lower-roman ：小写罗马字
		- upper-roman ：大写罗马字
		- lower-alpha ：小写字母
		- upper-alpha ：大写字母
- list-style-position 
	- 功能：设置列表符号的位置 
	- 属性值：
		- inside ：在 li 的里面
		- outside ：在 li 的外边
- list-style-image 
	- 功能：**自定义**列表符号 
	- 属性值：`url(图片地址)`
- list-style 
	- 功能：复合属性 
	- 属性值：没有数量、顺序的要求

```html
<head>
    <meta charset="UTF-8">
    <title>列表相关属性</title>
    <style>
        ul {
            /* 列表符号 */
            list-style-type: decimal;
            /* 列表符号的位置 */
            list-style-position: inside;
            /* 自定义列表符号 */
            list-style-image: url("../images/video.gif");
            /* 复合属性 */
            list-style: decimal url("../images/video.gif") inside;
        }

        li {
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <ul>
        <li>《震惊！两男子竟然在教室做出这种事》</li>
        <li>《一夜暴富指南》</li>
        <li>《给成功男人的五条建议》</li>
    </ul>
</body>
```

## 表格属性

### 边框相关属性（通用）

> 其他元也可以用

- border-width 
	- 功能：边框宽度 
	- 属性值：CSS 中可用的长度值
- border-color
	- 功能：边框颜色
	- 属性值：CSS 中可用的颜色值
- border-style
	- 功能：边框风格
	- 属性值：
		- none 默认值
		- solid 实线
		- dashed 虚线
		- dotted 点线
		- double 双实线
- border
	- 功能：边框复合属性
	- 属性值：没有数量、顺序的要求

```html
<head>
    <meta charset="UTF-8">
    <title>01_边框相关属性</title>
    <style>
        table {
            /* border-width: 2px; */
            /* border-color: green; */
            /* border-style: solid; */
            border: 2px green solid;
        }

        /* 单元格 */
        td,
        th {
            border: 2px orange solid;
        }

        h2 {
            border: 3px red solid;
        }

        span {
            border: 3px purple dashed;
        }
    </style>
</head>

<body>
    <h2>边框相关的属性，不仅仅是表格能用，其他元素也能用</h2>
    <span>你要加油呀！</span>
    <table>
        <caption>人员信息</caption>
        <thead>
            <tr>
                <th>序号</th>
                <th>姓名</th>
                <th>年龄</th>
                <th>性别</th>
                <th>政治面貌</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1</td>
                <td>张三</td>
                <td>18</td>
                <td>男</td>
                <td>党员</td>
            </tr>
            <tr>
                <td>2</td>
                <td>李四</td>
                <td>19</td>
                <td>女</td>
                <td>团员</td>
            </tr>
            <tr>
                <td>3</td>
                <td>王五</td>
                <td>20</td>
                <td>男</td>
                <td>群众</td>
            </tr>
            <tr>
                <td>4</td>
                <td>赵六</td>
                <td>21</td>
                <td>女</td>
                <td>党员</td>
            </tr>
        </tbody>
    </table>
</body>
```

### 表格独有属性

- table-layout
	- 功能：设置列宽度
	- 属性值：
		- auto ：自动，列宽根据内容计算（默认值）
		- fixed ：固定列宽，平均分

- border-spacing
	- 功能：单元格间距 
	- 属性值：CSS 中可用的长度值
	- 生效的前提：单元格**边框不能合并**

- border-collapse
	- 功能：合并单元格边框
	- 属性值：
		- collapse ：合并
		- separate ：不合并

- empty-cells
	- 功能：隐藏没有内容的单元格
		- 没有内容：`<td></td>`
	- 属性值：
		- show ：显示，默认
		- hide ：隐藏
	- 生效前提：**单元格不能合并**

- caption-side
	- 功能：设置表格标题位置
	- 属性值：
		- top ：上面（默认值）
		- bottom ：在表格下面

```html
<head>
    <meta charset="UTF-8">
    <title>02_表格独有属性</title>
    <style>
        table {
            border: 2px green solid;
            width: 500px;
            /* 控制表格的列宽 */
            table-layout: fixed;
            /* 控制单元格间距（生效的前提是：不能合并边框） */
            border-spacing: 20px;
            /* 合并相邻的单元格的边框 */
            /* border-collapse: collapse; */
            /* 隐藏没有内容的单元格（生效的前提是：不能合并边框） */
            empty-cells: hide;
            /* 设置表格标题的位置 */
            caption-side: bottom;
        }

        td,
        th {
            border: 2px orange dashed;
        }

        .number {
            width: 50px;
            height: 50px;
        }
    </style>
</head>

<body>
    <table>
        <caption>人员信息</caption>
        <thead>
            <tr>
                <th class="number">序号</th>
                <th>姓名</th>
                <th>年龄</th>
                <th>性别</th>
                <th>政治面貌</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td>1</td>
                <td>张三</td>
                <td>18</td>
                <td>男</td>
                <td>党员</td>
            </tr>
            <tr>
                <td>2</td>
                <td>李四</td>
                <td>19</td>
                <td></td>
                <td>团员</td>
            </tr>
        </tbody>
    </table>
</body>
```

## 背景属性

background-color
- 功能：设置背景颜色
- 属性值：
	- 符合 CSS 中颜色规范的值
	- 默认：transparent（透明）

background-image
- 功能：设置背景图片
- 属性值：`url(图片的地址)`

background-repeat
- 功能：设置背景重复方式
- 属性值：
	- （默认）repeat：重复，铺满整个元素
	- repeat-x ：只在水平方向重复
	- repeat-y ：只在垂直方向重复
	- no-repeat ：不重复

background-position
- 功能：设置**背景图**位置
- 属性值：
	- 1）关键字
		- 水平： left 、 center 、 right
		- 垂直: top 、 center 、 bottom
	- 2）x 坐标和 y 坐标
		- 为坐标原点：元素左上角
	- 1）2）只写一个，另一个取 center

background
- 功能：复合属性

```html
<head>
    <meta charset="UTF-8">
    <title>背景相关属性</title>
    <style>
        body {
            background-color: gray;
        }

        div {
            width: 400px;
            height: 400px;
            border: 5px black solid;
            font-size: 20px;

            /* 设置背景颜色，默认值是transparent */
            background-color: skyblue;
            /* 设置背景图片 */
            background-image: url(../images/bg.gif);
            /* 设置背景图片的重复方式 */
            background-repeat: repeat;
            /* 控制背景图片的位置——第一种写法：用关键词 */
            background-position: center;
            /* 控制背景图片的位置——第二种写法：用具体的像素值 */
            background-position: 100px 200px;
            /* 复合属性 */
            /* background: url(../images/悟空.jpg) no-repeat 100px 200px; */
        }
    </style>
</head>

<body>
    <div>你好啊！</div>
</body>
```

## 鼠标属性

cursor
- 功能：设置鼠标**光标**的样式
- 属性值：
	- pointer ：小手
	- move ：移动图标
	- text ：文字选择器
	- crosshair ：十字架
	- wait ：等待
	- help ：帮助

```html
<head>
    <meta charset="UTF-8">
    <title>鼠标相关属性</title>
    <style>
        div {
            width: 400px;
            height: 400px;
            background-color: skyblue;
            cursor: url("../images/arrow.png"), pointer;
        }

        button {
            cursor: pointer;
        }

        input {
            cursor: move;
        }
    </style>
</head>

<body>
    <div>
        把鼠标放进来看一看
        <input type="text"><br>

        <a href="#">百度</a><br>
        
        <button>点我</button>
    </div>
</body>

```

# ---------- 盒子模型

## CSS 长度单位

- px ：像素。
- em ：相对于 当前元素 或 父元素 font-size 的倍数。
- rem ：相对于 根元素（html）的 font-size 的倍数
	- root em
- % ：相对 父元素 计算。

> CSS 中设置长度，必须加单位，否则样式无效！

```html
<head>
    <meta charset="UTF-8">
    <title>01_CSS中常用的长度单位</title>
    <style>
        html {
            font-size: 40px;
        }

        body {
            font-size: 100px;
        }

        #d1 {
            /* 第一种长度单位：px（像素） */
            width: 200px;
            height: 200px;
            font-size: 20px;
            background-color: rgb(111, 121, 125);
        }

        #d2 {
            /* 第二种长度单位：em（相对于当前元素或其父元素的font-size的倍数） */
            width: 10em;
            height: 10em;
            font-size: 20px;
            background-color: orange;
        }

        #d3 {
            /* 第三种长度单位：rem（相对于根元素的font-size的倍数） */
            /* 4*40px */
            width: 4rem;
            height: 4rem;
            font-size: 20px;
            background-color: green;
        }

        #d4 {
            width: 200px;
            height: 200px;
            font-size: 20px;
            background-color: gray;
        }

        .inside {
            /* 第四种长度单位：%（相对其父元素的百分比） */
            /* 200px*0.5 */
            width: 50%;
            height: 25%;
            /* 20px*1.5 */
            font-size: 150%;
            background-color: orange;
        }

        .test {
            font-size: 20px;
            /* 首行缩进 2*20px */
            text-indent: 2em;
            background-color: yellowgreen;
        }
    </style>
</head>

<body>
    <div id="d1">1</div>
    <hr>
    <div id="d2">2</div>
    <hr>
    <div id="d3">3</div>
    <hr>

    <div id="d4">
        <div class="inside">4</div>
    </div>
    <hr>

    <div class="test">好好学习，天天向上</div>
</body>
```

## 元素的显示模式

1）块元素 block
- 特点：
	- **独占一行**；从上到下排列
	- 默认 宽：**撑满** **父**元素
	- 默认 高：由**内容**撑开
	- 可通过 CSS 设置宽高

```html
1. 主体结构标签：<html>、<body>
2. 排版标签：<h1>~<h6>、<hr> 、<p> 、<pre> 、<div>
3. 列表标签：<ul> 、<ol> 、<li> 、<dl> 、<dt> 、<dd>
4. 表格相关标签：<table> 、<tbody> 、<thead> 、<tfoot> 、<tr> 、<caption>
5. 表单相关标签：<form> 与 <option>
```

> 浏览器默认给body加了样式，所以视觉上没有完全撑满
> 
> ![](assets/Pasted%20image%2020240303144519.png)

2）行内元素 inline（内联元素）
- 特点：
	- **不独占一行**；从左到右排列
		- 一行中容纳不下，会在下一行
	- 默认 宽高：由**内容**撑开
	- **无法**通过 CSS 设置宽高

```html
1. 文本标签：<span>、<br> 、<em> 、<strong> 、<sup> 、<sub> 、<del> 、<ins>
2. <a> 与 <label>
```

3）行内块元素 inline-block（内联块元素）
- 特点：
	- **不独占一行**；从左到右排列
		- 一行中容纳不下，会在下一行
	- 默认 宽高：由**内容**撑开
	- **无法**通过 CSS 设置宽高

```html
1. 图片：<img>
2. 单元格：<td> 、<th>
3. 表单控件：<input> 、<textarea> 、<select> 、<button>
4. 框架标签：<iframe>
```

> 元素早期只分为：行内元素、块级元素，区分条件也只有一条："是否独占一行"，如果按照这种分类方式，行内块元素应该算作行内元素

---
为什么无论有没有 br，span 都独占一行？
- 因为 div 独占一行，span 没人理他了，只能独占一行
- br 的出现刚好把 span 这行后面的占满了

```html
<span id="s1">123</span>
<br>
<div id="d2">456</div>
```

---

```html
<head>
    <meta charset="UTF-8">
    <title>02_元素的显示模式</title>
    <style>
        div {
            width: 300px;
            height: 100px;
        }

        #d1 {
            background-color: skyblue;
        }

        #d2 {
            background-color: orange;
        }

        #d3 {
            background-color: green;
        }

        .one {
            background-color: skyblue;
        }

        .two {
            background-color: orange;
        }

        span {
            width: 200px;
            height: 200px;
        }

        img {
            width: 200px;
        }
    </style>
</head>

<body>
    <div id="d1">山回路转不见君</div>
    <div id="d2">雪上空留马行处</div>
    <div id="d3">风里雨里我在尚硅谷等你</div>

    <hr>

    <span class="one">人之初</span>
    <span class="two">性本善</span>
    <span class="one">人之初</span>
    <span class="two">性本善</span>
    <span class="one">人之初</span>
    <span class="two">性本善</span>
    <span class="one">人之初</span>
    <span class="two">性本善</span>
    <span class="one">人之初</span>
    <span class="two">性本善</span>
    <span class="one">人之初</span>
    <span class="two">性本善</span>
    <span class="one">人之初</span>
    <span class="two">性本善</span>

    <hr>

    <img src="../images/悟空.jpg" alt="悟空">
    <img src="../images/悟空.jpg" alt="悟空">
    <img src="../images/悟空.jpg" alt="悟空">
    <img src="../images/悟空.jpg" alt="悟空">
</body>
```

### 修改显示模式

display 属性：修改元素的默认显示模式，常用值如下：
- none：隐藏
- block：块级元素
- inline：内联元素
- inline-block：行内块元素

```html
<head>
    <meta charset="UTF-8">
    <title>04_修改元素的显示模式</title>
    <style>
        div {
            width: 200px;
            height: 100px;
            font-size: 20px;
            display: inline-block;
        }
        #d1 {
            background-color: skyblue;
        }
        #d2 {
            background-color: orange;
        }
        a {
            width: 200px;
            height: 100px;
            font-size: 20px;
            display: block;
        }
        #s1 {
            background-color: skyblue;
        }
        #s2 {
            background-color: orange;
        }
        #s3 {
            background-color: green;
        }
    </style>
</head>
<body>
    <div id="d1">你好1</div>
    <div id="d2">你好2</div>
    <hr>
    <a id="s1" href="https://www.baidu.com">去百度</a>
    <a id="s2" href="https://www.jd.com">去京东</a>
    <a id="s3" href="https://www.toutiao.com">去百头条</a>
</body>
```

## 盒子模型的组成

CSS 把所有的 HTML 元素都看成一个盒子，所有的样式都基于这个盒子
- margin（外边距）： 盒子与外界的距离
- border（边框）： 盒子的边框
- padding（内边距）： 紧贴内容的补白区域
- content（内容）：元素中的文本或后代元素都是它的内容。

**盒子的大小** = content + 左右 padding + 左右 border 

**margin 不影响盒子的大小，但影响盒子的位置**

![|300](assets/Pasted%20image%2020240303153104.png)

```html
<head>
    <meta charset="UTF-8">
    <title>05_盒子模型的组成部分</title>
    <style>
        div {
            /* 内容区的宽 */
            width: 400px;
            /* 内容区的高 */
            height: 400px;
            /* 内边距，设置的背景颜色会填充内边距区域 */
            padding: 20px;
            /* 边框，设置的背景颜色会填充边框区域 */
            border: 10px dashed black;
            /* 外边距 */
            margin: 50px;

            font-size: 20px;
            background-color: gray;
        }
    </style>
</head>
<body>
    <div>你好啊</div>
</body>
```

### content

> 属性值都是长度

content 属性：
- width：宽度
- max-width：最大宽度
- min-width：最小宽度
- height：高度
- max-height：最大高度
- min-height：最小高度

注意：max-width 、 min-width 一般不与 width 一起使用；高度同理

> 窗口比 min 还小的时候，会出现滚动条

```html
<head>
    <meta charset="UTF-8">
    <title>06_盒子的内容区_content</title>
    <style>
        div {
            width: 800px;
            /* min-width: 600px; */
            /* max-width: 1000px; */

            height: 200px;
            /* min-height: 50px; */
            /* max-height: 400px; */
            background-color: skyblue;
        }
    </style>
</head>

<body>
    <div>
        逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。逝者如斯夫！不舍昼夜。
    </div>
</body>
```

### 默认宽度

即不设置 width 属性时，元素呈现出的宽度

- 总宽度 = `父的 content — 自身的左右 margin`
- content 宽度 = `父的 content — 自身的左右 margin — 自身的左右 border — 自身的左右padding `

### padding

> 属性值都是长度

- adding-top：上内边距
- padding-right：右内边距
- padding-bottom：下内边距
- padding-left：左内边距
- padding：复合属性
	- 1 个值: 四个方向
	- 2 个值: 上下、左右
	- 3 个值: 上、左右、下
	- 4 个值: 上、右、下、左

注意：
- padding 不能为负
- **行内元素** 的 **上下**内边距不能完美设置
	- 不占位，和上下元素重叠

![](assets/Pasted%20image%2020240303170601.png)

> 块级元素、行内块元素，四个方向内边距都可以完美设置

```html
<head>
    <meta charset="UTF-8">
    <title>08_盒子的内边距_padding</title>
    <style>
        #d1 {
            width: 400px;
            height: 100px;

            /* 复合属性，写四个值，含义：上、右、下、左 */
            padding: 10px 20px 30px 40px;

            font-size: 20px;
            background-color: skyblue;
        }
        span {
            background-color: orange;
            font-size: 20px;

            padding-left: 20px;
            padding-right: 20px;

            padding-top: 20px;
            padding-bottom: 20px;
        }
        img {
            width: 200px;
            padding: 50px;
        }
    </style>
</head>
<body>
    <div id="d1">你好啊</div>
    <hr>
    <span>我很好</span>
    <div>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sunt, sint.</div>
    <hr>
    <img src="../images/小姐姐.gif" alt="">
    <div>小姐姐很想你呀</div>
</body>
```

# ---------- 浮动

# ---------- 定位

# ---------- 布局






