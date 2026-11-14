---
author: "TechVerser"
handle: "@realfxw"
source: "https://x.com/realfxw/status/2119203251673538837"
date: 2026-11-07T23:00:02.000Z
type: tweet
likes: 2
replies: 2
bookmarks: 3
views: 495
---
# TechVerser (@realfxw)

🚀 还在忍受臃肿缓慢的 Docker Desktop 吗？或者觉得配置内网穿透工具太繁琐？今天给大家推荐一个爆火的开源神器 —— Kiri！这是一款用 Rust 编写的轻量级、极速容器运行时，支持 macOS、Windows 和 Linux，完美平替 Docker Desktop 和 OrbStack。  
  
💡 核心亮点：  
  
1️⃣ 极致轻快： 守护进程仅约 15MB，闲置 60 秒后自动进入休眠，唤醒只需短短 0.15 秒！通过 Docker API 启动和退出容器的中位数时间仅为 2.5 毫秒，比 Docker Desktop 快了整整 80 倍！  
  
2️⃣ 自带免费内网穿透（平替 ngrok）： 只需要一行简单的命令 kiri share web，就能瞬间为运行中的容器生成一个公网链接（例如 [web.runkiri.cloud](http://web.runkiri.cloud)）。无需繁琐部署，无需设置端口转发，直接发给团队、客户测试，或者在手机上预览效果，简直是开发者的福音。  
  
3️⃣ 无缝兼容原有生态： 它直接使用 Docker Engine API，所以你平时习惯的 docker CLI、Docker Compose、buildx 等工具链完全可以无缝切换、开箱即用，甚至在 Mac 上也完美支持通过 Rosetta 运行 amd64 镜像。  
  
4️⃣ 本地开发利器： 自动为你分配 http://<name>.kiri.local 本地专属域名，而且还由 Kiri 自带的 CA 机构提供 HTTPS 证书支持，本地测试更加贴近真实生产环境。  
  
5️⃣ 不仅是容器： 支持一键启动 Kubernetes 集群。在最新的 0.1.18 版本中，更是在容器旁直接内置了终端（支持多标签页、分屏和搜索），K8s Pods 现在也支持通过 hostPath 使用本地文件夹。  
  
目前 Kiri 已经在 GitHub 上斩获了 500+ Stars，采用 Apache-2.0 协议开源。无论你是需要一个更快的本地开发环境，还是想快速向外部展示你的本地 Web 服务，Kiri 都绝对值得一试！  
  
🔗 项目地址：[github.com/libracoder/kiri](https://github.com/libracoder/kiri)  
  
#Docker #Rust #Kubernetes #DevTools #开源项目 #后端开发 #程序员日常 #Kiri

![Image](<TechVerser%20(@realfxw).assets/image-01.jpg>)