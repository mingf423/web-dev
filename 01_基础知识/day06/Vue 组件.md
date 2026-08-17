# Day06 Vue 组件 —— 看懂 `<script>` 里的东西

> Day05 你看懂了 `<template>` 区（页面长什么样：`{{ }}` 显示什么、`v-for` 循环什么、`@click` 绑什么）。
>
> 但问题来了：`{{ queryParam.name }}` 里的 `queryParam` 从哪来？`@click="loadData"` 里的 `loadData`
> 在哪定义？答案都在 `<script>` 区。
>
> 一个 `.vue` 文件的 `<script>` 区，本质就是一个 **JS 对象**（`export default { ... }`），
> 里面的每个「选项」对应一类职责。这些选项和你写 Java 类时用的概念**高度对应**——今天就是把这层对应关系打通。

---

## 学习目标

- [ ] 看懂 `export default { ... }` 组件整体结构（它就是一个对象）
- [ ] 掌握 `data()`：组件的本地状态（≈ 类的成员变量），并理解**为什么 data 必须是函数**
- [ ] 掌握 `methods`：组件的方法（≈ 类的方法）
- [ ] 掌握 `computed`：计算属性（≈ getter + 缓存），理解它和 `methods` 的区别
- [ ] 掌握 `watch`：监听数据变化（≈ 观察者 / 监听器）
- [ ] 掌握 `props`：父组件传值给子组件（≈ 构造器参数）
- [ ] 掌握 `$emit`：子组件通知父组件（≈ 回调函数）
- [ ] 掌握生命周期 `created` / `mounted`（≈ `@PostConstruct` / `init()`）
- [ ] 能把 `<template>` 和 `<script>` 两半拼起来，看懂项目里一个完整的列表页组件

---

## 1. 先看一个完整的组件（全景）

> 打开公司项目任意 `.vue` 文件，`<script>` 区大概率长这样。**先建立整体印象**，下面再逐项拆开讲。

```js
export default {
    name: "UserList",                    // 组件名（≈ 类名）
    mixins: [JeecgListMixin],            // 混入，复用公共逻辑（≈ 继承工具类）
    components: { CommonTable },         // 注册局部子组件（≈ 声明依赖）
    props: { userId: String },           // 父组件传进来的参数（≈ 构造器参数）
    data() {                             // 本地状态（≈ 成员变量）
        return {
            columns: [],
            loading: false,
            queryParam: { name: '' },
        };
    },
    computed: {                          // 计算属性（≈ getter 方法 + 缓存）
        displayName() { return this.user.name || '未知'; }
    },
    watch: {                             // 监听数据变化（≈ 观察者）
        userId(newVal) { this.loadData(); }
    },
    created() { this.loadData(); },      // 生命周期：组件创建后执行（≈ @PostConstruct）
    methods: {                           // 方法（≈ 类的方法）
        async loadData() { /* 发请求 */ },
        handleEdit(row) { /* 编辑 */ },
    },
}
```

### 🔑 最重要的一条规则：`this`

> 组件里的 `data` / `computed` / `methods` / `props` 里定义的东西，都能通过 `this.xxx` 访问。
> `this` 指向**当前组件实例**，等价于 Java 里的 `this`。

```js
methods: {
    handleEdit(row) {
        this.queryParam.name = row.name;   // 改 data
        this.loadData();                   // 调 methods
        this.displayName;                  // 读 computed
        this.userId;                       // 读 props
    }
}
```

> 你在 `<template>` 里写的 `{{ queryParam.name }}`、`@click="loadData"`，
> 本质就是 `this.queryParam.name`、`this.loadData`（模板里省略了 `this`）。

### 🎯 这一节回答了

> `.vue` 文件 `<script>` 区是一堆「选项」组成的对象，`this` 串起了它们。这就是整体结构。

### ✅ 检查点

- [ ] 知道 `<script>` 区 = `export default { 一堆选项 }`
- [ ] 知道 `this.xxx` 访问组件自己的东西，模板里可省略 `this`

---

## 2. `data()` —— 组件的本地状态（≈ 类的成员变量）

> 页面上的变量都定义在这里。template 里的 `{{ xxx }}` 和 `v-model="xxx"` 绑的都是这里的字段。

```js
data() {
    return {
        userName: '张三',
        queryParam: { name: '', status: 0 },   // 查询条件（对象）
        dataSource: [],                         // 表格数据（数组）
        loading: false,                         // 加载状态（布尔）
    };
}
```

### ⚠️ 重点：为什么 `data` 必须是**函数**，不能是对象？

