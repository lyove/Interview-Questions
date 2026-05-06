前端-css

1. 盒模型（Box Model）是什么？
答： CSS 中每个元素都由以下部分组成：
* content：内容区域
* padding：内边距
* border：边框
* margin：外边距
标准盒模型：width 只包含 content
IE盒模型：width 包含 content + padding + border
**设置方式：**
```css
box-sizing: content-box;  /* 标准盒模型 */
box-sizing: border-box;   /* IE盒模型 */
```

2. 如何水平居中一个块级元素？
答： 设置 margin: 0 auto 并指定 width。
```css
.center {
  width: 300px;
  margin: 0 auto;
}
```

2. 如何让一个行内元素水平居中？
答： 设置 `text-align: center` 给父元素。
```html
<div style="text-align: center;">
  <span>我是行内元素</span>
</div>
```

4. 清除浮动的方法有哪些？
答：
* 使用 clearfix（推荐）
* 设置父元素 overflow: hidden
* 添加空元素 clear: both
clearfix 示例：
```css
.clearfix::after {
  content: "";
  display: block;
  clear: both;
}
```

5. display: none 与 visibility: hidden 区别？
答：
* display: none：元素从文档流中移除，不占空间
* visibility: hidden：元素不可见，但仍占据空间

6. 绝对定位的元素是相对于谁定位的？
答： 相对于最近的“已定位”祖先元素（即设置了 position: relative/absolute/fixed 的祖先）。如果没有，则相对 body。
```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  top: 0;
  left: 0;
}
```

6. 如何让两个 div 左右排列？
答：
* 使用 float：老方法
* 使用 flex：推荐
Flex 示例：
.container {
  display: flex;
  justify-content: space-between;
}

8. 什么是 BFC？它的作用？
答： BFC（块级格式化上下文）是一个独立的布局环境。创建 BFC 可防止元素被浮动遮挡、清除 margin 合并等。
触发 BFC 方法：
overflow: hidden;
display: flow-root;

9. em 与 rem 的区别？
答：
* em：相对于父元素的字体大小
* rem：相对于根元素 html 的字体大小

10. 如何设置一个 div 的宽高比为 16:9？
答： 使用 padding-top 百分比的技巧：
.aspect-ratio {
  position: relative;
  width: 100%;
  padding-top: 56.25%; /* 9 / 16 = 0.5625 */
}

11. 如何实现文本溢出显示省略号？
答：
overflow: hidden;
text-overflow: ellipsis;
white-space: nowrap;

12. z-index 的使用注意点？
答：
* 只对 position 不为 static 的元素有效
* 数值越大越靠上

13. 如何禁用用户选择文字？
答：
user-select: none;

14. Flex 容器的主轴、交叉轴是怎么定义的？
答：
* flex-direction: row：主轴为水平方向，交叉轴为垂直
* flex-direction: column：主轴为垂直，交叉轴为水平

15. 如何使用 CSS 画一个三角形？
答： 使用透明边框技巧：
.triangle {
  width: 0;
  height: 0;
  border-left: 50px solid transparent;
  border-right: 50px solid transparent;
  border-bottom: 50px solid red;
}

16. 伪类与伪元素的区别？
答：
* 伪类（:hover、:nth-child()）：表示某种状态
* 伪元素（::before、::after）：表示虚拟的 DOM 节点

17. 如何给元素添加多个背景图？
答：
background-image: url(img1.jpg), url(img2.jpg);

18. 动画和过渡的区别？
答：
* transition：用于状态变化（如 hover）
* animation：可实现更复杂的关键帧动画，具有完整时间轴

19. 如何让一个元素垂直水平居中？
答（Flex 方式）：
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}

20. 如何使用媒体查询实现响应式布局？
答：
@media screen and (max-width: 600px) {
  body {
    background: lightblue;
  }
}

21.请解释Flex布局的基本概念和主要属性
容器属性：
display: flex;：开启Flex布局。
flex-direction：决定主轴方向（默认为row，可选值有row-reverse、column、column-reverse）。
flex-wrap：控制是否换行（默认nowrap，可选wrap、wrap-reverse）。
justify-content：沿主轴对齐方式（如flex-start、center、space-between等）。
align-items：沿交叉轴对齐方式（如stretch、center、baseline等）。
align-content：多根轴线的对齐方式（仅在flex-wrap: wrap时生效）。

项目属性：
order：定义项目的排列顺序。
flex-grow：定义项目的放大比例。
flex-shrink：定义项目的缩小比例。
flex-basis：定义在分配多余空间之前，项目占据的主轴空间。
flex：简写属性，相当于flex-grow, flex-shrink 和 flex-basis 的合并。
align-self：允许单个项目覆盖容器的align-items属性。

22.  CSS选择器优先级
权重排序‌：!important > 行内样式 > ID 选择器 > 类选择器 > 标签选择器 > 通配符。

23. 请解释CSS contain 属性的作用和可用值。


24. 请解释CSS clip-path 属性的作用及其实现形状裁剪的方法。 


25. 请解释什么是CSS Grid布局，并描述其关键特性与优势。

26、伪类与伪元素区别‌
伪类用单冒号（:hover），伪元素用双冒号（::before），
伪类不改变 DOM 内容，伪元素创建虚拟容器。


27、进阶知识点
* ‌响应式布局‌
    * ‌rem 与 em 区别‌：rem 相对根元素 html 字体大小，em 相对父元素字体大小。
    * ‌vw/vh 单位‌：vw 为视窗宽度 1/100，vh 为视窗高度 1/100，可克服 rem 的"阶梯性"弊端。
    * ‌媒体查询‌：CSS3 新属性，根据不同屏幕宽度设置样式，实现响应式设计。‌‌‌‌1
* ‌Flex 与 Grid 布局‌
    * Flex 常用属性‌：flex-direction（主轴方向）、justify-content（主轴对齐）、align-items（交叉轴对齐）、flex-wrap（换行）。
    * ‌Grid 布局‌：CSS3 新增网格布局模块，适合复杂二维布局。‌‌‌‌1
* ‌性能与兼容性‌
    * display:none 与 visibility:hidden 区别‌：前者脱离文档流不占空间，后者保留空间但隐藏。
    * ‌CSS 引入方式‌：link（同步加载、可 JS 控制）与@import（异步加载、仅 CSS）。
    * ‌浮动清除方法‌：overflow:hidden、clear:both、伪元素 clearfix 等。‌‌
