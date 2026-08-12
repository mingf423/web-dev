# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

面向 **Java 开发者**的前端 Web 开发学习项目，内容以中文编写。从纯 HTML/CSS/JS 开始，逐步过渡到 Vue 3 框架。每个 `day` 目录包含一个 HTML 示例和对应的入门指南（`.md`），用 Java 概念对照解释前端知识点。

## 目录结构

```
01_基础知识/
  day01/                # HTML/CSS/JS 基础 + DOM 操作
    hello.html          # 示例页面
    前端入门指南.md       # 本日知识点详解（Java 对照）
  day02/                # CSS 布局（盒模型、Flexbox、定位）
  day03/                # JS 深入（数组、对象、Promise、fetch）
  day04/                # 前端工具链（Node.js/npm/Vite）
  day05+/               # Vue 3 + Element Plus
```

新增学习内容遵循命名规范：`{序号}_{模块名}/day{序号}/`。

## 学习路线

```
纯 HTML/CSS/JS → CSS 布局 → JS 深入 → 工具链 → Vue 3 + Element Plus → 项目实战
```

## 技术栈

- 早期：纯 HTML/CSS/JavaScript，无构建工具、无包管理器、无框架
- 后期：Vue 3 + Element Plus + Vite
- 编码：UTF-8
- 语言：中文（zh）

## 开发方式

早期 `.html` 文件直接在浏览器中打开即可预览，无需安装依赖或启动开发服务器。进入 Vue 阶段后按对应 `package.json` 中的脚本执行。
