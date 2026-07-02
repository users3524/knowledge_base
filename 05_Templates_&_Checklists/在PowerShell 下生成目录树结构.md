# PowerShell 生成目录树方法汇总

在整理项目工程或知识库时，经常需要输出清爽的目录树结构。以下是几种在 Windows 下实现的方法。

---

## 1. Windows 自带传统 Tree 命令
最基础的命令，但功能较弱。
```powershell
# /F 代表显示每个文件夹中的文件名称
tree /F

PS D:\Work\important_doc> tree /F
卷 Work 的文件夹 PATH 列表
卷序列号为 0000022B 28A2:D4ED
D:.
│  README.md
│
├─.vscode
│      README.md
│      settings.json
│
├─01_Frameworks_&_Flows
│      BMS软件开发流程_详细版.md
│      MCU软件开发流程.md
│      README.md
│      ROS软件开发流程_详细版.md
│
├─02_Hardware_Engineering
│      README.md
│
├─03_Software_Architecture
│      README.md
│
├─04_Curriculum_&_Growth
│      README.md
│      嵌入式硬件课程大纲.md
│      嵌入式软件课程大纲.md
│
└─05_Templates_&_Checklists
        .gitkeep
        README.md
        在PowerShell 下生成目录树结构.md

没有子文件夹
```

## 2. PowerShell 原生命令（获取完整路径）

```powershell
Get-ChildItem -Recurse | Select-Object FullName

PS D:\Work\important_doc> Get-ChildItem -Recurse | Select-Object FullName

FullName
--------
D:\Work\important_doc\.vscode
D:\Work\important_doc\01_Frameworks_&_Flows
D:\Work\important_doc\02_Hardware_Engineering
D:\Work\important_doc\03_Software_Architecture
D:\Work\important_doc\04_Curriculum_&_Growth
D:\Work\important_doc\05_Templates_&_Checklists
D:\Work\important_doc\README.md
D:\Work\important_doc\.vscode\README.md
D:\Work\important_doc\.vscode\settings.json
D:\Work\important_doc\01_Frameworks_&_Flows\BMS软件开发流程_详细版.md
D:\Work\important_doc\01_Frameworks_&_Flows\MCU软件开发流程.md
D:\Work\important_doc\01_Frameworks_&_Flows\README.md
D:\Work\important_doc\01_Frameworks_&_Flows\ROS软件开发流程_详细版.md
D:\Work\important_doc\02_Hardware_Engineering\README.md
D:\Work\important_doc\03_Software_Architecture\README.md
D:\Work\important_doc\04_Curriculum_&_Growth\README.md
D:\Work\important_doc\04_Curriculum_&_Growth\嵌入式硬件课程大纲.md
D:\Work\important_doc\04_Curriculum_&_Growth\嵌入式软件课程大纲.md
D:\Work\important_doc\05_Templates_&_Checklists\.gitkeep
D:\Work\important_doc\05_Templates_&_Checklists\README.md
D:\Work\important_doc\05_Templates_&_Checklists\在PowerShell 下生成目录树结构.md
```

## 3. PowerShell 高级自定义脚本（带格式化缩进）

在当前目录下执行，会自动根据目录深度输出 |-- 结构：
```powershell
Write-Host "$($pwd.Path.Split('\')[-1])"; dir -Recurse | ForEach-Object { $depth = ($_.FullName.Split('\').Count) - ($pwd.Path.Split('\').Count); "  " * $depth + "|--" + $_.Name }

PS D:\Work\important_doc> Write-Host "$($pwd.Path.Split('\')[-1])"; dir -Recurse | ForEach-Object { $depth = ($_.FullName.Split('\').Count) - ($pwd.Path.Split('\').Count); "  " * $depth + "|--" + $_.Name }
important_doc
  |--.vscode
  |--01_Frameworks_&_Flows
  |--02_Hardware_Engineering
  |--03_Software_Architecture
  |--04_Curriculum_&_Growth
  |--05_Templates_&_Checklists
  |--README.md
    |--README.md
    |--settings.json
    |--BMS软件开发流程_详细版.md
    |--MCU软件开发流程.md
    |--README.md
    |--ROS软件开发流程_详细版.md
    |--README.md
    |--README.md
    |--README.md
    |--嵌入式硬件课程大纲.md
    |--嵌入式软件课程大纲.md
    |--.gitkeep
    |--README.md
    |--在PowerShell 下生成目录树结构.md
```

## 4. 固化为本地快捷命令 mytree
通过配置 PowerShell 用户 Profile，实现随时输入 mytree 直接输出结构。

配置步骤：
1. 创建配置文件：
    ```PowerShell
    New-Item -Type File -Path $PROFILE -Force
    ```
2. 编辑配置文件：
    ```PowerShell
    notepad $PROFILE
    ```
3. 在记事本中粘贴以下函数并保存：
    ```PowerShell
    #树状图遍历当前文件夹
    function mytree {
        param (
            [string]$Path = ".",
            [int]$Indent = 0
        )
        
        # 仅在最顶层（第一次执行）时，打印当前文件夹名
        if ($Indent -eq 0) {
            $resolvedPath = Get-Item $Path
            Write-Host $resolvedPath.Name -ForegroundColor Cyan
        }

        # 获取当前目录下的所有子项（先排文件夹，再排文件，按名称排序）
        $items = Get-ChildItem -Path $Path | Sort-Object @{Expression={$_.Attributes -match "Directory"}; Descending=$true}, Name

        foreach ($item in $items) {
            # 根据缩进层级生成前缀
            $spaces = "  " * $Indent
            Write-Host "${spaces}|-- $($item.Name)"

            # 如果是文件夹，则递归进入下一层
            if ($item.Attributes -match "Directory") {
                mytree -Path $item.FullName -Indent ($Indent + 1)
            }
        }
    }
    ```
4. 权限解除（若重启后提示权限不足，用管理员身份打开 PS 执行）：
    ```PowerShell
    Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
    ```