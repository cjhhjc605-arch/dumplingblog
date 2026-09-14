---
title: AI使用技巧与心得
published: 2026-09-14
tags:
  - AI
description: ""
---

## 笔记学习流程

### 1. 自动添加知识点
在使用 AI 的时候可以让 AI 自动添加不会的知识点到对应笔记位置。

### 2. 自动生成练习题
学完一个知识后，让 AI 在笔记末尾生成练习题：
- 每道题附带**复选框** `- [ ]`，做完改成 `- [x]`
- 每道题附带**参考答案**
- 题目针对 Windows 环境，代码可以直接运行

### 3. 同步创建练习文件
AI 会在 `E:\GithubProgect\MyRunProject\Daily-Learning\python` 下创建以笔记名命名的子文件夹，并在其中创建对应的 `.py` 文件：
- 子文件夹命名：笔记标题（如 `Python3数据类型转换`）
- 文件命名格式：`test_序号_知识点.py`
- 例如：`python/Python3数据类型转换/test_01_implicit.py`
- 每个文件包含题目要求和代码框架，可直接运行

### 4. 使用方式
告诉 AI：
```
出题
```
然后 AI 会自动：
1. 在笔记文件末尾添加练习题
2. 在 Daily-Learning/python 文件夹下创建对应的 .py 文件