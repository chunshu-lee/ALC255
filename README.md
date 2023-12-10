Applealc customization

ALC255(3234)for DELL Precision 3630

LayoutID 101

An unmodified CodecCommander.kext v2.4.0 needs to be loaded and the microphone is available.

Applealc customization ALC255(3234)for DELL Precision 3630 LayoutID 101 An unmodified CodecCommander.kext v2.4.0 needs to be loaded and the microphone is available.

此版本AppleALC拉取的1.8.6版本，支持macOS14，在此基础上，定制了DELL Precision 3630图形工作站的板载ALC255声卡，注入代码101，可驱动ALC255(3234)。

Resources文件夹中有制作好的文件，可加入AppleALC中运行。

由于通过Linux环境找到的前置二合一麦克风0x19节点在MacOS环境中没有电压，只能通过CodecCommander 2.40版本提供的额外的电压才能工作。
