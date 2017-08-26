---
title: atom编辑器
date: 2017-04-26 20:03:36
comments: true
categories: [other]
tags: [atom, sublime]
---
# atom编辑器
## 总体介绍
此前，我一直使用OneNote作为电子笔记本，大概使用一年左右，笔记的量比较大之后，OneNote就出出现卡顿的情况。当然我也探索过其他笔记管理软件，一直未找到一款让我满意的笔记管理软件。直到我碰到Atom编辑器，一开始接触Atom这个编辑器（ https://atom.io/ ），就深深的吸引了我。当晚，我探索这个编辑器的使用直到晚上2点多，还未有睡意。

那它有哪些吸引人的特性，接下来我就介绍一下Atom的总体特性。Atom编辑器支持Windows、Mac、Linux三大桌面平台，完全免费，并且已经在GitHub上开放全部的源代码。Atom采用类似Sublime的包管理功能，支持插件扩展，可配置性非常高，可以根据自身的使用特点，按照不同的插件。Atom有着各种流行编辑器都有的特性，功能上非常丰富，支持各种编程语言的代码高亮（HTML / CSS / Javascript / PHP / Python / C / C++ / Objective C / Java / JSON / Perl / CoffeeScript / Go / Sass / YAML / Markdown 等等）和代码补全功能，极大的提高了编程效率。具有项目管理、git代码管理等特性。采用文档管理的形式管理，这就不会出现笔记越多，编辑器就运行越慢。

** 注 ** 测试过100M左右的文件打开和编辑 ，反应速度确实比sublime要慢很多。大文件操作建议sublime操作，sublime编辑丢失的可能比较小，atom卡死后编辑会丢失。个人比较喜欢的是atom的插件和界面风格的简约，一般也不会编辑几十M的文件。

先上一个页面直观感受一下：
{% img full-image '/images/atom编辑器/graph1.jpg' %}

## 使用说明
Atom编辑器连接地址：https://atom.io/ 常见的使用功能和其他软件类似，本文就不做具体介绍。
本文主要介绍一下几点：
  1. 插件添加 Atom -> Preferences（Settings） -> install 搜索需要按照的插件，找到后直接按照就ok。在Settings里面可以对已安装的插件进行管理。点击对应的插件，可以查看插件的功能说明和使用说明，其中包括常用的快捷键。对于之前安装好的插件，可以在package里面查找，点击插件同样可以查看使用说明。（如果不能安装，可以git上下载包放入~/.atom/packages目录，志辰提供），不适用的插件可以停了，加快atom的运行速度。

{% img full-image '/images/atom编辑器/graph2.jpg' %}

  2. 快捷键的修改 Atom -> Preferences（Settings） -> KeyBindings 先找到需要修改的快捷键或者插件，然后点击快捷键前的复制按钮，最后将复制的代码粘贴到（your keymap file）。对于快捷键不要重复，重复可能导致快捷键无法使用。

{% img full-image '/images/atom编辑器/graph3.jpg' %}

## 常用插件
### markdown-preview
MarkDown文本预览器，有了这款插件可以把atom变成写markdown的神器。

### markdown-scroll-sync
markdown-scroll-sync将预览和编辑窗口同步滑动。

### file-type-icons
为各种不同的文件类型显示一个漂亮的ICON

### Sublime-Style-Column-Selection
Sublime的列编辑模式，atom默认对列编辑模式支持不够友好，安装这个插件可以获得很好的列编辑体验。

### project-manager
Atom默认关闭不保持当前编辑状态，下次需要自己打开对于的文件，project-manager可以帮助用户保持当前的工作窗口状态。

### markdown-image-paste
图片、截屏粘贴。默认快捷键ctrl-v；

### markdown-pdf
支持查看pdf文件

### markdown-table-editor
一直对Markdown的table语法很无爱，直到遇到了markdown-table-editor，这操作效率简直炸了！文字已经不能表达我的激动之情了，直接看图吧。编辑完以后自动对齐。

{% img full-image '/images/atom编辑器/graph4.jpg' %}

Atom有许多各种各样的插件，用户根据自己的需求，安装对应的插件。

最近因为使用atom，又研究了一下sublime的一些插件，发现sublime有很多功能未发现。atom相对是使用上还是比较卡，建议采用sublime。sublime的使用方法和atom类似，可以通过安装插件和设置增强sublime的功能。

下面添加一下sublime的常用插件：
- SideBar Enhancements　　这个插件改进了侧边栏，增加了许多功能
- tableedit   表格编辑器
- MarkdownEditing  markdown编辑
- file icon  文件icon
- Bracket Highlighter：匹配括号
- Alignment : 等号对齐
- image2tage : 图片标签
- omnimarkuppreviewer : markdown实时预览
- Boxy Theme : 主题
- side bar : 文件操作插件

sublime 常用命令mac：
- （command + shift + p）进入命令模式，可以按照插件，删除插件或者运行插件的命令；
- （command + p） 查找文件；

附上sublime的设置：
```
{
    "color_scheme": "Packages/Boxy Theme/schemes/Boxy Ocean.tmTheme",
    "draw_white_space": "all",
    "file_exclude_patterns":
    [
        "*.pyc",
        "*.pyo",
        "*.exe",
        "*.dll",
        "*.obj",
        "*.o",
        "*.a",
        "*.lib",
        "*.so",
        "*.dylib",
        "*.ncb",
        "*.sdf",
        "*.suo",
        "*.pdb",
        "*.idb",
        ".DS_Store",
        "*.class",
        "*.psd",
        "*.db",
        "*.sublime-workspace",
        "*.doc*",
        "*.xls*"
    ],
    "folder_exclude_patterns":
    [
        ".svn",
        ".repo",
        ".git",
        ".hg",
        "CVS"
    ],
    "font_size": 15,
    "ignored_packages":
    [
        "Vintage"
    ],
    "theme": "Boxy Ocean.sublime-theme",
    "translate_tabs_to_space": true,
    "wrap_width": 80
}
```

