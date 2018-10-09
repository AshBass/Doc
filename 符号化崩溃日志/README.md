# 符号化崩溃日志

一般的crash日志是没有被符号化的，我们根本看不懂，所以我们需要符号化日志。

我们需要找到一个工具帮我们符号化，Xcode中有 symbolicatecrash 来帮我们符号化。

我们可以在命令行输入以下命令寻找 symbolicatecrash。
```
find /Applications/Xcode.app -name symbolicatecrash -type f
```

当然，如果如无意外、Xcode没有太大改动的话，symbolicatecrash的路径是不变的。
```
/Applications/Xcode.app/Contents/Developer/Platforms/AppleTVSimulator.platform/Developer/Library/PrivateFrameworks/DVTFoundation.framework/symbolicatecrash
```

我们将这个symbolicatecrash复制出来。当然，文章仓库里已经有之前实践所使用过的 symbolicatecrash 和 crash 日志，你可以直接拿来用。

我们把我们 app 的崩溃日志复制出来，放入和 symbolicatecrash 同一个文件夹里。（如果是苹果审核时返回的日志，你可以复制到一个 .txt 文件中，然后把后缀改成 .crash ）。

然后我们需要提取我们项目的 dsyM 符号表。

**注意，请不要使用仓库中的符号表来给你自己的崩溃日志使用，符号表和app是对应存在的。具体请百度**

Xcode -> Window -> Organizer -> 你的app -> Archives -> 右键你崩溃的版本打开所在文件夹

之后我们就可以看到 app 的 .xcarchive 文件，右键点击显示包内容，我们可以在 dSYMs 文件夹里面找到和工程同名的 .dSYM 文件。复制到和 symbolicatecrash 、crash 日志同一个文件夹。

接下来依次执行两个命令

1.先是设置临时变量
```
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
```

2.符号化日志（其中 MyAppName.crash 和 MyAppName.app.dSYM 都要改成你自己的名字）
symbol.crash 是符号化生成的文件，你也可以生成 symbol.txt 文本文件
```
./symbolicatecrash ./MyAppName.crash ./MyAppName.app.dSYM > symbol.crash
```