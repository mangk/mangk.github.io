---
title: NFC基础知识与读写
comments: true
banner_img_height: 30
date: 2022-11-18 16:59:43
categories:
- 小程序
tags:
- NFC
- Ndef
- MifareClassic
---
## 序
最近的项目需求中，需要使用NFC芯片来保存一些类似于url的信息并通过iOS或Android手机能正常获取到相关信息，所以针对NFC相关技术与特点进行了学习和总结。我会把我看过教程链接也同步贴到文章中。因为手上只有M1卡，所以下面的主要内容也围绕M1卡来进行
## 什么是NFC
近场通信（英语：Near-field communication，NFC），又称近距离无线通信、近距离通信，是一套通信协议，让两个电子设备（其中一个通常是移动设备，例如智能手机）在相距几厘米之内进行通信。NFC，如同过去的电子票券智能卡一般，将允许移动支付取代或支持这类系统。NFC应用于社交网络，分享联系方式、照片、视频或文件。具备 NFC 功能的设备可以充当电子身份证和钥匙卡。NFC 提供了设置简便的低速连接，也可用于引导能力更强的无线连接。

近场通信技术由非接触式射频识别（RFID）演变而来，由飞利浦半导体（现恩智浦半导体）、诺基亚和索尼共同于2004年研制开发，其基础是RFID及互连技术。近场通信是一种短距高频的无线电技术，在13.56MHz频率运行于20厘米距离内。其传输速度有106 Kbit/秒、212 Kbit/秒或者424 Kbit/秒三种。目前近场通信已通过成为ISO/IEC IS 18092国际标准、EMCA-340标准与ETSI TS 102 190标准。NFC采用主动和被动两种读取模式。

每一个完整的NFC设备可以用三种模式工作：

- 卡模拟模式（Card emulation mode）：这个模式其实就是相当于一张采用RFID技术的IC卡。可以替代现在大量的IC卡（包括信用卡）场合商场刷卡、IPASS、门禁管制、车票、门票等等。此种方式下，有一个极大的优点，那就是卡片通过非接触读卡器的RF域来供电，即便是寄主设备（如手机）没电也可以工作。NFC设备若要进行卡片模拟（Card Emulation）相关应用，则必须内置安全组件（Security Element, SE）之NFC芯片或通过软件实现主机卡模拟(Host Card Emulation，HCE)。
- 读卡器模式（Reader/Writer mode）：作为非接触读卡器使用，比如从海报或者展览信息电子标签上读取相关信息。
- 点对点模式（P2P mode）：这个模式和红外线差不多，可用于数据交换，只是传输距离较短，传输创建速度较快，传输速度也快些，功耗低（蓝牙也类似）。将两个具备NFC功能的设备链接，能实现数据点对点传输，如下载音乐、交换图片或者同步设备地址薄。因此通过NFC，多个设备如数位相机、PDA、计算机和手机之间都可以交换资料或者服务。

