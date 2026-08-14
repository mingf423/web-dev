# Day05 Vue 模板语法 —— 看懂 `<template>` 里的东西

> 前四天都是纯 HTML/CSS/JS。今天进入 **Vue**。
>
> 你打开公司项目任意一个 `.vue` 文件，最先看到的就是 `<template>` 区。这一片长得像 HTML，
> 但夹着大量 `{{ }}`、`v-if`、`v-for`、`@click`、`:data` —— 看不懂这些，就看不懂页面。
>
> 好消息是：这些语法和你写后端时用的 **Thymeleaf / JSP / JSTL** 是一个思路，只是更强大。
> 你们项目用的是 **Vue 2.6**，所以本日演示也用 Vue 2（CDN 引入），模板语法在 Vue 2/3 里基本一致。

---

## 学习目标

- [ ] 理解 Vue 的核心理念：**声明式渲染 + 响应式**（数据变了页面自动更新）
- [ ] 掌握 `{{ }}` 插值：把变量显示到页面
- [ ] 掌握 `v-if` / `v-show`：条件渲染
- [ ] 掌握 `v-for`：循环渲染列表
- [ ] 掌握 `v-model`：双向绑定（表单输入）
- [ ] 掌握 `:prop`（v-bind）：动态绑定属性
- [ ] 掌握 `@event`（v-on）：绑定事件
- [ ] 了解 `v-html`：渲染 HTML 字符串
- [ ] 能看懂项目里的 `<el-table :data="..." @selection-change="...">` 这类写法

---

## 1. 为什么需要 Vue —— 从"手动操作 DOM"到"声明式渲染"

> 这是今天最重要的一个思想转变，理解了它，Vue 剩下的都是语法细节。

### 1.1 回想 Day01 是怎么改页面的

```js
// Day01 的手动 DOM 操作（命令式：一步步告诉浏览器"怎么做"）
document.getElementById('count').textContent = count;   // 手动找到元素，手动改文字
```

> 数据一变，你得**手动找到元素、手动改内容**。一个页面几十个元素，每次数据变化都要
> 挨个同步，改起来累、还容易漏。

### 1.2 Vue 的做法（声明式：只声明"页面长什么样"）

```html
<div>{{ count }}</div>   <!-- 声明：这里显示 count 的值，别的我不管 -->

<button @click="count++">+1</button>
<!-- 点一下，count 变了，上面的 {{ count }} 自动跟着变 -->
```

> 你只管写"页面上要显示 count"，**Vue 自动负责**：count 一变，就把对应的 DOM 更新掉。
> 全程没有一行 `getElementById`、`innerHTML`。

### 1.3 Java 类比：会"自己刷新"的模板引擎

```
你写后端时熟悉的模板引擎：
- JSP：     <h1>${user.name}</h1>        → 服务端渲染一次，数据变了要重新请求才刷新
- Thymeleaf：<h1 th:text="${user.name}">  → 同上，服务端渲染

Vue 是什么：
- <h1>{{ user.name }}</h1>               → 浏览器端渲染，数据一变页面【自动】更新，无需刷新
```

> **一句话**：Vue = 一个"数据变了会自动更新页面"的模板引擎。你声明"要显示什么"，
> Vue 负责"怎么更新"。Java 里你手动 `print` 数据到页面，Vue 里你只管改数据。

### 🎯 这一节回答了

> 为什么公司项目里几乎看不到 `document.getElementById`？因为 Vue 帮你做了 DOM 更新这件事。

### ✅ 检查点

- [ ] 知道"命令式"（手动改 DOM）和"声明式"（声明显示什么）的区别
- [ ] 知道 Vue 的"响应式" = 数据变了页面自动更新

---

## 2. `{{ }}` 插值 —— 把变量显示到页面

> ≈ Thymeleaf 的 `${...}` / JSP 的 `${...}`。把 `data` 里的变量值填到 HTML 里。

```html
<span>{{ userName }}</span>          <!-- 显示变量 -->
<span>{{ user.name }}</span>         <!-- 访问对象字段 -->
<span>{{ 1 + 1 }}</span>             <!-- 可以写表达式，显示 2 -->
<span>{{ status === 1 ? '在职' : '离职' }}</span>  <!-- 三元表达式 -->
```

```js
// 对应的 data（变量都定义在 data 里，Day06 会详细讲）
data() {
    return {
        userName: '张三',
        user: { name: '李四' },
        status: 1,
    };
}
```

> ⚠️ **注意**：`{{ }}` 里只能写**表达式**，不能写语句。`{{ if (...) }}` 是错的，
> 条件判断要用下一节的 `v-if`。

### 🎯 这一节回答了

> 页面上那个 `{{ dataSource.length }}`、`{{ queryParam.name }}` 是什么意思？

### ✅ 检查点

