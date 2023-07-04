# vscode

- [1. vscode使用markdown](#1-vscode使用markdown)
  - [1.1. Markdown All in One](#11-markdown-all-in-one)
    - [1.1.1. 配置](#111-配置)
    - [1.1.2. 操作](#112-操作)
    - [1.2. Markdown Preview Enhanced](#12-markdown-preview-enhanced)
    - [1.2.1. 配置](#121-配置)
    - [1.2.2. 操作](#122-操作)
- [vscode 配置 python](#vscode-配置-python)

## 1. vscode使用markdown

插件

1. Markdown All in One
    markdown插件，各种markdown常见优化

2. Markdown Preview Mermaid Support
   markdown绘图

3. markdownlint
    markdown语法检查器
    规则：<https://www.jianshu.com/p/51523a1c6fe1>

4. Markdown Preview Github Styling
   markdown在github page上的样式预览

5. Markdown Preview Enhanced
   和Markdown All in One是一种类型的插件，但预览功能和导出功能更为强大

### 1.1. Markdown All in One

用户手册：<https://github.com/yzhang-gh/vscode-markdown#auto-completions>

#### 1.1.1. 配置

1. 关于自动生成目录的配置
   在设置中搜索`Toc`，搜索出的选项就是关于自动生成目录的设置选项

#### 1.1.2. 操作

1. 自动生成目录
   1. 打开命令面板
      1. 右键，弹出面板最下面
      2. `shift + ctrl + P`
   2. 在命令面板上输入`TOC`
   3. 选择`Create Table of Contents`

2. 自动添加/删除章节号
   1. 打开命令面板
   2. 在面板上输入section
   3. 选择添加或删除章节号
3. 打开实时预览：Ctrl + Shift + V
4. 快速打开和关闭大纲：Ctrl + Shift + O

#### 1.2. Markdown Preview Enhanced

帮助手册: <https://www.bookstack.cn/read/mpe/zh-cn-pdf.md>

#### 1.2.1. 配置

1. 打开markdown中的代码运行功能

   对`Enable Script Execution`选项勾选

#### 1.2.2. 操作

1. 插入目录

   1. 插入`[TOC]`
   2. Html代码

      ```markdown
      <!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=3 orderedList=false} -->
      ```

2. 快速预览：`ctrl k,v`

```c++
#include <iostream>
using namespace std;

int main(){

   cout << "asdasd" <<endl;

   return 0;
}
```

## vscode 配置 python

问题：vscode 的 python 插件 对于python老版本不再支持

解决方法：安装老版本的python插件
>比如, python 2.7
>
> 点击 python 插件中卸载按键旁边的下拉箭头
>
> 选择安装另一个版本
>
> 在弹出窗口中选择版本"v2021.12.1559732655"

