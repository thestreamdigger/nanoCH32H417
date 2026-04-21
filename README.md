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
nanoCH32H417 is a development board made by MuseLab based on WCH CH32V305RBT6 with Dual TYPE-C USB interfaces, LCD interface and SD card interface onboard, can be programmed through the USB port, which is convenient for prototype verification and development.

![1](https://github.com/wuxx/nanoCH32H417/blob/main/doc/nanoCH32H417-1.jpg)
![4](https://github.com/wuxx/nanoCH32H417/blob/main/doc/nanoCH32H417-4.jpg)

# Feature
- Dual USB interface, USB1 is USB-FS device, USB2 is USB-HS device
- Can be downloaded directly via USB without additional downloader
- Onboard 8MHz and 32.768K crystal oscillator
- FPC-12P cable interface, can support common LCD (such as ili9341, st7789, etc.)
- SD card slot, support SD card reading and writing (SPI protocol)

# Chip Resources
![CH32H417](https://github.com/wuxx/nanoCH32H417/blob/main/doc/CH32H417_resource.jpg)

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
