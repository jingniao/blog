---
title: atom使用
mathjax: false
date: 2016-07-21 07:26:22
tags: [atom,python]
categories: python
---
# 开始使用atom
前几天使用了vscode这个微软出品的atom芯的编辑器，应该说限制有点大，但是工作中使用时没有问题的，只是希望提高sftp插件的水平，现在动不动就卡死有点受不了。  
然后我就开始尝试着使用atom这款github出品的编辑器。感觉就是另一种了，首先，两者的方向就不大一致。atom的目标是自由，按照网友的说法是，基本上所有的组建都是可以调整的。并且atom更新也比vscode勤快，插件数量，尤其是同类插件数量上，也比vscode强了好多。
<!-- more --> 
## 常用的插件
-  atom-beautify 格式化代码，支持的代码数量非常多，经实验，python的支持也算完善了。
-  autocomplete-python python的自动补全工具，应该说，还是挺智能的，也能搜索到所有本地文件里的类之类的。
-  script 简单的运行python脚本，应该是差不多了
-  linter-pep8 linter-pylint，pep8的检查只是检查pep8的风格，只要程序风格对了，其他的不管。pylint的检查则要精细化好多，检查的项目也多了好多，例如变量常量命名，未使用变量之类的都会进行一些语义的检查。坏处是对于代码引用的时候判断，如果不设置环境变量，则一般是找不到，还有检查项细致了太多，如果想要用好，配置可能比较麻烦。
-  simplified-chinese-menu 中文化插件，将大部分的核心菜单都翻译成了中文。看着顺眼了不少。
-  remote-ftp 必不可少的sftp同步插件，比vscode的同步插件靠谱了好多，而且atom给了更大的自由度，所以菜单里有同步选项了。

## 未完待续
