# Day03 JavaScript 深入 —— 看懂项目中的数据处理

> HTML/CSS 是"骨架和皮肤"，JS 才是"大脑"。项目中所有页面逻辑都在 `<script>` 区，
> 而 `<script>` 区里 80% 的代码都在**处理数据**：从接口拿到的 JSON 数组、对象，经过
> `map` / `filter` / 解构 / 展开 一顿操作，变成表格要显示的样子。
>
> 好消息是：这些操作你**基本都会**，因为它们几乎就是 Java Stream 和 Map 的翻版。

---

## 学习目标

- [ ] 掌握数组三件套：`map` / `filter` / `find`（≈ Stream 的 map/filter/findFirst）
- [ ] 会写"方法链"：`list.filter(...).map(...)` 这种组合拳
- [ ] 理解 JS 对象 `{}` 就是"万能容器"（≈ `Map<String, Object>`）
- [ ] 掌握解构赋值 `const { a, b } = obj`，一行提取字段
- [ ] 掌握展开运算符 `...`，会合并数组和对象
- [ ] 会用可选链 `?.` 优雅处理空值（≈ Java 的 null 判断）

---

## 1. 数组方法 —— 前端的 Stream API

> **这是今天的核心。** 你在 Java 里天天写的 `list.stream().map().filter().collect()`，
> 前端有几乎一模一样的写法，只是更简洁（不需要 `.stream()` 和 `.collect()` 那两下）。

### 1.1 先准备一份数据

```js
// 一个用户数组（项目里通常是接口返回的 data.records）
const users = [
    { id: 1, name: '张三', age: 30, status: 1 },   // status: 1=在职 0=离职
    { id: 2, name: '李四', age: 24, status: 1 },
    { id: 3, name: '王五', age: 35, status: 0 },
    { id: 4, name: '赵六', age: 28, status: 1 },
];
```

### 1.2 `.map()` —— 转换每个元素（≈ `stream().map()`）

```js
// 提取所有用户名
const names = users.map(u => u.name);
// 结果：['张三', '李四', '王五', '赵六']
```

```java
// Java 等价写法
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());
```

**一句话**：`map` 把数组里的**每个元素**都换成另一个东西，返回**同样长度**的新数组。

### 1.3 `.filter()` —— 过滤（≈ `stream().filter()`）

```js
// 只保留在职用户
const activeUsers = users.filter(u => u.status === 1);
// 结果：张三、李四、赵六（王五被过滤掉了）
```

```java
// Java 等价写法
List<User> activeUsers = users.stream()
    .filter(u -> u.getStatus() == 1)
    .collect(Collectors.toList());
```

**一句话**：`filter` 返回**满足条件**的元素，长度可能变短。

### 1.4 `.find()` —— 查找单个元素（≈ `stream().findFirst()`）

```js
// 找到 id 为 3 的用户
const user = users.find(u => u.id === 3);
// 结果：{ id: 3, name: '王五', age: 35, status: 0 }

// 找不到会返回 undefined（不是报错！）
const none = users.find(u => u.id === 999);
// 结果：undefined  ← 注意，这里要小心空值
```

```java
// Java 等价写法
User user = users.stream()
    .filter(u -> u.getId() == 3)
    .findFirst()
    .orElse(null);   // Java 用 orElse 兜底，JS 直接返回 undefined
```

**一句话**：`find` 返回**第一个**满足条件的元素，找不到返回 `undefined`。

### 1.5 `.sort()` —— 排序（≈ `sorted(Comparator)`）

```js
// 按年龄从小到大排序
const sorted = [...users].sort((a, b) => a.age - b.age);
// 结果：李四(24) → 赵六(28) → 张三(30) → 王五(35)
```

```java
// Java 等价写法
List<User> sorted = users.stream()
    .sorted(Comparator.comparingInt(User::getAge))
    .collect(Collectors.toList());
```

> ⚠️ 注意 `[...users]` 那一下：`.sort()` 会**原地修改**原数组，所以先复制一份再排。
> 这个 `[...xxx]` 就是下一节要讲的"展开运算符"，作用是把数组复制成新数组。

### 1.6 `.forEach()` —— 纯遍历（≈ for 循环）

```js
// 不返回新数组，只是挨个执行一次
users.forEach(u => {
    console.log(u.name + ' 今年 ' + u.age + ' 岁');
});
```

**一句话**：`forEach` 和 `map` 的区别 —— `map` 有返回值，`forEach` 没有，纯粹"做事"。

### 1.7 ⭐ 方法链：组合拳（项目里最常见的写法）

