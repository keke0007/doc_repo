---
author: "Hong&哈士鹅"
handle: "@clenusgrok"
source: "https://x.com/clenusgrok/status/2120369237009310111"
date: 2026-11-11T04:13:15.000Z
type: thread
likes: 721
reposts: 156
replies: 29
bookmarks: 1129
views: 39099
---
# Hong&哈士鹅 (@clenusgrok)

安卓手机净化：  
灭杀SystemHelper（泄露上传个人购买所属手机全部隐私的元凶🌚）  
适用其他任何你想灭杀的组件，只需换名称,小白操作无门槛😍.  
最后附视频。  
你去设置里找它，想停用——按钮是灰的焊死了。想卸载——根本没有这个选项。它会自动开启网络权限，无法禁止（禁止后会自动重开）。  
全程由muse在旁边定性、指挥、复验。  
下面是完整步骤，Windows、Mac、Linux 的电脑端命令，差异我都标出来了，照着做就行。  
第一步：开通道  
先下工具，谷歌官方的 platform-tools 下载页（Windows、Mac、Linux 都有）：  
[developer.android.com/tools/releases…](https://developer.android.com/tools/releases/platform-tools)  
下载对应你电脑系统的压缩包，解压到一个路径短、没有中文的地方。Windows 推荐直接解压到 C 盘根目录，得到 C:\platform-tools；Mac、Linux 解压到个人目录下就行。  
然后在这个文件夹里打开终端：  
  
Windows：打开文件夹，在资源管理器上方的地址栏里输入 cmd，回车，终端会直接开在这个目录里，不用敲一长串路径，也不用配环境变量。（如果你开的是 PowerShell，后面所有命令里的 adb 都要写成 .\adb，例如 .\adb devices。）  
•  
Mac：在访达里右键这个文件夹 →「服务」→「新建位于文件夹位置的终端窗口」；没有这一项的话，先打开终端，把文件夹拖进终端窗口再回车，效果一样。  
•  
Linux：在文件夹空白处右键，选「在终端中打开」。  
  
手机上：设置 → 关于手机，连点「版本号」7 次，打开开发者模式；再进开发者选项，打开「USB 调试」。用能传数据的数据线连上电脑（有些线只能充电，不能用），把 USB 用途切到「传输文件」，手机弹窗点「允许」。  
终端里输入（三个系统都一样）：  
  
adb devices  
能看到设备序列号，状态是 device，就说明通了。  
小贴士：如果显示 unauthorized，是手机上的授权弹窗没点，拔了重插、点「允许」；如果列表是空的，先换一根数据线或换个 USB 口再试。  
  
第二步：先存一份全量清单  
动手之前，先把手机里所有应用的包名导出存档，出任何问题都有据可查（三个系统都一样）：  
  
adb shell pm list packages -f > packages.txt  
确认目标在不在，这一步分系统：  
Windows：  
adb shell pm list packages | findstr systemhelper  
  
Mac、Linux：  
adb shell pm list packages | grep systemhelper  
  
查到 com.mobiletools.systemhelper，就是它。  
  
第三步：一条命令卸载  
三个系统都一样：  
  
adb shell pm uninstall -k --user 0 com.mobiletools.systemhelper  
回车，终端回一个 Success，就成了。  
说一下为什么这个卸法安全：--user 0 是只对当前用户卸载，不动系统分区里的本体；-k 是保留它的数据。所以这不是真删除，手机不会变砖，也随时能恢复——这正是它比 root 卸载稳的地方。  
第四步：验一遍，再收尾  
再跑一次第二步的查询命令（Windows 用 findstr，Mac、Linux 用 grep），查不到它，应用列表里也消失了，就说明干净了。拨号、短信、输入法、银行 App 各打开一遍确认正常，重启一次手机再查一遍。  
最后回开发者选项，关掉「USB 调试」，撤销 USB 调试授权。  
后悔药  
哪天想装回来，一条命令（三个系统都一样）：  
  
adb shell cmd package install-existing com.mobiletools.systemhelper  
（个别老系统如果这条报错，把 cmd package 换成 pm 再试。）  
提醒两句：恢复出厂设置会让它复活，这是这类卸法的通例；另外这套方法对大多数，你想卸载的预装应用都通用，换目标组件的名称就行，但只卸你查清楚的东西，扛着电话、短信这些功能的系统组件别碰。  
我录了完整的实操视频，配了讲解，贴在下面了。自己动手之前，可以先看一遍全程，心里有数再上手。

![🎥 Video](<Hong&哈士鹅%20(@clenusgrok).assets/image-01.jpg>)

---

操作命令在帖子里，复制粘贴即可  
先下工具，谷歌官方的 platform-tools 下载页（Windows、Mac、Linux 都有）：  
[developer.android.com/tools/releases…](https://developer.android.com/tools/releases/platform-tools)  
下载对应你电脑系统的压缩包，解压到一个路径短、没有中文的地方。Windows 推荐直接解压到 C 盘根目录，得到 C:\platform-tools；Mac、Linux 解压到个人目录下就行。

> 🔗 [**SDK Platform Tools release notes  |  Android Studio  |  Android Developers**](https://t.co/mRk1WkJPTx)
> 
> _From developer.android.com_

---

😂😂😂这个收藏震惊到我了，可惜昨天聚餐没有及时回复大家错过流量了。感谢大家的喜欢，如果遇到的  
困惑，留言共同解决。

![Image](<Hong&哈士鹅%20(@clenusgrok).assets/image-02.jpg>)