- [ ] `{{ }}` 里写的是变量或表达式，来自 `data`
- [ ] 能看懂 `{{ status === 1 ? '在职' : '离职' }}` 这种三元表达式

---

## 3. `v-if` / `v-show` —— 条件渲染

> ≈ JSTL 的 `<c:if>` / Thymeleaf 的 `th:if`。但两者有本质区别。

### 3.1 基本用法

```html
<el-button v-if="hasPermission">删除</el-button>   <!-- hasPermission 为 true 才显示 -->
<el-button v-else>无权限</el-button>                <!-- 配合 v-else 使用 -->
```

### 3.2 ⚠️ v-if 和 v-show 的区别（面试常问）

```html
<!-- v-if：条件不满足，元素【根本不渲染】，DOM 里没有这个节点 -->
<div v-if="false">这行不会出现在 DOM 里</div>

<!-- v-show：条件不满足，元素【还在】，只是加了 display:none 隐藏 -->
<div v-show="false">这行在 DOM 里，只是看不见</div>
```

| | v-if | v-show |
|---|---|---|
| 不满足时 | 不创建元素（DOM 里没有） | 创建了，只是 `display:none` |
| 切换开销 | 大（销毁/重建） | 小（只切换样式） |
| 首次渲染开销 | 小（不满足就不渲染） | 大（总是渲染） |
| 适用场景 | 条件**很少变**（如权限判断） | 条件**频繁切换**（如 tab 切换） |

> **Java 类比**：`v-if` ≈ 条件不满足就不 `new` 这个对象（根本不创建）；
> `v-show` ≈ 对象已经 `new` 出来了，只是 `setVisible(false)` 藏起来。

### 🎯 这一节回答了

> 为什么有些按钮在 DOM 里根本找不到？—— 因为 `v-if` 为 false 时它压根没渲染。

### ✅ 检查点

- [ ] `v-if` 不满足时元素不在 DOM 里，`v-show` 只是隐藏
- [ ] 知道权限控制常用 `v-if`，tab 切换常用 `v-show`

---

## 4. `v-for` —— 循环渲染列表

> ≈ JSTL 的 `<c:forEach>` / Java 的 `for (User u : list)`。项目里渲染表格行、下拉选项全靠它。

### 4.1 基本用法

```html
<!-- 遍历用户列表，每个 user 渲染一个 <li> -->
<li v-for="user in userList" :key="user.id">{{ user.name }}</li>

<!-- 也可以拿到下标（第二个参数） -->
<li v-for="(user, index) in userList" :key="user.id">{{ index }} - {{ user.name }}</li>
```

```js
data() {
    return {
        userList: [
            { id: 1, name: '张三' },
            { id: 2, name: '李四' },
            { id: 3, name: '王五' },
        ],
    };
}
```

### 4.2 `:key` 是什么？为什么必须加？

> `:key` 是给每个循环项一个**唯一标识**，让 Vue 能识别"哪个元素变了"，
> 从而高效地复用/更新 DOM，而不是整个列表重来。

**Java 类比**：就像 `HashMap` 的 key、数据库的主键。没有唯一 key，Vue 更新列表时
可能"认错人"（比如你删了第一行，Vue 误以为你改的是其他行）。

```html
<!-- ✅ 用 id 做 key（唯一且稳定） -->
<li v-for="user in userList" :key="user.id">

<!-- ❌ 用 index 做 key（列表增删时会导致错位） -->
<li v-for="(user, index) in userList" :key="index">
```

### 🎯 这一节回答了

> `<el-table-column v-for="col in columns" :key="col.prop">` 为什么要遍历、为什么要 `:key`？

### ✅ 检查点

- [ ] `v-for="item in list"` 遍历数组，`item` 是当前元素
- [ ] `:key` 要用唯一值（如 id），不能用 index
- [ ] 能看懂 `(item, index)` 第二个参数是下标

---

## 5. `v-model` —— 双向绑定（表单输入）

> Java 后端没有直接对应的概念，这是 Vue 最"爽"的特性之一：**输入框和数据自动同步**。

### 5.1 没有 v-model 时的痛苦（Java 后端你天天写的）

```java
// Java 后端：手动从请求里取参数，再手动 set 到对象上
String name = request.getParameter("name");   // 手动取
user.setName(name);                            // 手动赋值
```

### 5.2 v-model 让你一行搞定

```html
<!-- 输入框的内容，自动同步到 queryParam.name -->
<el-input v-model="queryParam.name" />

<!-- 等价于（v-model 是下面这行的语法糖）：
     :value="queryParam.name"  @input="queryParam.name = $event" -->
```

```js
data() {
    return {
        queryParam: { name: '' },
    };
}
```

> 用户在输入框敲 "张"，`queryParam.name` 立刻变成 "张"；你在代码里把
> `queryParam.name = '李'`，输入框也立刻显示 "李"。**双向**同步，不用手动 `getParameter`。

