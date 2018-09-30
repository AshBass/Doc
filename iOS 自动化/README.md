
iOS 自动打包

# 基础操作

## 关于 Xcode 命令

xcodebuild是一个命令行的工具，可以让你的工程通过 projects workspaces 进行编译，测试，分析，打包。他可以运行在包含一个或者多个Target的工程上面，或者在 projects workspaces 包含 scheme 上面。xcodebuild 提供了几个选项，可以在 Main Page 看到这些。默认情况下，xcodebuild会保存和输出在 Xcode 的本地定义的面板里面。

### 常用方法

#### 常用方法一

```
xcodebuild
[-project projectname ] #项目名称
[[-target targetname]...|-alltargets] #target名称
[-configuration configurationname] #configuration名称
[-arch architecture]...
[-sdk [sdkname | sdkpath]] #SDK名称
[-showBuildSettings]
[buildsetting=value]... 
[buildaction]...
```

#### 常用方法二

```
xcodebuild
[-project projectname] #项目名称
-scheme schemeName #scheme名称
[-destination destinationspecifier]... 
[-configuration configurationname] 
[-arch architecture]... 
[-sdk [sdkname | sdkpath]] 
[-showBuildSettings] 
[buildsetting=value]... 
[buildaction]...
```

#### 常用方法三

```
xcodebuild 
-workspace workspacename
-scheme schemeName
[-destination destinationspecifier]... 
[-configuration configurationname] 
[-arch architecture]... 
[-sdk [sdkname | sdkpath]] 
[-showBuildSettings] 
[buildsetting=value]... 
[buildaction]...
```

### 基本命令

#### 查看xcodebuild简洁用法
```
xcodebuild -usage
```

#### 查看已安装的SDK
```
xcodebuild -showsdks
```

#### 查看安装的版本号

```
xcodebuild
-version 
[-sdk [sdkfullpath | sdkname ] [ infoitem ]]
```

例子:
```
1.xcodebuild -version
2.xcodebuild -version -sdk iphonesimulator10.0
```

#### 查看项目中的 Targets、Configurations 和 Schemes

```
xcodebuild 
-list 
[[-project projectname]|[-workspace workspacename]]
```

例子:
```
1.xcodebuild -list
2.xcodebuild -list -workspace test.xcworkspace
```

#### 导出Archive
**exportOptionsPlist 在手动打包时会生成，若配置没有更改可以继续复用**
```
xcodebuild 
-exportArchive 
-archivePath xcarchivepath
-exportPath destinationpath
-exportOptionsPlist plistpath
```

#### 命令列表

