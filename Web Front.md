# Web Front

参考文档：[CDN](https://developer.mozilla.org)、[React](https://zh-hans.reactjs.org/)

# HTML

超文本标记语言（Hyper Text Markup Language，HTML）是一系列标签，而非编程语言。它使用各种标签把想要显示给用户的内容组织起来。Web 浏览器类似于解释器，根据标签的含义把 HTML 文档渲染成网页。不同浏览器处理 HTML 文档的能力千差万别，可把 Google Chrome 看作标准实现。

## 基本知识

**文档注释**

HTML 支持多行注释，注释内容使用  `<!--` 和 `-->` 括起。

**标签分类**

HTML 有两种重要的元素类别：

* 块级标签：在页面以块的形式呈现，独占一行，可嵌套在另一个块级标签内。
* 内联标签：通常出现在块级标签内，不换行，紧挨着呈现，不能内嵌其它元素。

元素类别与 CSS 盒子类型的术语同名，但它们不相关。改变 CSS 盒子类型，不影响元素的分类，也不影响它可以包含和被包含于哪些元素。

**标签结构**

完整的元素由开始标签、内容、结束标签构成，比如 `<p> Hello World </p>`。

空元素只有一个标签，比如 `<img src="images/cat.jpg" >`，空元素的末尾没有 `/` 在 HTML 合法。

**标签属性**

元素的（开始）标签可嵌入一些属性，这些信息不会呈现到页面，属性的值由单/双引号括起。

**布尔属性**

没有值的标签属性是合法的，这叫做布尔属性，我叫它标记属性，它的值一般是属性的名字。

## 文档结构

HTML 文档的所有元素组织为树形结构。

```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <link rel="icon" href="/images/icon.png">
    </head>
    <body>

    </body>
</html>
```

`<!DOCTYPE html>` 声明文档类型。

`<html>` 根元素，包含文档的所有内容。

`<head>` 容器，包含文档标题、引用样式、引用脚本等元数据。

`<title>` 文档标题，也就是出现在标题栏、标签页的文本。

`<meta>` 存放不能由其它元素定义的元数据。

`<link rel="icon" href="/images/icon.png">`：文档图标，也就是出现在标题栏、标签页的图片。

`<body>` 包含呈现到视窗的所有内容。

## 文本标签

文本类标签用于在页面显示文字，类型众多，绝大部分都可看作是设有固定样式的 `<div>` 或 `<span>` 标签。

**`<div>`**

通用型的流内容容器，主要用于元素分组，从而统一设置样式或进行 JS 操作。若不使用 CSS 修饰，则它对内容或布局没有任何影响。

**`<span>`**

文本内容的通用内联容器，没有任何特殊语义。

**`<h1>` - `<h6>`**

六个不同级别的标题，`<h1>` 级别最高，`<h6>` 级别最低。

**`<p>`**

文本段落，表现为一块与相邻元素分离的文本，以垂直空白隔离，或以首行缩进。

**`<pre>`**

预定义格式的文本，文本将按照源代码的编排，以等宽字体的形式呈现，空格和换行都会显示，但紧挨着标签的换行会被过滤。

**`<br>`**

换行。

**`<i>`**

需要与普通文本区分的文本，常呈现为斜体。

**`<b>`**

需要用户注意的文本，常呈现为粗体。若只是单纯的粗体，应使用 CSS 的 `font-weight` 属性。

**`del`**

已删除的文本，常以删除线修饰。

**`<ins>`**

已经插入的文本，常以下划线修饰。

**`hr`**

表示段落间的主题转换，呈现为一条横线。若只是单纯的横线，应使用 CSS 样式。

**`<a>`**

通往其它网页、文件、页面其它位置等的超链接，该元素可包含文本、图片等，点击后激活。

| 属性       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| `title`    | 显示的提示信息                                               |
| `href`     | 超链接指向的 URL 或 URL 片段。                               |
| `target`   | 指示在何处打开链接资源，默认 `_self` 在当前页面加载，`_blank` 在新标签加载。 |
| `download` | 指示浏览器下载链接资源，而非跳转。                           |

## 媒体标签

**`<img>`**

把图像嵌在文档。

| 属性     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| `src`    | 必须，图像的本地路径或网络链接。                             |
| `alt`    | 图像的文本描述，加载失败时显示该值。                         |
| `height` | 图像高度，在 H5 它的单位默认是 CSS 像素。                    |
| `width`  | 图像宽度。可以只设置 `width` 和 `height` 中的一个，浏览器将按原始比例进行拉伸。 |

**`<audio>`**

把音频嵌在文档，它能包含一个或多个音频资源。声明 `controls` 属性，浏览器将提供控制面板。

示例，使用 `src` 属性设置资源。

```
<audio controls src="/audios/bgm.mp3"></audio>
```

示例，使用 `<source>` 元素设置多个资源，浏览器按序尝试播放可加载的一个资源。

```
<audio controls>
	<source src="/audios/bgm.mp3">
	<source src="/audios/sound1.mp3">
	<source src="/audios/sound2.mp3">
</audio>
```

**`<video>`**

把视频嵌在文档，与 `<audio>` 相似，它能包含一个或多个视频资源，也使用 `src` 属性或 `<source>` 标签来设置资源，也能声明 `controls` 属性以提供控制面板。

```
<video controls src="/videos/video1.mp4" width="500"></video>
```

```
<video controls width="500">
	<source src="/videos/video3.mp4" type="video/mp4">
	<source src="/videos/video2.mp4" type="video/mp4">
</video>
```

## 格式标签

### 列表标签

**`<ul>` 与 `<li>`**

无序列表，常显示为项目符号列表。支持嵌套，`<ul>` 表示一层，`<li>` 表示单个项。

```
<ul>
    <li>Milk</li>
    <li>Cheese
        <ul>
            <li>Blue cheese</li>
            <li>Feta</li>
        </ul>
    </li>
</ul>
```

**`<ol>` 与 `<li>`**

有序列表，常显示为带编号的列表。与 `<ul>` 与 `<li>` 类似，有序、无序列表可相互嵌套。

```
<ol>
  <li>Mix flour, baking powder, sugar, and salt.</li>
  <li>In another bowl, mix eggs, milk, and oil.</li>
  <li>Stir both mixtures together.</li>
  <li>Fill muffin tray 3/4 full.</li>
  <li>Bake for 20 minutes.</li>
</ol>
```

**`<dl>`、`<dt>` 与 `<dd>`**

包含术语定义和描述的列表，类似键值对列表。`<dl>` 表示一层，`<dt>` 表示名称，`<dd>` 表示描述。

```
<p>Cryptids of Cornwall:</p>

<dl>
    <dt>Beast of Bodmin</dt>
    <dd>A large feline inhabiting Bodmin Moor.</dd>

    <dt>Morgawr</dt>
    <dd>A sea serpent.</dd>

    <dt>Owlman</dt>
    <dd>A giant owl-like creature.</dd>
</dl>
```

### 表格标签

**`<table>`**

整个表格。

**`<caption>`**

表格的标题，通常是 `<table>` 的第一个子元素。

**`<thead>`**

表头的行。

**`<tbody>`**

封装所有数据行。

**`<tr>`**

表格的一行。

**`<th>`**

表头单元格，粗体。

**`<td>`**

普通单元格。

```
<table>
   	<caption>Alien football stars</caption>
    <tr>
        <th scope="col">Player</th>
        <th scope="col">Gloobles</th>
        <th scope="col">Za'taak</th>
    </tr>
    <tr>
        <th scope="row">TR-7</th>
        <td>7</td>
        <td>4,569</td>
    </tr>
</table>
```

### 表单标签

**`<form>`**

整个表单，它包含各种交互控件，用于向 Web 服务器提交信息。

| 属性      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| `action`  | 提交地址。                                                   |
| `method`  | 提交方式。`post` 把表单数据放在请求体内，`get` 把表单数据追加到 URL 尾部，以 `?` 分隔。 |
| `enctype` | 当 `post` 提交时，表单内容提交给服务器的 MIME 类型，它决定浏览器对表单数据的编码方式。 |

对于 `enctype` 属性，可能的取值：

* `application/x-www-form-urlencoded`：默认，数据以键值对方式传送到服务器。
* `multipart/form-data`：提交文件时使用，把表单数据二进制编码，各控件以 `--${boundary}` 分割。
* `text/plain`：以纯文本形式编码，其中不包含任何控件或格式字符。H5 新增，用于调试。

还有其它可选值，比如常见的 `application/json` 类型。

**`<input>`**

表单的交互式控件，接收用户的输入信息。非常强大和复杂，拥有大量的输入类型和属性组合。

下表列举一些输入类型。

| 类型       | 说明                                               |
| ---------- | -------------------------------------------------- |
| `text`     | 默认，单行文本字段。                               |
| `password` | 单行文本字段，值会被遮盖。                         |
| `file`     | 选择文件进行上传，使用 `accept` 属性限制文件类型。 |
| `reset`    | 按钮，重置表单。                                   |
| `radio`    | 按钮，在多个拥有相同 `name` 的选项中进行单选。     |
| `submit`   | 按钮，提交表单。                                   |
| `button`   | 没有默认行为的按钮。                               |
| `checkbox` | 复选框，其值可设为选中或未选中。                   |
| `hidden`   | 隐藏控件，其值仍会提交。                           |

下表列举一些属性，不同输入类型支持不同的属性，具体查看[相关文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#%E5%B1%9E%E6%80%A7)。

| 属性          | 说明                       |
| ------------- | -------------------------- |
| `name`        | 提交时该控件数据的名字。   |
| `checked`     | 默认选中。                 |
| `required`    | 是否必须。                 |
| `maxlength`   | 值的最大长度。             |
| `minlength`   | 值的最小长度。             |
| `placeholder` | 没有值时，显示的提示文本。 |

**`<label>`**

表示某个元素的说明，按照习惯，表单的每个 `<input>` 都要关联一个 `<label>` 元素。使用 `<label>` 的 `for` 属性与 `<input>` 的 `id` 进行匹配。

**`<textarea>`**

多行纯文本编辑控件，用于输入一段相当长的、不限格式的文本。该元素不支持 `value` 属性，它的默认内容在开始标签和闭合标签之间设置。

| 属性   | 说明             |
| ------ | ---------------- |
| `rows` | 输入文本的行数。 |
| `clos` | 输入文本的列数。 |

**`<select>` 与 `<option>`**

`<select>` 表示一个提供选项菜单的控件，`<option>` 表示可选项。

控件数据的名字由 `<select>` 的 `name` 属性设置。`<option>` 的 `value` 属性是选项的值，若没有，则使用它包含的文本内容。`<select>` 可声明 `multiple` 属性以支持多选，`<option>` 可声明 `selected` 属性默认选中。

```
<label for="pet-select">Choose a pet:</label>
<select name="pets" id="pet-select">
	<option selected value="dog">Dog</option>
    <option value="cat">Cat</option>
    <option value="hamster">Hamster</option>
    <option value="parrot">Parrot</option>
    <option value="spider">Spider</option>
    <option value="goldfish">Goldfish</option>
</select>
```

## 其它标签

### 语义标签

语义标签没有特殊含义，效果与 `<div>` 类似，它们通过标签名表示这个块包含的内容，以提高文档的可读性。

| 标签        | 说明         |
| ----------- | ------------ |
| `<header>`  | 页头         |
| `<footer>`  | 页脚         |
| `<nav>`     | 导航栏       |
| `<aside>`   | 侧边栏       |
| `<section>` | 通用独立部分 |
| `<article>` | 通用独立部分 |

### 特殊符号

| 源码      | 显示 | 说明                   |
| --------- | ---- | ---------------------- |
| `&lt;`    | <    | 小于号或显示标记       |
| `&gt;`    | >    | 大于号或显示标记       |
| `&amp;`   | &    | 可用于显示其它特殊字符 |
| `&quot;`  | "    | 引号                   |
| `&reg;`   | ®    | 已注册                 |
| `&copy;`  | ©    | 版权                   |
| `&trade;` | ™    | 商标                   |
| `&nbsp;`  |      | 不断行的空白           |

# CSS

层叠样式表（Cascading Style Sheets，CSS）用于描述 HTML 文档的呈现效果。比如，表格的边框、图片的大小和位置。

CSS 支持多行注释，注释内容使用  `/*` 和 `*/` 括起。

## 定义方式

**内联样式表**

直接定义在标签的 style 属性，只影响所在的元素。

```
<img src="/images/mountain.jpg" style="width: 300px; height: 200px;">
```

**内部样式表**

定义在 `<style>` 标签，属性 type 可省，只影响当前文档的元素。

```
<style type="text/css">
	p{color:red;}
</style>
```

**外部样式表**

定义在 CSS 文件，HTML 文档通过 `<link>` 元素引用 CSS 文件，属性 type 可省。

```
<link rel="stylesheet" type="text/css" href="/static/css/color.css" />
```

## 选择器

决定样式将要渲染文档的哪些元素。

### 普通选择器

**标签选择器**

根据标签名选择元素。

```
div {...}
```

**ID 选择器**

根据 ID 属性选择元素。

```
#name {...}
```

**类选择器**

根据 class 属性选择元素。

```
.name {...}
```

### 伪类选择器

决定元素处于特殊状态时的样式。

**链接伪类选择器**

| 选择器     | 说明       |
| ---------- | ---------- |
| `:link`    | 链接访问前 |
| `:visited` | 链接访问后 |
| `:hover`   | 鼠标悬停   |
| `:active`  | 鼠标长按   |
| `:focus`   | 聚焦       |

示例，链接点击前为红色，点击后为灰色。

```
a:visited {color: red;}
a:hover {color: grey;}
```

示例，名称输入框聚焦时变宽。

```
input#name:focus {width: 100px;}
```

**位置伪类选择器 `:nth-child(n)`**

选择是其父标签的第几个子元素的所有元素。可用数字或 n 指定，这里 n 表示大于 0 的自然数。

示例，选择所有块的所有第奇数个子元素。

```
div:nth-child(2n+1) {...}
```

**目标伪类选择器 `:target`**

文档元素被 URL 链接时的样式。示例，选择处于被链接的 ID 为 "S1" 的元素。

 ```
 #S1:target {...}
 ```

### 混合选择器

由两个或多个基础选择器组成，以进行更精确的选择，以下是一些示例。

同时选择 `<div>` 和 `<span>`元素。

```
div, span {...}
```

选择指定类的 `<div>` 元素

```
div.classname {...}
```

选择指定 ID 的 `<div>` 元素。

```
div#id {...}
```

选择紧跟着 `<div>` 的 `<span>` 元素

```
div + span {...}
```

选择所有 `<div>` 的子孙 `<span>` 元素。

```
div span {...}
```

选择所有父标签是 `<div>` 的 `<span>` 元素。

```
div > span {...}
```

### 通配符选择器

| 选择器              | 说明                   |
| ------------------- | ---------------------- |
| `*`                 | 所有元素               |
| `[attribute]`       | 包含指定属性的元素     |
| `[attribute=value]` | 指定属性为指定值的元素 |

示例，选择 name 属性为 "Tony" 的 `<div>` 元素。

```
div[name=Tony] {...}
```

### 伪元素选择器

把标签的内容当作元素一样进行选择。

| 选择器           | 说明                     |
| ---------------- | ------------------------ |
| `::first-letter` | 第一个字母               |
| `::first-line`   | 第一行                   |
| `::selection`    | 被鼠标选中的内容         |
| `::after`        | 在被选中的元素后插入内容 |
| `::before`       | 在被选中的元素前插入内容 |

示例，选择每个段落的第一个字母。

```
p::first-letter {color: red;}
```

示例，在段落尾部插入红色的句号。

```
p::after {
    content: "。";
    color: red;
}
```

### 渲染的优先级

* 选择方式越具体，优先级越高。

  `!important` > 内联样式 > ID 选择器 > 类与伪类选择器 > 标签选择器 > 通用选择器

* 优先级相同时，后面定义的样式覆盖前面的定义。

* 继承自父元素的样式优先级最低。

> 若样式的优先级很低，但非要使用它，可使用 `!important` 关键字。
>
> ```
> div::first-letter {
>        color: red !important;
>        background-color: grey !important;
>    }
> ```

## 颜色

所有颜色都由红、绿、蓝组合而成，这里介绍 CSS 表示颜色的几种方式。

**颜色关键字**

```
span {color: red;}
```

**16 进制表示**

使用 6 位 16 进制数表示颜色，每两个数表示一种基础颜色，每个颜色的值在 0 到 255 之间。

```
span {color: #ff0000;}
```

**RGB 表示**

把上面的 16 进制转化为 10 进制。

```
span {color: rgb(255, 0, 0);}
```

**RGBA 表示**

前三个数同上，第四个数表示透明度。

```
span {color: rgba(255, 0, 0, 1);}
```

## 文本

**text-align**

定义块元素的内联内容（比如文字和内联元素）如何相对它的父元素进行对齐，对块元素本身和子块元素无影响。

| 取值        | 说明                                      |
| ----------- | ----------------------------------------- |
| start       | 与内容方向相同。                          |
| end         | 与内容方向相反。                          |
| left        | 左侧对齐。                                |
| right       | 右侧对齐。                                |
| center      | 居中。                                    |
| justify     | 两侧对齐，但最后一行无效。                |
| justify-all | 与 justify 相同，并强制最后一行两端对齐。 |

**line-height**

定义多行元素的空间量，比如多行文本的间距。对于块级元素，它指定 line boxes 的最小高度。对于非替代的内联元素，它用于计算 line box 的高度。这个属性常用于块级元素的内容竖直居中，即设值为块的高度。

> 可替代元素拥有固定的外观，CSS 只能影响它们的位置和部分内容，而无法控制样式。`<video>`、`<img>` 就是典型的可替代元素。

| 取值   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| normal | 取决于客户端，浏览器使用默认值，约 1.2，这取决于元素的 font-family 属性。 |
| 数字   | 应用值是该数字乘以元素的字体大小，推荐方式，不会在继承时产生不确定的结果。 |
| Length | 使用长度明确指定。                                           |
| 百分比 | 应用值是该百分比乘以元素字体大小的计算值。                   |

> **Length**
>
> 长度 Length 是用于表示距离、尺寸的 CSS 数据类型，由数字和单位组成，中间没有空格，有些单位支持负值。以下是部分 Length 单位。
>
> | 单位 | 说明                      |
> | ---- | ------------------------- |
> | px   | 像素                      |
> | em   | 当前元素的 font-size 值   |
> | rem  | 根元素的 font-size 值     |
> | vw   | 视窗初始包含块的宽度的 1% |
> | vh   | 视窗初始包含块的高度的 1% |
> | %    | 父元素的 font-size 的 1%  |

**letter-spacing**

文本字符的间距，正值使字符分散，负值使字符靠拢。默认 normal，可用 Length 自定义。

**text-indent**

块元素首行文本前面的缩进量。默认不缩进，可用 Length 自定义，推荐使用 em 单位。

**text-decoration**

文本外观修饰，比如下划线，它是 text-decoration-line、text-decoration-color、text-decoration-style，以及新出现的 text-decoration-thickness 的集成，这四种属性分别控制修饰线的位置、颜色、风格和粗细，可用这四个属性的取值任意组合，对 text-decoration 赋值。

示例，红色的粗下划线。

```
text-decoration: underline red 2px;
```

示例，清除 `<a>` 标签文本的下划线。

```
a {text-decoration: none;}
```

**text-shadow**

文字阴影，支持添加多个，值用逗号隔开即可。每个阴影值由元素在 X 和 Y 方向的偏移量、模糊半径和颜色组成，前三个都是 Length 值，颜色和模糊半径可省，模糊半径默认 0 值，越大阴影越淡。

## 字体

**font-size**

字体大小。因为该属性的值会用于计算 em 和 ex 长度单位，修改它可能改变子孙元素的大小。

**font-style**

字体风格。默认 normal，可选择 font-family 字体下的 italic 或 oblique 样式，前者是草书，后者是普通倾斜。

**font-weight**

字体粗细。可用 1 到 1000 之间的数字赋值，值越大字越粗。还能用关键字赋值，默认 normal，与 400 等值。bold 表示加粗，与 700 等值。lighter 表示比从父元素继承的值更低，bolder 表示比从父元素继承的值更高。有些字体只支持 normal 和 bold 两种。

**font-family**

字体家族。它的值是一个优先级从高到低的可选字体列表，逗号分隔，浏览器将选择列表中第一个计算机有安装的字体，或者是通过 `@font-face` 指定的可下载的字体。

注意，字体的选择是逐字进行的，所以可能存在多种字体并存的情况。

## 背景

**background-color**

背景颜色。默认 transparent 表示透明，可用 color 值自定义。

**background-image**

背景图像。默认 none，可用 [image](https://developer.mozilla.org/zh-CN/docs/Web/CSS/image) 赋值。支持多个背景图像，逗号分隔，前面的图像覆盖后面的图像。元素边框在图像上面绘制，但 background-color 在它们下面绘制，若图像半透明，底层的背景颜色会显示出来。

示例，指定背景图像。

```
background-image: url('/static/images/mountain.jpg');
```

> 若有多个背景图像，它们的大小、位置，要么分别设置，逗号分隔，要么都使用同一个值。

**background-size**

背景图像的大小。

| 取值    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| auto    | 默认，按比例拉伸图像。如果图像有固有比例，则指定的长度使用指定值，未指定的长度根据指定值和固有比例计算。如果图像没有固有比例，则指定的长度使用指定值，未指定的长度使用图像相应的固有长度，如果没有固有长度，则使用背景区的长度。 |
| cover   | 拉伸图像直到完全覆盖背景区，部分图像可能看不见。             |
| contain | 拉伸图像直到完全装入背景区，部分背景区可能空白。             |
| Length  | 显式指定长和宽，图像按值拉伸，若只有一个值，则表示宽，此时长隐式为 auto。 |
| %       | 背景图像相对于背景区的百分比。                               |

**background-repeat**

背景图像的重复方式，图像可沿横轴、纵轴进行重复，或者不重复。

| 取值      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| repeat    | 默认，横轴、纵轴都重复，与 repeat repeat 等值。              |
| no-repeat | 图像不重复                                                   |
| repeat-x  | 横轴重复                                                     |
| repeat-y  | 纵轴重复                                                     |
| space     | 图像尽可能重复，但不裁剪。                                   |
| round     | 图像不断重复，直到覆盖背景区，图像会自适应地进行拉伸以适应背景区。 |

**background-position**

背景图像的初始位置。值可以是 x/y 坐标，或关键字 center、top、bottom、left、right，它们表示居中或靠近哪个方向的边界，% 表示起始位置相对于背景区的百分比。以下是一些示例。

```
background-position: 50px 50px;
```

```
background-position: 50% 50%;
```

```
background-position: top right;
```

**background-attachment**

背景图像的附着状态。

| 取值   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| fixed  | 相对视窗固定，图像不随元素或窗口的滚动而变化。               |
| local  | 相对元素内容固定，图像随元素滚动而变化。此时图像的绘制区域和定位区域是相对于可滚动的区域而非包含它们的边框。 |
| scroll | 相对元素本身固定，不随它内容的滚动而变化。                   |

## 边框

**border-style**

边框样式。可按上、右、下、左顺序分别定义，或一个值统一设置。

| 取值   | 说明                                                   |
| ------ | ------------------------------------------------------ |
| none   | 无边框，优先级最低，若与其它边框重叠，则显示那个边框。 |
| solid  | 实线                                                   |
| dotted | 圆点                                                   |
| double | 双实线                                                 |
| dashed | 短的方形虚线                                           |

**border-width**

边框宽度。可按上、右、下、左顺序分别定义，或一个值统一设置。值是关键字 thin、medium、thick，或用长度 Length 明确指定。

**border-color**

边框颜色，可按上、右、下、左顺序分别定义，或一个值统一设置。

**border-radius**

边框半径，它使边框的角变成 1/4 的圆。值是 Length，表示半径。可从左上角开始，按顺时针方向，分别定义四个角，或一个值统一设置。

如果元素盒子是正方形，设置半径为 50% 可使它显示为一个完整的圆形，这常用于用户头像。

**border-collapse**

相邻表格的边框是否合并，默认 separate 表示分割，collapse 表示合并。

## 盒子模型

浏览器解析文档时，它的渲染引擎将根据 CSS 基础盒子模型（CSS basic box model）把所有元素表示为一个个矩形的盒子（box）。CSS 能决定这些盒子的大小、位置以及属性。

每个盒子由四个部分组成，如图，与这四个部分相对应的，每个盒子都有四个边界：内容边界 Content edge、内边距边界 Padding Edge、边框边界 Border Edge、外边框边界 Margin Edge。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/web-0121b44a.png)

**box-sizing**

定义浏览器如何计算盒子的总宽度和总高度，它有两个取值：

* content-box：默认，此时元素的 width 和 height 只表示内容区域，边框和内边距会影响盒子的大小。
* border-box：边框和内边距包含在元素的 width 和 height 之内，它们会挤占内容区域。

## 内边距外边距

**margin**

元素的外边距，它是 margin-top、margin-right、margin-bottom 以及 margin-left 的集成。可按上、右、下、左顺序分别定义，或一个值统一设置。

值是 Length，% 表示相对于包含块宽度的百分比，关键字 auto 让浏览器自己选择合适的外边距，在一些特殊场景它使元素居中。当块元素上下外边距非 atuo，此时设左右外边距为 atuo，可使之水平居中。

> **水平居中**
>
> 对于块元素，可用 `margin: 0 auto;` 或 `display: flex;` + `justify-content: center` 使之水平居中。
>
> 对于内联元素，可用 `text-align: center`。

> **外边距折叠**
>
> 两个元素垂直相邻，且分别设有上、下外边距，这两个边距将重叠，只取其中的最大值。
>
> 水平方向不存在外边距折叠，若两个元素水平相邻，它们的外边距都生效。

> **外边距溢出**
>
> 若父元素没有 border-top 和 padding-top，后代元素的上外边框将溢出，此时父元素的 margin-top 与后代元素取最大值。可在父元素设 `overflow: hidden` 消除后代元素外边距溢出的影响。

**padding**

元素的内边距，它是 padding-top、padding-right、padding-bottom 以及 padding-left 的集成。可按上、右、下、左顺序分别定义，或一个值统一设置。

内边距可能会影响元素盒子的大小，这取决于 box-sizing 属性。

## 元素展示格式

**display**

定义元素是否被视为块或内行元素，以及用于子元素的布局，这里讲解第一个功能，第二个功能见 Flex 节。

HTML 的标签可分为块元素、内行元素，以及内行块元素，图片 `<img>` 就是典型的内行块元素。

| 元素类型     | 特点                                                         |
| ------------ | ------------------------------------------------------------ |
| block        | 独占一行，宽、高、内边距、外边距均可控制，宽默认是整行。     |
| inline       | 允许共行，宽和高无效，内、外边距只在水平方向有效，宽默认是内容宽度。 |
| inline-block | 允许共行，宽、高、内边距、外边距均可控制，宽默认是内容宽度。 |

下例使 `<div>` 表现为内行元素，宽和高随之失效。

```
div {
    width: 100px;
    height: 100px;
    background-color: lightblue;
    display: inline;
}
```

**white-space**

定义元素如何处理内容中的空白。

| 取值     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| normal   | 合并连续空白，换行符被当作空白符处理，遇到 `<br/>` 或填充 line boxes 时换行。 |
| nowrap   | 合并连续空白，换行符无效。                                   |
| pre      | 保留连续空白，遇到换行符或 `<br/>` 时换行。                  |
| pre-line | 合并连续空白，遇到换行符或 `<br/>` 或填充 line boxes 时换行。 |
| pre-wrap | 保留连续空白，遇到换行符或 `<br/>` 或填充 line boxes 时换行。 |

> **行盒 live boxes**
>
> 块元素盒子的内容由许多行组成，当内容填充到某行的末尾时，要么换行，要么溢出块。

**text-overflow**

定义元素如何提示用户存在隐藏的溢出内容，其形式可以是裁剪、显示省略号或显示自定义的字符串。该属性只对那些在块级元素溢出的内容有效，且必须与块级元素的内联方向一致。

可赋一个或两个值，如果是一个值，表示行末溢出行为，如果是两个值，分别表示左、右端的溢出行为。

| 取值     | 说明                             |
| -------- | -------------------------------- |
| clip     | 默认，直接在内容区域的边界截断。 |
| ellipsis | 用省略号表示被截断的内容。       |

该属性只决定如何处理溢出，而不能强制溢出发生，为此可能需要设置两个 CSS 属性：

```
overflow: hidden;
white-space: nowrap;
```

**overflow**

定义元素溢出时的行为，它是 overflow-x 和 overflow-y 的集成。可以赋一个或两个值，如果是两个值，前者用于横轴，后者用于纵轴，否则 overflow-x 和 overflow-y 都用同一个值。

| 取值    | 说明                                                  |
| ------- | ----------------------------------------------------- |
| visible | 不裁剪，内容可能溢出到边界外。                        |
| hidden  | 可以裁剪内容以适应边距。                              |
| scroll  | 可以裁剪内容以适应边距，无论是否裁剪，都提供滚动条。  |
| auto    | 如果内容适应边距，则与 visible 相同，否则提供滚动条。 |

## 位置

**position**

元素在文档内的定位方式，它有以下几个取值：

* static：默认，元素使用正常的布局，按文档流从上到下进行放置。
* relative：元素先按正常文档流放置，然后在不改变页面布局的前提下进行偏移，原空间保留空白。
* absolute：元素被移出正常文档流，并不为其预留空间。通过指定元素相对于最近的非 static 祖先元素的偏移来确定位置，若没有那样的祖先元素，则相对于 ICB 初始包含块。绝对定位元素可设外边距，且不会与其它边距合并。
* fixed：元素被移出正常文档流，并不为其预留空间。通过指定元素相对于视窗的位置来确定位置。元素的位置在屏幕滚动时不会改变，打印时它会出现在每页的固定位置。fixed 元素会创建新的层叠上下文。
* sticky：元素先按正常文档流定位，然后相对于它的最近滚动祖先或最近块级祖先的偏移来确定位置，偏移不影响任何其它元素的位置。sticky 元素会创建新的层叠上下文。

元素使用 top、right、bottom、left 等属性进行偏移。从字面看，它们表示远离哪个方向，而不是向哪个方向偏移。比如 right，在 relative 表示远离原来位置的右边界，向左偏移；在 absolute 或 fixed 表示远离包含块的右边界，向左偏移。对于 sticky 元素，若其在 viewport 内，效果与 relative 相同，否则与 absolute 等效。

属性 z-index 不用于偏移，它决定元素是否创建堆叠上下文和堆叠层级。默认 auto 表示不创建新堆叠上下文，元素的堆叠层级与父容器相同。整数值表示创建一个新的堆叠上下文，值就是堆叠层级，层级越高越优先显示。

## 浮动

**float**

元素被移出正常文档流，然后向左或向右平移，直至碰到所处容器的边框，或碰到另一个浮动元素。它允许文本和内行元素环绕，从这点看，其还保有部分流动性。既然已被移出文档流，自然不预留空间，所以浮动元素可能与其它块元素重合。

| 取值  | 说明     |
| ----- | -------- |
| none  | 不浮动   |
| left  | 向左浮动 |
| right | 向右浮动 |

可用浮动方式把多个块元素放到同一行。

**clear**

指示元素是否移动到它前面的浮动元素下面，该属性可用于浮动和非浮动元素。应用于非浮动块时，它把非浮动块的边框边界移动到所有相关浮动元素外边界的下方，非浮动块的上外边距会折叠。应用于浮动块时，效果相同，但不会发生外边界折叠。

| 取值  | 说明                       |
| ----- | -------------------------- |
| none  | 元素不会向下移动以清除浮动 |
| left  | 元素向下移动以清除左浮动   |
| right | 元素向下移动以清除右浮动   |
| both  | 元素向下移动以清除左右浮动 |

## Flex 布局

样式 `display: flex` 使元素的行为类似块元素，并根据弹性盒子模型布局内容。

> 弹性盒子模型是普通盒子模型的优化，它的子元素可在任何方向排布，并进行弹性伸缩以填满未使用的空间，或避免父元素溢出。弹性元素的水平对齐和垂直对齐也都能很方便地进行操控。

***

以下属性用于 Flex 容器，即设 `display: flex` 的元素。

**flex-direction**

指定 Flex 容器主轴的方向，这与 Flex 元素的布局密切相关。

> 主轴与交叉轴垂直，类似于 X/Y 轴，弹性容器使用它们布局内容。

| 取值           | 说明                                     |
| -------------- | ---------------------------------------- |
| row            | 默认，主轴与文本方向相同。               |
| row-reverse    | 与 row 等效，但调转主轴的起点和终点。    |
| column         | 主轴与交叉轴相同                         |
| column-reverse | 与 column 等效，但调转主轴的起点和终点。 |

**flex-wrap**

是否允许 Flex 容器换行。元素的填充方向，以及行的堆叠方向，由主轴和交叉轴决定。

| 取值         | 说明                                         |
| ------------ | -------------------------------------------- |
| nowrap       | 默认，所有 Flex 元素在同一行，容器可能溢出。 |
| wrap         | 允许换行。                                   |
| wrap-reverse | 与 wrap 相似，但调转堆叠方向。               |

**flex-flow**

是 flex-direction 和 flex-wrap 的集成，默认 `row nowrap`。

**justify-content**

定义浏览器在主轴方向上如何分配元素之间和周围的空间（主轴上的对齐方式）。

| 取值          | 说明                                   |
| ------------- | -------------------------------------- |
| flex-start    | 默认，元素从行的起始位置开始排列。     |
| flex-end      | 元素从行的结束位置开始排列。           |
| space-between | 均匀排列，两侧没有空隙。               |
| space-around  | 均匀排列，两侧的空隙是元素间距的一半。 |
| space-evenly  | 均匀排列，两侧的空隙和元素间距一样大。 |

**align-items**

定义 Flex 元素在交叉轴上的对齐方式。

| 取值       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| flex-start | 向起点对齐。                                                 |
| flex-end   | 向终点对齐。                                                 |
| center     | 居中。如果元素在交叉轴上的高度大于容器，那么两边的溢出距离相同。 |
| stretch    | 弹性元素包含外边距的交叉轴尺寸被拉升至行高，前提是元素没有固定高度。 |

**align-content**

定义浏览器在交叉轴方向上如何分配元素之间以及周围的空间（交叉轴上的对齐方式）。

该属性的取值和作用与 align-items 非常相似，但区别是：它的 Flex 元素在交叉轴方向上没有间距，若 Flex 容器不允许换行，它甚至无法使单行在交叉轴方向居中。

***

以下属性用于 Flex 元素，即 Flex 容器内的元素。

**order**

Flex 元素布局的优先级。值是正整数，默认 0，值越小元素越靠前，它遵循严格排序。

**flex-grow**

Flex 元素主尺寸的增长系数，主尺寸是元素在主轴方向上的宽或高。当 Flex 元素的默认宽度之和小于容器，将把多出的空间根据 flex-grow 的值按比例分配给 Flex 元素，使它们增大。

值是正整数，默认 0，表示不随 Flex 容器变大而变大，值越大增大的越快。

**flex-shrink**

Flex 元素的收缩规则。仅当 Flex 元素的默认宽度之和大于容器时才发生，收缩大小根据 flex-shrink 的值。

值是正整数，默认 1，表示按比例缩小，0 表示不缩小，值越大缩小的越快。

**flex-basis**

Flex 元素在主轴方向上的初始大小。如果没用 box-sizing 修改盒子模型，那么该属性就决定 Flex 元素内容区域的尺寸。它的优先级比 width 或 height 更高。默认 auto，表示使用 width 和 height 的值。

**flex**

是 flex-grow、flex-shrink 以及 flex-basis 的集成，可用关键字设值：

* auto：默认，与 `flex: 1 1 auto` 等价。
* none：与 `flex: 0 0 auto` 等价。

## 响应式布局

响应式布局是一种概念，它的目的是使一个网站能够兼容多个终端，即自适应地根据终端（屏幕尺寸）来调整元素的布局，以为各种终端提供舒适的界面。

**@media**

根据一个或多个媒体查询的结果，决定是否使用某些样式表。当且仅当媒体查询的结果与正在使用该文档的设备匹配时，定义的 CSS 块才生效。

示例，屏幕宽度位于不同的区间时，卡片将显示不同的背景颜色。

```
@media(min-width: 400px) {
    .card {
        background-color: red;
    }
}

@media(min-width: 600px) {
    .card {
        background-color: blue;
    }
}

@media(min-width: 800px) {
    .card {
        background-color: green;
    }
}
```

# JavaScript

JavaScript 是具有函数优先特性的轻量级解释型编程语言，主要用于在浏览器环境作为内置的脚本语言，当然还有其它用途，比如在 Node.js 环境编写后端服务。

> **函数优先特性 First-class Function**
>
> 能把函数当作变量一样对待，函数能赋值给变量，也能作为其它函数的参数或返回值。

JavaScript  支持单行注释 `// 注释` 和多行注释 `/* 注释 */`。

## 嵌入脚本

HTML 文档使用 `<script>` 标签添加 JavaScript 代码或引入 JS 文件。

**内部代码**

直接在 `<script>` 内编写 JavaScript 代码。

```
<script>
	// Hello World!
</script>
```

**外部文件**

使用 `<script>` 的 `src` 属性引入外部 JS 文件。

```
<script src="script.js"></script>
```

**内联代码**

把 JavaScript 写到标签内，不推荐，这使 JavaScript 和 HTML 耦合。

```
<button onclick="createParagraph()">Go</button>
```

**脚本调用顺序**

浏览器从上到下解析 HTML 文档，遇到的 `<script>` 会被立即执行，这可能导致操作 HTML 元素的 JavaScript 加载于它要操作的元素之前。

对于内部代码，可使用 `DOMContentLoaded` 事件，它监听文档解析完成。

```
document.addEventListener("DOMContentLoaded", function() {. . .});
```

对于外部文件，可使用 `<script>` 的标记属性影响它们的加载顺序：

* `async`：告知浏览器继续解析后面的内容，并异步加载当前的 JS 文件。
* `defer`：整个文档解析完成后，再按序加载所有标记该属性的 JS 文件。

## 变量声明

使用关键字 `let`、`const` 和 `var` 申明变量，JavaScript 是动态语言，变量的类型随时可变。

`let` 声明一个块级作用域的本地变量，`const` 声明一个块级作用域的不可变常量。

```
let name = "Tony";
```

```
const NUM = 100;
```

`var` 已经过时，它声明的变量在整个函数可见，且允许重复声明。

> JavaScript 的语句块没有作用域，只有函数有作用域，通过 `var` 声明的变量的作用域是整个函数。

## 数据类型

### 查看类型

关键字 `typeof` 查看变量的类型，若变量还未声明，返回 `null`，若变量还未初始化，返回 `undefined`。

```
typeof varName;
```

### 数字 Number

JavaScript 没有整型，所有数都是浮点数。支持与其它语言相同的算术运算符和比较运算符，不同的是，它用 `**` 而非 `^` 作为求幂运算符。

> **严格比较运算符**
>
> 除 `==`、`!=` 外，JavaScript 还有严格的 `===`、`!==`，它不但比较值，还比较类型。

内置函数 `parseInt(String[, int])` 把字符串转换为整数，可选参数表示数字的进制，默认十进制。

内置函数 `parseFloat(String)` 把字符串转换为浮点数，只适用于十进制。

### 字符串 String

字符串被单引号或双引号围着，可用 `+` 进行拼接，它甚至能拼接其它类型，这将自动调用 `toString()` 函数。字符串有一个属性 `length` 表示长度，可用 `String[index]` 格式访问单个字符，下标基于零。

> **String**
>
> * `indexOf(String)`：查找子串的起始位置，返回 -1 表示子串不存在。
> * `slice(Start[, Length])`：截取子串。
> * `replace(String, String)`：用第二个参数替换第一个参数表示的子串。
> * `toLowerCase()`：所有字符转为大写。
> * `toUpperCase()`：所有字符转为小写。
> * `split(String)`：根据分隔符把字符串分割成数组。

### 数组 Array

数组是特殊的对象，使用 `Array[index]` 格式访问元素，它有一个属性 `length` 表示长度。数组可变，它包含的元素可以增减，类似于 Java 的 `ArrayList`。

传统方式创建数组。

```
let animal = new Array();
```

使用字面量创建数组

```
let goods = ['bread', 'milk', 'cheese', 'hummus', 'noodles'];
```

> **Array**
>
> * `push(Object)`：尾部插入元素。
> * `pop(Object)`：弹出尾部元素。
> * `unshift(Object)`：头部插入元素。
> * `shift(Object)`：弹出头部元素。
> * `toString()`：把所有元素 `toString()` 的值用逗号拼接并返回。
> * `join(String)`：把所有元素 `toString()` 的值用指定字符串拼接并返回。

### 布尔 Boolean

布尔类型有两个可选值：`true` 和 `false`。如果需要，JavaScript 将按以下规则对待其它类型：

* `0`、空字符串、`NaN`、`null` 和 `undefined` 被看作 `false`
* 其它值都被看作 `true`

JavaScript 支持与其它语言相同的逻辑运算符。

### 函数 Function

函数是可重复调用的代码集合，具有输入参数和返回结果。方法是定义在对象内的函数。

使用函数字面量定义函数。

```
function fun_name([arr,...]) {
  ...
}
```

定义匿名函数并赋值给变量。

```
let fun_var = function([arr,...]){...}
```

以下是 JavaScript 函数的特性：

* 如果没有 `return` 语句，或没有 `return` 值，默认返回 `undefined`。
* 允许使用 `...argName` 定义变长参数，以接收剩余的参数。
* 在定义处，可用 `=` 为参数设置默认值。
* 可无视函数定义，输入任意个参数，缺少的参数被默认值或 `undefined` 填充，多的则没有影响。
* 每个函数体内都有一个名为 `arguments` 的原生数组，它包含所有的输入参数。

## 事件响应

事件是系统内发生的动作或事情，系统监听到事件后，如果需要，能以某种方式进行回应。在 Web 环境，事件特指视窗内发生的活动，比如点击按钮、聚焦输入框。

可把 Web 事件与 JavaScript 函数进行绑定，使事件发生后执行指定的代码。在这种机制，被绑定的函数叫做事件处理器，或事件监听器。

### 绑定事件处理器

**事件属性**

表示元素的 DOM 对象，有一类默认为空的属性，它们指向当前元素在不同事件的处理器。通过把函数赋值给这些属性，能够实现事件与处理器的绑定，这类属性叫做事件属性。

```
const btn = document.querySelector('button');

btn.onClick = function() {
  ...
}
```

**函数注册**

DOM Level 2 Events Specification 提出使用函数的方式设置事件处理器。与事件属性相比，这种方式能对同一个元素的同一种事件绑定多个处理器，还能注销处理器。

* `addEventListener(String, Function)`：第一个参数是事件类型，第二个参数是处理器。
* `removeEventListener(String, Function)`

```
const btn = document.querySelector('button');

function bgChange() {
  ...
}

btn.addEventListener('click', bgChange);
```

**内联绑定**

最后一种方式，不推荐，在 HTML 标签内联 JavaScript 代码。

```
<button onclick="bgChange()">Press</button>
```

### 事件对象参数

处理器函数可声明一个参数，名字任意，表示事件对象本身，这个对象的 `target` 属性指向发生事件的 DOM 元素对象。

```
function bgChange(event) {
  const rndCol = 'rgb(' + random(255) + ',' + random(255) + ',' + random(255) + ')';
  event.target.style.backgroundColor = rndCol;
}

btn.addEventListener('click', bgChange);
```

### 阻止默认行为

某些元素对某些事件有默认的处理方案，比如 `<a>` 标签对点击事件将进行页面跳转。可声明事件对象参数，调用它的 `preventDefault()` 函数，来阻止默认行为。

```
form.onsubmit = function(e) {
  e.preventDefault();
  ...
}
```

### 事件的冒泡和捕获

当一个有父元素的元素发生了一个事件，现代浏览器将运行两个不同的流程：

* 捕获阶段：从最外层祖先开始，依次向内，检查是否注册有对应的处理器并执行。
* 冒泡阶段：从当前元素开始，依次向外检查并执行，直到 `<html>` 元素。

事件处理器默认注册在冒泡阶段，可将 `addEventListener()` 的第三个可选参数设为 `true`，以把处理器注册在捕获阶段。

若想阻止事件冒泡，可声明事件对象参数，调用它的 `stopPropagation()` 函数。

```
video.onclick = function(e) {
  e.stopPropagation();
  ...
};
```

## 对象和类

### 初步认识

直接使用对象字面量创建对象。

```
let objectName = {
  memberName : memberValue,
  memberName : memberValue,
}
```

可这样理解，JavaScript 对象就是键值对的集合，键是字符串，值可以是任何类型。其实，键还可以是 Symbol 类型，不多讨论。

使用 `ObjName.Key` 或 `ObjName["Key"]` 格式访问对象的属性和方法。

对象可以在内部使用 `this` 关键字表示自身。

### 对象原型



### 虚假的类

虽然原型能够实现面向对象，但着实不便，为此 JavaScript 提出基于类的 OOP 实现。这种特性在底层依旧是使用原型，它只不过是一种更简单的创建原型链的方式。

**类的定义和构造器**

关键字 `class` 声明一个类。关键字 `constructor` 声明构造方法，若没有定义，类将自动生成一个空构造器。类的属性直接写出，不需要 `let` 等关键字，可在声明处初始化默认值。关键字 `static` 声明静态属性和静态方法。

```
class Person {

  name;

  constructor(name) {this.name = name;}

  greet() {console.log(`Hi! I'm ${this.name}`);}
}
```

**继承**

关键字 `extends` 声明继承自另一个类。子类的构造器必须在最前面使用 `super` 关键字调用父类构造器。

示例，重写 `greet()` 方法，并新增 `grade(paper)` 方法。

```
class Teacher extends Person {

  teaches;

  constructor(name, teaches) {
    super(name);
    this.teaches = teaches;
  }

  greet() {...}

  grade(paper) {...}
}

```

**封装**

属性和方法默认公开，若想声明私有，只需在声明处的名字前加上 `#` 符号，中间没有空格。

```
#name;

#greet() {...}
```

### 使用 JSON

JSON 是一种流行的数据格式，基于 JavaScript 对象语法，但它和 JavaScript 是两种独立的东西。

在 JavaScript，JSON 表示为一个对象或字符串，前者用于解读 JSON 数据，后者用于在网络传输。

**JSON 结构**

可以认为，JSON 对象就是 JavaScript 对象。但是，JSON 只有属性，没有方法，且字符串和 Key 都必须用双引号括起。

```
{
    "name": "value",
    "name": "value"
}
```

对象数组也是合法的 JSON 对象。

```
[
    {
        "name": "value",
        "name": "value"
    },
    {
        "name": "value",
        "name": "value"
    }
]
```

**JSON 转换**

浏览器内置的 `JSON` 对象，包含转换 JSON 对象和 JSON 字符串的方法。

* `parse(String)`：把 JSON 字符串转换为 JSON 对象。
* `stringify(Object)`：把对象转换为 JSON 字符串。

## 控制结构

**条件语句**

```
if (condition) 
	{...}
else
	{...}
```

```
switch (condition) {
  case expression|variable: {
    //...
    break;
  }
  case expression|variable: {
    //...
    break;
  }
  default: {
    //...
  }
}
```

```
condition ? expression : expression
```

**循环语句**

支持 `break` 和 `continue` 关键字。

```
for (var i = 0; i < 5; i++) {
  // 将会执行五次
}
```

```
while (true) {
  // 一个无限循环！
}
```

```
do {
  input = get_input();
} while (inputIsNotValid(input))
```

**特殊的 For 循环**

为可迭代对象（`Array`、`Map`、`Set`、`String` 等）创建循环，对其包含的每个值执行语句。

```
for (let value of array) {
  // do something with value
}
```

以任意顺序迭代对象内除 `Symbol` 之外的可枚举属性，包括继承的可枚举属性。

```
for (let property in object) {
  // do something with object property
}
```

## 输入输出

**输入**

`process.stdin` 是标准输入流，默认设备是键盘。使用函数 `on()` 注册事件处理器，其中，`readable` 事件表示输入缓冲区有内容，`end` 事件表示输入结束。

```
let buf = '';

process.stdin.on("readable", function() {
    let chunk = process.stdin.read(); // 从缓冲区读数据
    if (buf) buf += chunk.toString();
})

process.stdin.on("end", function() {
    // 操作读的数据
})
```

**输出**

`console.log()` 向控制台打印信息，支持变量填充，需用飘号替代引号。

```
console.log("Hello World!");
```

```
console.log(`Hello I'm ${varName}`); // 把变量填充到字符串
```

使用飘号定义多行字符串。

```
let s = 
    `<div>
    	<p> Hello </p>
    </div>`
```

函数 `toFixed()` 设置保留多少位小数。

```
console.log(`${amount.toFixed(2)}`); // 保留两位小数
```

## 时间日期

`Date` 实例表示某个时刻，基于 Unix Time Stamp，即自 1970 年 1 月 1 日起经过的毫秒数。

下面是两个常用的静态方法：

* `now()`：返回自 1970-1-1 00:00:00 至今经过的毫秒数。

* `parse(String)`：解析一个表示日期的字符串。

  ```
  let time = Date.parse("2022-12-10 10:30:20");
  ```

两个 `Date` 实例的差值，是它们相距的毫秒数。

## 模块功能

所谓模块，是 JavaScript 代码的集合，它也定义在 JS 文件，内容与普通 JS 文件相似，但它能主动把某些成员暴露出去，以供其它模块或 HTML 文档导入和使用。对于普通 JS 文件，它的导入只能是全部，这也是模块与 JS 文件最明显的不同。现代浏览器基本都原生支持模块功能。

### 导出 export

关键字 `export` 导出成员，被导出的成员依然能在本地进行修改，且这些修改会同步更新到导入的地方。

无论是否声明，模块导出的成员，都运行在严格模式。

`export` 语句不能声明在嵌入式脚本。

**命名导出**

声明处导出单个成员。

```
export function () { ... };
```

批量导出多个成员。

```
export {funName, varName, ...};
```

使用 `as` 关键字重命名导出成员的标识符。

```
export {funName as newFunName, ...};
```

**默认导出**

模块只能声明一个默认导出，且默认导出的成员没有标识符，由导入的地方自命名。

```
export default function () { ... };
```

```
export {funName as default};
```

### 导入 import

关键字 `import` 导入其它模块定义的导出成员。

通过导出处指定的标识符，导入命名导出。

```
import {funName, varName as newName, ...} from './moduleName.js';
```

通过模块的文件名，导入默认导出，标识符自定义。

```
import defaultName from './moduleName.js';
```

混合导入，默认导出放在前面。

```
import defaultName, {funName, varName as newName, ...} from '/modules/my-module.js';
```

导入模块的所有导出，并放到自定义的命名空间。

```
import * as spaceName from '/modules/moduleName.js';
```

> 在浏览器，`import` 语句只能在有 `type="module"` 属性的 `<script>` 标签内使用，这个属性有以下作用：
>
> * 表示导入的是模块内容，而非 JS 文件。
> * 导入的模块内容运行在严格模式。
> * 模块的加载自动延迟，不需要 `defer` 属性。
> * 导入的模块内容的作用范围只在这个标签，无法全局获取。

> 严格模式使得 JavaScript 的运行具有限制：
>
> * 通过抛出错误，消除原来的某些静默错误。
> * 修复某些导致 JavaScript 引擎难以执行优化的缺陷，加快代码运行。
> * 禁用在 ECMAScript 未来版本可能定义的语法。

## ES6 语法

### 展开语法

使用 `...` 符号，把数组按值展开，或把 String 按字符展开，或把对象按 key-value 展开，以快速地使用现有成员的内容，构造新的成员或进行函数调用。

函数调用。

```
function(...obj);
```

构造数组。

```
[...obj, '4', ...'hello', 6];
```

克隆对象，浅拷贝。

```
let objClone = { ...obj };
```

### 解构赋值

这是一种表达式语法，用于快速地把数组或对象的内容取出，赋给其它变量。

两种解构模式：

* 绑定模式：以声明关键字开始，把值绑定到变量或进一步解构。
* 赋值模式：不以声明关键字开始，赋值目标是已有的变量或对象的属性。

对于数组，按序结构，若中间有想要忽略的值，不写赋值目标即可。

```
[N1, , N3] = [1, 2, 3]; // 第二个值被忽略
```

对于对象，先用相同的名字和结构取值，再用 `:` 把值转给目标。第二步可省，默认赋值给同名变量。

```
({N1 : M1, N2 : M2} = {N1 : 'N1', N2 : "N2"});
```

> 当对字面量对象进行解构且不带声明关键字时，赋值语句周围必须添加括号。

赋值目标可设默认值。当属性不存在或值为 `undefined` 时，使用默认值，属性值为 `null` 时不使用默认值。

```
({N1 = 'NO', N2 : M2 = 'NO'} = {N1 : 'N1', N2 : "N2"});
```

可用 `...var` 结束解构模式，该目标将接收剩余的所有内容。

```
let [N1, ...Last] = [1, 2, 3];
```

### 绑定 this

在 JavaScript，函数内 `this` 指向的是调用者，而非定义时所在的对象。函数调用 `bind()`，返回一个函数，这个函数的执行逻辑与原函数相同，但它的 `this` 指向的是 `bind()` 的第一个参数。

```
let newFun = objobjName.oldFun.bind(objName);
```

# Client Web API

客户端 API 由第三方提供，目的是使 JavaScript 具备额外的功能。客户端 Web API 指 JavaScript 在浏览器环境特有的非原生接口，它们或内置于浏览器，或由第三方提供，前者可在浏览器直接运行，后者需要专门获取。

JavaScript 的 API 都基于对象，对象是 API 所需数据以及方法的容器。通过一个或多个对象使用 API，有些 API 的所有功能都挂在一个对象，而有些需要额外的对象先创建上下文才能继续使用。

## 文档操控

下图表示浏览器界面的主要部分，本节介绍操作这些部位的客户端接口。

![](https://images-1305875271.cos.ap-chengdu.myqcloud.com/web-d2548bfb.png)

### 标签 Window

浏览器内置对象 `Window` 表示标签页，下面是 `Window` 的简单应用。

重新加载当前页面。

```
window.location.reload();
```

在新标签打开链接。

```
window.open("https://www.google.com");
```

在当前标签打开链接。

```
window.location.href = "https://www.google.com";
```

窗口绑定事件，窗口大小变化时触发，使块元素始终与视窗相同大小。

```
window.onresize = function() {
  WIDTH = window.innerWidth;
  HEIGHT = window.innerHeight;
  div.style.width = WIDTH + 'px';
  div.style.height = HEIGHT + 'px';
}
```

### 状态 Navigator

浏览器内置对象 `Navigator` 表示浏览器在 Web 的状态和标识，通过它可获取用户语言、地理位置等信息。

```
console.log(navigator.platform); // 操作系统
console.log(navigator.appName); // 浏览器产品名称
console.log(navigator.appVersion); // 浏览器版本号
console.log(navigator.userAgent); // 浏览器的用户代理
console.log(navigator.cookieEnabled); // 是否启用Cookies
```

### 文档对象模型 DOM

文档加载完后，浏览器将根据 DOM 对文档进行建模：把文档的所有内容，包括元素、属性、文本、注释等，按照包含关系，抽象成树结构。

浏览器内置有操作 DOM 的接口，基于 `Document` 单个对象，下面是 `Document` 和 `Node` 的简单应用。

**查找结点**

根据 CSS 选择器，获取第一个匹配的结点。

```
let root = document.querySelector(".root");
```

**增删结点**

创建元素结点。

```
let div = document.createElement('div');
```

创建文本结点。

```
let text = document.createTextNode('Hello World');
```

把文本结点插到子结点列表尾部。

```
div.appendChild(text);
```

把文本结点从子结点列表移除。

```
div.removeChild(text);
```

**修改样式**

通过 `HTMLElement.style` 属性，为结点添加内联样式。

```
div.style.color = 'white';
div.style.backgroundColor = 'black';
```

或者，修改元素属性，让其适配其它 CSS 选择器。

```
div.setAttribute('class', 'highlight');
```

## 数据存储

现代 Web 浏览器都有本地存储的功能，这使网站的数据不仅来自服务端，还可能来自本地设备。

### Cookie

最初 Web 浏览器使用 HTTP Cookie 进行客户端存储，Cookie 根据服务端响应头的 `Set-Cookie` 字段设置，浏览器保存它们，并在此后向同一地址/服务器发送请求时携带它们。

属性 `Document.cookie` 表示所有能从当前位置访问的 Cookie，这一个字符串，键值对以 `;` 分隔。

```
cookies = document.cookie;
```

写入一个新的 Cookie，看似赋值，实际是添加。

```
document.cookie = "name=tony";
```

### WebStorage

现代浏览器还有其它的客户端存储方式：WebStorage 和 IndexedDB，前者适合简单的键值对，而后者是个完整的数据库系统，能存储复杂数据。它们都比 Cookie 强大的多，浏览器也不会自动把它们装入请求。

这里讨论 WebStorage，它有两种实现：`LocalStorage` 和 `SessionStorage`，前者的数据持久有效，而后者的数据在关闭浏览器时将被清空，它们的 API 基本相同。下面是 `LocalStorage` 的简单应用。

> 注意 SessionStorage 和 Session 的区别，前者保存在客户端，后者保存在服务端。

存储键值对。

```
localStorage.setItem('name','Chris');
```

获取键的值。

```
let name = localStorage.getItem('name');
```

删除键值对。

```
localStorage.removeItem('name');
```

清空所有键值对。

```
localStorage.clear();
```

## 定时任务

**`Window.setTimeout(Function[, delay])`**

延迟指定时间后调用一次参数函数，单位毫秒，默认 0 表示立即执行。返回一个正整数 `timeoutID`，表示这个定时器的编号，把编号传给 `clearTimeout(timeoutID)` 可取消任务。

**`Window.setInterval(Function[, delay])`**

延迟指定时间后按固定时间间隔反复调用参数函数，单位毫秒，默认 0 表示 4 毫秒。返回一个正整数表示这个定时器编号，把编号传给 `clearInterval(intervalID)` 可取消任务。

**`Window.requestAnimationFrame(Function)`**

告诉浏览器，在下次重绘之前调用指定函数。所谓重绘，可理解为屏幕刷新。屏幕的刷新率固定，可在参数函数内回调 `requestAnimationFrame()`，将以稳定的频率反复执行程序，这非常适合于更新动画。

回调函数可声明一个参数，表示此次调用距离初次调用的时间间隔，单位毫秒。

为提高性能和电池寿命，在大多数浏览器，当该方法运行在后台标签页或隐藏的 `<iframe>` 里时，将暂停运行。

## JQuery

JQuery 是 JavaScript 库，包含一个 JS 文件，它有许多 JavaScript 语法糖，使得 JS 的编写更加简单。

> [JQuery Download](https://jquery.com/download/)

**强大的 $ 符号**

根据 CSS 选择器，获取第一个匹配的元素，返回的是 JQuery 对象而非 DOM 对象。

```
let $A = $(Selector);
```

创建 JQuery 对象。

```
let $B = $('<div> Hello World </div>');
```

**添加、删除元素**

* `append($A)`：把 `$A` 添加到子元素列表尾部。
* `prepend($A)`：把 `$A` 添加到子元素列表头部。
* `remove()`：删除当前元素。
* `empty()`：清空子元素列表。

**查找并筛选元素**

* `parent([Filter])`：查找父元素，可选参数是类似 CSS 选择器的字符串，用于筛选。
* `parents([Filter])`：向祖先元素查找。
* `children([Filter])`：在子元素查找。
* `find([Filter])`：向后代元素查找。

**元素的类 class**

* `addClass(className)`：添加类
* `removeClass(className)`：删除类
* `hasClass(className)`：判断元素是否属于某个类

**元素属性 attr**

* `attr(attrName)`：获取属性
* `attr(attrName, attrVal)`：设置属性

**事件处理器**

* `on(String, Function)`：绑定处理器，第一个参数是事件类型。处理器返回 `false` 阻止冒泡和默认行为。
* `off(String, Function)`：注销处理器。

**修改 CSS 样式**

* `css(attrName)`：获取 CSS 属性
* `css(attrName, attrVal)`：设置 CSS 属性

* `css(attrName, Object)`：使用对象同时设置多个 CSS 属性

**文本内容和 HTML 内容**

HTML 内容包括标签，文本内容只是元素内容。

* `html([String])`：获取、修改 HTML 内容。
* `text([String])`：获取、修改文本内容。
* `val([String])`：获取、修改元素的 `value` 属性，常用于 `<input>`、`<textarea>` 元素。

**Ajax，Asynchronous JavaScript And XML**

Ajax 用于异步地发送 HTTP 请求和接收响应，它能处理 JSON、XML 和 Text 等格式的数据，然后把数据更新到网站，而不用刷新整个页面。

Ajax 只是个语法糖，它是多个技术的整合，其中最重要的部分是 `XMLHttpRequest` 对象。

```
$.ajax({
    type: "get", // 请求方法，必须
    url: "http://www.google.com", // 请求路径，必须
    async: true, // 是否异步，默认True
    data: "", // 请求数据
    dataType: "json", // 响应类型
    contentType: "", // 请求头，编码类型
    success: function (data) { // 请求成功的回调函数

    },
    error: function () { // 请求失败的回调函数

    }
})
```

# React

不同于原来把文档、样式、脚本分离的开发习惯，React 的思想，是把网页划分为可复用的组件，组件是标签、样式和脚本的组合。

通过组件，可实现模板与数据的分离，这既减少重复代码，还能代码的提高可移植性。

## 开始使用

安装 [Node.js](https://nodejs.org/en)，这是 JavaScript 的运行环境，它使 JavaScript 能脱离 Web 浏览器独自运行。检查 Node 环境，执行命令 `node` 进入交互界面，或执行 `node [filePath]` 以运行 JS 文件。

安装 `create-react-app` 包，该包用于创建 React 项目。`npm` 是 Node 的包管理工具，附带安装。

```
npm i -g create-react-app
```

使用 `create-react-app` 创建 React 项目，对应的文件夹自动生成。

```
create-react-app [ProjectName]
```

使用 `npm` 安装 `bootstrap` 库，并在 React 项目导入。

```
npm i bootstrap
```

```
import 'bootstrap/dist/css/bootstrap.css';
```

启动 React 项目，使用浏览器访问网页。

```
npm start
```

VSCode 相关拓展：`ESLint`、`Prettier - Code formatter`、`Simple React Snippets`。

## 描述视图

组件 Component 是网页 UI 元素的抽象，React 通过组件构建网页，组件不但包含基本的 HTML 标签，还能设置样式和添加脚本。

### 创建组件

组件 Component 是返回一个标签的函数，其实它还能是类，但那种定义方式早已过时。

```
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  )
}
```

表示组件的函数的名字必须大写首字母，React 以此区分组件标签和 HTML 原生标签。

JavaScript 语句可以分号，若 `return` 的标签包括多行，应该用括号把它们围着，否则将忽略第一行之后的内容。

不要在组件内定义组件，这容易导致错误，重新渲染时，React 把两次函数调用得到的内部组件视为两种组件。

### 使用组件

**组件标签**

组件就是一个 UI 元素，它只能在 React 内使用，把它看作自定义的 HTML 标签即可，标签名就是函数名。

> 在 Node 环境，导入模块可以不添加 `.js` 后缀，有些打包工具也支持这种导入模块的方式。

```
import Profile from './profile'

