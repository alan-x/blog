### 概述

整个规格定义了层叠样式表，等级 2 修订 1（CSS 2.1）。CSS 2.1 是一个样式表语言，允许作者和用户去绑定样式（比如，字体和空格）到结构化文档（比如，HTML 文档和 XML 应用）。通过分离文档的表现样式和文档内容，CSS 2.1 简化了网页写作和站点维护。

CSS 2.1 基于 CSS2[CSS2]，CSS 2 基于 CSS1[CSS1]。它允许媒体指定样式脚本，这样作者可以调整他文档的表现到可视化浏览器，听力设备，打印机，盲文设备，手持设备，等。它同样支持内容定位，表格布局，国际化功能和用户界面相关属性。

CSS 2.1 纠正了一些 CSS2 中的错误（最重要的是绝对定位的元素的高/宽的新定义，HTML 的“style”属性更大的影响，和‘clip’属性新的计算），并添加了一些高请求功能，这些已经被大范围实现。但是大部分 CSS 2.1 代表 CSS 使用的“快照”：它由 Recommendation 发布的时候实现的所有 CSS 功能构成。

CSS 2.1 来自 CSS2，为了替代 CSS2。CSS2 的一部分在 CSS 2.1 中并没有改变，一部分被修改，一部分被移除。被移除的一部分可能被用在未来的 CSS 规格中。未来的规格应该引用 CSS2.1（除非需要 CSS2 的功能，但是被 CSS 2.1 移除，否则他们应该只为这些功能引用 CSS2，或者最好引用这些功能在分别包含这些功能的 CSS3 模块 ）。


### 这个文档的状态
这个章节描述了这个文档在它发布的时候的状态，其他文档可能替代这个文档。当前 W3C 发布的列表和这些技术报告最新的修订可以在 http://www.w3.org/TR/ 的W3C 技术报告索引找到。

这个文档已经经过 W3C 成员，软件开发者，和其他 W3C 组和相关团体审查，并被 Director 认可作为 W3C Recommendation。它是一个稳定文档，可能用作参考资料或者从其他文档引用。W3C 在创建 Recommendation 的角色是提醒注意规范并确保广泛部署。这增强了 Web 的功能行和可交互性。

（存档）公共邮件列表 www-style@w3.org（查阅指令）是讨论这个规格的首选。当发送 e-mail，请在标题中加上“CSS21”，最好像这样：“[CSS21]... 评论概述 ...”。

这个文档由 CSS Working Group（Style Activity 的一部分）编写。

这个文档是遵循 5 February 2004 W3C Paten Policy 的组织编写。W3C 维护一个和组织交付成果相关的任何专利披露的公开列表；页面也包含披露一个专利的说明。一个独立有专利知识的人认为包含 Essential Claim 必须根据 W3C Patent Policy 章节 6 披露信息。

工作组创建了一个测试套件和一个实现报告。

所有以前的 Working Draft，以前的 Cadidate Recommendation 和以前的 Recommendation 的改变被列在附录 C。

注意：这个规格的很多章节已经被其他规格更新。请，查阅“层级样式表（CSS）-官方定义”最新的 CSS 快照规格列表和他们替代的章节。

CSS 工作组也在开发 CSS 等级 2 修订 2（CSS 2.2）。

### 8 盒模型

CSS 盒模型描述文档树中元素生成的方形盒子，和基于可视格式化模型的布局。

### 8.1 盒尺寸

每一个盒子都有一个内容区域（比如，文字，一个图片，等）和可选的围绕的 padding，border，margin 区域；每一个区域的大小被下面定义的属性指定。下面的图像显示这些区域是怎样关联的和引用每一个 margin，border，和 paddidng 的术语：