```js
// ❌ 错误写法：data 是对象
data: {
    count: 0
}

// ✅ 正确写法：data 是函数
data() {
    return { count: 0 };
}
```

> 组件会被**复用**（比如 `v-for` 里渲染多个、页面里用多次）。如果 `data` 是对象，
> 所有实例**共享同一个对象引用**——你改第一个，第二个也跟着变（互相串数据）。
>
> 写成函数后，每个实例创建时都调用一次 `data()`，`return` 出一个**全新的对象**，实例之间互不影响。

**Java 类比**：

```
data: { count: 0 }        ≈ static int count = 0;   // 所有实例共享
data() { return {count:0} } ≈ 每个实例独立的成员变量   // new 一次独立一份
```

### 🎯 这一节回答了

> 项目里 `queryParam`、`dataSource`、`loading` 这些变量为什么都写在 `data() { return {...} }` 里？

### ✅ 检查点

- [ ] `data()` 返回一个对象，里面是本组件的所有状态变量
- [ ] 知道 `data` 必须是函数（每个实例要独立的数据）

---

## 3. `methods` —— 组件的方法（≈ 类的方法）

> `@click="handleEdit"` 里的 `handleEdit` 就定义在这里。所有「点按钮要干的事」都写在 methods。

```js
methods: {
    handleEdit(row) {                    // 编辑（点操作列的"编辑"触发）
        this.editData = { ...row };
        this.dialogVisible = true;
    },
    handleDelete(row) {                  // 删除
        this.$confirm('确认删除？').then(() => {
            this.dataSource = this.dataSource.filter(item => item.id !== row.id);
        });
    },
    async loadData() {                   // 加载数据（Day04 的 async/await）
        const res = await getAction('/user/list', this.queryParam);
        if (res.code === 0) {
            this.dataSource = res.data.records;
        }
    },
}
```

> 注意方法名和 Day04/Day05 完全打通：`loadData()` 里 `await getAction(...)` 发请求，
> `res.code === 0` 判成功，把结果 `this.dataSource = ...` 存进 data。页面就靠这个方法拿到数据。

### 🎯 这一节回答了

> `@click="handleEdit"` 的方法体在哪？在 `methods` 里，用 `this.xxx` 操作数据。

### ✅ 检查点

- [ ] `methods` 里定义方法，`@click` 等事件触发它们
- [ ] 方法内部用 `this.xxx` 读写 data、调用其他方法

---

## 4. `computed` —— 计算属性（≈ getter 方法 + 缓存）

> 有些值不是直接存起来的，而是**由其他数据算出来的**。比如"剩余待办数 = 总待办数 - 已完成数"。
> 这种「派生值」就用 `computed`。

```js
computed: {
    // 全名 = 姓 + 名（由 firstName、lastName 派生）
    fullName() {
        return this.firstName + this.lastName;
    },
    // 未完成数（由 todos 派生）
    remainCount() {
        return this.todos.filter(t => !t.done).length;
    },
}
```

```html
<!-- 模板里像用普通数据一样用，注意【不带括号】 -->
<span>{{ fullName }}</span>
<span>{{ remainCount }}</span>
```

### ⚠️ 重点：computed 和 methods 的区别（面试常考）

| | `computed` | `methods` |
|---|---|---|
| 写法 | `fullName()`，模板里**不带括号** `{{ fullName }}` | `getFullName()`，模板里**带括号** `{{ getFullName() }}` |
| 缓存 | ✅ **有缓存**，依赖没变就返回上次结果，不重算 | ❌ 每次访问都重新执行 |
| 用途 | 派生数据、需要缓存的计算 | 事件处理、有副作用的操作 |

```js
// ❌ 用 methods 算派生值：模板每次重新渲染，getFullName() 就重新执行一遍（浪费）
methods: {
    getFullName() { return this.firstName + this.lastName; }
}

// ✅ 用 computed 算派生值：firstName/lastName 没变，直接返回缓存
computed: {
    fullName() { return this.firstName + this.lastName; }
}
```

**Java 类比**：`computed` ≈ 一个**带缓存的 getter**。第一次访问时算一次，之后只要依赖的字段
没变就直接返回缓存值，不重复计算（类似懒加载 + 缓存）。

### 🎯 这一节回答了

> 项目里 `computed: { hasSelected() { return this.selection.length > 0 } }` 是什么？
> 是「选中了多少行」这个**派生值**，用 computed 写有缓存、更高效。

### ✅ 检查点

