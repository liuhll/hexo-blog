---
title: YAML语言
date: 2018-10-14 18:44:39
categories: "YAML"
tags:
- YAML
- 配置文件
---

## 简介
`YAML`是一个可读性高，用来表达资料序列的文件格式。`YAML`是**YAML Ain’t a Markup Language**（YAML不是一种标记语言）的缩写 。在开发的这种语言时，`YAML` 的意思其实是：**Yet Another Markup Language**（仍是一种标记语言），但为了强调这种语言以数据做为中心，而不是以标记语言为重点，而用反向缩略语重新命名。实际上，它是JSON的一个超集，因此，在使用的时候，你可能需要采用JSON风格的语法来跳出空格流。

YAML的发音为 /ˈjæməl/ ）,设计目标是方便人类读写。它实质上是一种通用的数据串行化格式。YAML主要的作用是用来写配置文件(例如：`docker-compose`编排文件,`string-boot`配置文件等),非常简洁和强大。

## 基本语法规则
- 大小写敏感
- 使用缩进表示层级关系
- 缩进时不允许使用`Tab`键，只允许使用空格。
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可

> **Notes**
> - `#` 表示注释，从这个字符一直到行尾，都会被解析器忽略


## YAML 支持的数据结构
`YAML`支持的数据结构有三种：
1. 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
2. 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
3. 纯量（`scalars`）：单个的、不可再分的值

## 对象
对象的一组键值对，使用冒号`:`结构表示。

```yaml
animal: pets
```
Yaml也允许另一种写法，将所有键值对写成一个行内对象:
```yaml
hash: { name: Steve, foo: bar } 
```

## 数组
一组连词线开头的行，构成一个数组:
```yaml
- Cat
- Dog
- Goldfish
```
数组也可以采用行内表示法:
```yaml
animal: [Cat, Dog]
```

数据结构的子成员是一个数组，则可以在该项下面缩进空格:
```yaml
- 
 - Cat
 - Dog
 - Goldfish
# 解析为: [[Cat, Dog, Goldfish]]
```

## 复合结构
对象和数组可以结合使用，形成复合结构:
```yaml
languages:
 - Ruby
 - Perl
 - Python 
websites:
 YAML: yaml.org 
 Ruby: ruby-lang.org 
 Python: python.org 
 Perl: use.perl.org 
```

## 纯量
纯量是最基本的、不可再分的值。

### 数值
数值直接以字面量的形式表示。
```yaml
number: 12.30
```

### 布尔值
布尔值用`true`和`false`表示。
```yaml
isSet: true
```

### 空值
`null`用`~`表示。
```yaml
parent: ~ 
```

### 日期和时间
时间和时间均采用 ISO8601 格式。
```yaml
iso8601: 2001-12-14t21:59:43.10-05:00 
date: 1976-07-31
```
### 字符串
字符串是最常见，也是最复杂的一种数据类型。
1. 字符串默认不使用引号表示。
  ```yaml
  str: 这是一行字符串
  ```
2. 如果字符串之中包含空格或特殊字符，需要放在引号之中。
  ```yaml
  str: '内容： 字符串'
  ```
3. 单引号和双引号都可以使用，双引号不会对特殊字符转义。
  ```yaml
  s1: '内容\n字符串'
  s2: "内容\n字符串"
  # 转为js为: { s1: '内容\\n字符串', s2: '内容\n字符串' }
  ```
4. 单引号之中如果还有单引号，必须连续使用两个单引号转义。
  ```yaml
  str: 'labor''s day' 
  ```
5. 字符串可以写成多行，从第二行开始，必须有一个单空格缩进。换行符会被转为空格。
  ```yaml
  str: 这是一段
   多行
   字符串
  # 转为js为： { str: '这是一段 多行 字符串' }
  ```
6. 多行字符串可以使用`|`保留换行符，也可以使用`>`折叠换行。 
  ```yaml
  this: |
    Foo
    Bar
  that: >
    Foo
    Bar
  # 转为js为： { this: 'Foo\nBar\n', that: 'Foo Bar\n' }
  ```
7. `+`表示保留文字块末尾的换行，`-`表示删除字符串末尾的换行。
  ```yaml
  s1: |
   Foo

  s2: |+
   Foo


  s3: |-
   Foo

  # 转为js为： { s1: 'Foo\n', s2: 'Foo\n\n\n', s3: 'Foo' }
  ```

8. 字符串之中可以插入 HTML 标记。
   ```yaml
   message: |
    
    <p style="color: red">
    段落
    </p>
   # 转为js为： { message: '\n<p style="color: red">\n  段落\n</p>\n' }
   ```
## 引用
锚点`&`和别名`*`，可以用来引用。
`&`用来建立锚点（defaults），`<<`表示合并到当前数据，`*`用来引用锚点
```yaml
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
```

等同于下面的代码:

```yaml
defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```