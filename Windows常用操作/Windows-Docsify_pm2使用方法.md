# Windows: Docsify工具pm2守护进程的配置方法


## 1. 安装 pm2
```bash
npm install -g pm2
```

## 2. 创建 start-docsify.bat
路径：D:\Docsify\start-docsify.bat
内容如下：
```bash
@echo off
cd /d D:\Docsify
docsify serve . --port 3000 --no-cache
```
<!-- python -m http.server 3000 --bind 127.0.0.1 —— NO!! -->

## 3. 创建 start-docsify.vbs（隐藏窗口运行）
路径：D:\Docsify\start-docsify.vbs
内容如下：
```bash
Set WshShell = CreateObject("WScript.Shell")
WshShell.Run "start-docsify.bat", 0
Set WshShell = Nothing
```
> WshShell.Run 是调用 .bat 文件的方式
> 第二个参数 0 表示：完全隐藏窗口
> 整个执行过程中不会弹出黑色终端

## 4. 用 PM2 启动 .vbs 脚本
打开 CMD 或 PowerShell，进入 D:\Docsify 目录，执行:
```bash
pm2 start wscript.exe --name docsify-server -- "start-docsify.vbs"
```

## 5. 保存 PM2 配置
```bash
pm2 save
```
> [PM2] Saving current process list...
> [PM2] Successfully saved in C:\Users\killer\.pm2\dump.pm2


## 6. 开机自启配置
手动添加任务计划：
- 打开「任务计划程序」
- 创建基本任务 → 名称：Restore PM2
- 触发器：登录时
- 操作 → 启动程序：
- 程序：cmd.exe
- 参数：/c pm2 resurrect
- 这样开机后会自动恢复你用 pm2 save 保存的服务。

## 7. 管理服务常用命令
```bash
pm2 list                          # 查看服务
pm2 stop docsify-server          # 停止服务
pm2 restart docsify-server       # 重启服务
pm2 delete docsify-server        # 删除服务（可重新注册）
pm2 logs docsify-server          # 查看日志
```