- [ ] `computed` 是「由其他数据算出来的值」，模板里不带括号
- [ ] 知道 computed 有缓存、methods 没缓存，派生值优先用 computed

---

## 5. `watch` —— 监听数据变化（≈ 观察者）

> 有些场景是：「**当某个数据变了**，我要自动做一件事」。比如 `userId` 变了，重新加载这个用户的数据。
> 这就是 `watch`。

```js
watch: {
    // 监听 userId：它一变，就调用 loadData
    userId(newVal, oldVal) {          // 新值、旧值都给你
        console.log('userId 从', oldVal, '变成了', newVal);
        this.loadData();
    },
    // 监听 queryParam.name（深度监听对象里的字段）
    'queryParam.name'(newVal) {
        this.searchQuery();
    },
}
```

**Java 类比**：`watch` ≈ `PropertyChangeListener` / `@EventListener` / 观察者模式。
你告诉 Vue「盯着 `userId`，变了就执行这个回调」，Vue 负责在它变化时通知你。

### ⚠️ watch vs computed 什么时候用哪个？

```
computed：想【算出一个新值】用 computed（有缓存）
watch：  想【数据一变就执行某个动作】用 watch（如重新发请求）
```

### 🎯 这一节回答了

> 项目里 `watch: { userId(newVal) { this.loadData() } }` 是干嘛的？
> 是「监听 userId，一变就重新加载数据」。

### ✅ 检查点

- [ ] `watch` 监听某个数据，变了自动执行回调（参数是 newVal / oldVal）
- [ ] 区分：算新值用 computed，数据一变做动作用 watch

---

## 6. `props` —— 父组件传值给子组件（≈ 构造器参数）

> 组件可以像函数一样接收「入参」，这个入参就是 `props`。父组件用 `:xxx` 传进来。

```js
// 子组件：声明自己接收哪些 props
props: {
    title: String,        // 接收一个字符串
    userId: {             // 更完整写法：类型 + 默认值
        type: Number,
        default: 0,
    },
}
```

```html
<!-- 父组件：用 :title 把值传给子组件 -->
<child-card :title="pageTitle" :userId="user.id" />
<!--                   └── 传的是变量 pageTitle 的值 -->
```

```js
// 子组件内部：props 像 data 一样用（通过 this.title 读）
template: '<div>{{ title }}</div>'
```

### ⚠️ 单向数据流（重要规则）

> props 是**父 → 子**单向传递的。子组件**不能直接修改** props（类似方法参数是只读的）。
> 如果子组件想改，应该通过第 7 节的 `$emit` 通知父组件去改。

```js
// ❌ 错误：子组件直接改 props
this.title = '新标题';

// ✅ 正确：子组件发事件，让父组件改
this.$emit('update:title', '新标题');
```

**Java 类比**：`props` ≈ **构造器参数**。父组件 `new 子组件(title, userId)` 时把值传进去，
子组件里把它当只读字段用。

### 🎯 这一节回答了

> 项目里 `props: { visible: Boolean, editData: Object }` 是什么意思？
> 是「这个弹窗组件接收两个父组件传来的参数：是否显示、要编辑的数据」。

### ✅ 检查点

- [ ] `props` 是父组件传给子组件的参数，父用 `:xxx` 传，子用 `props` 声明接收
- [ ] 单向数据流：子组件不能直接改 props

---

## 7. `$emit` —— 子组件通知父组件（≈ 回调函数）

> 第 6 节说了数据单向「父 → 子」。那子组件想反过来通知父组件怎么办？用 `$emit` 抛一个事件。

```js
// 子组件：抛出一个叫 delete 的事件，带上参数
methods: {
    handleDelete() {
        this.$emit('delete', this.id);   // 事件名 + 参数
    },
}
```

```html
<!-- 父组件：监听子组件的 delete 事件，触发自己的方法 -->
<child-card @delete="handleChildDelete" />
<!--             └── 子组件 $emit('delete') 时，调用 handleChildDelete -->
```

```js
// 父组件
methods: {
    handleChildDelete(id) {
        console.log('子组件通知我删除 id =', id);
    },
}
```

**Java 类比**：`$emit` ≈ **回调函数 / 事件监听**。父组件把一个"回调"（`@delete="handleChildDelete"`）
注册到子组件上，子组件在合适时机 `$emit('delete', ...)` 调用它，就像 `listener.onDelete(id)`。

### 🎯 这一节回答了

> 项目里 `this.$emit('update:visible', false)` 是什么？是「子组件告诉父组件：把 visible 改成 false」。

### ✅ 检查点

