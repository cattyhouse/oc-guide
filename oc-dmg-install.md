# OpenCore从Recovery的DMG安装macOS各种版本
## 优点
- 无需从App Store下载macOS安装文件
- 对操作平台没有要求, 只要能运行php命令就可以, 所以可以在Linux/Windows/macOS 甚至在Linux服务器上操作.
- 下载量非常小, Recovery只有400~500M
- 支持各种版本的macOS的安装,解决了旧版本macOS很难获取的问题

## 缺点
- 需要知道如何配置OpenCore, 要提前配置好. 其实不算缺点, 因为就算是用制作启动盘的方法, 依然是要配置好OpenCore才可以启动电脑.
- 进入到了安装界面之后, 需要联网, 后面的安装过程需要从Apple的服务器获取所需文件, 通常都会跑满你的带宽, 所以其实并不算一个缺点.

## 步骤
1. [配置好OpenCore的EFI](https://github.com/cattyhouse/oc-guide)
2. 下载Recovery的DMG
    1. 获取[php脚本](https://raw.githubusercontent.com/acidanthera/OcSupportPkg/master/Utilities/Recovery/obtain_recovery.php)
    1. 获取下载DMG的命令的[参考文件](https://raw.githubusercontent.com/acidanthera/OcSupportPkg/master/Utilities/Recovery/recovery_urls.txt)
    1. 以Linux/macOS平台下面, 下载Mojave的DMG为例子, 进入obtain_recovery.php所在的文件夹,参考recovery_urls.txt 运行:
    
        `php obtain_recovery.php 00000000000KXPG00 Mac-7BA5B2DFE22DDD8C`

    1. 会下载2个文件, 分别为 **RecoveryImage.dmg** 和 **RecoveryImage.chunklist**

3. 准备好一个容量1GB以上的U盘, 格式化成 **Fat32** 格式.
4. 将OpenCore的EFI复制到U盘的 **根目录**
5. 在U盘的 **根目录** 创建一个文件夹名字叫做: **`com.apple.recovery.boot`**
6. 将 **RecoveryImage.dmg** 和 **RecoveryImage.chunklist** 复制到 **`com.apple.recovery.boot`** 里面
7. 给启动文件命名
    1. 在文件夹 **`com.apple.recovery.boot`** 下面新建一个隐藏文件 **`.contentDetails`**
    1. 内容可以随便写, 比如 `Mojave Boot From Recovery`, 这个名字会出现在OpenCore的启动菜单上.
8. 最终U盘上的文件结构看起来像这样:

    ````
    EFI
        ├── BOOT
        ├── OC
        └── Tools
    com.apple.recovery.boot
        ├── RecoveryImage.chunklist
        └── RecoveryImage.dmg
        └── .contentDetails
    ````
9. 用U盘启动电脑选择 `Mojave Boot From Recovery`, 接下来就进入正常的安装界面