
# 更新记录
- 最近更新: 2019.12.17
- 本文最开始写的时候是 OpenCore 0.03 版本, 到现在已经变化了不少.
- 建议下载最新的 OpenCore 开始, 并留意我的注释
- OpenCore 0.04 版本开始, 有比较大的变化:
    - AptioMemoryFix.efi 已经弃用, 功能移到了 OpenCore/config.plist/Booter 
    - FwRuntimeServices.efi 取代了 VariableRunEmutimeDxe.efi 和 EmuVariableRuntimeDxe.efi. 
    - 但总体结构上保持一致.
- OpenCore 0.5 版本开始公测
    - 尝试模拟白苹果行为, 比如按住CMD+R的同时开机, 进入恢复模式, 等等.

# 已知问题和解决方法

## iGPU 在解码编码过程中频率不超过 0.5 Ghz. 相关 [issue](https://github.com/acidanthera/bugtracker/issues/546)
>  针对 iMac19,2 和 iMac19,1 的 SMBIOS. 

> 总结: 如果有 AMD 独立显卡, 直接使用 iMacPro1,1, 关闭 iGPU, 一了百了.
- 原因: WhateverGreen 屏蔽了 Intel GuC 的加载.
- 解决方法:
    > 如果用 iMacPro1,1 就没必要采用下面的操作, 因为 iGPU 已经关闭不起作用.

    1. Disable 掉 WhateverGreen.kext 的加载
    1. 加入 AGDP patch, 在 **config.plist/Kernel/Patch**

        ````
        Identifier: com.apple.driver.AppleGraphicsDevicePolicy
        Find: 62 6f 61 72 64 2d 69 64
        Replace: 62 6f 61 72 64 2d 69 78
        Comment: Ranem board-id to board-ix
        Count: 1
        ````
    1. ACPI 设备重命名, 在 **config.plist/ACPI/Patch**
        - GFX0 to IGPU, 因为 BIOS 的集成显卡叫做 GFX0, macOS 需要它叫做 iGPU
            ````
            Comment: Rename GFX0 to IGPU
            Find: 47 46 58 30
            Replace: 49 47 50 55
            Count: 0
            TableSignature: 0
            ````
        - PEGP to GFX0, 因为BIOS的独立显卡叫做PEGP, macOS需要它叫做GFX0
            ````
            Comment: Rename PEGP to GFX0
            Find: 50 45 47 50
            Replace: 47 46 58 30
            Count: 0
            TableSignature: 0
            ````
## iGPU 无法播放 Apple TV 里面的电视剧和电影, 相关 [issue1](https://github.com/acidanthera/bugtracker/issues/519), [issue2](https://github.com/acidanthera/bugtracker/issues/582)
>  针对 iMac19,2 和 iMac19,1 的 SMBIOS. 

> 总结: 如果有 AMD 独立显卡, 直接使用 iMacPro1,1, 关闭 iGPU, 一了百了.
- 原因: 因为没有 Apple Firmware, 黑苹果上的 iGPU 无法硬件解码 Apple TV 的 DRM 内容
- 解决方法 (方法三最好): 
    - 方法一: BIOS 关闭 iGPU, SMBIOS 采用 iMacPro1,1
    - 方法二: 无需关闭 iGPU, SMBIOS 依旧采用可爱的 iMac19,2 iMac19,1 加入启动参数 **shikigva=32 shiki-id=Mac-7BA5B2D9E42DDD94**, 这个目的就是让 AppleGVA 用 iMacPro1,1 的方式处理硬件解码和编码,也就是用 AMD的显卡, 但与此同时, iGPU 变成彻底无用了. 因为这个方法是对所有 App 生效
    - 方法三: 等待 WhateverGreen 更新, 计划是将 Apple TV 单独列出来, 让它使用 AMD 的 GPU 去解码 DRM 视频, 这样就不会影响其他 App 使用 iGPU 的解码编码功能. **Update: 1.3.5版 已经更新, 通过 shikigva=16 参数让部分 app 走 AMD GPU 解码. 1.3.6版 则可以用 shikigva=80 更进一步的支持Netflix等**
