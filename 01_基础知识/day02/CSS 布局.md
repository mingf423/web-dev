# Day02 CSS 布局 —— 看懂页面是怎么排列的

> 后端转前端的第一道坎：不管你逻辑多强，CSS 布局不过关，页面就是歪的。
> 今天学完，你能看懂项目中所有 `display`、`flex`、`position` 的含义。

---

## 学习目标

- [ ] 理解盒模型：知道 width/padding/border/margin 谁包着谁
- [ ] 分清块级 vs 行内：知道 `<div>` 和 `<span>` 什么时候该用
- [ ] 掌握 Flexbox：`display: flex` 三件套搞定 90% 的排列需求
- [ ] 理解定位：relative/absolute/fixed 三个值的区别
- [ ] 实践：手写一张"个人名片"卡片

---

## 1. 盒模型 —— CSS 世界的"俄罗斯套娃"

### 1.1 什么是盒模型

> 在 CSS 眼里，**每个元素都是一个矩形盒子**。理解盒子从外到内有四层，就理解了为什么元素之间有间距、为什么元素会比预期的宽。

```
┌─────────────────────────────────────────────┐
│  margin（外边距）                              │  ← 盒子与其他盒子的距离
│  ┌───────────────────────────────────────┐  │
│  │  border（边框）                         │  │  ← 盒子的"墙壁"
│  │  ┌─────────────────────────────────┐  │  │
│  │  │  padding（内边距）                 │  │  │  ← 墙壁到内容的距离
│  │  │  ┌───────────────────────────┐  │  │  │
│  │  │  │  content（内容区）          │  │  │  │  ← width/height 控制的区域
│  │  │  │  文字、图片、子元素         │  │  │  │
│  │  │  └───────────────────────────┘  │  │  │
│  │  └─────────────────────────────────┘  │  │
│  └───────────────────────────────────────┘  │
└─────────────────────────────────────────────┘
```

### 1.2 Java 类比

```java
// 想象你在写 Swing
JPanel panel = new JPanel();
panel.setSize(200, 100);          // ← width/height（内容区）
panel.setBorder(BorderFactory.createLineBorder(Color.BLACK));  // ← border
// panel 里的内容离边框 10px           // ← padding
// panel 离其他组件 20px               // ← margin
```

### 1.3 关键属性速记

```css
.box {
    /* 内容区大小 */
    width: 200px;
    height: 100px;

    /* 内边距：内容到边框 */
    padding: 16px;           /* 四边统一 */
    padding: 10px 20px;      /* 上下 左右 */
    padding: 10px 20px 5px 15px;  /* 上 右 下 左（顺时针） */

    /* 边框 */
    border: 1px solid #ccc;  /* 粗细 样式 颜色 */

    /* 外边距：盒子到其他盒子的距离 */
    margin: 20px;            /* 四边统一 */
    margin: 100px auto;      /* 上下100px，左右自动=水平居中！ */
}
```

### 1.4 ⚠️ 最容易踩的坑：box-sizing

```css
/* 默认情况（content-box）：width 只算内容区 */
.box-default {
    width: 200px;
    padding: 20px;
    border: 5px solid #333;
}
/* 实际占用宽度 = 200 + 20×2 + 5×2 = 250px！不是你预期的 200 */

/* 推荐（border-box）：width 包含了 padding + border */
.box-safe {
    box-sizing: border-box;   /* ← 项目中一般全局设置这个 */
    width: 200px;
    padding: 20px;
    border: 5px solid #333;
}
/* 实际占用宽度 = 200px，内容区自动缩小为 150px */
```

> 📂 **项目中的代码**：你们的 `common.css` 里大概率有一行 `* { box-sizing: border-box; }`，所以不用担心这个坑。

### 🎯 这一节回答了

> 为什么加了 padding 之后元素变宽了？为什么 `margin: 0 auto` 能居中？

### ✅ 检查点

