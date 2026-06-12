# Chromium-iOS
Github Action的[runner](https://docs.github.com/en/enterprise-cloud@latest/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories)硬盘只有14G,很难编出工件,这里提供一个实机编译方法

## 设置Xcode
`Xcode`的版本需要比编译目标版本高,可以在[ios_sdk_overrides.gni](https://chromium.googlesource.com/chromium/src.git/+/refs/heads/main/build/config/ios/ios_sdk_overrides.gni)看到目前主线开启`blink`需要`ios26.0`,也就是[Xcode 26.0](https://developer.apple.com/services-account/download?path=/Developer_Tools/Xcode_26/Xcode_26_Universal.xip)或更高版本

打开(解压)下载的`.xip`文件,将`Xcode.app`移到`应用程序`中

查看并同意协议
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
>`root用户`环境与`普通用户`不一样,`Xcode`会找不到sdk

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
mkdir -p ~/chromium/src
cd ~/chromium
```

拉取`ios`主线
```shell
fetch --no-history ios
```
>下载约`8G`,占用约`28G`

<details>
<summary>按版本号拉取</summary>

```shell
gclient config --spec 'solutions = [    
  {
    "name": "src",
    "url": "https://chromium.googlesource.com/chromium/src.git@[目标版本号]",
    "managed": False,
    "custom_deps": {},
    "custom_vars": {},
  },
]
target_os = ["ios"]
target_os_only = "True"
'
gclient sync --no-history
```
>将`[目标版本号]`改成需要的版本号,在[这里](https://chromium.googlesource.com/chromium/src.git/+refs)查询版本号

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

生成`args.gn`以确立编译目标
```shell
gn gen out/Release-iphoneos --args='is_debug=false target_os="ios" ios_enable_code_signing=false is_component_build=false target_environment="device" target_cpu="arm64" use_blink=true'
```
>占用约`2G`

## 开始编译
```shell
autoninja -C out/Release-iphoneos chrome
```
>分配的cpu核心越多,编译越快

<details>
<summary>lld报错</summary>

不使用工具链中的lld,重新确立编译目标
```shell
gn gen out/Release-iphoneos --args='is_debug=false target_os="ios" ios_enable_code_signing=false is_component_build=false target_environment="device" target_cpu="arm64" use_lld=false use_blink=true'
```

</details>

## 打包ipa
```shell
mkdir Payload
mv ~/chromium/src/out/Release-iphoneos/Chromium.app Payload
zip -r chromium.ipa Payload
```
>未签名,需要侧载