```js
// 一句话完成：过滤在职用户 → 提取 id → 排序
const activeIds = users
    .filter(u => u.status === 1)      // 1. 先过滤出在职用户
    .map(u => u.id)                   // 2. 再提取出 id
    .sort((a, b) => a - b);           // 3. 最后排序

// 结果：[1, 2, 4]
```

```java
// Java 完全等价
List<Long> activeIds = users.stream()
    .filter(u -> u.getStatus() == 1)
    .map(User::getId)
    .sorted()
    .collect(Collectors.toList());
```

> **看懂这行，你就看懂了项目里一半的数据处理逻辑：**
> ```js
> const ids = list.filter(i => i.status === 1).map(i => i.id);
> ```

### 🎯 这一节回答了

> 项目中 `dataSource.filter(i => i.checked).map(i => i.id)` 是什么意思？为什么能一个方法接一个方法地写？

### ✅ 检查点

- [ ] `map` = 转换（长度不变），`filter` = 过滤（可能变短）
- [ ] `find` 找不到返回 `undefined`，而不是报错
- [ ] 能看懂并手写 `list.filter(...).map(...)` 方法链

---

## 2. 箭头函数 —— 你已经会的 Lambda

> Day01 里你已经见过 `() => {}`，这里再补一个项目里最常用的"简写"。

```js
// 完整写法（两个参数，有函数体）
users.map(function(u) { return u.name; });

// 箭头函数（等价）
users.map((u) => { return u.name; });

// 只有一个参数：省略括号
users.map(u => { return u.name; });

// 函数体只有一行 return：省略 return 和花括号
users.map(u => u.name);        // ← 项目里 99% 都是这种写法
```

| 写法 | 说明 |
|------|------|
| `(u) => { return u.name; }` | 最完整 |
| `u => { return u.name; }` | 单参数省略 `()` |
| `u => u.name` | 单行 return 全省略 ⭐ 最常用 |

> **Java 对照**：`u => u.name` 就是 Java 的 `u -> u.getName()`，只是 JS 的箭头更简洁、没有类型。

### 🎯 这一节回答了

> 为什么 `.map(u => u.name)` 里没有 `return`？为什么 `u` 前面没有类型？

---

## 3. 对象 `{}` —— JS 的万能容器

### 3.1 JS 对象 ≈ Java 的 Map + 类 的结合体

```js
// 一个对象
const user = {
    id: 1,
    name: '张三',
    age: 30,
    status: 1,
    // 甚至可以放函数（对象的方法）
    greet() {
        return '你好，我是' + this.name;
    }
};

// 读取属性：两种写法等价
user.name;          // 点号写法（最常用）
user['name'];       // 方括号写法（动态 key 时用）

// 修改 / 新增属性（不用提前声明字段！）
user.email = 'zhangsan@test.com';   // 直接加，没有就自动创建
```

```java
// Java 里你要么建个 User 类，要么用 Map
Map<String, Object> user = new HashMap<>();
user.put("id", 1);
user.put("name", "张三");
user.get("name");   // 没有 user.name 这种点号写法
```

**关键区别**：
| | JS 对象 | Java |
|---|---------|------|
| 定义 | 直接 `{}` 字面量，无需类 | 要建类 / 用 Map |
| 字段 | 随时增删，无固定结构 | 类字段固定 |
| 访问 | `obj.name`（点号） | `obj.getName()` / `map.get("name")` |

> **一句话**：JS 对象就是"不用提前定义类、字段随意增删、用点号访问"的 `Map<String, Object>`。

### 🎯 这一节回答了

> 项目里 `res.data.records`、`row.id`、`this.queryParam.name` 这些点号是什么意思？

### ✅ 检查点

- [ ] 知道 `obj.name` 等价于 Java 的 `map.get("name")`
- [ ] 知道 JS 对象字段可以随时新增，不用声明

---

## 4. 解构赋值 —— 一行代码提取字段

> **这是读项目代码必须跨过的坎。** 接口返回的响应对象，前端几乎都用解构来"拆包"。

### 4.1 对象解构

```js
// 接口返回的响应（项目里统一是 { code, type, message, data }）
const res = { code: 0, type: 'success', message: '成功', data: { list: [...] } };

// ❌ 老写法：一个个取
const code = res.code;
const data = res.data;
const message = res.message;

// ✅ 解构写法：一行搞定
const { code, data, message } = res;
// 现在 code、data、message 三个变量都直接能用了
```

```java
// Java 没有解构，只能一个个取
int code = res.getCode();
Object data = res.getData();
String message = res.getMessage();
```

### 4.2 项目中最常见的三个解构场景