- [ ] 能画出盒模型的四层结构
- [ ] 能解释 `margin: 0 auto` 为什么能水平居中
- [ ] 知道 `box-sizing: border-box` 解决了什么问题

---

## 2. 块级 vs 行内 —— `<div>` 和 `<span>` 的区别

### 2.1 三种 display 类型

| 类型 | 行为 | 典型标签 | 类比 |
|------|------|----------|------|
| `block`（块级） | 独占一行，可设宽高 | `<div>` `<p>` `<h1>` `<section>` | `JPanel` |
| `inline`（行内） | 不换行，**不可**设宽高 | `<span>` `<a>` `<em>` | `JLabel` 里的文字 |
| `inline-block` | 不换行，**可**设宽高 | `<button>` `<input>` | 最小的 `JButton` |

```
块级元素（block）：
┌──────────────────────────────────────┐
│  div 1                               │
├──────────────────────────────────────┤
│  div 2                               │
└──────────────────────────────────────┘
↑ 每个独占一整行，上下堆叠

行内元素（inline）：
┌──┬──┬──┬──┐
│a │b │c │d │  ← 在同一行内排列，不换行
└──┴──┴──┴──┘

行内块元素（inline-block）：
┌────┐ ┌────┐ ┌────┐
│btn1│ │btn2│ │btn3│  ← 同一行，但每个有固定宽高
└────┘ └────┘ └────┘
```

### 2.2 代码演示

```css
.block-box   { display: block; width: 200px; height: 50px; }   /* ✅ 宽高生效 */
.inline-box  { display: inline; width: 200px; height: 50px; }  /* ❌ 宽高无效！ */
.ib-box      { display: inline-block; width: 200px; height: 50px; } /* ✅ 宽高生效，不换行 */
```

### 🎯 这一节回答了

> 为什么 `<div>` 总是一行一个，而 `<span>` 可以跟在别人后面？为什么 `<button>` 可以设宽高还能跟别的元素排在同一行？

### ✅ 检查点

- [ ] 知道 `<div>` 是 block，`<span>` 是 inline
- [ ] 知道 `inline-block` = 不换行 + 可以设宽高
- [ ] 能解释为什么给 `<span>` 设 `width` 没用

---

## 3. Flexbox —— 现代布局的核心

> **这是今天最重要的内容。** 项目中 90% 的排列需求靠 Flexbox 解决。学完它你就能看懂项目里几乎所有的布局代码。

### 3.1 一句话理解

```
display: flex 把容器变成"弹性容器"，容器里的子元素会自动排成一行，
你可以轻松控制：水平怎么对齐？垂直怎么对齐？要不要换行？
```

### 3.2 核心三件套（记住这三个就够了）

```css
.container {
    display: flex;                    /* 启动弹性布局 */
    justify-content: center;          /* 水平方向怎么对齐 */
    align-items: center;              /* 垂直方向怎么对齐 */
    gap: 16px;                        /* 子元素之间的间距 */
}
```

### 3.3 justify-content（水平对齐）

```
justify-content: flex-start（默认，靠左）
[子1] [子2] [子3] ___________________

justify-content: center（居中）
______ [子1] [子2] [子3] ______

justify-content: flex-end（靠右）
___________________ [子1] [子2] [子3]

justify-content: space-between（两端对齐，中间平分）
[子1] _____________ [子2] _____________ [子3]

justify-content: space-around（每个元素周围有相等间距）
___ [子1] _____ [子2] _____ [子3] ___
```

### 3.4 align-items（垂直对齐）

```
align-items: center        ← 垂直居中（用最多！）
align-items: flex-start    ← 顶部对齐
align-items: flex-end      ← 底部对齐
align-items: stretch       ← 拉伸填满高度（默认）
```

### 3.5 一个公式搞定居中

```css
.center-box {
    display: flex;
    justify-content: center;   /* 水平居中 */
    align-items: center;       /* 垂直居中 */
    /* 这个组合 = 你找了半天的 "万能居中" */
}
```

