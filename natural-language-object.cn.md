# **NOBJ** (Natural Object) - Natural Language Object Notation

## 简介

`NOBJ` 旨在提供一种更接近自然语言的结构化数据格式， 同时保留 JSON 的简洁性和易于解析的特点，
目的是让 LLM 输出更高质量的结果供程序解析，无须二次转换为JSON。

**优势：**

* **易读性:**  使用更接近日常语言的词语和结构，方便LLM阅读和理解。
* **可维护性:**  更直观的设计，更容易理解和修改数据结构。
* **跨语言易用性:**  自然语言元素可以更容易跨越语言障碍进行理解。

**劣势：**

* 当前仅支持Flat简单对象或数组。
* 缺少层级结构，无法表示嵌套对象或数组。
* 不支持可选项。
* 需要提示词保证LLM输出正确的顺序和格式。

## 举例

**NOBJ 表示简单对象**:

```yaml
name: John Doe
age: 30
city: New York
# 数组值
tags: developer, engineer
```

**NOBJ 表示简单数组**:

```yaml
name: John Doe
age: 30
city: New York

name: Mike Doe
age: 20
city: New York
```

**备注**：

* NOBJ  还在开发阶段，具体语法规则和解析方式后续会进一步完善。
* 字符串值不包括换行符。
* 数组值用逗号分隔
* 如果 `key` 中包含空格，那么返回的json对象中 `key`中的空格会被替换为 `_`。
