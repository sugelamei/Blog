---
title: IDEA 快捷键大全
date: 2019-08-08 18:05:01
tags: 
    - IDEA
    - 快捷键
---

### 自动代码 ###

   常用的有fori/sout/psvm+Tab即可生成循环、System.out、main方法等boilerplate样板代码 。
    
   例如要输入for(User user : users)只需输入user.for+Tab ；
    
   再比如，要输入Date birthday = user.getBirthday()只需输入user.getBirthday().var+Tab即可。
    
   代码标签输入完成后，按Tab，生成代码。


<!--more-->

1. Ctrl+Alt+O 优化导入的类和包
1. Alt+Insert 生成代码(如get,set方法,构造函数等) 或者右键（Generate）
1. fori/sout/psvm + Tab
1. Ctrl+Alt+T 生成try catch 或者 Alt+enter
1. CTRL+ALT+T 把选中的代码放在 TRY{} IF{} ELSE{} 里
1. Ctrl + O 重写方法
1. Ctrl + I 实现方法
1. Ctr+shift+U 大小写转化
1. ALT+回车 导入包,自动修正
1. ALT+/ 代码提示
1. CTRL+J 自动代码
1. Ctrl+Shift+J，整合两行为一行
1. CTRL+空格 代码提示
1. CTRL+SHIFT+SPACE 自动补全代码
1. CTRL+ALT+L 格式化代码
1. CTRL+ALT+I 自动缩进
1. CTRL+ALT+O 优化导入的类和包
1. ALT+INSERT 生成代码(如GET,SET方法,构造函数等)
1. CTRL+E 最近更改的代码
1. CTRL+ALT+SPACE 类名或接口名提示
1. CTRL+P 方法参数提示
1. CTRL+Q，可以看到当前方法的声明
1. Shift+F6 重构-重命名 (包、类、方法、变量、甚至注释等)
1. Ctrl+Alt+V 提取变量

### 查询快捷键 ###

1. Ctrl＋Shift＋Backspace可以跳转到上次编辑的地
1. CTRL+ALT+ left/right 前后导航编辑过的地方
1. ALT+7 靠左窗口显示当前文件的结构
1. Ctrl+F12 浮动显示当前文件的结构
1. ALT+F7 找到你的函数或者变量或者类的所有引用到的地方
1. CTRL+ALT+F7 找到你的函数或者变量或者类的所有引用到的地方
1. Ctrl+Shift+Alt+N 查找类中的方法或变量
1. 双击SHIFT 在项目的所有目录查找文件
1. Ctrl+N 查找类
1. Ctrl+Shift+N 查找文件
1. CTRL+G 定位行
1. CTRL+F 在当前窗口查找文本
1. CTRL+SHIFT+F 在指定窗口查找文本
1. CTRL+R 在 当前窗口替换文本
1. CTRL+SHIFT+R 在指定窗口替换文本
1. ALT+SHIFT+C 查找修改的文件
1. CTRL+E 最近打开的文件
1. F3 向下查找关键字出现位置
1. SHIFT+F3 向上一个关键字出现位置
1. 选中文本，按Alt+F3 ，高亮相同文本，F3逐个往下查找相同文本
1. F4 查找变量来源
1. CTRL+SHIFT+O 弹出显示查找内容
1. Ctrl+W 选中代码，连续按会有其他效果
1. F2 或Shift+F2 高亮错误或警告快速定位
1. Ctrl+Up/Down 光标跳转到第一行或最后一行下
1. Ctrl+B 快速打开光标处的类或方法
1. CTRL+ALT+B 找所有的子类
1. CTRL+SHIFT+B 找变量的类
1. Ctrl+Shift+上下键 上下移动代码
1. Ctrl+Alt+ left/right 返回至上次浏览的位置
1. Ctrl+X 删除行
1. Ctrl+D 复制行
1. Ctrl+/ 或 Ctrl+Shift+/ 注释（// 或者/…/ ）
1. Ctrl+H 显示类结构图
1. Ctrl+Q 显示注释文档
1. Alt+F1 查找代码所在位置
1. Alt+1 快速打开或隐藏工程面板
1. Alt+ left/right 切换代码视图
1. ALT+ ↑/↓ 在方法间快速移动定位
1. CTRL+ALT+ left/right 前后导航编辑过的地方
1. Ctrl＋Shift＋Backspace可以跳转到上次编辑的地
1. Alt+6 查找TODO


### 其他快捷键 ###

1. SHIFT+ENTER 另起一行
1. CTRL+Z 倒退(撤销)
1. CTRL+SHIFT+Z 向前(取消撤销)
1. CTRL+ALT+F12 资源管理器打开文件夹
1. ALT+F1 查找文件所在目录位置
1. SHIFT+ALT+INSERT 竖编辑模式
1. CTRL+F4 关闭当前窗口
1. Ctrl+Alt+V，可以引入变量。例如：new String(); 自动导入变量定义
1. Ctrl+~，快速切换方案（界面外观、代码风格、快捷键映射等菜单）

### 调试快捷键 ###

其实常用的 就是F8 F7 F9 最值得一提的就是Drop Frame 可以让运行过的代码从头再来。

1. alt+F8 debug时选中查看值
1. Alt+Shift+F9，选择 Debug
1. Alt+Shift+F10，选择 Run
1. Ctrl+Shift+F9，编译
1. Ctrl+Shift+F8，查看断点
1. F7，步入
1. Shift+F7，智能步入
1. Alt+Shift+F7，强制步入
1. F8，步过
1. Shift+F8，步出
1. Alt+Shift+F8，强制步过
1. Alt+F9，运行至光标处
1. Ctrl+Alt+F9，强制运行至光标处
1. F9，恢复程序
1. Alt+F10，定位到断点

### 重构 ###

1. Ctrl+Alt+Shift+T，弹出重构菜单
1. Shift+F6，重命名
1. F6，移动
1. F5，复制
1. Alt+Delete，安全删除
1. Ctrl+Alt+N，内联