### 3.6 flex-direction（排列方向）

```css
.container {
    display: flex;
    flex-direction: row;            /* 默认：左→右 */
    flex-direction: column;         /* 上→下 */
    flex-direction: row-reverse;    /* 右→左 */
}
```

### 3.7 flex 属性（子元素怎么分空间）

```css
.child-a { flex: 1; }   /* 占 1 份 */
.child-b { flex: 2; }   /* 占 2 份（是 a 的两倍宽） */
.child-c { flex: 1; }   /* 占 1 份 */
/* 结果：a 占 25%，b 占 50%，c 占 25% —— 类似 GridBagLayout 的 weight */
```

### 🎯 这一节回答了

> 怎么让一个东西水平居中？垂直居中？又水平又垂直居中？左边放 logo、右边放菜单怎么做？三个卡片平分一行怎么做？

### ✅ 检查点

- [ ] `display: flex` + `justify-content: center` + `align-items: center` = 居中
- [ ] 能区分 `justify-content`（主轴/水平）和 `align-items`（交叉轴/垂直）
- [ ] 知道 `space-between` 什么时候用（两端对齐，常见于 header 布局）
- [ ] 知道 `flex: 1` 意思是"平分剩余空间"

---

## 4. 定位 —— 让元素脱离正常流

> Flexbox 是在"正常排列规则"里排，定位是让元素"跳出规则"，放到你想放的任意位置。

### 4.1 五种 position 值

| 值 | 行为 | 使用场景 | 优先级 |
|----|------|----------|--------|
| `static` | 默认值，正常排列 | 99% 的元素 | — |
| `relative` | 相对于**自己原位置**偏移 | 微调位置、作为 absolute 的锚点 | ⭐⭐⭐ |
| `absolute` | 相对于**最近的定位祖先**定位 | 下拉菜单、弹窗、角标 | ⭐⭐⭐ |
| `fixed` | 相对于**浏览器窗口**定位 | 固定顶部导航、回到顶部按钮 | ⭐⭐ |
| `sticky` | 滚动到阈值后固定 | 表头固定、侧边栏跟随 | ⭐ |

### 4.2 relative vs absolute（必须分清）

```html
<!-- 组合用法（出现频率最高） -->
<div style="position: relative;">          <!-- 锚点：relative，自己不乱跑 -->
    <span>正常内容</span>
    <span style="position: absolute; top: 0; right: 0;">   <!-- 角标：absolute -->
        🔴 红点
    </span>
</div>
```

**规则**：`absolute` 元素会去找**最近一个** `position` 不是 `static` 的祖先元素来定位。如果找不到，就以 `<body>` 为参考。

```css
/* 常见模式 */
.parent {
    position: relative;   /* 父元素设置 relative，自己不偏移 */
}
.child {
    position: absolute;   /* 子元素 absolute，相对父元素定位 */
    top: 0;
    right: 0;             /* 放在父元素右上角 */
}
```

### 4.3 fixed 的典型场景

```css
.header {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;          /* 固定在屏幕顶部，滚动时不动 */
    z-index: 100;         /* 保证在最上层 */
}
```

### 4.4 z-index（谁在上面）

```css
/* z-index 越大越靠前（靠近用户），只在定位元素上生效 */
.front  { position: relative; z-index: 10; }
.behind { position: relative; z-index: 1; }
```

### 🎯 这一节回答了

> 角标（红点、未读数）怎么放到元素右上角？弹窗怎么居中浮在页面上？顶部导航怎么固定不随滚动消失？

### ✅ 检查点

- [ ] 知道 `relative` = 相对自己偏移
- [ ] 知道 `absolute` = 相对最近的定位祖先
- [ ] 知道 `relative` 父 + `absolute` 子 是最常见组合
- [ ] 知道 `fixed` = 相对视口固定

---

## 5. 📂 在你的项目中找这些模式

打开项目中任意 `.vue` 文件，你会看到：