```js
// 场景1：拆接口响应（你们项目到处都是）
const { code, data } = res;
if (code === 0) {
    this.dataSource = data.records;
}

// 场景2：从对象里取字段赋值给同名变量
const { name, age } = row;   // 等价于 const name = row.name; const age = row.age;

// 场景3：重命名（取个不一样的变量名）
const { name: userName } = row;   // 把 row.name 存到 userName 变量
```

### 4.3 数组解构

```js
const arr = [1, 2, 3];
const [first, second] = arr;
// first = 1, second = 2

// 交换两个变量（不用临时变量！）
let a = 1, b = 2;
[a, b] = [b, a];   // a=2, b=1
```

### 🎯 这一节回答了

> 为什么项目里到处都是 `const { code, data, message } = res` 而不是 `const code = res.code`？

### ✅ 检查点

- [ ] 能看懂 `const { code, data } = res` 干了什么
- [ ] 知道解构只是"批量取字段"的简写，没有新概念

---

## 5. 展开运算符 `...` —— 合并与复制

> `...` 是个"魔法符号"，出现的频率极高，但含义其实很简单：**把数组/对象"摊开"**。

### 5.1 复制数组（不改变原数组）

```js
const arr1 = [1, 2, 3];
const arr2 = [...arr1];   // 复制一份（arr2 和 arr1 是两个独立数组）
// 注意：不用 ... 的话，arr2 = arr1 只是"引用"，改一个另一个也变
```

### 5.2 合并数组

```js
const a = [1, 2];
const b = [3, 4];
const c = [...a, ...b];   // [1, 2, 3, 4]
const d = [0, ...a, 5];   // [0, 1, 2, 5]（可以插在任意位置）
```

### 5.3 合并对象（项目里最常用！）

```js
// 场景：修改查询参数时，不想动原来的对象
this.queryParam = { ...this.queryParam, pageNo: 1 };
// 含义：把原来 queryParam 的所有字段展开，再把 pageNo 改成 1
// 如果原来有 pageNo，则被后面的 1 覆盖；如果没有，则新增
```

```js
const baseConfig = { host: 'localhost', port: 8080, timeout: 3000 };
const devConfig = { ...baseConfig, port: 9090 };   // 覆盖 port
// 结果：{ host: 'localhost', port: 9090, timeout: 3000 }
```

> **一句话**：`{ ...obj, key: newValue }` 就是"复制一份 obj，顺便改个字段"。等于 Java 里 clone 后 set。

### 5.4 函数参数：剩余参数

```js
// 把多个参数收集成一个数组
function sum(...nums) {
    return nums.reduce((total, n) => total + n, 0);
}
sum(1, 2, 3, 4);   // 10
```

### 🎯 这一节回答了

> 项目里 `{ ...this.queryParam, pageNo: 1 }` 是什么意思？为什么不用 `this.queryParam.pageNo = 1`？

### ✅ 检查点

- [ ] `[...arr]` 是复制数组，`[...a, ...b]` 是合并数组
- [ ] `{ ...obj, k: v }` 是复制对象并覆盖字段
- [ ] 知道 `...` 为什么在项目里高频出现（避免直接改原对象）

---

## 6. 可选链 `?.` —— 优雅的空值处理

> 还记得 Day01 那个 `Cannot read properties of null` 报错吗？可选链就是**从根上预防它**的语法。

### 6.1 问题：深层访问容易炸

```js
// 想拿用户所在公司的地址
const city = user.company.address.city;
// 如果 user.company 是 undefined，这里直接抛异常：
// ❌ Cannot read properties of undefined (reading 'address')
```

```java
// Java 里你早就习惯了这种防御
String city = null;
if (user != null && user.getCompany() != null && user.getCompany().getAddress() != null) {
    city = user.getCompany().getAddress().getCity();
}
```

### 6.2 解法：可选链 `?.`

```js
// 一路问下去，任何一环是 null/undefined，直接返回 undefined，不报错
const city = user?.company?.address?.city;
// 等价于上面 Java 那一大串 if 判断
```

### 6.3 常用场景

```js
// 场景1：安全访问
const name = user?.name;                    // user 为 undefined 也不报错
const id = row?.id;

// 场景2：安全调用方法
obj?.method?.();                             // 方法存在才调用

// 场景3：结合空值合并 ??（提供默认值）
const nickname = user?.nickname ?? '未设置';   // undefined 时给个默认值
```

> `??` 是"空值合并"：左边是 `null` 或 `undefined` 时取右边。类似 Java 的 `Optional.ofNullable(x).orElse("默认值")`。

### 🎯 这一节回答了

> 为什么有的代码用 `a.b.c`，有的用 `a?.b?.c`？问号是干嘛的？

### ✅ 检查点

