---
author: "Geek Lite"
handle: "@QingQ77"
source: "https://x.com/QingQ77/status/2056029473511002177"
date: 2026-05-17T15:10:00.000Z
type: tweet
likes: 369
reposts: 76
replies: 6
bookmarks: 387
views: 15803
---
# Geek Lite (@QingQ77)

一套生产级的开源水培自动化控制系统，填补业余 DIY 装置和昂贵工业自动化之间的差距，支持 ESP32 硬件、MQTT 遥测和实时 Web 仪表板。  
  
[github.com/40rbidd3n/Hydr…](https://github.com/40rbidd3n/Hydro0x01)  
  
HydroponicOne 是水培种植的 IoT 全套方案。ESP32 负责读 pH、EC、水位、温湿度这些传感器数据，走加密 MQTT 送到 Fastify 后端，React 前端通过 WebSocket 实时刷新。存数据用 PostgreSQL 管配置、InfluxDB 管时序数据。固件升级走 RSA 签名 OTA，安全可靠。项目不光有代码，还给了 PCB 设计、3D 打印外壳和接线图，能直接对接 Home Assistant 和 Telegram 告警。

![Image](<Geek%20Lite%20(@QingQ77).assets/image-01.jpg>)