```html
<!-- Flexbox 水平布局 -->
<div style="display: flex; justify-content: space-between; align-items: center;">
    <span>标题</span>
    <el-button type="primary">操作</el-button>
</div>

<!-- Flexbox 垂直居中 -->
<el-form-item label="姓名" style="display: flex; align-items: center;">

<!-- 绝对定位角标 -->
<div style="position: relative;">
    <el-badge :value="12">    <!-- 角标内部就用了 absolute！ -->
        <el-button>消息</el-button>
    </el-badge>
</div>

<!-- padding 间距 -->
<div style="padding: 16px; margin-bottom: 12px; background: #fff;">

<!-- margin: auto 居中 -->
<div style="max-width: 1200px; margin: 0 auto;">
```

---

## 6. 实践：手写一张"个人名片"

> 打开 [business-card.html](./business-card.html)，用刚学的 CSS 布局做一张名片。

**要求**：
- 一张卡片，白色背景、圆角、阴影
- 左侧：头像占位符（圆形）
- 右侧：姓名、职位、技能标签列表
- 技能标签用 Flexbox 排列，可换行
- 卡片在页面中**垂直+水平居中**

**提示**：
```
整体居中 → body 用 flex + justify-content + align-items
卡片布局 → 内部用 flex 分左右两栏
标签排列 → ul 用 flex + flex-wrap: wrap
```

---

## 7. 补充知识速查表

### 常用 CSS 属性一览

| 分类 | 属性 | 常用值 | 作用 |
|------|------|--------|------|
| 尺寸 | `width` `height` | `200px` `100%` `auto` | 宽高 |
|  | `max-width` | `1200px` | 最大宽度 |
|  | `min-height` | `100vh` | 最小高度（vh=视口高度百分比） |
| 间距 | `margin` | `16px` `auto` | 外边距 |
|  | `padding` | `16px` | 内边距 |
| 边框 | `border` | `1px solid #ccc` | 边框 |
|  | `border-radius` | `4px` `50%` | 圆角（50%=圆形） |
| 背景 | `background` | `#fff` `transparent` | 背景色 |
|  | `box-shadow` | `0 2px 8px rgba(0,0,0,0.1)` | 阴影 |
| 文字 | `color` | `#303133` | 文字颜色 |
|  | `font-size` | `14px` | 字号 |
|  | `font-weight` | `700` `bold` `normal` | 字重 |
|  | `text-align` | `center` `left` `right` | 水平对齐 |
|  | `line-height` | `1.5` `40px` | 行高 |
| 布局 | `display` | `flex` `block` `inline-block` `none` | 显示类型 |
|  | `flex-wrap` | `wrap` | 允许换行 |
|  | `gap` | `16px` `10px 20px` | 子元素间距 |
|  | `position` | `relative` `absolute` `fixed` | 定位方式 |

### 颜色表示法

```css
color: #303133;                     /* 十六进制（最常用） */
color: rgb(48, 49, 51);            /* RGB */
color: rgba(0, 0, 0, 0.5);        /* RGBA，最后一位是透明度 0~1 */
background: transparent;            /* 透明 */
```

---

## ✅ Day02 检查清单

- [ ] 盒模型：能画出四层结构，解释 `box-sizing: border-box`
- [ ] 块级 vs 行内：知道 `<div>`（block）和 `<span>`（inline）的区别
- [ ] Flexbox：能手写 `display: flex; justify-content: center; align-items: center;`
- [ ] Flexbox：知道 `space-between` 和 `flex: 1` 的用法
- [ ] 定位：知道 `relative` 父 + `absolute` 子 的组合
- [ ] 定位：知道 `fixed` 什么时候用
- [ ] 实践：完成个人名片卡片

---

## 下一步

> **Day03**：JavaScript 深入 —— 数组操作（`.map`/`.filter`/`.find`）、对象、解构赋值、展开运算符。这些是读懂 Vue 组件 `<script>` 区逻辑的前提。

---

> 📅 生成日期：2026-08-12
