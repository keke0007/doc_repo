---
author: "李不川"
handle: "@Libuchuan"
source: "https://x.com/Libuchuan/status/2120005987603972462"
date: 2026-11-10T04:09:49.000Z
type: article
---
# 实测！国行三星成功卸载SystemHelper！其他安卓也能用！

*By 李不川 (@Libuchuan)*

![Banner](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-01.jpg>)

前阵子有开发者拆了小米手机里一个叫 SystemHelper 的预装组件，包名 com.mobiletools.systemhelper，是联通的。

按他的逆向分析，这个组件会读取手机上装了哪些 App、过去 24 小时每个 App 用了多久、跑了多少流量，再连同 IMEI、IMSI、ICCID 一起发到联通的服务器。装了哪些 App 都在清单里，包括你用来上外网的那几个。而且并不限运营商，移动电信的卡也会上报，特别是翻墙上网的时候。

代码里列出的品牌有 OPPO、vivo、小米、三星等。我的主力机是国行三星，按照图示流程果然也能找到。

![图像](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-02.jpg>)

对于经常在外网冲浪的我还是非常害怕的😱所以搓了这篇帖子，教大家怎么彻底地清除掉安卓机里的 systemhelper组件

虽然抖音里边新闻报道说是为了防范电诈用的，但是我觉得我并不需要，我还是挺害怕他们会上报我的信息。

**所以按需设置，如果你觉得这个反诈组件还挺有用的，可以不管他。**

![图像](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-03.jpg>)

折腾了半个小时，三星主系统里的组件成功卸载，安全文件夹卸载不掉，但是成功停用。安全文件夹是三星独有的功能，其他安卓手机不用管这个。卸载完觉得手机都变流畅了😂

不用 root，不用电脑。其他安卓里边基本也都有，方法也大差不差，可以参考一下。实测我的VIVO老备用机里边也有这个组件，**也能成功卸载。**

![图像](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-04.jpg>)

## 准备工作

需要两个 App：Shizuku 和 Canta。用安全文件夹的话，再加一个「雹」。下载地址（都是开源软件，GitHub 原版）：  
Shizuku：[github.com/RikkaApps/Shizuku/releases](https://github.com/RikkaApps/Shizuku/releases)  
Canta：[github.com/samolego/Canta/releases](https://github.com/samolego/Canta/releases)  
雹：[github.com/aistra0528/Hail/releases](https://github.com/aistra0528/Hail/releases)

## 一、激活 Shizuku

先确认手机连着 WiFi，无线调试离不开 WiFi。

① 打开开发者选项  
设置 → 关于手机 → 软件信息，连续点「编译编号」7 次，按提示输入锁屏密码。看到「开发者模式已启用」就行。  
返回设置首页，拉到最底下，会多出一个「开发者选项」。

② 打开无线调试  
进开发者选项，往下翻到「调试」那一栏，打开「无线调试」，弹窗点允许。  
先别退出这个页面。

③ 在 Shizuku 里发起配对  
打开 Shizuku，找到「通过无线调试启动」，点「配对」。  
Shizuku 会发一条通知，里面有个输入框。第一次用如果提示要通知权限，一定要允许，不然收不到这条通知。

④ 拿到配对码  
回到开发者选项 → 无线调试，点进去（点文字，不是点开关），再点「使用配对码配对设备」。屏幕上会弹出一个 6 位数字。  
这个弹窗别关。

⑤ 输入配对码  
从屏幕顶部下拉通知栏，在 Shizuku 那条通知里点「输入配对码」，填刚才那 6 位数字，提交。  
通知显示配对成功就行。

⑥ 启动  
回到 Shizuku 首页，点「启动」，等几秒，顶部显示「Shizuku 正在运行」就成功了。

![图像](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-05.jpg>)

## 二、主系统卸载

在 Shizuku 的应用管理里，把 Canta 的开关打开。  
打开 Canta，留在左边的标签页，搜 systemhelper，会出来两个：  
· SystemHelper，包名 com.mobiletools.systemhelper，标绿色 RECOMMENDED，删它  
· System Helper Service，包名 com.qualcomm.qti 开头，这个是高通的组件，标红色 EXPERT，**别删！**  
勾选 SystemHelper，点垃圾桶卸载。

![图像](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-06.jpg>)

![图像](<实测！国行三星成功卸载SystemHelper！其他安卓也能用！.assets/image-07.jpg>)

## 三、安全文件夹

这个是三星独有的功能，其他安卓手机不用管  
Canta 都动不了安全文件夹，三星会直接拦下来，所以只能用「雹」来停用这个组件。

把「雹」添加进安全文件夹，打开后在右上角勾选「系统」，找到 SystemHelper，长按 → 应用信息 → 底部点「禁用」。这里点卸载没用，禁用就够了。

收尾：回开发者选项，把无线调试关掉。系统大版本更新后它可能会回来，再走一遍就行。

这种卸载只对当前用户生效，安装包还留在系统分区。如果想恢复，在 Canta 右边的已卸载列表里还能装回来。  
  
真是卸载完觉得手机都变流畅了😂