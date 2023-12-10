Applealc customization
ALC255(3234)for DELL Precision 3630
LayoutID 101
An unmodified CodecCommander.kext v2.4.0 needs to be loaded and the microphone is available.
Applealc customization ALC255(3234)for DELL Precision 3630 LayoutID 101 An unmodified CodecCommander.kext v2.4.0 needs to be loaded and the microphone is available.
此版本AppleALC拉取的1.8.6版本，支持macOS14，在此基础上，定制了DELL Precision 3630图形工作站的板载ALC255声卡，注入代码101，可驱动ALC255(3234)。
Resources文件夹中有制作好的文件，可加入AppleALC中运行。
具体方法是：
第一步：
App Store下载Xcode
拉AppleALC
git clone https://github.com/vit9696/AppleALC
拉SDK
git clone https://github.com/acidanthera/MacKernelSDK
会在小房子里把SDK文件夹放在AppleALC根目录
按 https://github.com/acidanthera/MacKernelSDK 所写配置Xcode，其实也不用配置啥。
下载lilu的debug版本放在AppleALC根目录
安装homebrew编译环境
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
选中科大版本。
完成后，就配置好了编译环境。
第二步：
使用verbit.sh，可生成节点信息
用法
verbit.sh codec#0 > ALC256_dump.txt
这样，ALC256_dump.txt里面就包含了节点信息，其中
Codec: Realtek ALC3234
是告诉了你的声卡型号
Address: 0
会告诉你生成configdata的数据的前缀是0,比如上面显示输出信息最后一行的Modified Verbs in One Line:后面所有的数据中,每组数据的第一位就是这个 0 ,如果Address: 2,那么每组数据的第一位就是 2 ,这个后面我们会用到它
DevID: 283902549 (0x10ec0255)
这是10进制（十六进制）值，0x10ec代表vendorID(芯片供应商ID)，指REALTEK，0255指型号，所以声卡型号是ALC255。


Verbs from Linux Codec Dump File: /Users/sige/Desktop/codec#0

Codec: Realtek ALC3234   Address: 0   DevID: 283902549 (0x10ec0255)

   Jack   Color  Description                  Node     PinDefault             Original Verbs
--------------------------------------------------------------------------------------------------------
Unknown Unknown  Line Out at Ext N/A         18 0x12   0x40000000   01271c00 01271d00 01271e00 01271f40
 Analog Unknown  Speaker at Int N/A          20 0x14   0x90170130   01471c30 01471d01 01471e17 01471f90
    1/8   Black  Speaker at Ext Rear         23 0x17   0x411111f0   01771cf0 01771d11 01771e11 01771f41
    1/8   Black  Speaker at Ext Rear         24 0x18   0x411111f0   01871cf0 01871d11 01871e11 01871f41
    1/8   Black  Speaker at Ext Rear         25 0x19   0x411111f0   01971cf0 01971d11 01971e11 01971f41
    1/8   Black  Speaker at Ext Rear         26 0x1a   0x411111f0   01a71cf0 01a71d11 01a71e11 01a71f41
 Line Out at Ext Rear    0x1b 0x1b                        16846880 01b71c20 01b71d10     01b71e01 01b71f01  
    RCA UNKNOWN  SPDIF Out at Ext N/A        29 0x1d   0x4044c029   01d71c29 01d71dc0 01d71e44 01d71f40
    1/8   Black  Speaker at Ext Rear         30 0x1e   0x411111f0   01e71cf0 01e71d11 01e71e11 01e71f41
    1/8   Black  HP Out at Ext Front         33 0x21   0x0221103f   02171c3f 02171d10 02171e21 02171f02
--------------------------------------------------------------------------------------------------------


   Jack   Color  Description                  Node     PinDefault             Modified Verbs
--------------------------------------------------------------------------------------------------------
Unknown Unknown  Line Out at Ext N/A         18 0x12   0x40000000   01271c00 01271d00 01271e00 01271f40
 Analog Unknown  Speaker at Int N/A          20 0x14   0x90170130   01471c30 01471d00 01471e17 01471f90
 Line Out at Ext Rear    0x1b 0x1b                        16846880 01b71c20 01b71d10     01b71e01 01b71f01  
    RCA UNKNOWN  SPDIF Out at Ext N/A        29 0x1d   0x4044c029   01d71c40 01d71dc0 01d71e44 01d71f40
    1/8   Black  HP Out at Ext Front         33 0x21   0x0221103f   02171c50 02171d10 02171e21 02171f01
--------------------------------------------------------------------------------------------------------

