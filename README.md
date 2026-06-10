# Chromium-iOS
github action的runner硬盘只有14G,很难编出工件,这里提供一个实机编译方法

## 设置Xcode
下载[Xcode 26.3](https://developer.apple.com/services-account/download?path=/Developer_Tools/Xcode_26.3/Xcode_26.3_Universal.xip)

使用以下命令查看并同意协议
```shell
sudo xcodebuild -license
```
>回车展示协议,输入`agree`同意协议

首次运行Xcode以完成安装
```shell
xcodebuild -runFirstLaunch
```

下载ios sdk
```shell
xcodebuild -downloadPlatform ios
```
>root用户环境与普通用户不一样,xcode会找不到sdk

## 配置工具链
拉取工具链仓库
```shell
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git ~/depot_tools
```

添加工具链到环境变量
```shell
export PATH="/Users/[用户名]/depot_tools:$PATH"
```
>将`[用户名]`换成mac os用户名

## 拉取chromium
创建文件夹并进入
```shell
mkdir -p ~/chromium/src/out/Release-iphoneos
cd ~/chromium
```

拉取ios分支
```shell
fetch --no-history ios
```

## 生成编译目标
进入源码目录
```shell
cd ~/chromium/src
```

同步定义
```shell
gclient sync
```

生成ninja文件
```shell
gn gen out/Release-iphoneos --args='is_debug=false target_os="ios" ios_enable_code_signing=false is_component_build=false target_environment="device" target_cpu="arm64" use_blink=true'
```

## 开始编译
```shell
autoninja -C out/Release-iphoneos chrome
```
>根据核心数,编译时间会有不同,1个核心会分配512个线程