| 命令 | 说明 |
|:------:|:------:|
| -usage | 查看xcodebuild简洁的用法 |
| -help | 查看帮助 |
| -verbose | 提供额外的状态输出 |
| -license | 显示Xcode和SDK许可协议 |
| -checkFirstLaunchStatus | 检查是否有任何初启动任务需要执行 |
| -project NAME | 编译项目名称，例如:xcodebuild -project XXX.xcodeproj |
| -targets | 编译全部目标 |
| -target NAME | 编译目标名称 |
| -workspace NAME | 编译全部目标 |
| -scheme NAME | 编译计划名称 |
| -configuration NAME	| 为构建每一个目标使用build配置名称 |
| -xcconfig PATH | 在PATH作为替代应用文件中定义的构建设置 |
| -arch ARCH | 建立每个目标的架构ARCH;这将覆盖在项目中定义的架构 |
| -sdk SDK | 使用指定的SDK编译项目 |
| -toolchain NAME | 使用工具链与标识或名称 |
| -destination DESTINATIONSPECIFIER | 使用由目标说明（用逗号分隔的一系列的key =描述目的地使用值对）中描述的目的地 |
| -destination-timeout TIMEOUT | 等待TIMEOUT秒，而搜索的目标设备 |
| -parallelizeTargets | 建立并行独立目标 |
| -jobs NUMBER | 指定并发生成操作的最大数量 |
| -dry-run | 做一切，除了实际运行的命令 |
| -hideShellScriptEnvironment |	不显示在构建日志shell脚本中的环境变量 |
| -showsdks | 显示已安装的SDK的列表 |
| -showBuildSettings | 显示构建设置和值的列表 |
| -list	| 列出了在一个工作空间中的一个项目的目标和配置，或方案 |
| -find-executable NAME | 在所提供的SDK和工具链显示的完整路径可执行文件名称 |
| -find-library NAME | 在所提供的SDK和工具链显示的完整路径库名 |
| -version | 显示的Xcode的版本;与-sdk将显示一个或所有已安装的SDK信息 |
| -enableAddressSanitizer YES/NO | 测试时打开或关闭地址过滤 |
| -resultBundlePath PATH | 指定在描述什么发生了捆绑的结果将被放置的目录 |
| -derivedDataPath PATH	| 指定的目录中生成产品和其他衍生数据 |
| -archivePath PATH	| 被指定任何创建的档案将被放置的目录，或应导出存档 |
| -exportArchive | 指定归档应导出 |
| -exportOptionsPlist PATH | 指定用于配置归档导出plist文件的路径 |
| -enableCodeCoverage YES/NO | 打开代码覆盖率或关闭时的测试 |
| -exportPath PATH	| 指定从存档导出的产品的目标 |
| -skipUnavailableActions | 指定不能执行计划的行动应被跳过而不是导致失败 |
| -exportLocalizations | 出口完成优秀项目本地化 |
| -importLocalizations | 进口本地化项目，假设任何必要的本地化资源在Xcode中已创建 |
| -localizationPath	| 指定XLIFF本地化文件路径 |
| -exportLanguage | 规定包括在本地化出口多个可选ISO 639-1语言 |

## 上传到 app store 

通过 altool 上传 App 的二进制文件
您可以使用 Application Loader 的命令行工具 altool，验证 App 二进制文件并将其上传至 App Store。

若要在上传之前验证构建版本或将有效构建版本自动上传至 App Store，您可在您的持续集成系统中包含 altool。altool 位于以下文件夹中：Application Loader.app/Contents/Frameworks/ITunesSoftwareService.framework/Versions/A/Support/。

若要运行 altool，请在命令行指定以下一项操作：
```
$ altool --validate-app -f file -u username [-p password] [--output-format xml]
$ altool --upload-app -f file -u username [-p password] [--output-format xml]
```

| 位置 | 详细说明 |
|:------:|:------:|
| --validate-app | 您要验证指定的 App |
| --upload-app | 您要上传指定的 App |
| -f file | 正在验证或上传的 App 的路径和文件名 |
| -u username | 您的用户名 |
| -p password | 您的用户密码 |
| --output-format [xml 或 normal] | 您想让 Application Loader 以结构化的 XML 格式还是非结构化的文本格式返回输出信息。默认情况下，Application Loader 以文本格式返回输出信息 |

# 远程打包

**使用远程 mac 服务器打包。此文档的所有操作建立在本地工程打包成功的情况下**

## 安装Xcode

### 1.图形界面操作

直接使用 app store 搜索 Xcode 下载

### 2.使用命令行安装

安装 xcode-install
```
gem install -n /usr/local/bin xcode-install
```

安装，选择一个 XCode 版本安装（下面例子为9.2）
```
xcversion install 9.2
```

## 安装 CocoaPods

百度上有许多教程，搜索一下即可。若项目没有使用 CocoaPods 管理项目，则跳过。

**下面为安装时的注意事项**

### 更新 gem Ruby 源

gem 是一个用于对 Ruby 组件进行打包的 Ruby 打包系统。 它提供一个分发 ruby 程序和库的标准格式，还提供一个管理程序包安装的工具。gem是一个管理 ruby 库和程序的标准包，它通过 ruby gem（如http://rubygems.org ）源来查找、安装、升级和卸载软件包，非常的便捷。

由于墙的原因和现在 taobao 的 ruby 源已经不维护了，所以我们使用 https://gems.ruby-china.com 