- [ ] 子组件 `$emit('事件名', 参数)` 抛事件，父组件 `@事件名="方法"` 监听
- [ ] props（父→子）+ $emit（子→父）组成父子通信的完整闭环

---

## 8. 生命周期 —— `created` / `mounted`（≈ `@PostConstruct` / `init()`）

> 组件从创建到显示，有一系列「时间点」。你可以在这些时间点插入自己的代码。

```
new Vue() → 初始化 data/props → created（数据就绪，DOM 还没渲染）
                              → 挂载 DOM → mounted（DOM 渲染完成）
```

```js
created() {
    // data 已就绪，但 DOM 还没渲染。
    // 通常在这里发请求加载数据（项目里几乎每个列表页都是 created() { this.loadData() }）
    this.loadData();
},
mounted() {
    // DOM 已渲染完成，可以操作 DOM、初始化图表（如 ECharts）
    this.initChart();
},
```

| 生命周期 | 时机 | 典型用途 |
|----------|------|----------|
| `created` | data 就绪，DOM 未渲染 | **发请求加载数据**（最常见） |
| `mounted` | DOM 已渲染 | 操作 DOM、初始化图表 |
| `beforeDestroy` | 组件销毁前 | 清理定时器、移除监听（了解） |

**Java 类比**：`created` ≈ Spring 的 `@PostConstruct`（bean 属性注入完成后执行）；
`mounted` ≈ 页面加载完成后的 `init()`。都是「框架在某个时刻自动回调你写的方法」。

### 🎯 这一节回答了

> 项目里每个列表页都有的 `created() { this.loadData() }` 是干嘛的？
> 是「组件一创建，就先加载一次数据」，这样页面打开就有内容。

### ✅ 检查点

- [ ] `created`：data 就绪、DOM 未渲染，常用它调 `loadData()`
- [ ] `mounted`：DOM 已渲染，可操作 DOM

---

## 9. `name` / `mixins` / `components` —— 组件结构的其他项

> 这三个在项目里也常见，认识即可。

```js
export default {
    name: "UserList",                // 组件名。路由映射、keep-alive 缓存、调试时用
    mixins: [JeecgListMixin],        // 混入：把另一个对象的方法"合并"进当前组件
    components: { CommonTable },     // 注册局部子组件：注册后才能用 <common-table>
}
```

**Java 类比**：

```
name       ≈ 类名
mixins     ≈ 继承抽象父类（把父类的 loadData/searchQuery 等方法合并进来）
components ≈ 依赖注入（声明要用 CommonTable 这个组件）
```

> 项目里 `mixins: [JeecgListMixin]` 特别重要：它给组件**注入**了 `loadData()`、`searchQuery()`、
> `searchReset()` 等方法。所以你会在代码里看到 `this.loadData()` 但翻遍当前文件都找不到
> `loadData` 的定义——它来自 mixin（Day12 会专门讲）。

### ✅ 检查点

- [ ] 认识 `name` / `mixins` / `components` 的作用
- [ ] 知道 `this.loadData()` 可能是 mixin 注入的，当前文件里找不到定义

---

## 10. 📂 在你的项目中找这些模式（完整列表页逐行拆解）

> 现在把 Day05 的 `<template>` 和今天的 `<script>` **拼起来**，看一个完整的列表页。
> 这是你的核心目标：**看懂一个 `.vue` 文件从头到尾在干什么**。

```vue
<template>
  <div>
    <!-- ① 查询表单：v-model 绑 data 里的 queryParam -->
    <el-form :model="queryParam">
      <el-input v-model="queryParam.name" placeholder="姓名" />
      <!-- ② 查询按钮：@click 绑 methods 里的 searchQuery（来自 mixin） -->
      <el-button @click="searchQuery">查询</el-button>
    </el-form>

    <!-- ③ 表格：:dataSource 传 data 里的数组，:loading 传加载状态 -->
    <common-table :dataSource="dataSource" :columns="columns" :loading="loading" />
  </div>
</template>

<script>
import CommonTable from "@/components/views/table/CommonTable";
import { JeecgListMixin } from "@/mixins/JeecgListMixin";

export default {
    name: "UserList",
    mixins: [JeecgListMixin],           // 注入 loadData/searchQuery/searchReset
    components: { CommonTable },
    data() {
        return {
            queryParam: { name: '' },   // ① 的 v-model 绑这里
            columns: [],                // ③ 的 :columns 绑这里
            dataSource: [],             // ③ 的 :dataSource 绑这里
            loading: false,             // ③ 的 :loading 绑这里
        };
    },
    created() {
        this.loadData();                // 组件一创建就加载数据（loadData 来自 mixin）
    },
    methods: {
        handleEdit(row) { /* 编辑 */ },
    },
}
</script>
```

