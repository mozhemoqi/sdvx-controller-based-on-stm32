# Sdvx controller based on stm32
# 一.方案简介 
使用stm32f103c8t6制作的控制器，包含6个按键以及两个旋钮。  
支持1000hz回报率以及高速的响应，8ms无阻塞式消抖，按下松开立即响应。  
使用硬件编码器接口，需保证编码器能产生波形，如使用ec11等应上拉改为电压输出。  
将程序烧录进stm32中，即可变成usb设备，将按键以及编码器正确连接即可。  
此方案重点提供代码部分以及亚克力切割面板文件，代码花费数小时编写完成并调试，目前运行稳定并未发现任何bug，手台具体制作过程不过于描述。
# 二.材料清单
1.购买stm32f103c8t6最小核心板 10  
2.购买烧录工具如 stlink（推荐），jlink 10         并百度keil烧录方法！！！       
3.360ppr的编码器或者其他线数的编码器 两个 60   
4.sdvx按键，推荐tb购买喵呜手台店的sdvx一套按键 40  
5.微动开关，霍尼韦尔v15s05-ez025-k02，欧姆龙的也可以，需购买7个 35  
6.xh2.54端子，2p x7 4p x2（建议多买不贵，端子容易焊坏），其他连接器也可，也可以飞线 10  
7.购买xh2.54的连接线同 2p和4p的连接线  价格和6中的端子加起来一个十块左右
8.切割亚克力面板 40  
9.购买木盒。40  
预估成本 245  
# 三.代码烧录以及接线顺序  
**注意：只给出了代码源码以及工程文件，pcb，亚克力草图，如何烧录代码和pcb怎么打印需要百度！！！** 
下载解压压缩包  
![Image](https://user-images.githubusercontent.com/105113020/266985680-279d3b45-5d15-4137-8ac3-d55ff37bda7d.png)   
引脚对应  
PA0 左编码器A相  
PA1 左编码器B相  
PA2 BT-A  
PA3 BT-B  
PA4 BT-C  
PA5 BT-D  
PA6 右编码器A相  
PA7 右编码器B相  
PB0 FX-L  
PB1 FX-R  
PB10 START  
使用stlink 或者jlink，打开工程文件(提前提前安装好mdk-arm)，选择调试设备并勾选调试模式为swd，swdio与swdclk对应引脚连接。  
也可以烧录hex文件，准备ch340，改为bootloader启动，使用isp烧录。  
下载整个压缩包即可，代码烧录部分可以百度stm32f103代码烧录方法，文件工程在压缩文件中，代码烧录完成后可以将端子的两个引脚短接一下看看电脑输入法是否有相应的反应，旋钮引脚由于悬空的原因，在未连线的情况下鼠标可能会乱跳属于正常现象，连接后即可正常使用，代码烧录完成基本就大功告成。
pcb部分在末尾，建议使用第一个核心板版本（目前文件只有第一个版本，后面那个大多数人不会用还很麻烦所以就不传了），嘉立创免费打板，百度搜索一下立创eda，打开工程文件可以查看板子原理图和pcb，搜索嘉立创每月领券打板两次即可白嫖，拿到板子后只有排母和xh座子需要焊接非常容易，编码器和按键微动开关的线焊接需要注意（微动开关焊接常开两个引脚）
淘宝买2p和4p的线和端子，2p用于按键不分方向只起到引脚和gnd导通的作用，4p用于编码器，一定要注意方向，编码器上面vcc和信号线颜色都有说明不要接反了，检查无误后再插电否则接反编码器容易烧毁
![image](https://github.com/mozhemoqi/sdvx-controller-based-on-stm32/assets/105113020/5d954a20-3bc7-4674-b702-8b4f821df375)

**建议使用pcb中的核心板版本，不容易出问题方便焊接** 

# 四.代码部分实现原理  
## 1.hid通信相关  
### 设备描述   
usbd_desc.c中  
控制器名称为sdvx controller，烧录代码后将usb连接至电脑可以看到设备名称  
![Image](https://user-images.githubusercontent.com/105113020/266962156-17c727f2-bb86-475c-8b84-fbceb94e2876.png)
![Image](https://user-images.githubusercontent.com/105113020/266962573-768430f2-dd36-4782-b629-139d830d68b8.png)  
### 配置描述符  
usbd_hid.h中  
![Image](https://user-images.githubusercontent.com/105113020/266999765-1ef8370e-6f6e-4384-9f73-67014508fa34.png)  
配置缓存为64字节（最大报文长度）。    
usbd_config.h中  
![Image](https://user-images.githubusercontent.com/105113020/267000699-1cd8b671-6435-4312-98ab-06efcf7ac4e7.png)  
定为fs设备传输速率，回报率为1000hz。  
## 报告描述符  
__ALIGN_BEGIN  static uint8_t HID_MOUSE_ReportDesc[HID_MOUSE_REPORT_DESC_SIZE]  __ALIGN_END =  
{  
 //鼠标  
    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)  
    0x09, 0x02,                    // USAGE (Mouse)  
    0xa1, 0x01,                    // COLLECTION (Application)  
    0x09, 0x01,                    //   USAGE (Pointer)  
    0xa1, 0x00,                    //   COLLECTION (Physical)  
    0x85, 0x01,					     //		ID(1)  
    0x05, 0x01,                    //     USAGE_PAGE (Generic Desktop)  
    0x09, 0x30,                    //     USAGE (X)  
    0x09, 0x31,                    //     USAGE (Y)  
    0x15, 0x81,                    //     LOGICAL_MINIMUM (-127)  
    0x25, 0x7f,                    //     LOGICAL_MAXIMUM (127)  
    0x75, 0x08,                    //     REPORT_SIZE (8)  
    0x95, 0x02,                    //     REPORT_COUNT (2)  
    0x81, 0x06,                    //     INPUT (Data,Var,Rel)  
    0xc0,                          //   END_COLLECTION  
    0xc0,                           // END_COLLECTION  

//键盘  
    0x05, 0x01,                    // USAGE_PAGE (Generic Desktop)  
    0x09, 0x06,                    // USAGE (Keyboard)  
    0xa1, 0x01,                    // COLLECTION (Application)  
    0x85, 0x02,					            //	 ID(2)  
  
    0x05, 0x07,                    //   USAGE_PAGE (Keyboard)  
    0x19, 0xe0,                    //   USAGE_MINIMUM (Keyboard LeftControl)  
    0x29, 0xe7,                    //   USAGE_MAXIMUM (Keyboard Right GUI)  
    0x15, 0x00,                    //   LOGICAL_MINIMUM (0)  
    0x25, 0x01,                    //   LOGICAL_MAXIMUM (1)  
    0x75, 0x01,                    //   REPORT_SIZE (1)  
    0x95, 0x08,                    //   REPORT_COUNT (8)  
    0x81, 0x02,                    //   INPUT (Data,Var,Abs)  
  
    0x95, 0x01,                    //   REPORT_COUNT (1)  
    0x75, 0x08,                    //   REPORT_SIZE (8)  
    0x81, 0x03,                    //   INPUT (Cnst,Var,Abs)  

    // 0x95, 0x05,                    //   REPORT_COUNT (5)
    // 0x75, 0x01,                    //   REPORT_SIZE (1)
    // 0x05, 0x08,                    //   USAGE_PAGE (LEDs)
    // 0x19, 0x01,                    //   USAGE_MINIMUM (Num Lock)
    // 0x29, 0x05,                    //   USAGE_MAXIMUM (Kana)
    // 0x91, 0x02,                    //   OUTPUT (Data,Var,Abs)
    // 0x95, 0x01,                    //   REPORT_COUNT (1)
    // 0x75, 0x03,                    //   REPORT_SIZE (3)
    // 0x91, 0x03,                    //   OUTPUT (Cnst,Var,Abs)

    0x95, 0x07,                    //   REPORT_COUNT (7)
    0x75, 0x08,                    //   REPORT_SIZE (8)
    0x15, 0x00,                    //   LOGICAL_MINIMUM (0)
    0x25, 0x65,                    //   LOGICAL_MAXIMUM (101)
    0x05, 0x07,                    //   USAGE_PAGE (Keyboard)
    0x19, 0x00,                    //   USAGE_MINIMUM (Reserved (no event indicated))
    0x29, 0x65,                    //   USAGE_MAXIMUM (Keyboard Application)
    0x81, 0x00,                    //   INPUT (Data,Ary,Abs)
    0xc0,                          // END_COLLECTION
}; 
报告描述符我去掉了一些无关的数据，删除了鼠标按键，键盘发送最大允许7个按键同时按下（修改REPORT_COUNT 可增加数目）  

发送格式为  
鼠标：  
第一个字节固定为鼠标ID，0x01。  
第二个字节表示鼠标的X轴移动，取值范围为-127到127。  
第三个字节表示鼠标的Y轴移动，取值范围为-127到127。  
发送长度固定为3位。  
USBD_HID_SendReport(&hUsbDeviceFS,hidmouse_buffer,3);  
键盘;  
第一个字节固定为键盘ID，0x02。  
第二个字节表示特殊按键是否按下，无关写0x00;  
第三个字节定义为空，写0x00；  
第四到十个字节每一个字节表示按下的键值，例如0x04为A按下，0x00为未按下，  
发送长度为10位。**注意：长度由报告描述符决定，过长会报错，过短即使数据正确主机也不会做出反应。**  
USBD_HID_SendReport(&hUsbDeviceFS,hidkey_buffer,10);  
## 按键及旋钮扫描相关  
![Image](https://user-images.githubusercontent.com/105113020/267007524-0d403899-5b71-49c6-8e70-97eb3a70fc2f.png)  
**按键扫描机理**  
![Image](https://user-images.githubusercontent.com/105113020/267007931-bd024651-fa45-4912-aa00-494c697bc160.png)  
判断引脚电平状态为按下并且屏蔽计数为0则说明按键已经按下  
此时将按键标志位置位并且开始按下屏蔽计数同时对计数限幅  
按下屏蔽计数大于8（8ms）并且电平为松开时立即清除标志位并且开始松开屏蔽计数  
按下屏蔽计数和松开屏蔽计数同时大于8时将所有计数清零，此时按键处于复位状态  
**旋钮扫描机理**  
![Image](https://user-images.githubusercontent.com/105113020/267009250-b01c7273-44ce-4b8b-952d-93e6a5536954.png)  
使用了计数更加精准的编码器模式，cnt寄存器存储计数值，dir存储在cr寄存器的d4中  
**按键和旋钮发送机理**  
![Image](https://user-images.githubusercontent.com/105113020/267010787-a73c869f-67ba-407b-8e5f-cc725e6c40c6.png)  
按键发送只有当按键按下和松开瞬间发送标志位才会置1  
旋钮只有当cnt不为0时才会置标志位  
只有当所有发送标志位都为0时stm32才会向主机发送数据。  
## 轮询过程  
![Image](https://user-images.githubusercontent.com/105113020/267012185-d86e692c-d4f6-426b-8f1d-c20549f0e46e.png)  
**通过定时1ms在主循环执行发送过程**  
![image](https://github.com/mozhemoqi/Sdvx-controller-based-on-stm32/assets/105113020/8f7573a5-b40e-4e3a-bf98-e5f9b9ead332)
![image](https://github.com/mozhemoqi/Sdvx-controller-based-on-stm32/assets/105113020/8b75e11c-97c1-44ac-8f16-66702a975565)
# 五.CAD前面板
![Image](https://user-images.githubusercontent.com/105113020/267015096-b15d80ae-9837-4a34-8b0c-5eb7f599e57d.png)
**推荐切割5mm厚的亚克力板**
# 六.pcb  
![Image](https://user-images.githubusercontent.com/105113020/267015551-ba7941a9-583d-4637-9eab-c8e92844c5b2.png)  
**核心板版本，仅方便连接，使用立创白嫖既可，板子双面铺铜铺铜间距较短，注意焊接时不要烫坏阻焊导致引脚与gnd接触**  
![image](https://github.com/mozhemoqi/sdvx-controller-based-on-stm32/assets/105113020/734d7f1e-f461-4a05-b9c8-9fcb2d891dc1)
**此版本可用于diy或者自行增加其他外设或者功能，大部分引脚完全引出，rtc的32.768晶振可以不焊** 