export default function Gallery() {
  return (
    <section>
      <Profile /> // 两个标签，就是两个组件实例
      <Profile />
    </section>
  );
}
```

**引用 JavaScript**

可在返回标签内引用 JavaScript 的数据，这是通过 `{}` 实现，花括号内填写 JavaScript 表达式，式子的返回值将作为文本填充到标签。

只有两种场景能使用 `{}`：标签文本内容、标签属性赋值。

```
return (
  <img
    className="avatar"
    src={avatar}
    alt={description}
  />
);
```

**条件渲染**

可使用 JavaScript 的 `if`、`?`、`&&` 等语句，选择性的渲染标签。

```
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

逻辑表达式：

* 对于 `&&`，当第一个表达式为真，则返回第二个表达式的值。
* 对于 `||`，当第一个表达式为假，则返回第二个表达式的值。
* React 把 `false` 返回值当作 `null` 处理。

```
return (
  <li className="item">
    {name} {isPacked && '✔'}
  </li>
);
```

**响应事件**

React 的特点就是把 JavaScript 与 HTML 混合，所以，推荐以内联的方式设置事件处理器。

```
<button onClick={function handleClick(e) {
  e.stopPropagation();
  e.preventDefault();
  // ...
}}>
```

### 组件通信

父组件可向嵌套的子组件传递信息。

