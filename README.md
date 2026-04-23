nanoCH32H417
-----------
[中文](./README_cn.md)

* [Introduce](#Introduce)
* [Feature](#feature)
* [Chip Resources](#chip-resources)
* [How To Use](#how-to-use)
* [Product Link](#product-link)
* [Reference](#reference)


# Introduce
nanoCH32H417 is a development board made by MuseLab based on WCH CH32H417QEU6 with Dual TYPE-C USB interfaces, USB 3.0 5Gbps and 100Mbps ethernet, LCD interface and SD card interface onboard, programmed through the onboard debugger WCHLinkE, which is convenient for prototype verification and development.

![1](https://github.com/wuxx/nanoCH32H417/blob/main/doc/nanoCH32H417-1.jpg)
![4](https://github.com/wuxx/nanoCH32H417/blob/main/doc/nanoCH32H417-4.jpg)

# Feature
- Onboard WCHLink-E support SWD debug and serial port
- 5Gbps USB 3.0 super speed interface
- USB 2.0 OTG interface, support PD
- FPC-12P cable interface, can support common LCD (such as ili9341, st7789, etc.)
- SD card slot, support SD card reading and writing (SDIO protocol)

# Chip Resources
![CH32H417](https://github.com/wuxx/nanoCH32H417/blob/main/doc/CH32H417_Resource.jpg)

# How To Use
## MounRiver Studio IDE
WCH officially provides MounRiver Studio IDE development environment, which supports Windows/Linux/Mac. The instructions are as follows
 
### MounRiver Studio Download
download the MounRiver Studio IDE from the official website [MounRiver Studio](http://www.mounriver.com), and just select the latest version to download (MRS2).

### Compile
Take the GPIO project as an example, double-click GPIO_Toggle.wvproj to open the project
![MRS-1](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-1.png)
![MRS-2](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-2-EN.png)  
Click Project -> Build Project to compile the project  
![MRS-3](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-3-EN.png)


## Program
connect the WCHLinkE USB port, select the V5F project(the Merge.Bin is generated in V5F project), then click Flash -> Download to program the flash.   
![MRS-4](https://github.com/wuxx/nanoCH32H417/blob/main/doc/MRS-4-EN.png)


# Product Link
[Aliexpress](https://www.aliexpress.com/item/1005005033298927.html?spm=a2g0s.12269583.0.0.20535947csm0Sw
)  
[Tindie](https://www.tindie.com/products/johnnywu/nanoch32h417-development-board/)

# Reference
### WCH
https://www.wch.cn/

### LCD source code
https://github.com/wuxx/CH32V305-makefile-example
