---
title: "升级后的系统是macOS 10.15 (Catalina) ，安装cartopy"
author: "Kai Zheng"
tags: ["Cartopy"]
date: 2021-03-10T11:15:31+08:00
draft: true
---




```diff
报错内容：No such file or directory “limits.h”
```


```diff
csrutil disable # 需要在恢复模式下运行命令，具体请自行搜索。
$ xcode-select --install # 安装常用开发工具，如：git等。
$ sudo mount -uw / # 根目录挂载为可读写，否则无法在/usr/下建立文件，本修改重启前有效。
$ sudo ln -s "$(xcrun --show-sdk-path)/usr/include" /usr/include
$ export SDKROOT="$(xcrun --show-sdk-path)" # 设置环境变量
$ echo "export SDKROOT=\"\$(xcrun --show-sdk-path)\"" >> ~/.bash_profile # zsh的自行搞定
$ sudo DevToolsSecurity -enable # 将系统置于开发模式
```

Reference：https://zhile.io/2018/09/26/macOS-10.14-install-sdk-headers.html