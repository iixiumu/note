# OpenCore折腾小记

## 缘起

说来话长，长话短说。事情的起因还要追溯到三年前了，确切地说是2020年的双十一。那时候macOS还只支持x86，没有Apple Silicon，mbp还在单管压i9.现在有了Apple Silicon，苹果还在出8GB内存的电子垃圾，Only Apple can do.

更久以前，在我还买不起苹果的时候，听说有个东西叫黑苹果，那个时候还是四叶草，后来有了更强大的OpenCore了。简而言之，OpenCore可以在非苹果硬件上安装macOS系统，在Sonoma之前，几乎可以完美黑苹果，AirDrop啊什么的也都是没啥问题。另外OpenCore还有一个有意思的用法，Sonoma放弃了对一些旧x86设备的支持，借助OpenCore，可以让这些设备支持最新的Sonoma。

折腾OpenCore的初衷，首先是需要一台可以日常使用的PC（没有游戏需求），Windows和Linux又不怎么好用，遂瞄上了macOS，也可以和工作用的设备保持一致。鉴于白苹果的价格、内存、SSD，又瞄上了黑苹果。

不过OpenCore也有一些限制，首先是，最完美的是10代酷睿，10代之后和AMD也可以支持，但是核显肯定是用不了了。然后是显卡，不再支持N卡，A卡也不是全都支持，有限几个型号，最后的完美支持是RX6xxx了。主板的话，理论上CPU能插上的基本都可以支持，只不过挑一个大众型的，可以直接用别人做好的EFI，内存就无所谓了，主板能支持就行。SSD也还好，避开三星，问题就不大了。网卡的话，Sonoma之前用博通的网卡，可以完美AirDrop，Sonoma之后，好像也可以支持，只是比较麻烦，intel的话，可以上网，但是AirDrop就没办法了，不过AirDrop这玩意儿也没啥好的，经常性抽风——对方已拒绝，我也换成了有线上网，又快又稳定，多香啊。

2020年的双十一买了硬件，装机，最初自己照着教程做EFI，失败了几次，就放弃了，网上找现成的，我用的主板还算主流，不难找。然后去年年底突然又想起这事来了，虽然随着Apple Silicon的普及，x86的黑苹果已经快要走到头了，但还是想再研究一下，一方面把这个遗憾给了结了，另一反面，用黑苹果的人越来越少了，我用的EFI快没人更新了，只能自己上了，以后可以自己升级OpenCore版本了，准备迎接macOS 15 16，黑苹果再战三年。用六年，完全够本了，最后还有残值，这个配置淘汰下来跑跑服务啊什么的，应该也够了。

Apple Silicon有一个缺陷，没办法跑x86的Windows，当然了我不会裸机装Windows，但是万一碰到需要Windows环境的时候，开个虚拟机就行了，Apple cann't do.

## 硬件

| 类型  | 型号&规格 |
| ---- | ---- |
| 主板 | MAG B460M MORTAR |
| CPU | i5-10400 |
| 散热器 | Thermalright(利民) AK120 SE |
| 内存 | 4 * 16GB 2666Mhz |
| SSD | 西数SN730 |
| HD | 2 * 4TB紫盘 + 2 * 4TB酷狼 |
| 网卡 | Fenvi T919 1750M + 蓝牙4.0 |
| 机箱 | 鞋盒 |

时间太久，具体记不太清楚了，全套大概5-6k吧，已经用了3年了，目之所及，再用个两三年问题也不大，CPU性能足够，内存64GB还是可以造一造的，扩硬盘相对于苹果官方来说，成本低太多了。日常写写代码，处理一些工作上的事情，刷刷视频，vmware跑一些虚拟机什么的，毫无压力，唯一后悔的可能就是当初该加钱上i7的。

## 安装&踩坑

### 制作安装U盘

https://dortania.github.io/OpenCore-Install-Guide/installer-guide/

macos、Windows、Linux都可以制作，不过还是macOS方便。

系统下载，推荐gibMacOS

写入U盘

``` sh
sudo /Applications/Install\ macOS\ Big\ Sur.app/Contents/Resources/createinstallmedia --volume /Volumes/MyVolume
```

### EFI初始化

https://dortania.github.io/OpenCore-Install-Guide/installer-guide/opencore-efi.html

照着教程，只保留最基本的就可以，别的也没啥用

### 收集文件

https://dortania.github.io/OpenCore-Install-Guide/ktext.html

https://github.com/acidanthera/OpenCorePkg/releases

三个部分，Drivers、Kexts和SSDTs，看说明，那些需要哪些不需要，有些和平台有关系，EFI分release和debug，刚上手建议debug，出错了会有比较详细的错误提示，便于排查问题。

很久以前踩过一个坑，以为WhateverGreen只有独显需要，没仔细看文档，其实核显也需要。

其中比较麻烦的两个东西是USB和无线网卡：

1. USB有定制教程，略复杂，懒得做，我直接用了OS-X-USB-Inject-All（https://github.com/daliansky/OS-X-USB-Inject-All），大概看了下USB基本上都是可用的。

2. 无线网卡，博通的网卡，在Sonoma下已经不能默认支持了，intel我试过加驱动，效果也不太行，就放弃了，有线就挺好，又快又稳，不过有个小问题，一个蓝牙鼠标，接收器丢了，只能用蓝牙了，博通和intel网卡的蓝牙，加上驱动是可以用的，不过不想插个网卡就为了一个蓝牙，遂狗东买了个蓝牙接收器，不到十块钱，挺好用的。

### EFI定制

https://dortania.github.io/OpenCore-Install-Guide/config.plist/#creating-your-config-plist

EFI文件编辑，ProperTree就可以，另外还有个OpenCore Configurator，界面更友好一些，不过感觉没必要。

照着自己平台的教程慢慢改就行了，一定要仔细，任何一个地方没改对，都可能启动不了。文档的最后还有一些需要调整的bios设置项，跟着做就行了。

设备号生成，用GenSMBIOS就可以。

出了问题看日志，网上搜索。官方也给了一些常见问题及其解决办法，可以查阅。

https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/opencore-issues.html#stuck-on-a-black-screen-before-picker

黑苹果就不建议单盘多系统了，有多系统需求加硬盘，台式机加硬盘成本不高，每个系统用独立的盘，互不干扰，挂一个影响也不大。

### 安装

U盘启动，硬盘格式化，安装。

### 善后

https://dortania.github.io/OpenCore-Install-Guide/troubleshooting/extended/post-issues.html

一些优化调整什么的，我都没做，好像也没啥问题。不过也算是遇到了一个问题，HDMI只能4K 30hz，测试了线和接口都是正常的，但是DP可以，就不折腾HDMI了。

### 补充

EFI文件，安装的时候要放到U盘的EFI分区，挂载的话用diskutil就可以了，装完之后拷贝到系统的EFI分区就可以了。

## 最后的最后

目测macOS应该会在两三个版本之后彻底放弃对x86的支持，最近的版本，可能也在对x86做负优化了。随着macOS放弃对x86的支持，软件开发者对x86的支持也会越来越差，两三年后黑苹果应该就要彻底玩完了，再追两个大版本，用六年是够本了。

苹果的所有设备，mac、ipad，iPhone，对内存和硬盘的抠门，真是一言难尽，Only Apple can do！