### 5.3 项目里的常见搭配

```html
<el-input v-model="queryParam.name" placeholder="请输入姓名" />
<el-select v-model="queryParam.status" placeholder="请选择状态">
    <el-option label="在职" :value="1" />
    <el-option label="离职" :value="0" />
</el-select>
<el-date-picker v-model="queryParam.startDate" type="date" />
```

> 查询表单里的每个输入项，几乎都是 `<xxx v-model="queryParam.字段">` 这种写法。

### 🎯 这一节回答了

> 为什么页面上输入框敲几个字，`this.queryParam.name` 就跟着变了？因为 v-model 双向绑定。

### ✅ 检查点

- [ ] `v-model` 双向绑定输入框和 data 变量
- [ ] 能看懂 `<el-input v-model="queryParam.name" />` 是"输入框 ↔ queryParam.name"同步

---

## 6. `:prop`（v-bind）—— 动态绑定属性

> `:prop` 是 `v-bind:prop` 的简写。作用：把 **JS 变量** 传给 HTML 属性（而不是字符串）。

### 6.1 关键区别：加不加冒号

```html
<!-- ❌ 不加冒号：data 的值是字符串 "dataSource" 这五个字 -->
<el-table data="dataSource">

<!-- ✅ 加冒号：data 的值是变量 dataSource（一个数组） -->
<el-table :data="dataSource">
```

```js
data() {
    return {
        dataSource: [ { name: '张三' }, { name: '李四' } ],   // 真数组
    };
}
```

> **Java 类比**：`:data="dataSource"` 就像方法调用 `setData(dataSource)`（传变量）；
> `data="dataSource"` 就像 `setData("dataSource")`（传字符串字面量）。一字之差，天壤之别。

### 6.2 项目里最常见的 `:` 用法

```html
<el-table :data="dataSource" :loading="loading">
    <el-table-column :prop="col.prop" :label="col.label" :width="col.width" />
</el-table>

<el-dialog :visible.sync="dialogVisible" title="编辑" />
```

### 6.3 动态绑定样式（也常用）

```html
<!-- class 可以是动态的：isActive 为 true 时加 active 类 -->
<div :class="{ active: isActive, error: hasError }">
```

### 🎯 这一节回答了

> `:data="dataSource"` 里的冒号是干嘛的？不加冒号会怎样？

### ✅ 检查点

- [ ] 知道 `:prop` 是 `v-bind:prop` 的简写，传的是变量不是字符串
- [ ] 能看懂 `:data="dataSource"`、`:loading="loading"`、`:visible.sync="..."`

---

## 7. `@event`（v-on）—— 绑定事件

> `@event` 是 `v-on:event` 的简写。作用：给元素绑定事件处理器。≈ `addEventListener`。

### 7.1 基本用法

```html
<el-button @click="handleSave">保存</el-button>       <!-- 点击时调用方法 -->
<el-input @change="handleChange" />                    <!-- 值改变时调用 -->
<el-select @change="handleSelectChange" />             <!-- 选择变化时调用 -->
<el-table @selection-change="handleSelect" />          <!-- 勾选变化时调用 -->
```

```js
methods: {                                            // 方法定义在 methods 里（Day06 讲）
    handleSave() {
        console.log('点了保存');
    },
    handleSelect(selection) {                          // 事件会传入参数
        console.log('勾选的行：', selection);
    },
}
```

### 7.2 常见事件对照

| 事件 | 触发时机 | 项目中的典型用法 |
|------|----------|------------------|
| `@click` | 点击 | 按钮保存/查询/删除 |
| `@change` | 值改变且失焦 | 下拉选择变化 |
| `@input` | 输入时 | 输入框实时响应 |
| `@selection-change` | 表格勾选变化 | 拿勾选的行 |

### 7.3 传参

```html
<!-- 不带参数：直接写方法名 -->
<el-button @click="loadData">查询</el-button>

<!-- 带参数：写箭头函数或方法名(参数) -->
<el-button @click="handleEdit(scope.row)">编辑</el-button>
```

### 🎯 这一节回答了

> `@click="handleSave"` 为什么点一下就会执行 handleSave？@ 是 v-on 的简写。

### ✅ 检查点

- [ ] `@event` 是 `v-on:event` 的简写，绑定事件
- [ ] 能看懂 `@click="handleEdit(scope.row)"` 是"点击时调用 handleEdit，传当前行数据"

---

## 8. `v-html` —— 渲染 HTML 字符串（了解即可）

> 前面 `{{ }}` 会把内容当**纯文本**显示，`<` 会被转义。如果要渲染真正的 HTML，用 `v-html`。

```html
<div v-html="htmlContent"></div>
```

```js
data() {
    return { htmlContent: '<span style="color:red">红色文字</span>' };
}
```