Modified Verbs in One Line: 01271c00 01271d00 01271e00 01271f40 01471c30 01471d00 01471e17 01471f90 01b71c20 01b71d10 01b71e01 01b71f01 01d71c40 01d71dc0 01d71e44 01d71f40 02171c50 02171d10 02171e21 02171f01
--------------------------------------------------------------------------------------------------------



通过verbit.sh工具，上边列出的结果显示，声卡共有10个节点，其中下边的5个节点是有效节点，5个节点中，PinDefault一列0x40打头的是屏蔽设置。
剩下3个小效节点

   Jack   Color  Description                  Node     PinDefault             Modified Verbs
--------------------------------------------------------------------------------------------------------

 Analog Unknown  Speaker at Int N/A          20 0x14   0x90170130   01471c30 01471d00 01471e17 01471f90
                 Line Out at Ext Rear        27 0x1b     16846880   01b71c20 01b71d10 01b71e01 01b71f01  
    1/8   Black  HP Out at Ext Front         33 0x21   0x0221103f   02171c50 02171d10 02171e21 02171f01
--------------------------------------------------------------------------------------------------------

再通过黑果小兵的方法，在linux环境下，使用demsg命令得知
0x19       25    Headset Mic 前置音频二合一麦克风插口（黑色） 

这里的问题是：通过Linux环境找到的0x19节点在MacOS中没有电压，只能通过CodecCommander 2.40版本提供的额外的电压才能工作。
最终确定：
有效节点    十进制  设备名称                                 路径
------------------------------------------------------------------------------------
0x21       33    HP out at Ext Front 前置耳机插口中（黑色）  33-12-2
0x1b       27    Line Out at Ext 线路输出（后置插口，绿色）   27-13-3
0x14       20    Speaker at Int 内置扬声器                 20-12-2
再通过linux的demsg命令得知
0x19       25    Headset Mic 前置音频二合一麦克风插口（黑色）  8-35-25,9-34-25
以下内容为configdata计算
 Node     PinDefault               Description                 
---------------------------------------------------------------
18 0x12   0x40000000         [N/A] Line Out at Ext N/A         
20 0x14   0x90170130       [Fixed] Speaker at Int N/A          
23 0x17   0x411111f0         [N/A] Speaker at Ext Rear         
24 0x18   0x411111f0         [N/A] Speaker at Ext Rear         
25 0x19   0x411111f0        [Jack] Headset Mic         
26 0x1a   0x411111f0         [N/A] Speaker at Ext Rear         
27 0x1b   0x16846880        [Jack] Line Out at Ext N/A        
29 0x1d   0x4044c029         [N/A] SPDIF Out at Ext N/A        
30 0x1e   0x411111f0         [N/A] Speaker at Ext Rear         
33 0x21   0x0221103f        [Jack] HP Out at Ext Front         
---------------------------------------------------------------
分离，去掉无用的字符
 Node      c  d  e  f       Description                 
----------------------------------------------------------------
12         40 00 00 00        [N/A] Line Out at Ext N/A        
14         90 17 01 30      [Fixed] Speaker at Int N/A         
17         41 11 11 f0        [N/A] Speaker at Ext Rear        
18         41 11 11 f0        [N/A] Speaker at Ext Rear        
19         41 11 11 f0       [Jack] Headset Mic         
1a         41 11 11 f0        [N/A] Speaker at Ext Rear        
1b         16 84 68 80       [Jack] Line Out at Ext N/A        
1d         40 44 c0 29        [N/A] SPDIF Out at Ext N/A       
1e         41 11 11 f0        [N/A] Speaker at Ext Rear        
21         02 21 10 3f       [Jack] HP Out at Ext Front        
---------------------------------------------------------------- 
小端转换
 Node      c  d  e  f       Description                 
----------------------------------------------------------------
12         00 00 00 40        [N/A] Line Out at Ext N/A      
14         30 01 17 90      [Fixed] Speaker at Int N/A       
17         f0 11 11 41        [N/A] Speaker at Ext Rear      
18         f0 11 11 41        [N/A] Speaker at Ext Rear      
19         f0 11 11 41       [Jack] Headset Mic              
1a         f0 11 11 41        [N/A] Speaker at Ext Rear      
1b         80 68 84 16       [Jack] Line Out at Ext N/A      
1d         29 c0 44 40        [N/A] SPDIF Out at Ext N/A     
1e         f0 11 11 41        [N/A] Speaker at Ext Rear      
21         3f 10 21 02       [Jack] HP Out at Ext Front      
----------------------------------------------------------------
屏蔽无效节点
 Node      c  d  e  f       Description                 
