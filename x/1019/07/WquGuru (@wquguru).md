---
author: "WquGuru"
handle: "@wquguru"
source: "https://x.com/wquguru/status/2112084878842085436"
date: 2026-10-19T07:34:10.000Z
type: tweet
likes: 230
reposts: 27
replies: 22
bookmarks: 242
views: 11587
---
# WquGuru (@wquguru)

ESP32 这两周是真火，乐鑫自己都下场做了模拟器。  
  
我把能找到的 ESP32 模拟器翻了一遍，挑出 3 个，又用自己的固件实测了一下官方那个模拟器，手里还没板子的先收藏👇  
  
1. Velxio（3.3k 星，AGPL）  
浏览器里拖线画电路，40 块板，ESP32 到 C6 都有。ESP32 系列在服务器上跑 QEMU，再把引脚和串口推回网页，也能自己部署。  
[github.com/davidmonterocr…](https://github.com/davidmonterocrespo24/velxio)  
  
2. esp32sim（364 星，MIT）  
从芯片自带的 ROM 一路启动，屏幕、触摸、摄像头、声音都做了，S3 最成熟，连 Linux 都跑起来了。网页版打开就能玩。  
[github.com/mikroverk/esp3…](https://github.com/mikroverk/esp32sim)  
  
3. 乐鑫官方 esp-emulator（190 星）  
芯片最全：C3、C5、C6、H2、P4、S3，蓝牙、Thread、GDB 都有，本地版还能连真实互联网。  
有三点要知道：经典 ESP32 不在列表里；网页版不联网；仓库目前只放了二进制，开不开源，官方在 issue 里回复还没决定。  
[github.com/espressif/esp-…](https://github.com/espressif/esp-emulator)  
  
官方那个我拿自己的验收固件跑了一遍：点灯、双核、NVS、PSRAM、LVGL 都能跑，芯片和协议做得很全，但是外设这块还得等等。  
  
最后预告一下我自己在做的：试板 TryBoard。  
同样在浏览器里跑你编译好的 ESP-IDF / Arduino 固件（C3、S3），全在本地跑。板子、面包板、OLED、灯带、舵机摆在 3D 实验台上，跑完就知道该买哪块板🫡

![Image](<WquGuru%20(@wquguru).assets/image-01.jpg>)