# Linux bash 脚本常用操作

## 1. 启动脚本

每次开启终端时，会自动执行这个启动脚本

```bash
vi ~/.bashrc
source ~/.bashrc
```

## 2. 别名定义（alias definition）

是 Bash shell 中的一种快捷方式机制，可以在 ~/.bashrc 中加入如下两句：

```bash
# 设置编辑环境脚本的快捷方式
alias vibash='vi ~/.bashrc'
# 设置运行环境脚本的快捷方式
alias runbash='source ~/.bashrc'
```

后续需要修改环境脚本时，只需要

```bash
vibash
runbash
```