----------------------------------------------------------------
12         f0 00 00 40        [N/A] Line Out at Ext N/A 
14         40 01 17 90      [Fixed] Speaker at Int N/A  
17         f0 00 00 40        [N/A] Speaker at Ext Rear 
18         f0 00 00 40        [N/A] Speaker at Ext Rear 
19         70 10 ab 02       [Jack] Headset Mic         
1a         f0 00 00 40        [N/A] Speaker at Ext Rear 
1b         80 40 01 01       [Jack] Line Out at Ext N/A 
1d         f0 00 00 40        [N/A] SPDIF Out at Ext N/A
1e         f0 00 00 40        [N/A] Speaker at Ext Rear 
21         30 10 2b 02       [Jack] HP Out at Ext Front 
---------------------------------------------------------------- 
configdata计算
 Node              c        d       e        f    Description                 
----------------------------------------------------------------
01271f         01271cf0 01271d00 01271e00 01271f40                [N/A] Line Out at Ext N/A 
01471f         01471c40 01471d01 01471e17 01471f90 01470c02     [Fixed] Speaker at Int N/A  
01771f         01771cf0 01771d00 01771e00 01771f40                [N/A] Speaker at Ext Rear 
01871f         01871cf0 01871d00 01871e00 01871f40                [N/A] Speaker at Ext Rear 
01971f         01971c70 01971d10 01971eab 01971f02               [Jack] Headset Mic         
01a71f         01a71cf0 01a71d00 01a71e00 01a71f40                [N/A] Speaker at Ext Rear 
01b71f         01b71c80 01b71d40 01b71e01 01b71f01 01b70c02      [Jack] Line Out at Ext N/A 
01d71f         01d71cf0 01d71d00 01d71e00 01d71f40                [N/A] SPDIF Out at Ext N/A
01e71f         01e71cf0 01e71d00 01e71e00 01e71f40                [N/A] Speaker at Ext Rear 
02171f         02171c30 02171d10 02171e2b 02171f02 02170c02      [Jack] HP Out at Ext Front 
---------------------------------------------------------------- 
计算结果
01271cf0 01271d00 01271e00 01271f40 01471c40 01471d01 01471e17 01471f90 01470c02 01771cf0 01771d00 01771e00 01771f40 01871cf0 01871d00 01871e00 01871f40 01971c70 01971d10 01971eab 01971f02 01a71cf0 01a71d00 01a71e00 01a71f40 01b71c80 01b71d40 01b71e01 01b71f01 01b70c02 01d71cf0 01d71d00 01d71e00 01d71f40 01e71cf0 01e71d00 01e71e00 01e71f40 02171c30 02171d10 02171e2b 02171f02 02170c02
去掉40结尾的屏蔽端口最终结果可精减为
01471c40 01471d01 01471e17 01471f90 01470c02 01971c70 01971d10 01971eab 01971f02 01b71c80 01b71d40 01b71e01 01b71f01 01b70c02 02171c30 02171d10 02171e2b 02171f02 02170c02

Platforms101.xml的PathMap键值下
0是耳机输入
1下的0是内置扬声器1是耳机输出
2下的0是线路输出
这是PathMap中的结构
PathMaps
   |
   |
   |
   |----0（输入设备1）
   |    |
   |    |
   |    |----0（前置耳机输入）----0（线路1）----0（节点8）
   |         |                 |
   |         |                 |------------1（节点35）
   |         |                 |
   |         |                 |------------2（节点25）
   |         |
   |         |
   |         |-----------------1（线路2）----0（节点9）
   |                           |
   |                           |------------1（节点34）
   |                           |
   |                           |------------2（节点25）
   |
   |
   |
   |
   |----1（输出设备1，里边有内置扬声器和卫机输出）
   |    |
   |    |
   |    |----0（内置扬声器）------0（线路1）----0（节点20）
   |    |                       |
   |    |                       |------------1（节点12）
   |    |                       |
   |    |                       |------------2（节点2）
   |    |
   |    |
   |    |----1（前置耳机输出）----0（线路1）----0（节点33）
   |                            |
   |                            |------------1（节点12）
   |                            |
   |                            |------------2（节点2）
   |
   |
   |
   |
   |----2（输出设备2，里边有线路输出）
        |
        |
        |----0（线路输出）----0（线路1）----0（节点27）
                             |
                             |------------1（节点13）
                             |
                             |------------2（节点3）