> ⚠️ **危险**：`v-html` 有 **XSS 注入风险**。如果内容是用户输入的，千万别用 `v-html`。
> 项目里极少用到，你只要"认识它"就行。

### ✅ 检查点

- [ ] 知道 `v-html` 渲染 HTML，但了解它有 XSS 风险，尽量少用

---

## 9. 📂 在你的项目中找这些模式

打开任意 `.vue` 文件的 `<template>` 区，你会看到（逐行拆解）：

```html
<el-table :data="dataSource" @selection-change="handleSelect">
<!--        └── 传变量        └── 勾选时调用方法     -->

    <el-table-column v-for="col in columns" :key="col.prop"
    <!--              └── 遍历 columns 数组      └── 唯一标识 -->
        :prop="col.prop" :label="col.label" />
        <!-- └── 动态传列字段  └── 动态传列标题 -->

    <el-table-column label="操作">
        <template slot-scope="scope">
            <el-link type="primary" @click="handleEdit(scope.row)">编辑</el-link>
            <!--                        └── 点击传当前行数据 -->
        </template>
    </el-table-column>
</el-table>

<el-pagination
    :current-page="ipagination.current"
    :page-sizes="[10, 20, 50]"
    :total="ipagination.total"
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange" />
```

**一句话总结整个 template 区**：`:` 传变量，`@` 绑事件，`v-` 是控制逻辑（if/for/model）。

---

## 10. 实践：跑起来看效果

> 三个演示文件，浏览器直接打开（需联网加载 Vue CDN）：

1. **[counter.html](./counter.html)** —— 计数器。理解"数据变了页面自动更新"，并对比 Day01 手动 DOM 操作的写法。
2. **[directives.html](./directives.html)** —— 七大指令逐个演示（插值、v-if/v-show、v-for、v-model、:bind、@event、v-html）。
3. **[todo.html](./todo.html)** —— 待办列表综合练习（v-model 输入 + v-for 列表 + v-if 空状态 + @click 删除 + 剩余数统计）。

**验证**：打开 F12 → Elements，在 counter 页点 +1，观察 `<div>` 里的数字自动变；
再试试把 `v-if` 条件改成 false，看元素从 DOM 里消失。

---

## 11. 速查表

### 模板指令一览

| 指令 | 作用 | Java 类比 | 简写 |
|------|------|-----------|------|
| `{{ }}` | 插值，显示变量 | `${...}`（JSP/Thymeleaf） | - |
| `v-if` | 条件渲染（不创建元素） | `<c:if>`（JSTL） | - |
| `v-show` | 条件显示（display:none） | `setVisible(false)` | - |
| `v-for` | 循环渲染 | `<c:forEach>` / `for` | - |
| `v-model` | 双向绑定 | 手动 getParameter + set | - |
| `:prop` | 动态属性（传变量） | 传方法参数 | `v-bind:prop` |
| `@event` | 绑定事件 | `addEventListener` | `v-on:event` |
| `v-html` | 渲染 HTML | `<c:out escapeXml="false">` | - |

### 记忆口诀

```
{{ }} 显示值
v-if   条件渲染（DOM 里没有）
v-for  循环渲染（记得加 :key）
v-model 双向绑定（表单）
:xxx   传变量（v-bind 简写）
@xxx   绑事件（v-on 简写）
```

---

## ✅ Day05 检查清单

- [ ] 理解声明式渲染：数据变了页面自动更新，不用手动改 DOM
- [ ] `{{ }}` 插值：能看懂 `{{ queryParam.name }}`
- [ ] `v-if` / `v-show`：知道两者区别，`v-if` 不渲染 DOM
- [ ] `v-for`：能看懂 `v-for="col in columns" :key="col.prop"`
- [ ] `v-model`：能看懂 `<el-input v-model="queryParam.name" />`
- [ ] `:prop`：知道冒号 = 传变量，不是字符串
- [ ] `@event`：能看懂 `@click="handleSave"`
- [ ] 综合：能看懂 `<el-table :data="dataSource" @selection-change="handleSelect">`

### 🎯 本节回答了什么问题

> 打开公司项目 `.vue` 文件的 `<template>` 区，能看懂 `{{ }}`、`v-if`、`v-for`、
> `v-model`、`:prop`、`@event` 这些指令，知道每一行在"声明渲染什么、绑定什么、监听什么"。

---

## 下一步

> **Day06**：Vue 组件 —— 看懂 `<script>` 里的东西。模板看懂了，但数据从哪来、方法在哪定义？
> `data()`、`methods`、`computed`、`watch`、`props`、`$emit` 这些才是逻辑核心。学完 Day06，
> 你就能把 `<template>` 和 `<script>` 两半拼起来，看懂一个完整的 `.vue` 文件。

---

> 📅 生成日期：2026-08-14