## 开启网络唤醒后, Wi-Fi ping 延迟在睡眠唤醒后非常高
- 原因: 未知
- 解决方法: 安装 AirportBrcmFixup.kext ( **其中未发布的 2.0.5 版本对10.15 做了支持** ), 这个 kext 默认禁用 wowlan 的功能. 这样不影响 wolan的功能, 这样之后, 设置-节能- wake for network access 变成 wake for ethernet network access.
- 唤醒macOS的方法: linux下面安装一个软件 wol, 例如 `pacman -S wol` , 然后运行 `wol macOS的有线网卡的MAC地址`, 不依赖BIOS的设置, 可以直接唤醒
- 唤醒后测试 Wi-Fi 的延迟的方法: `ping 路由器ip地址 -S Wi-Fi的ip地址` 一般来说 5ms 以内都正常, 几十甚至上百ms 都是不正常的

## 唤醒后蓝牙硬件找不到
- 原因: 转接板设计问题
- 解决方法: 更换转接板. 


# 资源:

1. OpenCore 的官方下载地址: [OpenCore](https://github.com/acidanthera/OpenCorePkg)
2. 与 OpenCore 配合使用的 kexts, efi 下载地址: [kexts](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/Kexts.md)

# Wiki
[wiki](https://github.com/cattyhouse/oc-guide/wiki) 会放一些OpenCore使用过程中的一些经验

# 关于本文
- ***通篇没有废话, 请务必认真读每一个字*** 
- 尽可能用最简单的语言描述OpenCore安装和启动黑苹果.
- 不提供最终的启动文件,而是提供方法.有了正确的方法, 就可以完全掌控自己的电脑.
- 黑苹果用到的所有工具,kexts以及efi文件的链接都指向原作者,可以及时的了解原作者的更新内容, 同时确保看到这篇文章的时候, 使用的是最新.
- 建议通读本文并理解后,再开始黑苹果之旅.

# 黑苹果与白苹果的主要区别
> 2006年乔布斯将苹果电脑使用的处理器从IBM转向Intel之后, 在普通PC上安装macOS的大门才算正式开启, 随着硬件的发展, PC与苹果电脑的差异越来越小, 黑苹果越来越接近苹果电脑, 目前的区别主要在于EFI以及ACPI.
- EFI
    - Bootloader所在的分区, [OpenCore](https://github.com/acidanthera/OpenCorePkg), Clover等都安装在这里. 苹果电脑则是采用私有的bootloader
    - 分区格式为FAT32, 所以Windows, Linux, macOS等等都可以读写这个分区的文件
    - OpenCore可以放在SSD的EFI分区里, 也可以放在FAT32格式的USB盘上, 建议调试阶段使用USB盘,调试完毕复制到SSD的EFI分区

- ACPI
    - ACPI是保存在BIOS里面的描述硬件的一张表
    - PC与苹果电脑的ACPI内容大致一样,但也有不少的区别,这些少量区别足以导致一个原生的macOS安装盘在PC上启动失败.
    - OpenCore作为一个booloader,除了负责启动macOS之外的另一个主要功能就是修改PC的ACPI表兼容macOS

> 所以接下来的内容就非常明了:
- 制作一个 macOS 启动U盘
- 生成一个 OpenCore 的EFI放在U盘, 将 macOS 安装到PC并启动
- 复制U盘上的EFI到安装好 macOS 的 SSD 上, 从 SSD 启动 macOS, 脱离U盘.

# OpenCore的优缺点

## 优点
- 结构非常简单,没有安装文件pkg,跟搭积木一样简单,就算没有任何黑苹果经验, 也很容易上手
- 体积非常小, 只有9k的`BOOTx64.efi`启动文件和~200KB的`OpenCore.efi`主文件
- 速度非常快
- 文档非常完善,每一个配置都有详尽的说明
- 全新的 Bootloader, 没有历史包袱
- 稳定, 非常稳定
- 安全, 可以选择对自身做签名,防止篡改
- 配置文件有严格的一一对应的要求, 知道自己在干什么.

## 缺点
- 目前处于开发阶段, 更新会比较勤, 变动可能会比较大, 不过作者会做详细的变动记录,文档都及时更新

# 从 Recovery DMG 安装 macOS

- 无需从 apple store 下载巨大的安装文件, 无需写盘.
- 此方法源自 OpenCore 的文档, 用 python 脚本从苹果服务器下载 BaseSystem.dmg, OpenCore 可以自动识别这个文件, 所以可以在任意操作系统下面操作.
- 制作 OpenCore EFI 也可以任意操作系统下面操作, 于是现在安装 macOS 不需要事先拥有 macOS 系统了. 
-  [OpenCore从Recovery的DMG安装macOS各种版本](oc-dmg-install.md)

# OpenCore安装与配置

## OpenCore的文件结构

> 首先, 我们看看一个已经配置好的 OpenCore 的文件结构, 在 EFI 分区 (/Volumes/EFI/) 下面, 有一个 EFI 文件夹, 内容如下

```bash

EFI
│   ├── BOOT
│   │   └── BOOTx64.efi
│   └── OC
│       ├── ACPI
│       │   ├── SSDT-PLUG.aml
│       ├── Drivers
│       │   ├── ApfsDriverLoader.efi
│       │   ├── FwRuntimeServices.efi
│       │   ├── HFSPlus.efi
│       ├── Kexts
│       │   ├── AppleALC.kext
│       │   ├── IntelMausi.kext
│       │   ├── Lilu.kext
│       │   ├── SMCProcessor.kext
│       │   ├── SMCSuperIO.kext
│       │   ├── VirtualSMC.kext
│       │   └── WhateverGreen.kext
│       ├── OpenCore.efi
│       ├── Tools
│       │   └── Shell.efi
│       └── config.plist

```

> 简单说明如下, 蓝色高亮的是下载链接, 都是指向原作者的最新版.

- `EFI/BOOT/BOOTx64.efi` 电脑启动的时候支持UEFI的主板会去读取这个文件
- `EFI/OC` OpenCore 存放目录
    - `ACPI` 存放自定义的ssdt aml文件, 比如 (aml为二进制文件, 是我们需要的最终文件. dsl为源文件, 需要用 [MaciASL](https://github.com/acidanthera/MaciASL/releases)打开另存为aml.)
        - [`SSDT-PLUG.aml`](EFI/OC/ACPI/SSDT-PLUG.aml) 开启硬件变频功能, 作用于CPU, iGPU, dGPU. 注意此文件里面的 CPU 的位置和名称需要与 DSDT.aml 里面的一致, 可能的位置: _SB _PR 等, 可能的名称: PR00 CPU0 等, 不同的主板各不相同.
        - [`SSDT-AWAC.dsl`](https://github.com/acidanthera/OpenCorePkg/tree/master/Docs/AcpiSamples) 300系列的主板用最近更新的 BIOS 后, RTC 失效, 这个 SSDT 作用是启用 RTC, 需要自行编译为 aml
        - [`SSDT-EC-USBX.dsl`](https://github.com/acidanthera/OpenCorePkg/tree/master/Docs/AcpiSamples) Fake EC 和 USBX, 给 macOS 提供一个虚假的EC设备, 同时提供 USB 大电流支持, 需要自行编译为 aml
    - `Drivers` 存放文件系统驱动文件, 比如
        - [`ApfsDriverLoader.efi`](https://github.com/acidanthera/AppleSupportPkg/releases) 用于加载 macOS 内置的 apfs.efi , 读取APFS分区, **必要文件**
        - [`FwRuntimeServices.efi`](https://github.com/acidanthera/AppleSupportPkg/releases) 提供模拟 nvram 等其他功能,  **必要文件**
        - [`HFSPlus.efi`](EFI/OC/Drivers/HFSPlus.efi) 提供HFS+文件系统的支持, 读取macOS的安装U盘以及Recovery分区需要此efi,  **必要文件**
    - `Kexts` 存放各种设备和硬件的驱动或者补丁.
        - [`Lilu.kext`](https://github.com/acidanthera/Lilu/releases) 一个框架式的kext,自身单独使用没有作用, 是其他kext的依赖, 必须第一个被加载.  **必要文件**
        - [`AppleALC.kext`](https://github.com/acidanthera/AppleALC/releases) 让macOS可以正确识别大部分主板上的集成声卡
        - [`VirtualSMC.kext`](https://github.com/acidanthera/VirtualSMC/releases) 模拟SMC,  **必要文件**
        - [`WhateverGreen.kext`](https://github.com/acidanthera/WhateverGreen/releases) 解决集成/独立显卡的各种问题  **必要文件**
        - 其他的几个kext, 根据需要使用, 比如: 
        - [`IntelMausi.kext`](https://github.com/acidanthera/IntelMausi/releases), Intel 有线网卡驱动. 
        - [`RealtekR1000SL.kext`](https://github.com/SergeySlice/RealtekLANv3/releases), Realtek 有线网卡驱动
        - [`SMCProcessor.kext SMCSuperIO.kext`](https://github.com/acidanthera/VirtualSMC/releases) 让macOS下的监控软件可以读取主板上的传感器信息温度,频率等
    - `Tools` 工具类efi, 这些工具在 OpenCore 启动界面可以看到, 目前只有下面2个工具, **不可以放入Drivers文件夹**
        - ~~`CleanNvram.efi`~~ 作用为清空nvram, 新版本 OpenCore 已经用内置的选项 AllowNvramReset=YES 取代这个 efi, 等效于启动到macOS恢复模式之后, 运行 `nvram -c`
        - [`Shell.efi`](https://github.com/acidanthera/OpenCoreShell/releases) 一个修改版的 `UEFI SHELL`, 可以做很多有趣的事情.
        - [`VerifyMsrE2`](https://github.com/acidanthera/AppleSupportPkg/releases), 检查主板是否有 CFG LOCK
    - `OpenCore.efi` OpenCore的主引导文件, 体积约200KB
    - `config.plist` OpenCore的主要配置文件, 可以使用PlistEdit Pro 或者 Xcode 可视化编辑.

> 所以, OC下面总共有4个文件夹, 1个主文件, 和一个配置文件, 是不是非常简洁

## 搭积木 - 从零开始组建OpenCore

> 相信看完 [OpenCore的文件结构](https://github.com/cattyhouse/oc-guide#opencore的文件结构), 心里已经有底了, 我们从 0 开始玩, 以下终端操作, 当然也可以在 Finder 里面鼠标操作, 结果是一样.

- 我们在任意地方开始操作, 比如桌面
    ```sh
    cd ~/Desktop
    ```
- 然后建立一个叫做 EFI 的文件夹
    ```sh
    mkdir EFI && cd EFI
    ```
- 然后建立 OpenCore 的文件结构
    ```sh
    mkdir BOOT
    mkdir -p OC/{ACPI,Drivers,Kexts,Tools}
    # 注意大小写
    ```
- 结构建立好了, 接下来将下载好的最新版 OpenCore 解压, 取出需要的文件放入对应的地方:
    - BOOTx64.efi 放入 BOOT 文件夹
    - OpenCore.efi 放入 OC 文件夹
    - Docs 里面的 Sample.plist 命名为 config.plist, 放入 OC 文件夹
    - 将 `OpenCore的文件结构` 中提到的 *.aml, *.kext, *.efi 按自己的需求需复制到对应的 `ACPI , Drivers , Kexts , Tools` 文件夹.
- 积木搭建完毕, 这个 EFI 文件夹, 复制到 FAT32 格式的U盘的根目录, 就可以在BIOS选择U盘引导它, 当然要配置下, 配置见下文.

## 配置config.plist

- config.plist 请使用 PlistEdit Pro 或者 Xcode 进行可视化编辑, 避免出错.
- 无论首次安装或者更新, 请务必以你下载的 OpenCore 里面的 sample.plist 为基础做设置. 对于更新来说, 请自行比较新旧版的 sample.plist 的变化, 然后适配到你的 config.plist 里面去
- 请勿删除 sample.plist 里面的条目, 很可能会破坏结构, 大部分配置都可以设置 YES/NO (PlistEdit Pro) 或者 true/false (Xcode) 来 启用/禁用
- 请勿使用来自 Clover 的 kexts, 他们并不兼容.
- 关于配置, 作者有非常详细的英文文档, 解压 OpenCore 在 `Docs/Configuration.pdf` 如果有兴趣,可以从头到尾看一遍, 由于篇幅有限,我不打算每一个项目都过一遍, 只列出注意事项:

1. 推荐值: 
    - 重要的事情说三遍: **`用U盘做测试 用U盘做测试 用U盘做测试`** 
    - **`AvoidRuntimeDefrag=YES`** , **必要项目**
    - **`DisableVariableWrite=YES`**, 100/200/300系列主板没有nvram的, 需要YES, 如果设置为 NO, 那么表现就是睡眠会自动重启. 通常 **z370** 主板不需要开启此选项.
    - **`EnableWriteUnprotector=YES`**,  **必要项目**
    - **[`SSDT-AWAC.dsl`](https://github.com/acidanthera/OpenCorePkg/blob/master/Docs/AcpiSamples/SSDT-AWAC.dsl)**, 300系列主板, 新版本BIOS必须要的SSDT, 需要编译为 aml 才可以使用. 具体google搜索如何把 dsl 编译为 aml. 加载方法见下面的说明. 如果想了解它的作用, 可以看这里的 [原理分析](https://github.com/cattyhouse/oc-guide/wiki/我的一些黑苹果笔记#解决华擎主板新bios无法启动macos)
    - **`Kernel/Add Lilu.kext`** 必须永远在第一条
    - **`AppleCpuPmCfgLock=YES, AppleXcpmCfgLock=YES, AppleXcpmExtraMsrs=YES`** 如果主板有 CFG LOCK 且无法从 BIOS 里面关掉的话,如果可以 BIOS 关掉 CFG LOCK, 这三个选项都设置为 NO
    - **`PanicNoKextDump=YES`** 启动过程中如果崩溃了, 禁止 Kext Dump, 这样可以看到具体引起崩溃的原因 (backtrace)
    - 启动参数在 **`config.plist/NVRAM/Add/7C436110-AB2A-4BBB-A880-FE41995C9F82/boot-args`** 这里加入或者修改. 建议的启动参数为 `-v alcid=1 keepsyms=1 debug=0x100` , 其中: `-v` 跑码, `alcid=1` 声卡id注入, `keepsyms=1 debug=0x100` 系统崩溃不自动重启, 方便查看崩溃原因.
    - **`XhciPortLimit=YES`**, 取消 macOS 15个 USB 端口的限制
    - **`ConnectDrivers=YES`**, 让 *.efi 文件可以顺利加载
    - **`ConsoleControl=YES`**, 更好的控制opencore菜单
    - **`IgnoreTextInGraphics=YES`**, 修复一些图形界面显示部分文字消息的问题
    - **`ProvideConsoleGop=YES`**, 一般需要为YES, 否则看不到 apple logo
    - **`RequireSignature=NO, RequireVault=NO`**, 关闭 OpenCore 的文件校验功能, 否则启动不了, 等你玩熟悉了可以去玩玩这个功能. **必要项目**
    - **`PollAppleHotKeys=NO`**, 关闭菜单界面的快捷键功能, 这个功能目前兼容性不是很好.
    - **`Automatic=YES`**, 根据 **`Generic`** 里面的信息自动注入 SMBIOS 所需要的其他信息.
    - **`shikigva=16`** , 加入这个启动参数, 实现用 AMD GPU 解码 Apple Tv, 解决无法播放的问题. 

1. EFI下面的每一个 kext, efi, aml, 都必须在config.plist里面有对应的条目, 且设置为`Enabled=YES`, 否则他们不会加载
    - `OC/ACPI/*.aml` 对应 `config.plist/ACPI/Add`
    - `OC/Drivers/*.efi` 对应 `config.plist/UEFI/Drivers`
    - `OC/Kexts/*.kext` 对应 `config.plist/Kernel/Add`
    - `OC/Tools/*.efi` 对应 `config.plist/Misc/Tools`
    - 附图举例, `config.plist/UEFI/Drivers/` 下面配置了5个条目, 要保证 `OC/Drivers/` 下面有这5个efi, 否则启动会提示出错. 同样 `OC/Drivers/` 下面如果有efi没有加入config.plist里面,是不会被加载的. 所以我上面提到的这4个一一对应的地方, 要反复核对, 确保没有疏忽. ![1to1](pics/1to1.png)

1. 如果config.plist里面有条目, 但是OC文件夹下面的子文件夹没有对应的文件, 启动会报错, 所以两者必须是一一对应.条目被设置为`Enabled=NO`,除外.
1. 如果想新增一个条目, 那么可以右键点击已有条目,选择 Duplicate, 然后做相应的修改
1. 新增kext的注意事项
    - 以附图为例![附图](pics/addkext.png)
    - 注意kext里面是否有可执行文件, 如果有, 需要按图填入 `ExecutablePath` 如果没有,这个地方留空.
    - 查看kext的内容, 可以右键点击kext, 然后选择`show package contents`
1. 如果是从Clover过来的, 使用了比如`rename EHC1 to EH01` (此处为举例, 这个 rename 只在很旧的主板上需要), 这样的补丁, 可以将他们添加到config.plist/ACPI/Patch, 并设置Enabled=YES 让其生效. 注意Count=0 表示搜索整个DSDT表,直到搜不到为止, Skip=0 表示从头搜到尾. TableSignature=44534454 表示搜索DSDT表(因为DSDT的hex为44534454), TableSignature=0 表示搜索整个ACPI表, 包括SSDT表.
1. config.plist里面有很多Quirks, 可以理解为作者预设好的补丁, 减轻使用者的负担, 每个Quirks的作用, 可以查阅`Docs/Configuration.pdf`
1. DeviceProperties/Add 里面的参数, 可以设置比如iGPU的`AAPL,ig-platform-id`等等, 具体阅读 [whatevergreen.kext github 页面的文档](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.cn.md)
1. 最后, 这个config.plist是没有序列号等等信息的, 只需要填 PlatformInfo/Generic 里面的5个项目, 可以 [通过macserial生成](https://github.com/acidanthera/MacInfoPkg/releases)
    1. macserial --help 获取帮助
    1. macserial -m iMacPro1,1 示例生成iMacPro1,1的 `SystemSerialNumber` 和 `MLB`
    1. 找个在线生成 UUID 的地方, 可以生成 `SystemUUID`
    1. 找个在线生成 MAC Address 的地方, 生成一个网卡地址, 去掉 `:` 或者 `-`, 转换为大写, 填入 `ROM`
1. 如果选择玩启动项目的数字就卡住, 开了debug显示 [Failed to find first BOOT_MODE_SAFE | BOOT_MODE_ASLR sequence](https://github.com/acidanthera/AptioFixPkg/blob/e33f044fb966045eb37cdf1b978dd67ef3d8d1eb/Platform/AptioMemoryFix/CustomSlide.c#L503) 有可能是MSR寄存器的问题, 可以尝试设置 **`IgnoreInvalidFlexRatio=YES`**

