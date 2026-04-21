nanoCH32H417
-----------
[English](./README.md)

* [nanoCH32H417介绍](#nanoCH32H417介绍) 
* [特性](#特性)
* [芯片资源](#芯片资源)
* [使用教程](#使用教程)
* [产品链接](#产品链接)
* [参考](#参考)


# nanoCH32H417介绍
nanoCH32H417 是MuseLab基于沁恒CH32H417QEU6芯片推出的开发板，RISC-V双核结构，最高主频可达400MHz，板载WCHLink调试器，支持USB 3.0接口(5Gbps)，百兆以太网，LCD接口，SD卡接口，可通过USB口下载烧录，方便进行快速的原型验证及开发。

![1](https://github.com/wuxx/nanoCH32H417/blob/main/doc/nanoCH32H417-1.jpg)
![4](https://github.com/wuxx/nanoCH32H417/blob/main/doc/nanoCH32H417-4.jpg)


# 特性
- 板载WCHLinkE下载器，支持下载与一路串口通信
- 5Gbps超高速USB 3.0 接口
- USB 2.0 OTG接口，支持USB PD
- 百兆以太网接口
- FPC-12P接口，可支持常见LCD （如ili9341、st7789等）
- SD卡座，支持SD卡读写（SDIO协议）

# 芯片资源
![CH32H417QEU6](https://github.com/wuxx/nanoCH32H417/blob/main/doc/CH32H417_Resource.jpg)
CH32H417 是基于青稞 RISC-V5F 和 RISC-V3F 双内核设计的互联型通用微控制器。CH32H417 集成了
USB 3.2 Gen1 控制器和收发器、百兆以太网 MAC 及 PHY、SerDes 高速隔离收发器、Type-C/PD 控制器
及 PHY，提供 SD/EMMC 控制器、500MBytes 通用高速接口 UHSIF、DVP 数字图像接口、单线协议主接口
SWPMI、可编程协议 I/O 控制器 PIOC、灵活存储控制器 FMC、DFSDM、LTDC、GPHA、DMA 控制器、多组
定时器、8 组串口、I3C、4 组 I2C、2 组 QSPI、4 组 SPI，2 组 I2S、3 组 CAN 等外设资源，内置了 5M
采样率双 12 位 ADC 单元、20M 采样率 10 位高速 HSADC 单元、16 路 Touchkey、双 DAC 单元、3 组运放
OPA、电压比较器 CMP 等模拟资源，支持 10M/100M 以太网通讯，支持 USB 2.0 和 USB 3.0，支持 USB Host
主机和 USB Device 设备功能、Type-C 和 PDUSB 快充功能，支持 SerDes 高速隔离及远距离传输，支持
双内核分工提升网络协议处理效率和通讯响应速度。

# 使用教程
## MounRiver Studio IDE
沁恒官方提供MounRiver Studio IDE开发环境，支持Windows/Linux/Mac，具体使用说明如下
 
### MounRiver Studio 下载
可在官网[MounRiver Studio](http://www.mounriver.com)下载IDE，选择最新版本下载即可。

### 编译
以GPIO工程为例，双击GPIO_Toggle.wvproj打开工程  
![MRS-1](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-1.png)
![MRS-2](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-2-CN.png)  
点击 工程 -> 编译全部 对工程进行编译  (会分别编译两个核心V3F和V5F的代码)
![MRS-3](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-3-CN.png)

## 烧录
工程已经内置脚本，将V3F和V5F两个项目的代码自动拼接成单独文件Merge.bin, 选择V5F工程，然后点击 闪存 -> 下载 即可完成烧录
![MRS-4](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-4-CN.png)


# 产品链接
[nanoCH32H417 Board](https://item.taobao.com/item.htm?spm=a1z10.3-c.w4002-21349689064.10.516b773dX0i6on&id=693917864064)

# 参考
### WCH
https://www.wch.cn/
