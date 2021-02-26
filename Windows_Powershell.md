# Windows PowerShell常用操作命令及语法
## 日志查看
### 列出系统中所有日志列表
```powershell
Get-WinEvent -ListLog "*"
```
```text
LogMode   MaximumSizeInBytes RecordCount LogName
-------   ------------------ ----------- -------
Circular            15728640        1715 Windows PowerShell
Circular            20971520        9036 System
Circular            20971520       25990 Security
Circular            20971520           0 Key Management Service
Circular             1052672           0 Internet Explorer
Circular            20971520           0 HardwareEvents
Circular            20971520       15093 Application
Circular             1052672             Windows Networking Vpn Plugin Platform/OperationalVerbose
Circular             1052672             Windows Networking Vpn Plugin Platform/Operational
Circular             1052672           0 SMSApi
Circular             1052672          44 Setup
Circular             1052672           0 OpenSSH/Operational
Circular             1052672           0 OpenSSH/Admin
Circular             1052672             Network Isolation Operational
Circular             1052672           0 Microsoft-WindowsPhone-Connectivity-WiFiConnSvc-Channel
.....................此处省略大概400条日志信息.....................
Circular             1052672        2266 Microsoft-Client-Licensing-Platform/Admin
Circular            10485760           0 Microsoft-AppV-Client/Virtual Applications
Circular            10485760           0 Microsoft-AppV-Client/Operational
Circular            10485760           0 Microsoft-AppV-Client/Admin
Circular             1052672          88 Lenovo-Power-BaseModule/Operational
Circular            20971520             ForwardedEvents
```
### 查看指定类型和指定日志等级的日志
```powershell
Get-WinEvent -FilterHashtable @{Logname='system';level='2'}
```
```text
Logname: 是Get-WinEvent -ListLog "*"查询出来的LogName
level:
|  No  | Level |
| :-:  |  :-:  |
|  1   |  关键  |
|  2   |  错误  |
|  3   |  警告  |
|  4   |  信息  |
```
## 命令结果输出内容格式化
### 格式化为json
```powershell
Get-WinEvent -ListLog "*" | ConvertTo-Json
```
* 输出结果:
```json
{
        "LogName":  "ForwardedEvents",
        "LogType":  1,
        "LogIsolation":  2,
        "IsEnabled":  false,
        "IsClassicLog":  false,
        "SecurityDescriptor":  "O:BAG:SYD:(A;;0x2;;;S-1-15-2-1)(A;;0x2;;;S-1-15-3-1024-3153509613-960666767-3724611135-2725662640-12138253-543910227-1950414635-4190290187)(A;;0xf0007;;;SY)(A;;0x7;;;BA)(A;;0x7;;;SO)(A;;0x3;;;IU)(A;;0x3;;;SU)(A;;0x3;;;S-1-5-3)(A;;0x3;;;S-1-5-33)(A;;0x1;;;S-1-5-32-573)",
        "LogFilePath":  "%SystemRoot%\\System32\\Winevt\\Logs\\ForwardedEvents.evtx",
        "MaximumSizeInBytes":  20971520,
        "LogMode":  0,
        "OwningProviderName":  "Microsoft-Windows-EventCollector",
        "ProviderNames":  [
                              "Microsoft-Windows-EventCollector"
                          ],
        "ProviderLevel":  null,
        "ProviderKeywords":  null,
        "ProviderBufferSize":  64,
        "ProviderMinimumNumberOfBuffers":  0,
        "ProviderMaximumNumberOfBuffers":  64,
        "ProviderLatency":  1000,
        "ProviderControlGuid":  null,
        "FileSize":  null,
        "IsLogFull":  null,
        "LastAccessTime":  null,
        "LastWriteTime":  null,
        "OldestRecordNumber":  null,
        "RecordCount":  null
    }
```
* 命令解释:
```text
ConvertTo-Json: 格式化输出为Json
ConvertTo-Csv:  格式化输出为Csv
ConvertTo-Html: 格式化输出为Html
ConvertTo-Xml:  格式化输出为Xml
......还有其他格式，此处忽略......
```
## 命令结果输出内容替换
### replace替换输出内容

* 替换前示例命令:
```powershell
$a=Get-EventLog application -after (get-date).adddays(-1) | Where-Object{($_.EntryType -eq "error") -or ($_.EntryType -eq "warning")} | select EventID,EntryType,TimeGenerated,Source | Sort-Object -Property Source | Sort-Object -Property source -Unique | ConvertTo-Json
$a
```
* 替换前输出内容:
```text
[
    {
        "EventID":  1000,
        "EntryType":  1,
        "TimeGenerated":  "\/Date(1613790497000)\/",
        "Source":  "Application Error"
    },
    {
        "EventID":  1002,
        "EntryType":  1,
        "TimeGenerated":  "\/Date(1613792277000)\/",
        "Source":  "Application Hang"
    },
    {
        "EventID":  455,
        "EntryType":  1,
        "TimeGenerated":  "\/Date(1613814037000)\/",
        "Source":  "ESENT"
    },
    {
        "EventID":  1023,
        "EntryType":  1,
        "TimeGenerated":  "\/Date(1613803838000)\/",
        "Source":  "Microsoft-Windows-Perflib"
    }
]
```
* 替换后示例命令:
```powershell
$a=Get-EventLog application -after (get-date).adddays(-1) | Where-Object{($_.EntryType -eq "error") -or ($_.EntryType -eq "warning")} | select EventID,EntryType,TimeGenerated,Source | Sort-Object -Property Source | Sort-Object -Property source -Unique | ConvertTo-Json
$a=$a -replace "\\\/Date\(" -replace "\)\\\/"
$a
```
* 替换后输出内容:
```text
[
    {
        "EventID":  1000,
        "EntryType":  1,
        "TimeGenerated":  "1613804834000",
        "Source":  "Application Error"
    },
    {
        "EventID":  1002,
        "EntryType":  1,
        "TimeGenerated":  "1613792277000",
        "Source":  "Application Hang"
    },
    {
        "EventID":  455,
        "EntryType":  1,
        "TimeGenerated":  "1613817637000",
        "Source":  "ESENT"
    },
    {
        "EventID":  1020,
        "EntryType":  1,
        "TimeGenerated":  "1613803838000",
        "Source":  "Microsoft-Windows-Perflib"
    }
]
```
* 反义解析:
```text
Windows反义字符: `和\
\\\/Date\(    #反义之前\/Date(
\)\\\/        #反义之前)\/
```
## Windows设备类型
### 列出Windows逻辑设备信息
* 执行命令:
```powershell
Get-WmiObject [-Class] Win32_logicaldisk
```
* 执行结果:

```text
DeviceID     : C:
DriveType    : 3
ProviderName :
FreeSpace    : 38964342784
Size         : 85218238464
VolumeName   : 系统

DeviceID     : D:
DriveType    : 3
ProviderName :
FreeSpace    : 253565566976
Size         : 394200608768
VolumeName   : 软件
```
* 设备类型(DriveType)说明:
```text
|Drive            |Device No|
|-----------------|---------|
|Unknown          |    0    |
|No Root Directory|    1    |
|Removable Disk   |    2    |
|Local Disk       |    3    |
|Network Drive    |    4    |
|Compact Disc     |    5    |
|RAM Disk         |    6    |
```
## 执行结果计数
* 执行命令:
```powershell
@(Get-WmiObject win32_logicaldisk).Count
```
* 执行结果:
```text
2
```