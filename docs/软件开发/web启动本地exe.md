---
tags: ["exe", "web", "注册表"]
---

# 生成注册表文件（shell.bat）

**注意：exe文件需要和shell.bat在同一目录下，且不要轻易更换exe的路径和名称。**

```cmd
@echo off
setlocal enabledelayedexpansion

:: 获取当前路径
set "current_path=%~dp0"

:: 设置文件名
set "exe_name=exe_name.exe"
set "exe_path=%current_path%%exe_name%"

:: 确保路径中不含特殊字符，并处理反斜杠
set "escaped_path=%exe_path:\=\\%"

:: 创建注册表文件内容
(
echo Windows Registry Editor Version 5.00
echo.
echo [HKEY_CLASSES_ROOT\qfx]
echo @="audioRecord"
echo "URL Protocol"=""
echo.
echo [HKEY_CLASSES_ROOT\qfx\DefaultIcon]
echo @="!escaped_path!"
echo.
echo [HKEY_CLASSES_ROOT\qfx\shell]
echo @=""
echo.
echo [HKEY_CLASSES_ROOT\qfx\shell\open]
echo @=""
echo.
echo [HKEY_CLASSES_ROOT\qfx\shell\open\command]
echo @="\"!escaped_path!\" \"%%1\""
) > register.reg
echo "OK!"
pause
```

# 使用

**双击shell.bat生成register.reg，双击register.reg即可完成注册** 。

**使用下列代码即可 _唤醒本地_ exe**。

```html
<a href="qfx:">打开程序</a>
```
