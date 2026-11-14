---
author: "Shlcy"
handle: "@CY160909"
source: "https://x.com/CY160909/status/2121066015564140632"
date: 2026-11-13T02:22:00.000Z
type: tweet
likes: 206
reposts: 33
replies: 3
bookmarks: 193
views: 5969
---
# Shlcy (@CY160909)

挖到一个硬核的开源Skill  
  
它能把手绘草图、扫描图纸，甚至 CAD 截图，重新绘制成可编辑的 DWG。  
  
而且它是不只负责画，还会检查重绘结果  
  
以前遇到没有源文件的老图纸，往往只能打开 AutoCAD，一根线一根线重新画。  
  
这个项目直接把整套流程交给 AI：  
  
识别图纸中的线条、圆弧和尺寸信息  
自动生成 CAD 绘图数据  
调用 AutoCAD 绘制可编辑 DWG  
整理图层，导出 DXF 和预览图  
对比原图，检查重绘偏差  
  
最让我感兴趣的是它的尺寸处理机制。  
  
哪些尺寸有明确标注，哪些是按比例推算，哪些只是 AI 推断，甚至哪些根本无法识别，都会区分处理  
  
这比单纯把图片转换成线条实用得多。  
  
需要整理老图纸、重建 CAD 文件的人，这就是神器  
  
我建议关键尺寸最好人工复核，别当成工程级精准转换。  
  
GitHub：

> ![Link card preview](<Shlcy%20(@CY160909).assets/image-01.jpg>)
> 
> 🔗 [**GitHub - pengxiaoan/autocad-dwg-redraw-skill**](https://t.co/MGyihzhq3v)
> 
> _From github.com_