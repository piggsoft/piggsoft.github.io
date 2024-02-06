---
title: powershell常用命令
date: 2024-02-06 13:38:45
keyword:
tags:
  - powershell
  - powershell常用命令
categories:
  - [ "IT", "命令", "Powershell" ]
---

PowerShell是一种强大的命令行工具和脚本语言，它为系统管理员和开发人员提供了广泛的功能。在本文中，我们将介绍一些PowerShell常用命令，这些命令可以帮助您更高效地完成各种任务。无论是文件操作、网络管理还是系统管理，PowerShell都能为您提供强大的支持。现在，让我们一起探索这些实用且高效的PowerShell命令吧！

<!--more-->

## 命令导航

- [makeFileLink.ps1](#makeFileLink.ps1)
- [查看端口监听](#查看端口监听)
- [杀掉进程](#杀掉进程)

## makeFileLink.ps1

对文件夹建立`link`，并将内容迁移到其他`D`盘的对应目录，例如将`C:\Users\${user}\AppData\Roaming\JetBrains`
迁移到`D:\Users\${user}\AppData\Roaming\JetBrains`

### 用法

````pwoershell
./makeFileLink.ps1 -source "C:\Users\${user}\AppData\Roaming\JetBrains"
````

### 命令定义

````powershell
# https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.management/copy-item?view=powershell-7.2
param($source)
$OutputEncoding = 'utf8'

$target = $source.Substring(0).Replace("C:\", "D:\").Replace("C:/", "D:/")

# https://learn.microsoft.com/zh-cn/windows-server/administration/windows-commands/robocopy
#robocopy $source $target /copyall /e /xj
#Write-Output $source

Write-Host "the source folder is: "$source
Write-Host "the new link folder is: "$target

$flag = Read-Host -Prompt "Please confirm whether to proceed with file link? Y or N"

if($flag -ne "Y") 
{
    Pause
    exit 0
}

Write-Host "robocopy the " $source " into " $target
robocopy $source $target /E /COPYALL /XJ


Write-Host "Remove-Item the "$source
Remove-Item $source -Recurse -Force

Write-Host "Make link"
New-Item -ItemType SymbolicLink -Path $source -Target $target
````

## 查看端口监听

命令有两种`Get-NetTCPConnection`和`Get-NetUDPEndpoint`,两者用法一致。

### 使用Where-Object

查看状态是监控，本地端口是4000的网络连接情况

```powershell
Get-NetTCPConnection | Where-Object {$_.State -eq 'listen'} | Where-Object {$_.LocalPort -eq '4000'}
```

### 直接使用`Get-NetTCPConnection`

```powershell
Get-NetTCPConnection -State listen -LocalPort 4000
```

### 如何知道有哪些属性可以选择呢？

```powershell
Get-NetTCPConnection -State listen -LocalPort 4000 | Select-Object -Property OwningProcess
```

### 查看所有的端口和对应的进程
```powershell
Get-NetTCPConnection | 
    Where-Object { $_.State -eq "listen" } |
    Select-Object RemoteAddress,
        RemotePort,
        @{Name="PID";         Expression={ $_.OwningProcess }},
        @{Name="ProcessName"; Expression={ (Get-Process -Id $_.OwningProcess).ProcessName}}, 
        @{Name="UserName";    Expression={ (Get-Process -Id $_.OwningProcess).UserName }} |
    Sort-Object -Property ProcessName, UserName |
    Format-Table -AutoSize
```

## 杀掉进程

### 根据名称杀掉进程
```shell
Get-Process -Name node | Stop-Process
```

### 根据端口和状态杀掉进程
```powershell
 Get-NetTCPConnection -State listen -LocalPort 4000 | Select-Object -ExpandProperty OwningProcess | ForEach-Object { Get-Process -Id $_ } | Stop-Process
```

## 参考
- https://www.sysgeek.cn/windows-check-listening-ports/
- https://stackoverflow.com/questions/44509183/powershell-get-nettcpconnection-script-that-also-shows-username-process-name
- https://stackoverflow.com/questions/75833086/how-to-get-id-of-a-process-by-port-number-using-get-process-powershell-command