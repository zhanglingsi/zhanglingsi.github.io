---
title: 我的 Python 笔记（一）
categories: python
copyright: true
---

# 我的 Python 笔记（一）

## 初识Python
1. Python 越来越火，各种领域都在发力，大数据，机器学习，人工智能等。
2. 让我这个JAVA Coder开始关注这门语言，听说高中、大学毕业都要开始考试科目里加上这门语言，成为必修语言。
3. 只是在Python介绍里面，知道python和JavaScript一样是解释型的语言，也是和JAVA一样，需要被预编译成字节码执行的，也需要像JVM一样的环境，做到可移植性。
    
### 安装Python 选择V2.7 ？还是V3.6 ？ 

 1. 这两个版本完全不能兼容，所以我安装了windows版的python V3.6
 2. 突然发现学习的时候很多程序都基于V2.7，就去学V2.7 ，又卸载V3.6 
 3. 搞环境的时候，发现有个软件Anaconda可以管理什么python的虚拟环境？
 4. 下载安装来玩玩，Anaconda3可以方便的切换python运行时版本环境。
 5. 教程选择廖雪峰大神的Python3教程，每个语言都是从基础语法开始。发现很多好用的东东。激发自己的学习兴趣。https://www.liaoxuefeng.com/

### Anaconda3 安装配置
    
> 介绍： Anaconda 是一个用于科学计算的 Python 发行版，支持 Linux, Mac, Windows,包含了众多流行的科学计算、数据分析的 Python 包。

> Anaconda 安装包可以到 https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/ 下载。

1. 去官网下载安装包，windowns和Mac 安装步骤都一样，一路下一步即可。
   > 注：安装后比较占用空间，我安装到C盘，居然有9个G，还好我C盘比较大，安装Anaconda之前不用安装python，自带的。
2. 验证安装，cmd进去，分别输入python、ipython、conda等命令，会看到相应的结果，说明安装成功。
3. Anaconda3安装成功之后，我们需要修改其包管理镜像为国内源。执行以下命令
```
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --set show_channel_urls yes
```
4. 在 Windows 上，会随 Anaconda 一起安装一批应用程序：
 - Anaconda Navigator，它是用于管理环境和包的 GUI
 - Anaconda Prompt 终端，它可让你使用命令行界面来管理环境和包
 - Spyder，它是面向科学开发的 IDE
5. 为了避免报错，我推荐在默认环境下更新所有的包。打开 Anaconda Prompt （或者 Mac 下的终端），键入：`conda upgrade --all`  
<!-- more -->

### Anaconda3 命令操作

> 1. 安装包命令 
    $> **conda install pymysql**
   
> 2. 安装多个包命令 安装时anaconda会自动管理依赖无需干预
    $> **conda install pymysql django** 
    
> 3. 指定版本 
    $> **conda install numpy=1.10** 

> 4. 如果使用pip是这样滴，注意区别
    $> **pip install numpy==1.10** 

> 5. 列出环境下已安装的包
    $> **conda list**     

> 6. 更新包
    $> **conda update django**     

> 7. 搜索包
    $> **conda search djan** 

> 8. conda 创建虚拟环境  -n / -name 创建环境名称 list packages 为新建的环境安装包 
    $> **conda create -n learn_env django** 

> 9. conda 指定创建的环境的python版本
    $> **conda create -n learn_env python=3.6** 

> 10. 进入创建的虚拟环境 在cmd和Mac中
    $> **source activate learn_env** 

> 11. 进入创建的虚拟环境 在Anaconda Prompt 终端
    $> **activate learn_env** 

> 12. 可查看当前的激活的环境
    $> **conda env list** 

> 13. 保存导出环境信息文件
    $> **conda env export > environment.yaml** 

> 14. 加载导入环境信息文件
    $> **conda env create -f environment.yaml** 
    
> 15. 删除虚拟环境
    $> **conda env remove -n learn_env** 


 - 另：为我从事的每个项目创建环境很有用。这对于与数据不相关的项目（例如使用 Flask 开发的 Web 应用）也很有用。例如，个人博客Pelican [http://docs.getpelican.com/en/stable/#] Python的静态网站生成器   
 
 - 共享环境 : 在 GitHub上共享代码时，最好同样创建环境文件并将其包括在代码库中。这能让其他人更轻松地安装你的代码的所有依赖项。
 
 - 对于不使用 conda的用户，我通常还会使用 pip freeze [https://pip.pypa.io/en/stable/reference/pip_freeze/] 将一个 pip requirements.txt 文件导出并包括在其中。

