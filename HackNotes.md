
# Hackintosh Notes

## 无核显开启硬件解码编码

必要条件:
- AMD RX 4xx/5xx Vega 56/64
- SMBIOS iMacPro1,1
    - 如果用其他的SMBIOS需要如下启动参数
        
        - shikigva=32 (enable board-id swap)
        - shiki-id=Mac-7BA5B2D9E42DDD94 (replace the board-id with iMacPro 1,1)

        [Source](https://github.com/acidanthera/WhateverGreen/blob/master/WhateverGreen/kern_shiki.hpp#L29)
- macOS 10.14.5 
    

## 某些主板需要敲2下键盘或者鼠标才可以唤醒

加入下面的启动参数

````
Darkwake=no
````

## 允许运行任意来源的app 
- Allow from Anywhere (Turns off the Gatekeeper checks)

````
sudo spctl --master-disable
````

## 休眠修复 (不好用,建议直接关闭休眠)


- 当点击系统的sleep Or 睡眠后, 或者系统自动在 N分钟睡眠 [pmset -g | grep sleep (得到 sleep N , N的单位为分钟)]后,

- macOS 默认 8小时进入深层次的睡眠, 参考:

    `pmset -g | grep autopoweroffdelay 数值28800秒 (也就是8小时)`

- 然后 24小时进入休眠, 参考 

    `pmset -g | grep standbydelay 数值 86400秒 (也就是24小时)`

- 休眠会将RAM的文件写入硬盘.

- 通常情况下在睡眠后的24小时之前唤醒是从内存直接唤醒, 一般是没什么问题的, 但是在此之后, 唤醒就会从硬盘的休眠文件唤醒, 会出现问题, 解决办法

    `无`

- 直接禁用休眠:

    `sudo pmset -a hibernatemode 0 ; sudo pmset -a standby 0 ; sudo pmset -a autopoweroff 0`

- 测试
    - 如果你等不了24小时, 那么测试办法: 
    ````    
    sudo pmset -a standbydelaylow 0
    sudo pmset -a standbydelayhigh 0
    sudo pmset -a autopoweroffdelay 0
    ````
    - 然后点睡眠(中文),或者Sleep (英文). 如果唤醒成功, 表示测试成功.

    - 恢复macOS默认的pm策略, 去设置-Energy Saver (节能)- Restore Default


## 重置文件的大小
- 比如一些log文件

````
gtruncate -s 0 v2ray.log // 将file的大小设置为0, or truncate on linux
````

## 删除小老鼠

````
sudo xattr -rc
````

## 解除proc and file limit

````
/etc/sysctl.conf
kern.maxfiles=5242880
kern.maxfilesperproc=5242880

sudo chown root:wheel limit.maxfiles.plist limit.maxproc.plist
sudo cp limit.maxfiles.plist limit.maxproc.plist /Library/LaunchDaemons/
sudo launchctl load -w /Library/LaunchDaemons/limit.maxproc.plist
sudo launchctl load -w /Library/LaunchDaemons/limit.maxfiles.plist
````
## aria2 webui 设置语言为中文
````
vim webui-aria2/docs/app.js
  //.determinePreferredLanguage(),
              .preferredLanguage('zh_CN'),
````

## 关闭Realtek网卡的EEE

````
Support for Energy Efficient Ethernet (EEE) which can be disabled by setting enableEEE to NO in the drivers Info.plist without rebuild. The default is YES.
````

## SIP 建议设置 0x3, 允许安装未签名的kexts
````
Relevant user options for SIP:

csr-active-config 0x0 = SIP Enabled (Default)
csr-active-config 0x3 = SIP Partially Disabled (Loads unsigned kexts)
csr-active-config 0x67 = SIP Disabled completely
````

## Sleep秒唤醒解决办法
- 蓝牙的USB端口设置问题

````
蓝牙的USB自定义端口必须是255，否则sleep会秒唤醒
````

## chrome 多线程下载

````
chrome://flags/
搜索download,
开启 Parallel downloading 为 enable
````

## 在B360M-HDV 下面有待解决的问题:

- IKBC 键盘 以及 hwmonitor3 导致iojones和 ioregistryexplorer 非常慢,甚至闪退 // hwmonitor3  作者说这是正常现象.
- 蓝牙 无法唤醒电脑 (蓝牙供电走PCI-E), 鼠标唤醒后, 蓝牙并不是马上连接, 有几秒的延迟.
- Wi-Fi 在电脑唤醒后需要几秒才会连接 (Wi-Fi也是走PCI-E) // 这个并不是问题. 
- 如果打开 网络唤醒 功能, Wi-Fi在唤醒后掉速度.

````
初步分析认为: 电脑在Sleep后, PCI-E接口供电丢失. 具体的PCI-E接口为: RP12@1D
改名 _STA 为 XSTA , 并不管用.
00001420 5F535441 00414442            00001420 58535441 00414442
0000141A 5F535441 00414442            0000141A 58535441 00414442
````

## 解决华擎主板新BIOS无法启动macOS

BIOS 3.2,4.0, 4.1

- 方法 1 (Justin自己摸索的方法):  

````
find: 79001415 5F535441 00A00A93  replace: 79001415 58535441 00A00A93 // 原理是将 RTC设备 PNP0B00 的 _STA 改名为 XSTA, 让RTC的Status失效. 
````
- 方法2 (网络上的方法) :

````
Find: A00A93 53544153 01 Replace: A00A91 0AFF0BFF FF // 将 RTC 设备 PNP0B00 的 _STA  If ((STAS == One)) 改为  If ((0xFF || 0xFFFF))
````
- 方法3 (tgtbridge)

> 此方法是用clover的tgtbridge, 精确的定位到 Device (RTC), 然后在这个下面搜寻 \_STA 改名为 XSTA, 由于RTC只有3位数, 需要下划线补全, 也就是 RTC_ , tgtbrigde这边填写RTC_的hex, 也就是5254435F, 所以具体的就是

````
Comment: Rename RTC._STA to XSTA(Tgt RTC is RTC_ for 4bit)
Find: 5F535441 (hex of _STA )
Replace: 58535441 (hex of XSTA)
TgtBridge: 5254435F
````

- 方法4 设置STAS=1  
> 对于 BIOS 4.1 用modified grub 启动电脑, 运行

````
setup_var 0x582 0x1 
````

- 方法5 SSDT 设置 STAS=1

````
DefinitionBlock ("", "SSDT", 1, "HACK", "SET-STAS", 0x00000000)
{
    External (STAS, IntObj)

    Scope (_SB)
    {
        Method (_INI, 0, NotSerialized)  // _INI: Initialize
        If (_OSI ("Darwin"))
        {
            STAS = One
        }
    }
}
````

- 题外话, 如果要设置多个_INI参数, 只需要合并, 注意每个scope下面只可以有一个_INI

````
DefinitionBlock ("", "SSDT", 2, "HACK", "SET-STAS", 0x00000000)
{
    External (GPHD, IntObj)
    External (STAS, IntObj)

    Scope (_SB)
    {
        Method (_INI, 0, NotSerialized)  // _INI: Initialize
        {
            If (_OSI ("Darwin"))
            {
                STAS = One
                GPHD = Zero
            }
        }
    }
}
````

## DSDT 细节 

> 具体方法, 首先我们的目标是把 _STA 改为 XSTA 让RTC设备的 Status失效, 但是DSDT里面有很多 _STA, 我们用 Hex Fiend 打开 DSDT.aml, 搜索txt _STA, 从右侧的乱码中基本上可以看到些许的东西,这部分与 DSDT.dsl对比下, 基本上可以定位 RTC的 _STA的位置, _STA 的 Hex 是 5F535441, 确定了位置之后, 为了保证这个_STA唯一, 我们在找到它的前后的HEX代码, 前面是79001415 后面是 00A00A93, 这样整串就是 79001415 5F535441 00A00A93, 为了100% 确保, 用 Hex Fiend 再次搜索 79001415 5F535441 00A00A93, 如果只搜到一个, 那么唯一.  然后我们只需要替换 5F535441 为 58535441 (也就是XSTA的hex), 所以这个是方法1的原理.

> 如果在定位_STA 或者其他东西的时候的时候遇到了困难, 我们可以先将DSDT的_STA改为XSTA(XSTA在DSDT是唯一的), 然后 Hex Fiend打开 DSDT, 搜索 XSTA,定位到XSTA前后的 HEX代码, 记录下来, 一般来说前面 8个bit后面8个bit就可以精确定位.  


## DSDT find repl 实例 (BIOS更新大部分情况下不会失效)

> 举例: 如何将 Return (0x0F) 改为  Return (0x0B) 

- 先保存一份 Return (0x0F) 的DSDT 比如 0F.aml

- 然后修改  Return (0x0F) 为  Return (0x0B), 保存为0B.aml

- 然后 Hex Fiend 打开两者, 做比较, 会提示不同点在0F和0B, 取包含他们的4bytes左右的HEX代码 (PMheart说, 太短会不唯一, 太长会搜不到) 一般情况下取4bytes的HEX代码比较合适, 目的是保证find 和 replace都唯一.
- Clover或者 Opencore中做find repl
    ````
    0F.aml 0A0FA103

    0B.aml 0A0BA103
    ````


##   opencore 的 Bitmasking

> 簡單講就是 需要 precise matching 的 放 FF , fuzzy 就 00 (PMheart 语录)

举例:
find
01 02 03 04 xx xx 0A 0B 0C
find mask
FF FF FF FF 00 00 FF FF FF

具体例子, 比如 有的bios 搜出来是 0A0FA103 有的 是 0C0FA103 , 前面2位不同, 那么 bitmask就是 00FFFFFF

https://github.com/acidanthera/bugtracker/issues/365
   
##  ACPI error 查看方法 
    ````
    log show --predicate "processID=0" --last boot | grep -i "ACPI" | grep -i -C5 error
    ````

## 修复设置错误的显示器配置

````
cd ~/Library/Preferences/ByHost/
rm -f com.apple.windowserver.*
rm -f com.apple.preference.displays*
````

## Grub mod BIOS

- USB disk, GUID, msFAT
- Tree:
````
    EFI/
        BOOT/
            bootx64.efi (renamed from grub mod)
````
- Commands:

    `setup_var [setup_var_2,setup_var_3] offset value `


## using rdisk for dd 速递快很多

````
sudo dd if=/Users/jst/Downloads/archlinux-2019.04.01-x86_64.iso of=/dev/rdisk4 bs=4m
````
##  隐藏 hide 不自动 mount 磁盘

- 找分区的UUID

    `diskutil info /Volumes/Toshiba | grep -E "Volume UUID" | cut -d ":" -f2 | xargs`
- 编辑fstab

    `sudo vifs` 

- 填入内容 (UUID为举例)

    `UUID=2EA3EF0A-0819-4AF9-B1E9-84E99A619241 none auto rw,noauto`

## OpenCore 相关

- 如果加载了 VboxHFS.efi, 那么无法启动Recovery分区.
https://github.com/acidanthera/bugtracker/issues/399
    - Update: 实际上可以启动, 但是需要花3分钟的时间才开始加载Recovery

- 编译最新的opencore, 无需安装Xcode

```sh
git clone https://github.com/acidanthera/OpenCorePkg 
cd OpenCorePkg
./macbuild.tool
cp Binaries/RELEASE/OpenCore-*-RELEASE.zip ~/Desktop
```
- 编译 VariableRuntimeDxe.efi

```sh
git clone https://github.com/acidanthera/audk UDK
cd UDK
source edksetup.sh
make -C BaseTools
build -a X64 -b RELEASE -t XCODE5 -p MdeModulePkg/MdeModulePkg.dsc -m MdeModulePkg/Universal/Variable/RuntimeDxe/VariableRuntimeDxe.inf --pcd=PcdEmuVariableNvModeEnable=1﻿
cp Build/MdeModule/RELEASE_XCODE5/X64/VariableRuntimeDxe.efi ~/Desktop/
```
- 如何开启OpenCore的debug模式

    - 使用debug版本的 opencore.efi, bootx64.efi 以及其他的debug版的 efi
    - 参数设置 (Number)
        ````
        Target=67 (0x1 + 0x2 + 0x40 的hex转换)
        DisplayLevel=2147483714 (0x00000002 + 0x00000040 0x80000000 的hex转换)
        HaltLevel=2147483648 (0x80000000 的hex转换)
        ````

- Kernel Panic 的 Debug 

    - 启动参数的设置
    ````
    -v debug=0x100 keepsyms=1 (遇到KP的时候, 系统不自动重启
    ````
    - config.plist的设置
    ````
    PanicNoKextDump=YES, debug不显示 kext的dump, 这样可以看到backtrace
    ````


## 磁盘修复

````
不需要进入recovery,
sudo diskutil verifydisk disk0
sudo diskutil repairdisk disk0
````

## USB limit patch for 10.15 Catalina

````
com.apple.iokit.IOUSBHostFamily
Find: 83FB0F0F
Replace: 83FB3F0F
````
````
com.apple.driver.usb.AppleUSBXHCI
Find: 83F90F0F
Replace: 83F93F0F
````

## 查看macOS installer 的版本

````
grep -A1 version /Volumes/Install\ macOS\ Mojave/Install\ macOS\ Mojave.app/Contents/SharedSupport/InstallInfo.plist
````