# vscode tips
- 对python而言，run和debug下sys.path为运行当前文件所在目录+默认指定的目录。
- os.path.abspath('.‘)为终端pwd,vscode运行python代码时，会进入到当前python文件所在路径
```sys.path.append(os.path.abspath('.'))```

# python模块调用
- 在__init__.py里添加
    ```python
        import os, sys
        sys.path.append(os.path.dirname(os.path.realpath(__file__)))
    ```
- 在其他路径就可以直接import ```__init__.py```对应文件夹下的py文件

# 论文代码Tips
- [avod](https://github.com/staillyd/avod/blob/master/info/%E8%AF%B4%E6%98%8E.md)
- [deep-sort](https://github.com/staillyd/deep_sort/blob/master/note/code.md)
- [cosine_metric_learning](https://github.com/staillyd/cosine_metric_learning/blob/master/note/code.md)

# github tips
- 提升github clone速度,将网址替换为:```github.com.cnpmjs.org```