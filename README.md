# Chromium-iOS
github action的runner硬盘只有14G,很难编出工件,这里提供一个实机编译方法

## 设置Xcode
下载[Xcode 26.3](https://developer.apple.com/services-account/download?path=/Developer_Tools/Xcode_26.3/Xcode_26.3_Universal.xip)
>需要登录apple id

打开(解压)下载的xip文件,将`Xcode.app`移到`应用程序`中

使用以下命令查看并同意协议
```shell
sudo xcodebuild -license
```
>输入密码,按回车查看协议,输入`agree`同意协议

首次运行Xcode以完成安装
```shell
xcodebuild -runFirstLaunch
```

下载ios sdk
```shell
xcodebuild -downloadPlatform ios
```
>root用户环境与普通用户不一样,Xcode会找不到sdk

## 配置工具链
拉取工具链仓库
```shell
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git ~/depot_tools
```
>下载约`75M`,占用约`87M`

添加工具链到环境变量
```shell
export PATH="$HOME/depot_tools:$HOME/depot_tools/python-bin:$PATH"
```

## 拉取chromium
创建文件夹并进入
```shell
mkdir -p ~/chromium/src/out/Release-iphoneos/Payload
cd ~/chromium
```

拉取ios版
```shell
fetch --no-history ios
```
>下载约`8G`,占用约`28G`

<details>
<summary>拉取特定版本</summary>

手动fetch
```shell
gclient config --spec 'solutions = [    
  {
    "name": "src",
    "url": "https://chromium.googlesource.com/chromium/src.git@[]",
    "managed": False,
    "custom_deps": {},
    "custom_vars": {},
  },
]
target_os = ["ios"]
target_os_only = "True"
'
```

</details>

## 生成编译目标
进入源码目录
```shell
cd ~/chromium/src
```

同步定义
```shell
gclient sync
```
>下载约`700M`

生成ninja文件
```shell
gn gen out/Release-iphoneos --args='is_debug=false target_os="ios" ios_enable_code_signing=false is_component_build=false target_environment="device" target_cpu="arm64" use_lld=false use_blink=true'
```
>占用约`2G`

## 开始编译
```shell
autoninja -C out/Release-iphoneos chrome
```
>分配的cpu核心越多越快

## 打包ipa
```shell
mv ~/chromium/src/out/Release-iphoneos/Chromium.app ~/chromium/src/out/Release-iphoneos/Payload
zip -r ~/chromium.ipa ~/chromium/src/out/Release-iphoneos/Payload
```
>