```
gem sources -a https://gems.ruby-china.com 
```

然后看看我们是否添加成功

```
gem sources -l
```

如果有多余的 ruby 源，最好删掉。
例如：
```
gem sources -r https://ruby.taobao.org
```

更新 gem 本身
```
gem update --system
```

更新所有程序包
```
gem update
```

## 更新证书

由于使用脚本打包，不依赖 jenkins 的环境，所以我们需要给服务器安装证书。

**若以后证书有更新，这一步需要重新配置**

首先我们从电脑导出证书的 .p12 文件。

使用 scp 拷贝到服务器上：scp 本地的.p12文件地址 用户名@服务器地址:服务器存放文件的路径
```
scp 本地的.p12文件地址 用户名@服务器ip:/Users/xxkj/Desktop
```

```
security unlock-keychain -p mac_password /Users/lizeyang/Library/Keychains/login.keychain
security list-keychains -s /Users/lizeyang/Library/Keychains/login.keychain
security import 你的.p12文件路径 -k /Users/lizeyang/Library/Keychains/login.keychain -P 你设置的密码 -T /usr/bin/codesign
```

当然，最简单粗暴的方式就是把自己电脑的 login.keychain 覆盖到服务器上。

## 更新描述文件

**若以后有新的描述文件，这一步需要重新配置**

使用 scp 把所有描述文件拷贝到服务器上
**注意：如果有些需要的描述文件并不在本地，则要一个一个拷贝过去**
```
scp /Users/xxkj/Library/MobileDevice -r 用户名@服务器ip:/Users/xxkj/Library
```

## 编写远程脚本

```

git pull origin 你的git分支

#工程路径
project_path=$(cd `dirname $0`; pwd)

#工程名
project_name=你的工程名字

#build文件夹路径
build_path=${project_path}/build

#plist文件所在路径
exportOptionsPlistPath=${project_path}/developmentOptionsPlist.plist

#导出 xcarchive 的地址
archivePath=${build_path}/${project_name}.xcarchive

#导出ipa所在路径
exportFilePath=${build_path}

#项目没有采用自动匹配证书，需要证书和描述文件信息。建议项目采用自动匹配证书。
CODE_SIGN_IDENTITY=证书名
PROVISIONING_PROFILE=描述文件名

#若使用 CocoaPods 管理的，请先编译 Pods 库
pod install 或者 pod update 

#清理工程 （第一种为用 xcodeproj 打开的工程，而非 workspace ）
xcodebuild -project ${project_path}/${project_name}.xcodeproj clean || exit
xcodebuild -workspace ${project_name}.xcworkspace -scheme ${project_name} clean || exit

#编译工程 （第一种为用 xcodeproj 打开的工程，而非 workspace ）
xcodebuild archive -project ${project_path}/${project_name}.xcodeproj -scheme ${project_name} -archivePath ${archivePath} || exit
xcodebuild archive -workspace ${project_name}.xcworkspace -scheme ${project_name} -archivePath ${archivePath} || exit

#打包 (项目没有采用自动匹配证书)
#xcodebuild \
#-exportArchive \
#-archivePath ${archivePath} \
#-exportPath "${exportFilePath}" \
#-exportOptionsPlist exprotOptionsPlist.plist \
#CODE_SIGN_IDENTITY=CODE_SIGN_IDENTITY \
#PROVISIONING_PROFILE=PROVISIONING_PROFILE || exit

#打包 (项目采用自动匹配证书)
xcodebuild \
-exportArchive \
    -archivePath ${archivePath} \
    -exportPath "${exportFilePath}" \
    -exportOptionsPlist developmentOptionsPlist.plist || exit

```

### 关于证书名 
点击钥匙串(keychain) -> 找到自己的证书 -> 右键选择显示简介 -> 最上面的黑色大标题就是证书名

### 关于描述文件名
可在存储描述文件名的文件夹中找到 /Users/xxkj/Library/MobileDevice