**数据流串起来**：

```
页面打开
  → created() 触发 loadData()          (methods/mixin)
    → getAction(url, queryParam)       (Day04 发请求)
      → res.data.records               (后端返回的数据)
        → this.dataSource = ...        (存进 data)
          → :dataSource="dataSource"   (传给表格)
            → 表格渲染出每一行          (Day05 的 v-for)
```

> 看懂上面这条链，你就看懂了公司项目 80% 的列表页。

### ✅ 检查点

- [ ] 能把 template 的 `v-model`/`@click`/`:data` 和 script 的 `data`/`methods` 对应起来
- [ ] 能说清「页面打开 → 加载数据 → 渲染表格」这条完整数据流

---

## 11. 实践：跑起来看效果

> 三个演示文件，浏览器直接打开（需联网加载 Vue CDN）：

1. **[script-options.html](./script-options.html)** —— 组件 `<script>` 区各选项逐个演示（data/methods/computed/watch/生命周期）。
2. **[computed-vs-methods.html](./computed-vs-methods.html)** —— 重点：用「计算次数」直观演示 computed 的缓存 vs methods 的重复计算。
3. **[props-emit.html](./props-emit.html)** —— 父子组件通信综合练习（props 下传 + $emit 上抛）。

**验证**：在 computed-vs-methods 页反复点「触发重渲染」按钮，观察 `methodCount` 不断暴涨、
`computedCount` 却不变——这就是缓存的威力。

---

## 12. 速查表

### `<script>` 区选项一览

| 选项 | 作用 | Java 类比 | 优先级 |
|------|------|-----------|--------|
| `name` | 组件名 | 类名 | ⭐⭐ |
| `data()` | 本地状态（必须函数） | 成员变量 | ⭐⭐⭐ |
| `methods` | 方法 | 类的方法 | ⭐⭐⭐ |
| `computed` | 计算属性（有缓存） | getter + 缓存 | ⭐⭐⭐ |
| `watch` | 监听数据变化 | `PropertyChangeListener` | ⭐⭐ |
| `props` | 父传子参数 | 构造器参数 | ⭐⭐⭐ |
| `$emit` | 子传父事件 | 回调函数 | ⭐⭐⭐ |
| `created` | 生命周期（数据就绪） | `@PostConstruct` | ⭐⭐⭐ |
| `mounted` | 生命周期（DOM 就绪） | `init()` | ⭐⭐ |
| `mixins` | 混入公共逻辑 | 继承抽象父类 | ⭐⭐⭐ |
| `components` | 注册子组件 | 依赖注入 | ⭐⭐ |

### 记忆口诀

```
data()     存状态（必须是函数）
methods    写方法（@click 触发）
computed   算新值（有缓存，模板不带括号）
watch      盯变化（一变就执行动作）
props      父传子（只读，像构造器参数）
$emit      子传父（抛事件，像回调）
created    发请求加载数据
mounted    操作 DOM / 初始化图表
```

---

## ✅ Day06 检查清单

- [ ] 看懂 `export default { ... }` 组件结构，知道 `this` 串起各选项
- [ ] `data()`：知道为什么必须是函数（实例间数据独立）
- [ ] `methods`：方法定义处，`@click` 触发
- [ ] `computed`：派生值 + 缓存，模板不带括号，和 methods 区别清楚
- [ ] `watch`：监听变化执行动作
- [ ] `props` + `$emit`：父子通信闭环（父传子 / 子传父）
- [ ] `created` / `mounted`：能解释 `created() { this.loadData() }`
- [ ] 综合：能把 template 和 script 拼起来，讲清列表页数据流

### 🎯 本节回答了什么问题

> 打开公司项目 `.vue` 文件的 `<script>` 区，能看懂 `data()`、`methods`、`computed`、
> `watch`、`props`、`$emit`、`created` 每个选项在干什么，并能把 template（Day05）和
> script（Day06）拼起来，讲清楚「页面打开 → 发请求 → 存数据 → 渲染表格」这条完整链路。

---

## 下一步

> **Day07**：第 1 周回顾。不学新东西，把 Day01~Day06 的知识用起来，**独立**做一个纯 HTML 的
> 用户列表页（查询表单 + 表格 + 搜索过滤 + 删除行）。这是检验第 1 周成果的综合练习。

---

> 📅 生成日期：2026-08-14
