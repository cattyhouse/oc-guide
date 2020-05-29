# 解决网络唤醒Wi-Fi问题

## 概述

> 在开启网路唤醒的情况下, macOS在睡眠唤醒后, Wi-Fi变得非常慢, 具体表现为
- ping 路由器的延迟高达 50ms~500ms
    - ping 路由器的ip地址 -S Wi-Fi的ip地址
- Wi-Fi的速度非常慢, 低至 5Mbps
    - 局域网另外一台设备开 iperf3 服务, 本地用 `iperf3 -c iperf3服务器ip地址 -p 端口 -B Wi-Fi的ip地址` 测试

## 原因

> 在开启网络唤醒的情况下, macOS 睡眠后, 需要保持 Wi-Fi 的某种状态, 以便局域网的设备可以通过 Wi-Fi 唤醒 macOS. 但是在 macOS 唤醒后, 这种状态需要切换成 Wi-Fi on 的状态, 这个过程中, 某种未知原因导致 Wi-Fi 延迟和掉速

## 解决思路

> 根据以上的原因的猜测, 一个很直接的解决思路就是睡眠前关闭 Wi-Fi, 唤醒后, 再打开 Wi-Fi

> 先手动测试一遍, 发现这个思路是可行的. 那么接下来就是如何自动实现

## 解决方法

### 工具

- 关闭 Wi-Fi 的命令 ` networksetup -setairportpower en1 off`, 通常有线网络是 en0, Wi-Fi 是 en1, 查看具体名称, OPT+鼠标点击Wi-Fi图标, 获取到这个`网卡名称`

- 打开 Wi-Fi 的命令 ` networksetup -setairportpower en1 on`

- 睡眠和唤醒自动执行脚本的程序, [SleepWatcher](https://www.bernhard-baehr.de)

### 步骤

1. 下载 SleepWatcher, [官网](https://www.bernhard-baehr.de), [本文备份](sleepwatcher_2.2.1.tar), 并解压

1. 删除讨厌的安全印记:

    ````
    xattr -d -r com.apple.quarantine ~/Downloads/sleepwatcher_2.2.1/sleepwatcher
    ````
1. 复制 SleepWatcher 到可执行目录

    ````
    cp -af ~/Downloads/sleepwatcher_2.2.1/sleepwatcher /usr/local/sbin/
    ````

1. 创建 .sleep 和 .wakeup 文件

    ````
    echo 'networksetup -setairportpower en1 off' > ~/.sleep
    echo 'sleep 5' > ~/.wakeup
    echo 'networksetup -setairportpower en1 on' >> ~/.wakeup

    chmod +x ~/.sleep
    chmod +x ~/.wakeup
    ````
1. 测试

    ````
    - 终端执行下面的命令
    /usr/local/sbin/sleepwatcher  -V -s ~/.sleep -w ~/.wakeup
    - 开启网络唤醒
    - 将 macOS 睡眠
    - 唤醒 macOS
    - 测试 Wi-Fi 的 ping 和 iperf3, 前文所述
    ````

1. 开机启动

    ````
    cp -af ~/Downloads/sleepwatcher_2.2.1/config/de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist ~/Library/LaunchAgents/

    launchctl load -w ~/Library/LaunchAgents/de.bernhard-baehr.sleepwatcher-20compatibility-localuser.plist

    # 查看是否运行成功
    ps aux | grep sleepwatcher
    ````