> [维基百科-近场通信](https://zh.wikipedia.org/zh-cn/%E8%BF%91%E5%A0%B4%E9%80%9A%E8%A8%8A)
> [百度-NFC](https://baike.baidu.com/item/%E8%BF%91%E5%9C%BA%E9%80%9A%E4%BF%A1/9741433?fromtitle=nfc&fromid=5684&fr=aladdin)

## 各种NFC卡的区别
| 卡       | 功能描述                                                                                                                |
| :------- | :---------------------------------------------------------------------------------------------------------------------- |
| 普通IC卡 | 0扇区不可以修改，其他扇区可反复擦写，我们使用的电梯卡、门禁卡等智能卡发卡商所使用的都是 M1 卡，可以理解为物业发的原卡。 |
| UID 卡   | 普通复制卡，可以重复擦写所有扇区，主要应用在IC卡复制上，遇到带有防火墙的读卡器就会失效。                                |
| CUID 卡  | 可擦写防屏蔽卡，可以重复擦写所有扇区，UID卡复制无效的情况下使用，可以绕过防火墙。                                       |
| FUID 卡  | 不可擦写防屏蔽卡，此卡的特点0扇区只能写入一次，写入一次变成 M1 卡，CUID 复制没用的情况下使用，可以绕过防火墙。          |
| UFUID 卡 | 高级复制卡，我们就理解为是 UID 和 FUID 的合成卡，需要封卡操作，不封卡就是 UID 卡，封卡后就变为 M1 卡。                  |
>[知乎-司小凯-UID卡、IC卡、ID卡、CUID 卡、FUID 卡、UFUID 卡 的区别](https://zhuanlan.zhihu.com/p/351266514)
## NFC标签类型
目前iOS系统并没有开放过多的NFC权限，所以这里讨论通过Android系统操作NFC标签，以下是Android系统操作NFC标签支持的标签技术类型
| Class            | Description                                                                            |
| :--------------- | :------------------------------------------------------------------------------------- |
| TagTechnology    | 这是所有标签技术类都必须实现的接口。                                                   |
| NfcA             | 提供对 NFC-A (ISO 14443-3A) 属性和 I/O 操作的访问权限。                                |
| NfcB             | 提供对 NFC-B (ISO 14443-3B) 属性和 I/O 操作的访问权限。                                |
| NfcF             | 提供对 NFC-F (JIS 6319-4) 属性和 I/O 操作的访问权限。                                  |
| NfcV             | 提供对 NFC-V (ISO 15693) 属性和 I/O 操作的访问权限。                                   |
| IsoDep           | 提供对 ISO-DEP (ISO 14443-4) 属性和 I/O 操作的访问权限。                               |
| Ndef             | 提供对 NDEF 格式的 NFC 标签上的 NDEF 数据和操作的访问权限。                            |
| NdefFormatable   | 为可设置为 NDEF 格式的标签提供格式化操作。                                             |
| MifareClassic    | 提供对 MIFARE Classic 属性和 I/O 操作的访问权限（如果此 Android 设备支持 MIFARE）。    |
| MifareUltralight | 提供对 MIFARE Ultralight 属性和 I/O 操作的访问权限（如果此 Android 设备支持 MIFARE）。 |
>[Android开发文档-高级NFC概览](https://developer.android.google.cn/guide/topics/connectivity/nfc/advanced-nfc?hl=zh_cn)

## NDEF数据格式
这里讲的很详细：
>[NDEF技术规范](http://article.iotxfd.cn/RFID/NDEF)

## Mifare Classic标签（M1卡）
NFC 有很多协议，其中 MIFARE Classic 基于 ISO 14443-3 Type A 标准，里面有一些 MIFARE 的命令。通过这些命令，就可以控制 MIFARE Classic 卡的内容。[MIFARE Classic EV1 4K S70](https://www.nxp.com/docs/en/data-sheet/MF1S70YYX_V1.pdf)
>[杰哥的{运维,编程,调板子}小笔记-MIFARE Classic 上配置 NDEF](https://jia.je/hardware/2020/05/10/mifare-classic-ndef/)
### Sector&Block 标签的内存布局
在 MIFARE Classic 中，有 Sector 和 Block 的概念，每个 Sector 有若干个 Block，其中最后一个 Block 是特殊的（称为 Sector Trailer），保存了这个 Sector 的一些信息：Key A、Access Bits、GPB 和 Key B。对于 Classic 4K，首先是 32 个有 4 blocks 的 sector，（M1卡）整体的内存布局大概是：
``` shell
Sector 0:
	Block 0 (出厂时写入了标签ID，厂商信息等，不可修改)
	Block 1
	Block 2
	Block 3(Sector Trailer)
Sector 1:
	Block 4
	Block 5
	Block 6
	Block 7(Sector Trailer)
...
Sector 15:
	Block 60
	Block 61
	Block 62
	Block 63(Sector Trailer)
```
Sector Trailer 的布局如下：
| Key A | Access Bits | GPB   | Key B |
| :---- | :---------- | :---- | :---- |
| 6字节 | 3字节       | 1字节 | 6字节 |
### A、B密钥与控制位
其中Access Bits（控制位）决定了密钥A、B对每个Block的读写权限，关于控制位详解参见：
>[CSDN-小流氓哥哥-IC卡 M1卡 各个扇区 控制块 密码 详解](https://blog.csdn.net/hanlinhe111/article/details/121100198)
### 读取写入等操作命令
- 认证：
    一条认证指令：0x60 0x05 0xFF 0xFF 0xFF 0xFF 0xFF 0xFF（0x60：使用密钥A[0x61：使用密钥B]，0x05：sector1-block1就是指从0开始编号的第n个块，后面6个16进制数即是密钥A）
- 读取：
    一条读取指令：0x30 0x05 （0x30：读取命令，0x05：sector1-block1就是指从0开始编号的第n个块）
    这里是要读取扇区1块0的内容
- 写入：
    一条写入指令：0xA0 0x06 0x64 0x6C 0x2F 0x62 0x75 0x73 0x69 0x6E 0x65 0x73 0x73 0x2F 0x3F 0x74 0x3D 0x6E（0xA0：写入命令，0x06：sector1-block2就是指从0开始编号的第n个块，后面为16个16进制数组成的要写入的内容）
    这里要写入扇区1块1的内容为：646C2F627573696E6573732F3F743D6E
- 内容格式：
    文字内容需要将文字对应的Ascii码值转化为16进制

借用微信的能力使用小程序对NFC标签进行读写的时候需要将命令内容转化为二进制内容：
>[微信开放社区提问的评论内容](https://developers.weixin.qq.com/community/develop/doc/0004e61e77ccb8187a5b325415b400)
### 认证与读写流程
针对扇区的读取与写入操作都需要使用密钥进行验证，验证成功后就可以执行相关的读写命令

## NDEF on Mifare Classic
NDEF 只定义了数据格式，但为了实际使用，还得看具体情况。就好像文件内容保存在硬盘上的时候，并不是直接保存，而是通过文件系统，人为定义一个路径，这样大家才知道要从 /etc/shadow 文件去读 Linux 的用户密码信息，NDEF 也需要人为定义一些规则，再作为数据存放在智能卡里的某个地方，这样大家去读取 metadata，发现上 NDEF Tag，然后才会去解析 NDEF 信息。

有些时候，这个定义很简单，比如直接把 NDEF 数据放在某个 block 里面；有的时候又很复杂，因为可能同时存在很多应用，NDEF 只是其中的一种，所以要有一种类似目录的东西去索引 NDEF“文件”。

MIFARE Classic 上采用的方法上，在特定的 Sector（比如 Sector 0）放一些元数据，元数据里注明了其他的 Sector（从 1 开始的其它 sector）分别用于什么用途，然后 NDEF 是其中一种用途。这个结构叫做 [MIFARE Application Directory](https://www.nxp.com.cn/docs/en/application-note/AN10787.pdf)。具体来说，在 MIFARE Classic 里面，它规定 Block 1 和 Block 2 的内容如下：
| 0-1        | 2-3  | 4-5  | 6-7  | 8-9  | 10-11 | 12-13 | 14-15 |
| :--------- | :--- | :--- | :--- | :--- | :---- | :---- | :---- |
| Info & CRC | AID  | AID  | AID  | AID  | AID   | AID   | AID   |
| AID        | AID  | AID  | AID  | AID  | AID   | AID   | AID   |

第一个字节是 CRC 8，它的定义可以在这里的 [CRC-8/MIFARE-MAD](https://reveng.sourceforge.io/crc-catalogue/1-15.htm) 里找到：初始值 0xC7，多项式上 0x1D。参与 CRC 计算的是按顺序从第二个字节开始的 31 个字节。

第二个字节是 Info Byte，用处不大，见 MAD 的文档。

之后每两个字节对应一个 Sector 的 AID（Application ID），比如 Block 1 的 2-3 字节对应 Sector 1 的 AID，Block 1 的 4-5 字节对应 Sector 2 的 AID，最后 Block 2 的 14-15 字节对应 Sector 15 的 AID。NDEF 的 AID 就是 0x03 0xE1。当软件发现这里的 AID 是 0x03E1 的时候，它就会去相应的 Sector 去读取 NDEF 信息。

省流助手：[0x14 0x01]可以理解为标记扇区0为索引，[0x03 0xE1]可以理解为标记扇区为NDEF消息储存空间。所以一个将NDEF信息记录在Mifare Classic标签上的数据形式大概类似这样：
``` shell
Sector0
    Block0 E7 CA C1 B3 5F 08 04 00 02 78 B1 F6 C9 6F FA 1D（芯片ID与厂商信息）
    Block1 14 01 03 E1 03 E1 03 E1 03 E1 03 E1 03 E1 03 E1（相当于所有扇区的类型索引）
    Block2 03 E1 03 E1 03 E1 03 E1 03 E1 03 E1 03 E1 14 01（相当于所有扇区的类型索引）
    Block3 A0 A1 A2 A3 A4 A5 78 77 88 C1 B0 B1 B2 B3 B4 B5（密钥A：A0A1A2A3A4A5 控制位：787788[表示扇区0密钥A只读，密钥B读写] C1[C1实际上不参与控制，可以用来替换成其他的内容保存用户数据] 密钥B：B0B1B2B3B4B5）
Sector1
    Block0 48 45 4C 4C 4F 59 4F 52 4C 44 00 00 00 00 00 00（自定义的数据内容，翻译：HELLOWORLD）
    Block1 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00（没有填写内容）
    Block2 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00（没有填写内容）
    Block3 FF FF FF FF FF FF 78 77 88 C1 FF FF FF FF FF FF（密钥与控制位）
Sector2
    Block0
    Block1
    Block2
    Block3
......

Sector16
    Block0
    Block1
    Block2
    Block3
```
## 通过微信小程序读写标签
以上内容，主要是基础知识的补充，下面是真正的干货demo

[这里是我开源的用来对MifareClassic芯片（M1芯片）进行读写的项目](https://github.com/MangK/NFCTools-MiniProgram)

## The end
>再次感谢以上文章引用中提到的博主与作者！！！