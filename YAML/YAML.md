## 1 概念

YAML（YAML Ain't Markup Language）高可读的数据序列化语言。

可以看成精简版的 JSON 文件，相对于 JSON 文件，省略了一些字符：

1. 省略掉了键的引号
2. 使用缩进代替了大括号，来控制层级关系

可以看成升级版的 JSON 文件，相对于 JSON 文件，增加了更多特性：

1. 可以注释
2. 锚点和引用

YAML的作用：

1. 配置文件
2. 对象序列化

```YAML
# 将 ip 定义成锚点，并在下面进行引用
server:
 ip: &ip 127.0.0.1
 port: 8080

server1:
 ip: *ip
 port: 8081

server2:
 ip: *ip
 port: 8082
```

## 2 基础语法

键值对

```YAML
key: value
key1:
 key2: value2
 key3: value3
 key4:
  key5: value5
  key6: value6
# key1:value1 # 错误！
```

1. 同一层级不能同名key
2. 区分大小写
3. 用缩进表示层级
4. 缩进不允许使用tab，只允许空格
5. 缩进的空格数不重要，只需要相同层级左对齐就行（相同层级必须左对齐！！！）
6. `#`表示注释
7. `:`和`value`之间需要有一个空格。但对象的第一个`:`后，由于没有`value`，因此不需要空格

## 3 数据类型

YAML数据类型：

+ 对象：键值对的集合，又称为映射（mapping），对应JSON中的属性
+ 数组：一组有序的值，又称为列表
+ 标量：单个、不可再分的值，如整数、浮点数、布尔值等基本数据类型，以及字符串

### 3-1 对象

本质为键值对，只不过值并非标量，而是一组键值

```YAML
# 对象
key:
 key1:value1
 key2:value2
 key3:
  key4:value4
```

> 对象可以嵌套

### 3-2 数组

```YAML
# 1
数组key:
 - value1
 - value2
 - value3

# 2
数组key: [value1,value2]
```

> [!note]
> 1. `-`后有空格，且其层级要低于`数组key`一层
> 2. `:`后有空格

## 4 锚点和引用

+ 使用`&别名`定义锚点
+ 使用`*别名`引用锚点
以标量为锚点

```YAML
# 标量锚点
key:
 key01: &alias value1
 key02: value02
 key03: value02

key1:
 key11: *alias
 key12: value12
 key13: value13

key2:
 key21: *alias
 key22: value22
 key23: value23
```

以键值对组合为锚点

```YAML
# 对象锚点
common: &alias
 key01: value1
 key02: value02

key:
 <<: *alias
 key03: value02

key1:
 <<: *alias
 key13: value13

key2:
 <<: *alias
 key23: value23
```

>[!note]
>`<<`为连接符，将其对应的value直接拼接

>[!question] `key: value`中的`:`后面必须有空格的原因
>为了将`key: value`中的`:`与value字符串中的`:`做区分，也就是说：

## 5 SnakeYaml工具

1. 支持完整的YAML.1.1规范
2. 支持UTF-8/UTF-16
3. 支持Java对象的序列化/反序列化
4. 支持所有的YAML定义的类型