- [ ] `?.` = 前面是 null/undefined 就短路返回 undefined，不报错
- [ ] `??` = 左边为空时用右边默认值
- [ ] 知道 `?.` 和 `??` 是替代一堆 if 判断的利器

---

## 7. 📂 在你的项目中找这些模式

打开项目中任意 `.vue` 文件的 `<script>` 区，你会高频看到：

```js
// ① 解构接口响应
const { code, data, message } = res;
if (code === 0) {
    this.dataSource = data.records;
    this.ipagination.total = data.total;
}

// ② 数组方法链（勾选项提取 id）
const selectedIds = this.selection.filter(i => i.checked).map(i => i.id);

// ③ 展开合并查询参数
this.queryParam = { ...this.queryParam, pageNo: 1 };

// ④ 展开复制行数据（编辑时避免污染原对象）
this.editData = { ...row };

// ⑤ 可选链安全访问
const orgName = this.userInfo?.org?.orgName ?? '未知机构';

// ⑥ 数组 find 查字典项
const dictItem = this.dictList.find(d => d.code === value);
```

---

## 8. 实践：处理一份用户数据

> 打开 [practice.html](./practice.html)，完成下面的数据处理练习。

**任务**：给一个用户数组，依次完成：

1. **筛选活跃用户**（`status === 1`）→ 用 `filter`
2. **提取姓名** → 用 `map`
3. **按年龄排序** → 用 `sort`
4. **找出年龄最大的在职用户** → 组合 `filter` + `sort` 或 `filter` + 数组尾部
5. **把所有字段解构出来打印** → 用解构

**参考代码**：

```js
// 1. 筛选在职用户
const active = users.filter(u => u.status === 1);

// 2. 提取姓名
const names = active.map(u => u.name);

// 3. 按年龄排序（复制后再排，避免改原数组）
const byAge = [...users].sort((a, b) => a.age - b.age);

// 4. 年龄最大的在职用户
const oldestActive = active.sort((a, b) => b.age - a.age)[0];

// 5. 解构打印
active.forEach(({ name, age }) => {
    console.log(`${name} 今年 ${age} 岁`);
});
```

---

## 9. 速查表

### 数组方法（≈ Stream）

| 方法 | 作用 | Java 等价 | 是否返回新数组 |
|------|------|-----------|----------------|
| `map(fn)` | 转换每个元素 | `stream().map()` | ✅ 新数组 |
| `filter(fn)` | 过滤元素 | `stream().filter()` | ✅ 新数组 |
| `find(fn)` | 查第一个 | `findFirst()` | 单个元素/`undefined` |
| `sort(fn)` | 排序 | `sorted()` | ⚠️ 原地改，返回原数组 |
| `forEach(fn)` | 遍历 | for 循环 | ❌ 无返回 |
| `reduce(fn, init)` | 累加/汇总 | `reduce()` | 单个值 |
| `includes(x)` | 是否包含 | `contains()` | 布尔值 |
| `indexOf(x)` | 位置索引 | `indexOf()` | 数字/-1 |

### 运算符速记

| 符号 | 名字 | 作用 | Java 类比 |
|------|------|------|-----------|
| `...` | 展开/剩余 | 复制、合并、收集 | 无（需手写循环） |
| `?.` | 可选链 | 安全访问，防空指针 | `if (x != null)` |
| `??` | 空值合并 | 空时给默认值 | `Optional.orElse()` |
| `{}` 解构 | 解构赋值 | 批量提取字段 | 无（逐个 get） |
| `=>` | 箭头函数 | 匿名函数 | Lambda `->` |

---

## ✅ Day03 检查清单

- [ ] 数组：能区分 `map`（转换）和 `filter`（过滤）
- [ ] 数组：知道 `find` 找不到返回 `undefined`
- [ ] 数组：能看懂并手写 `list.filter(...).map(...)` 方法链
- [ ] 对象：知道 `obj.name` 等价于 Java 的 `map.get("name")`
- [ ] 解构：能看懂 `const { code, data } = res`
- [ ] 展开：知道 `{ ...obj, k: v }` 是复制并覆盖字段
- [ ] 可选链：知道 `?.` 能防 `Cannot read properties of null`
- [ ] 实践：完成用户数组的筛选/提取/排序练习

### 🎯 本节回答了什么问题

> 打开项目 `.vue` 文件的 `<script>` 区，能看懂里面 `.map()`、`.filter()`、`...`、`{}` 解构、`?.` 在干什么。知道这些都是 Java Stream 和 Map 的"简写版"。

---

## 下一步

> **Day04**：异步请求 —— Promise / async-await / axios。看懂 `getAction(url).then(res => {...})` 这种前后端通信代码，以及 `async/await` 同步写法。

---

> 📅 生成日期：2026-08-13