![](https://www.w3.org/TR/CSS2/images/boxdim.png)

marging，border，padding 可以分离为 top，right，bottom，和 left 片段（比如，在图像中，“LM”为 left margin，“RP”为 right padding，“TB”为 top border，等）。

四个区域的边界（content，padding，border，和 margin）称为“edge”，所以每一个盒子都有四个 edge：

- content edge 或者 inner edge
    content edge 包围在盒子的 width 和 height 指定的长方形，通常基于元素的渲染的内容。4 个 content edge 定义盒子的 content box。
- padding edge
    padding edge 包围 box padding，如果 padding 的 width 是 0，paddng edge 和 content edge 一致。4 个 padding edge 定义盒子的 padding box。
- border edge
    border edge 包围在盒子的 border。如果 borde 的宽度是 0，border edge 和 padding edge 一致。4 个 border edge 定义盒子的 border box。
- margin edge 或者 outer edge
    margin edge 包围盒子的 margin。如果 margin 的宽度是 0，margin edge 和 border edge 一致。4 个 margin edge定义盒子的 margin box。

每一个 edge 可能划分为 top，right，bottom，和 left edge。


盒子内容区域的尺寸 -- content width 和 heigth width -- 依赖于一系列的因素：元素生成的盒子是否有‘width’和‘height’属性集合，盒子是否包含文字或者其他盒子，盒子是不是表格，等。盒子宽度和高度在可视化格式模型章节详细讨论。

盒子的 content，pading，和 border 区域的背景样式通过生成的元素的‘background’属性指定。margin 背景总是透明的。

### 8.2 margin，padding，和 border 的例子

这个例子描述了 margin，padding，和 border 是如何交互的。例子的 HTML 文档：
```
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<HTML>
  <HEAD>
    <TITLE>Examples of margins, padding, and borders</TITLE>
    <STYLE type="text/css">
      UL { 
        background: yellow; 
        margin: 12px 12px 12px 12px;
        padding: 3px 3px 3px 3px;
                                     /* No borders set */
      }
      LI { 
        color: white;                /* text color is white */ 
        background: blue;            /* Content, padding will be blue */
        margin: 12px 12px 12px 12px;
        padding: 12px 0px 12px 12px; /* Note 0px padding right */
        list-style: none             /* no glyphs before a list item */
                                     /* No borders set */
      }
      LI.withborder {
        border-style: dashed;
        border-width: medium;        /* sets border width on all sides */
        border-color: lime;
      }
    </STYLE>
  </HEAD>
  <BODY>
    <UL>
      <LI>First element of list
      <LI class="withborder">Second element of list is
           a bit longer to illustrate wrapping.
    </UL>
  </BODY>
</HTML>
```

导致一个文档树（在其他关系中）携带一个 UL 元素有两个 LI 子元素。

下面的第一张图描述了例子产生。第二张图描述了 UL 元素和这些子 LI 元素的 marging，pading，和 border 之间的关系。（图片不是按比例的）

![](https://www.w3.org/TR/CSS2/images/boxdimeg.png)

![](https://www.w3.org/TR/CSS2/images/boxdimeg.png)

注意：
- 每一个 LI 盒子的 content width 是从上到下计算的；每一个 LI 盒子的包含块由 UL 元素建立。

- 每一个 LI 盒子的 margin 盒子的高度依赖于它的内容高度，加上 top 和 bottom padding，border，和 margin。注意 LI 盒子的垂直 margin 会折叠。

- LI 盒子的右 padding 被设置为 0 宽度（‘padding’属性）。效果在第二张图显示。

- LI 盒子的 margin 是透明的 -- margin 总是透明的 -- 所以 UL padding 和 content 区域的背景颜色（黄色）透过他们。

- 第二个 LI 元素指定虚线标签（’border-style‘属性）。  

### 8.3 Margin 属性：‘margin-top’，‘margin-right’，‘margin-bottom’，‘margin-left’，和‘margin’。

margin 属性指定盒子 margin 区域的宽度，‘margin’缩写属性设置 4 边的 margin，其他 margin 属性只设置各自的边。这些属性应用于所有的元素，但是垂直 margin 对于非替换行内元素没有任何效果。

- <length>
    指定固定的宽度

- <percentage>
    百分比是根据生成的盒子包含块的 width 计算的。注意这对于‘margin-top’和‘margin-bottom’也是真的。如果包含块的宽度依赖于这个元素，则导致的布局在 CSS 2.1 没有定义。

- auto
    查阅计算宽度和编剧的行为的章节。

marin 属性负值也是允许的，但是这是实现指定的限制。


‘margin-top’，‘margin-bottom’
    Value：         <margin-width> | inherit
    Initial:        0
    Applies to:     所有除了 table 显示类型不是 table-cation，table 和 inline-table 的元素
    Inherited:      no
    Percentages:    参考包含块的宽度
    Media:          visual
    Computed value: 指定的百分比或者绝对长度

这些属性对于非替代行内元素无效。

‘margin-right’，‘margin-left’
    Value：         <margin-width> | inherit
    Initial:        0
    Applies to:     所有除了 table 显示类型不是 table-cation，table 和 inline-table 的元素
    Inherited:      no
    Percentages:    参考包含块的宽度
    Media:          visual
    Computed value: 指定的百分比或者绝对长度

这些属性设置盒子的 top，right，bottom，和 left margin。
```
h1 { margin-top: 2em }
```
‘margin’
    Value：         <margin-width>{1,4} | inherit
    Initial:        查看各自的属性
    Applies to:     所有除了 table 显示类型不是 table-cation，table 和 inline-table 的元素
    Inherited:      no
    Percentages:    参考包含块的宽度
    Media:          visual
    Computed value: 查看各自的属性

‘margin’属性是在样式表同样位置设置‘margin-top’，‘margin-right’，‘margin-bottom’，和‘margin-left’的缩写属性。



如果只有一个组件值，它应用于所有边。如果有两个值，top 和 bottom margin 被设置为第一个值，right 和 left margin 设置为第二个值。如果有三个值，top 设置为第一个值，left 和 right 设置为第二个，bottom 设置为第三个。如果有 4 个值，他们应用于 top，right，bottom，和 left，各自。
```
body { margin: 2em }         /* all margins set to 2em */
body { margin: 1em 2em }     /* top & bottom = 1em, right & left = 2em */
body { margin: 1em 2em 3em } /* top=1em, right=2em, bottom=3em, left=2em */
```
前面例子最后的规则和下面的例子相同：
```
body {
  margin-top: 1em;
  margin-right: 2em;
  margin-bottom: 3em;
  margin-left: 2em;        /* copied from opposite side (right) */
}
```

### 8.3.1 折叠边距

在 CSS 中，两个或者多个盒子相邻的边距（可能或者可能不是邻居）可以合并组成一个单独的边距。这种方式的边距组合成为折叠，边距结合的结果叫做边距折叠。

相邻的垂直边距折叠，除了
- 跟元素的盒子的边距不折叠。
- 如果带有清除的元素的顶部和底部边距相邻，它的边距和后续相邻边距折叠，但是这导致边距和父块的底部边距不折叠。【疑惑？？】

水平边距不会折叠。

两个边距是相邻的，如果并且只有如果：
- 都属于流氏块级盒子，并参与相同的块级格式上下文。

- 没有行内盒子，没有清除，没有 padding 且没有 border 分离他们（注意某些 0 高行内盒子（查阅 9.4.2）为了这个目的被忽略）

- 都属于垂直相邻盒子边缘，比如，由下面其中一对组成：
    - 一个盒子的顶部边距和它的第一个流式子元素的顶部边距
    - 盒子的底部边距和它的下一个流式后续相邻元素
    - 最后一个流氏自元素的底部边距和它的父元素的底部边距，如果父元素有‘auto’计算高度
    - 一个没有建立一个新的块级格式上下文的盒子和有 0 计算的‘min-heigth’，0 或者 ‘auto’ 计算’height‘，和非流式子元素的顶部和底部边距。

一个折叠的边距认为和其他边距相邻如果它的组件边距和这个边距相邻。

注意：相邻边距可以被不是兄弟或者祖先的元素生成。

注意：前面的规则暗示：

- 浮动盒子和其他盒子的边距不折叠（甚至在一个浮动和它的流式自元素之间都不行）。

- 建立一个新的块级格式上下文（比如浮动和‘overflow’不是‘visible’的元素）的元素和他们的流式子元素不折叠。

- 绝对定位盒子的边距不折叠（设置和他们的流式子元素也不行）。

- 行内块级盒子的边距不折叠（甚至和他们的流式子元素也不行）。

- 一个流式块级元素的底部边距总是和下一个流式块级邻居的顶部边距折叠，除非这个邻居有清除。

- 流式块级元素的顶部边距和它的第一个流式块级子元素的顶部边距折叠，如果元素没有顶部边框，没有顶部填充，并且子元素没有清除。

- ‘height’为‘auto’，‘min-height’为 0 的流式块盒子的底部边距和它的 最后一个流式块级子元素的底部边距折叠，如果盒子没有底部填充并且没有底部边框并且子元素的底部边距没有和有清除的顶部边距折叠。

- 一个盒子的自己的边距折叠，如果‘min-height’属性是 0，并且它没有顶部或者底部边框，或者底部或者顶部填充，并且它的‘height’是 0 或者‘auto’，并且它没有包含一个行内盒子，并且所有它的流式子元素（如果有）边距都折叠。

当两个或者更多边距折叠，最后的边距宽度是折叠的边距的宽度的最大值。在负值边距的场景中，负数相邻边距的最大绝对值从正数相邻边距的最大绝对值扣除。如果没有正数边距，相邻边距的最大绝对值从 0 扣除。

如果一个盒子的顶部和底部边距相邻，则它可能通过它折叠。在这个场景中，元素的位置依赖于它和其他边距被折叠的元素的关系。

- 如果元素的边距和它的父元素顶部边距折叠，盒子顶部边框边缘和父元素的定义一致。

- 否则，元素的父元素不参与边距折叠，或者只有父元素底部边距参与，元素的顶部边框边缘的位置和如果元素有一个非 0 底部边框一致。

注意：被折叠的元素的定位不影响其他边距被折叠的元素的定位；顶部边框边缘定位只用于这些元素的子孙元素布局。

### 8.4 填充属性：‘padding-top’，‘padding-right’，‘padding-bottom’，‘padding-left’，和‘padding’


填充属性指定盒子填充区域的宽度。‘padding’缩写属性设置所有 4 边的填充，其他填充属性只设置他们各自的边。

- <length>
    - 指定一个固定的宽度

- <percentage>
    - 百分比根据生成盒子的包含块的宽度来计算，就算是‘padding-top’和‘padding-bootom’，如果包含看的宽度依赖于这个元素，则 CSS 2.1 中的布局没有定义。

不像 margin 属性，padding 值不能是负数。不像 margin 属性，填充属性的百分比值参考生成的很子的包含块的宽度。

’padding-top’，‘padding-right’，‘padding-bottom’，‘paddng-left’
    Value：         <padding-width> | inherit
    Initial:        0
    Applies to:     所有元素除了 table-row-group，table-header-group，table-footer-group，table-row，table-column-group 和 table-column
    Inherited:      no
    Percentages:    参考包含块的宽度
    Media:          visual
    Computed value: 指定的百分比或者绝对长度

这些属性设置一个盒子的顶部，右边，底部和左边填充
```
blockquote { padding-top: 0.3em }
```


‘padding’
    Value：         <padding-width>{1,4} | inherit
    Initial:        查看各自的属性
    Applies to:     所有元素除了 table-row-group，table-header-group，table-footer-group，table-row，table-column-group 和 table-column
    Inherited:      no
    Percentages:    参考包含块的宽度
    Media:          visual
    Computed value: 查看各自的属性

‘padding’属性是在样式表相同位置设置‘padding-top’，‘padding-right’，‘padding-bottom’和‘pading-left’的缩写属性。
如果只有一个组件值，它应用于所有边。如果有两个值，顶部和底部填充设置为第一个值，右边和左边设置为第二个值。如果有三个值，顶部设置为第一个值，左边和右边设置为第二个，底部设置为第三个，如果有四个值，它应用于顶部，右边，底部和左边，各自。

表面颜色或者填充区域的图片通过‘background’属性指定：
```
h1 { 
  background: white; 
  padding: 1em 2em;
} 
```
前面的例子指定一个‘1em’垂直填充（‘padding-top’和‘padding-bottom’）和一个‘2em’水平填充（）。‘em’单位是‘相对’于元素的字体大小：‘1em’和使用的字体的大小相同。

### 8.5 边框属性


边框属性指定一个盒子的边框区域的宽度，颜色和样式。这些属性应用于所有元素。

注意：特别是 HTML，用户代理可能为某些用户接口元素渲染边框，不同于“普通”元素。

### 8.5.1 边框宽度：‘border-top-width’，‘border-right-width’，‘border-bottom-width’，‘border-left-width’和‘border-width’

边框宽度属性指定边框区域的宽度。这个章节定义的属性参考<border-width>值类型，可能使用下面一个值：

- thin
  一个细边框
- medium
  一个中等边框
- thick
  一个厚边框
- <length>
  边框粗细有一个精确的值。精确的边框宽度不能是负数。

前三个值的解释依赖于用户代理。下面的关系必须被遵循，然而：

‘thin’ <= 'medium' <= 'thick'

甚至，这些宽度必须贯穿文档，始终如一。

’border-top-width’，‘border-right-width’，‘border-bottom-width’，‘border-left-width’
    Value：         <border-width> | inherit
    Initial:        medium
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 绝对长度；‘0’ 如果边框样式是‘none’或者‘hidden’
  
这些属性设置盒子顶部，右变，底部和左边的边框的宽度。

‘border-width’
    Value：         <border-width>{1,4} | inherit
    Initial:        查看各自的属性
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 查看各自的属性

这个属性是在样式表相同位置设置‘border-top-width’，‘border-right-width’，‘border-bottom-width’和‘border-left-width’的缩写属性。

如果只有一个组件值，它应用于所有边。如果有两个值，顶部和底部填充设置为第一个值，右边和左边设置为第二个值。如果有三个值，顶部设置为第一个值，左边和右边设置为第二个，底部设置为第三个，如果有四个值，它应用于顶部，右边，底部和左边，各自。

在下面的例子中，评论只是了最后的顶部，右边，底部，左边边框的宽度。
```
h1 { border-width: thin }                   /* thin thin thin thin */
h1 { border-width: thin thick }             /* thin thick thin thick */
h1 { border-width: thin thick medium }      /* thin thick medium thick */
```

### 8.5.2 边框颜色：‘border-top-color’，‘border-right-color’，‘border-bottom-color’，‘border-left-color’和‘border-color’

’border-top-color’，‘border-right-color’，‘border-bottom-color’，‘border-left-color’
    Value：         <color> | transparent | inherit
    Initial:        'color'属性的值
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 当从‘color’属性取值的时候，‘color’的计算值；否则，正如指定。
  
‘border-color’
    Value：         <color> | transparent | inherit
    Initial:        查看各自的属性
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 查看各自的属性

‘border-color’属性设置四个边框的颜色。值又下面的意义：

<color>
  指定一个颜色值

transparent
  边框是透明的（尽管他可能有宽度）

‘border-color’属性可以有有一到四个组件值，并且值设置不同的边，就像‘’border-width。

如果一个元素的边框颜色没有通过边框属性指定，用户代理必须使用元素的‘color’属性的值作为边框颜色的计算值。

在这个例子，边框将会是实体黑线。
```
p { 
  color: black; 
  background: white; 
  border: solid;
}
```

### 8.5.3 边框样式：‘border-top-style‘，‘border-right-style‘，‘border-bottom-style‘，‘border-left-style‘，和‘border-style‘

边框样式属性指定盒子的边框的行内样式（实线，双实线，虚线，等。）。定义在这个章节的属性参考 <border-style> 值类型，可能此阿勇下面的值之一：

- none
  没有边框，计算的边框宽度是 0；
- hidden
  和‘none’一样，但是除了在表格元素的边框冲突解决
- dotted
  边框是一系列的点
- dashed
  边框是一系列的短线片段
- solid
  边框是一个单独的线片段
- double
  边框是两个实体线。两线和他们之间的空格的和和‘border-width’相同
- groove
  边框看起来向内凹陷
- ridge
  和‘groove’相反：边框看起来向外突出
- inset
  边框看起来像是嵌入
- outset
  和‘inset’相反：边框让盒子看起来向外突出

所有的边框都是绘制在盒子的背景之上的。为边框值‘groove’，‘ridge’，‘inset’和‘outset’绘制的颜色依赖于元素的边框颜色属性，但是 UA 可能选择他们自己的算法去计算实际使用的颜色。比如，如果‘border-color’有值‘silver’，则 UA 可以使用一个从白到深灰的渐变颜色表示倾斜的边界。

’border-top-style’，‘border-right-style’，‘border-bottom-style’，‘border-left-style’
    Value：         <border-style>  | inherit
    Initial:        none
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 正如指定


‘border-style’
    Value：         <border-style>{1,4} | inherit
    Initial:        查看各自的属性
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 查看各自的属性

‘border-style’属性设置边框的样式。可以由一个到四个组件值，并且值设置不同的边，就像前面的‘border-width’。
```
#xy34 { border-style: solid dotted }
```

在前面的例子中，水平边框将会是‘solid’并且垂直边框是‘dotted’。

因为边框样式的初始值是’none‘，边框将会不被看到，除非边框样式没有设置。

### 8.5.4 边框缩写属性：


’border-top’，‘border-right’，‘border-bottom’，‘border-left’
    Value：         [ <border-width> || <border-style> || <border-top-color>] || inherit
    Initial:        查看各自的属性
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 查看各自的属性

这是一个设置一个盒子顶部，右边，底部，左边边框的宽度，样式和颜色的简写属性。
```
h1 { border-bottom: thick solid red }
```
前面的规则将会设置 H1 元素下的边框的宽度，样式，和颜色。缺少值将会设置成他们的初始值。因为后续的规格没有指定边框颜色，边框将会有‘color’属性执行的颜色。
```
H1 { border-bottom: thick solid }

```

‘border’
    Value：         [ <border-width> || <border-style> || <border-top-color>] || inherit
    Initial:        查看各自的属性
    Applies to:     所有元素
    Inherited:      no
    Percentages:    N/A
    Media:          visual
    Computed value: 查看各自的属性
  
`border`属性设置盒子四个边框相同宽度，样式，和颜色的简写属性。不同于‘margin’和‘padding’的简写属性，‘border’属性不能为四个边框设置不同的值。为了做到它，一个或者多个其他边框属性必须使用。

比如，下面的第一个规则和后面显示的设置四个规则相同：
```
p { border: solid red }
p {
  border-top: solid red;
  border-right: solid red;
  border-bottom: solid red;
  border-left: solid red
}
```

因为，在某种程度上，属性有覆盖的功能，规则中的顺序是很重要的。

考虑下面的例子：
```
blockquote {
  border: solid red;
  border-left: double;
  color: black;
}
```

在前面的例子中，左边框的颜色是黑色，其他边框是红色。这是因为‘border-left’设置宽度，样式，和颜色。因为颜色值没有在‘border-left’属性中给定，他将从‘color’属性中提取。事实上，‘color’属性设置在‘border-left’属性是没有关系的。


### 8.6 双向上下文内的行内元素的盒模型


对于每一个行内盒子，UA 必须采取为每一个元素生成的行内盒子并渲染边距，边框和填充，在可视化顺序（不是逻辑顺序）。


当元素的‘direction’属性是‘lrt’，第一个行内盒子最左边的生成的盒子有左边距，左边框和左填充，最右边的生成的最后一行盒子有右填充，右边框和右边距。

当元素的‘direction’属性是‘rtl’，第一行盒子最右边生成的盒子有右填充，右边框和右边距，最后一样最左边生成的盒子有左边距，左边框和左填充。