# Chromium-iOS
Github Action的[runner](https://docs.github.com/en/enterprise-cloud@latest/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories)硬盘只有14G,很难编出工件,这里提供一个实机编译方法

## 配置Xcode
`Xcode`的版本需要比编译目标版本高,可以在[ios_sdk_overrides.gni](https://chromium.googlesource.com/chromium/src.git/+/refs/heads/main/build/config/ios/ios_sdk_overrides.gni)看到目前主线开启`blink`需要`ios26.0`,也就是[Xcode 26.0](https://developer.apple.com/services-account/download?path=/Developer_Tools/Xcode_26/Xcode_26_Universal.xip)或更高版本

打开(解压)下载的`.xip`文件,将`Xcode.app`移到`应用程序`中

打开`Xcode`,`Agree`用户协议,输入密码,勾选`IOS 26.0`,`Download & Install`,等待sdk完成安装

## 配置证书

在`Xcode`的菜单栏选择`Settings`,转到`Apple Accounts`选项卡,`Add Apple Account...`

登入后,进入账号,选择你的团队(默认是`Personal Team`),点`Manage Certificates`,点左下角`+`号申请`Apple Development`证书

打开`钥匙串访问.app`,找到刚刚申请的证书,双击打开,检查证书状态

如有`证书不受信任`,需要前往[Apple PKI](https://www.apple.com/certificateauthority)下载`签发者名称`中的对应证书

完成导入后,Apple Development 证书状态应为`此证书有效`

>[!IMPORTANT]
>申请`Apple Development`证书后,需要吊销证书才能重新申请,前往[Apple Developer](https://developer.apple.com/account/resources)吊销证书(仅限付费开发者)

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
>将`[目标版本号]`改成需要的版本号,在[chromium src refs](https://chromium.googlesource.com/chromium/src.git/+refs)查询版本号

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

设置构建
```shell
echo '[gn_args]\nuse_blink = true' >~/.setup-gn
~/chromium/src/ios/build/tools/setup-gn.py
```
>占用约`2G`

## 自动签名

在`Xcode`中打开项目`~/chromium/src/out/build/all.xcodeproj`,按图中顺序选取选项,在选择团队后,`Xcode`会自动注册包名

<img width="1512" height="902" alt="截屏" src="https://github.com/user-attachments/assets/07e8b4e9-96eb-49b4-af65-12bcfd37fd08" />


## 开始编译
```shell
autoninja -C out/Release-iphoneos chrome
```
>cpu核心越多,编译越快

<details>
<summary>lld报错</summary>

不使用工具链中的lld,重新设置构建
```shell
echo '\nuse_lld = false' >~/.setup-gn
~/chromium/src/ios/build/tools/setup-gn.py
```

</details>

## 打包ipa
```shell
mkdir Payload
mv ~/chromium/src/out/Release-iphoneos/Chromium.app Payload
zip -r chromium.ipa Payload
```