**标签属性**

每个组件的函数，都有一个唯一的参数 `props`，这是一个对象，它包括从父组件传过来的所有标签属性。通常不直接使用 `props` 对象，而是将其结构，明确声明使用哪个属性。

```
function Avatar({ person, size }) {
  // ...
}

export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

**标签内容**

父组件在子组件标签内填写的内容，将被赋给 `props` 对象的 `children` 属性。

```
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar/>
    </Card>
  );
}
```

组件在不同时刻可能收到不同的 `props`，React 察觉在到变化后，将会重新渲染页面。

子组件不能修改 `props`，当需要不同的 `props` 时，应请求父组件传递新的 `props`。

### 渲染列表

使用 JavaScript 的数组方法，可把一个数据集转换成多个相似的组件。

**`map(Funcation)`**

遍历数组所有元素，对每个元素执行指定的操作，把所有返回结果组合成新的数组返回。

```
export default function List() {
  const listItems = people.map(person =>
  	<li key={person.id}>
  	  {person}
    </li>
  );
  return <ul>{listItems}</ul>;
}
```

通过 `map()` 得到的新数组，其包含的每个项都应该有一个在组内唯一的 key 属性。若没有显式设置，React 将根据数组下标为其设置 key，这容易导致错误。

**`filter(Funcation)`**

遍历数组所有元素，执行逻辑判断，把返回 `true` 的元素组合成新的数组返回。

```
const chemists = people.filter(person =>
  person.profession === 'teacher'
)
```

### JSX 语法

这种能在 JavaScript 代码中写 HTML 的语法，叫做 JSX（JavaScript XML）。还有其它写组件的方式，但 JSX 最大众。React 项目的 JS 文件，其实都是 JSX 文件，React 使用 Babble 解释器把它们转换成 JS 代码。

React 支持 JSX 语法，这是两个相互独立的东西。React 项目的 JS 文件，其实都是 JSX 文件，运行时，React 使用 Babel 把 JSX 转换为 JavaScript 语句。

下面是 JSX 几个重要的规则：

* 函数只能返回一个根标签，如果需要，可用 `<Fragment>` 或 `<></>` 包着同级标签。
* 标签必须闭合。
* 大部分带 `-` 的原生属性，都转换为驼峰命名。
* 类似 `class` 这样的属性，与 JavaScript 关键字重名，在 JSX 转为 `className`。

如果有大量现成的 HTML 内容要转换为 JSX 格式，可使用[转换器](https://transform.tools/html-to-jsx)工具。

## 组件状态

网页交互时，组件可能需要记住某些信息，比如购物清单，这可通过状态 state 实现，它有两种功能：

* 在组件的多次渲染之间，保留数据。
* 通过 `setter` 函数更新状态，将触发组件使用新数据重新渲染。

### 创建 state

调用 Hook 函数 `useState()`，参数是 state 的初始值，返回包含两个元素的数组，第一个元素是 state 变量，第二个元素是该 state 的 `setter` 函数。

```
const [index, setIndex] = useState(0);
```

调用 `setter` 函数更新 state 的值，参数是一个值，表示用这个值重新渲染组件。

```
setIndex(index + 1);
```

每个组件可声明多个 state 变量，相同组件的不同实例，它们的 state 变量相互独立。

### 更新 state

**state 快照**

状态 state 在组件内的表现就像快照。每次渲染，React 都会为组件创建一张快照，记录状态的值。组件内所有用到 state 的地方，其值都来自于当前的快照。

示例，多次调用 `setter` 函数，但当前快照不会改变，这相当于调用 `setNumber(1 + 1)` 三次。

```
const [number, setNumber] = useState(1);
for(let n = 0; n < 3; n++) {
  setNumber(number + 1);
}
```

**state 批处理**

渲染过程，React 把遇到的所有 state 更新操作按序插入队列，直到事件处理函数的代码都执行完，再一并执行所有 state 更新，这就是批处理。

若想在下次渲染之前多次更新同一个 state，可向 `setter` 函数传入一个函数，而非值，这告诉 React 使用队列的前一个 state 而非当前快照来进行计算。

```
const [number, setNumber] = useState(1);
for(let n = 0; n < 3; n++) {
  setNumber(n => n + 1);
}
```

**修改 state 对象**

若 state 是一个对象，不要直接修改它，这无法影响下次渲染的 state 快照，也不能触发组件的渲染，还会破坏当前的 state 快照。无论 state 是什么类型，都应该把它视为只读。

修改 state 对象的正确做法：创建一个新的对象，或拷贝已有对象，调用 `setter` 函数替换旧对象。若要修改嵌套对象，那这个嵌套对象同样需要被替换，因为解构赋值是浅拷贝。

```
const newObj = {...oldObj};
// update newObj
setObje(newObje);
```

**修改 state 数组**

若 state 是一个数组，修改它的方式与对象相似，即用一个新数组替换旧数组。

可用 `[...array]`、`map()`、`filter()`、`slice()` 等方法，实现 state 数组的更新。

### 保留/重置 state

虽然 state 在组件内创建，但它是由 React 在组件外管理。React 根据组件在 UI 树的位置，把它管理的 state 与组件进行关联。

**保留**

若发生渲染的组件，与上次渲染相比，是在相同位置的相同组件，则保留它的 state 快照。

React 判断组件与上次渲染时是否相同，是根据定义（函数）而非实例。所以，如果此次渲染是用相同组件的新实例替换旧实例，那么 state 不会被重置。

示例，重新渲染时无论显示哪个实例，在 React 看来组件都没有变化，不会导致 state 被重置。

```
{exist ? <Counter /> : <Counter />}
```

**重置**

若此次渲染，发生变化的位置，无论是组件更替、增加、删除，都会导致对应位置的 state 被重置，且整棵子树也会被重置。

如果想用相同组件的新实例替换旧实例，要么使它们渲染在不同位置：

```
{exist && <Counter />}
{!exist && <Counter />}
```

要么，为不同实例设置 key 标识符：

```
{exist ? <Counter key="A" /> : <Counter key="B" />}
```

父组件内唯一的 key 属性，是 React 区分不同组件的依据，它们默认按在父组件内的顺序赋值。

### 组件渲染的过程

浏览器用树形结构 DOM 描述 HTML 元素，React 也有类似的结构，即 React DOM，也叫 UI 树。React 通过它对基于 JSX 构建的 UI 元素进行管理和建模，同时，React 也根据这棵树对浏览器的 DOM 元素进行更新。

根据组件当前的内容，修改 React DOM，然后把修改更新到浏览器的 DOM 元素，这个过程叫做渲染。只有两种情况会导致组件的渲染：组件初次渲染，组件 state 改变。

以 state 更新为例，渲染分为三个步骤：

* 触发：调用 `setter` 函数，state 被修改。

* 渲染：根据新的快照，修改 React DOM。
* 提交：检查 React DOM 与 DOM 是否有区别，若有则修改 DOM 元素。

初次渲染，React 从根组件开始调用。后续渲染，React 从发生改变的组件开始递归调用。所谓递归，即遇到的子孙组件也会依次重新渲染。

## 其它特性

### Reducer

朴素的 state 应用方式，是把 state 的更新逻辑写到事务处理程序当中，若组件包含太多的状态更新事件，将使代码的维护变得困难，也不易读。

Reducer 提供了另一种维护 state 的方案，它把所有更新 state 的逻辑整合到一个外部函数，需要时，事件处理器只需告诉 Reducer 函数发生了什么事情，让它执行对应的操作。

**定义 Reducer 函数**

Reducer 函数包含所有 state 更新的逻辑，它接收两个参数，分别是当前的 state 和 action 对象，返回更新后的状态值。参数对象 action 包含事件处理器传递过来的信息，通常都有一个 `type` 属性，描述当前发生的事件。

```
function dataReducer(state, action) {
  switch (action.type) {
  	case "add": {
  	  //...
  	}
  	case "delete": {
  	  //...
  	}
  	default: {
  	  //...
  	}
  }
}
```

**创建 Reducer 状态**

导入 Reduer 组件。

```
import {useReducer} from 'react';
```

调用 `useReducer()`，这是一个 Hook 函数，起声明作用。它接收两个参数，第一个是 Reducer 函数，第二个是状态的初始值。返回包含两个元素的数组，第一个元素是状态的值，第二个元素是 dispatch 函数。

```
const [data, dispatchTask] = useReducer(dataReducer, initialdata);
```

**触发 Reducer 更新**

事件处理器使用前面的 dispatch 函数，向 Reducer 传递信号，让其进行状态更新。它接收一个 action 对象，这个对象就是 Reducer 函数的第二个参数。

```
function handleAdd(itemId) {
  dispatchTask(
    {
      type: 'add',
      id: itemId,
    }
  );
}
```

### Context

## Route 路由

