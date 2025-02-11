# MetaBOC - 意义

- 对于脑机接口有什么意义？

- 普通AI会到达什么样的瓶颈？

  > 普通AI已经达到可以很好的控制乒乓球，片上脑机接口相较而言有什么样的研究意义？

- 侧重于片上脑机接口的研究意义。

  > 无伦理限制，低能耗不具有说服性。
  >
  > 学的更快。

- 800000 个细胞的芯片是多大？

- Cortical Labs

- 人类神经元是自我编程的，具有无限的灵活性。

- 更低的电力成本。

- 原生智能

- 在进行任务之前要进行训练？



# 设计模式

### 委托事件机制

> 它允许一个函数在某个事件发生或某个操作完成时通知另一个函数。

**委托** - 类型

**事件** - 回调函数集合

> 使用委托类型来声明事件
>
> 基于事件实现**发布-订阅**模式

**回调函数** - 事件触发时的回调函数，订阅者

> 所有回调函数类型需要与委托类型的返回值与函数参数一致.
>
> 当事件触发时，依次调用所有注册的回调函数.

##### 3.1 委托事件机制执行流

**定义委托 `MessageHandler`**：

- 描述了事件处理方法的签名，接受一个 `string` 类型的参数，没有返回值。

**声明事件 `OnMessageReceived`**：

- 使用 `event` 关键字和 `MessageHandler` 委托类型，表示发布者可以触发 `OnMessageReceived` 事件。

**订阅事件 `HandleMessage`**：

- 订阅者通过 `pub.OnMessageReceived += HandleMessage` 将自己的方法 `HandleMessage` 绑定到发布者的事件上。

**触发事件**：

- 发布者调用 `OnMessageReceived(msg)` 触发事件，所有订阅了该事件的处理方法都会被调用。

# MetaBOC - 源码剖析

- 先跑通刺激和记录流。
- 后结合MetaBOC - main 代码查询如何做到与任务结合的闭环控制。

### Currrent 

1. 梳理刺激 demo 执行流程。
2. **剖析师兄如何基于该执行流程完成的刺激参数配置以及刺激施加。 - Next**

### 1 McsUsbNet.dll 接口

`McsUsbNet.dll` .NET库

> 需要安装.
>
> NET Framework 4.7.2
>
> Microsoft Visual C++ Redistributable for Visual Studio 2019
>
> 才能使用该 dll。

提供了与 MCS 设备通信的功能，包括数据采集和刺激控制等。

### 2 Stimulation 刺激 demo

``` python
import time
import os
import clr
# import clr：导入 pythonnet 提供的 Common Language Runtime 模块，允许 Python 调用 .NET 库。
clr.AddReference(os.getcwd() + '\\..\\..\\McsUsbNet\\x64\\\McsUsbNet.dll')
from System import Action
from System import *

from Mcs.Usb import CMcsUsbListNet
from Mcs.Usb import DeviceEnumNet
from Mcs.Usb import CStg200xDownloadNet
from Mcs.Usb import McsBusTypeEnumNet
from Mcs.Usb import STG_DestinationEnumNet

# 刺激发生器状态变化处理回调函数
def PollHandler(status, stgStatusNet, index_list):
    print('%x %s' % (status, str(stgStatusNet.TiggerStatus[0])))

# 搜索并连接设备：列举所有连接的 MCS USB 设备，并连接到第一个设备。
deviceList = CMcsUsbListNet(DeviceEnumNet.MCS_DEVICE_USB)

print("found %d devices" % (deviceList.Count))

for i in range(deviceList.Count):
    listEntry = deviceList.GetUsbListEntry(i)
    print("Device: %s   Serial: %s" % (listEntry.DeviceName,listEntry.SerialNumber))

# 实例化刺激器对象，订阅设备状态事件后，连接到设备刺激器。
device = CStg200xDownloadNet();
device.Stg200xPollStatusEvent += PollHandler;
device.Connect(deviceList.GetUsbListEntry(0))

# 获取设备参数：获取设备的电压和电流范围及分辨率，并打印出来。
voltageRange = device.GetVoltageRangeInMicroVolt(0);
voltageResulution = device.GetVoltageResolutionInMicroVolt(0);
currentRange = device.GetCurrentRangeInNanoAmp(0);
currentResolution = device.GetCurrentResolutionInNanoAmp(0);

print('Voltage Mode:  Range: %d mV  Resolution: %1.2f mV' % (voltageRange/1000, voltageResulution/1000.0))
print('Current Mode:  Range: %d uA  Resolution: %1.2f uA' % (currentRange/1000, currentResolution/1000.0))

# 设置刺激参数：定义刺激的通道映射、同步输出映射、重复次数、振幅和持续时间。
channelmap = Array[UInt32]([1,0,0,0])
syncoutmap = Array[UInt32]([1,0,0,0])
repeat = Array[UInt32]([10,0,0,0])

amplitude = Array[Int32]([-100,100]);
duration = Array[UInt64]([10000,10000]);

# 配置并发送刺激：设置触发器，指定电压模式，准备并发送刺激数据，然后启动刺激。
device.SetupTrigger(0, channelmap, syncoutmap, repeat)
device.SetVoltageMode();
device.PrepareAndSendData(0, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)
device.SendStart(1) 
time.sleep(10)

device.Disconnect()



# Out:
"""
PS D:\ZY\McsUsbNet_Examples\Examples\Python> & D:/DownloadCache/Anaconda/envs/MetaBOC/python.exe d:/ZY/McsUsbNet_Examples/Examples/Python/Stimulation.py
found 1 devices
Device: MEA2100-Mini   Serial: 81083-A
Voltage Mode:  Range: 10000 mV  Resolution: 0.50 mV
Current Mode:  Range: 1000 uA  Resolution: 0.05 uA
0 Idle
1000001 Running
1000000 Idle
PS D:\ZY\McsUsbNet_Examples\Examples\Python> 
"""
```



### 3 Namespace Mcs.usb 命名空间

``` python
namespace Mcs.Usb
{
    # stimulation 相关
    public class CMcsUsbListNet {}
    public enum DeviceEnumNet {}
    public class CStg200xDownloadNet {}
    public enum McsBusTypeEnumNet {}
    public enum STG_DestinationEnumNet {}
    
    # recording 相关
}
```

1. **CMcsUsbListNet**

- **类型**: 类（Class）

- **作用**: 主要用于列举和管理连接到系统的 MCS USB 设备。

- 常用方法:

  - `GetUsbListEntry(index)`: 获取指定索引的设备信息。
  - `Count`: 返回连接的设备数量。

- 使用示例:

  ```
  deviceList = CMcsUsbListNet(DeviceEnumNet.MCS_DEVICE_USB)
  print("Found %d devices" % deviceList.Count)
  ```

2. **DeviceEnumNet**

- **类型**: 枚举（Enum）

- **作用**: 定义设备枚举类型，用于指定不同类型的设备或连接方式。

- 可能的枚举值:

  - `MCS_DEVICE_USB`: 表示通过 USB 连接的 MCS 设备。
  - `MCS_DEVICE_ETH`: 表示通过以太网连接的 MCS 设备。
  - 等等。

- 使用示例:

  ```
  deviceList = CMcsUsbListNet(DeviceEnumNet.MCS_DEVICE_USB)
  ```

3. **CStg200xDownloadNet 重要**

- **类型**: 类（Class）

- **作用**: 这个类是 **McsUsbNet.dll** 库中用于控制 **STG200x** 系列刺激器设备的**核心类**。它提供了一系列方法和属性，用于连接设备、配置参数、发送刺激信号等操作。 

- 常用方法:

  - `Connect(entry)`: 连接到指定的设备。
  - `Disconnect()`: 断开设备连接。
  - `SetVoltageMode()`: 设置电压模式。
  - `PrepareAndSendData(...)`: 准备并发送刺激数据。
  - `SendStart(triggerIndex)`: 启动刺激信号。

- 使用示例:

  ```
  device = CStg200xDownloadNet()
  device.Connect(deviceList.GetUsbListEntry(0))
  ```

**4. McsBusTypeEnumNet**

- **类型**: 枚举（Enum）
- **作用**: 定义 MCS 总线类型，用于指定设备的连接方式（如 USB、以太网等）。

**5. STG_DestinationEnumNet**

- **类型**: 枚举（Enum）

- **作用**: 定义刺激信号的数据类型或目标，指定是发送电压数据还是电流数据。

- 可能的枚举值:

  - `channeldata_voltage`
  - `channeldata_current`
  - 等等。

- 使用示例:

  ```
  device.PrepareAndSendData(0, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)
  ```



### 4 Demo 刺激执行流

1. **搜索并连接设备**：列举所有连接的 MCS USB 设备，并连接到第一个设备。

```python
self.deviceList = CMcsUsbListNet(DeviceEnumNet.MCS_DEVICE_USB)
...
device = CStg200xDownloadNet();
device.Stg200xPollStatusEvent += PollHandler;
device.Connect(deviceList.GetUsbListEntry(0))
# CMcsUsbListNet(DeviceEnumNet.MCS_DEVICE_USB)：获取所有连接的 MCS USB 设备列表。
# device.Connect(deviceList.GetUsbListEntry(0))：连接到第一个设备。
```

2. **获取设备参数**：获取设备的电压和电流范围及分辨率，并打印出来。

> Voltage Mode:  Range: 10000 mV  Resolution: 0.50 mV
> Current Mode:  Range: 1000 uA  Resolution: 0.05 uA

``` python
voltageRange = device.GetVoltageRangeInMicroVolt(0)
voltageResulution = device.GetVoltageResolutionInMicroVolt(0)
currentRange = device.GetCurrentRangeInNanoAmp(0)
currentResolution = device.GetCurrentResolutionInNanoAmp(0)
# GetVoltageRangeInMicroVolt(0)：获取设备的电压范围（单位是微伏）。
# GetVoltageResolutionInMicroVolt(0)：获取设备的电压分辨率（单位是微伏）。
# GetCurrentRangeInNanoAmp(0)：获取设备的电流范围（单位是纳安）。
# GetCurrentResolutionInNanoAmp(0)：获取设备的电流分辨率（单位是纳安）。
```

3. **设置刺激参数**：定义刺激的通道映射、同步输出映射、重复次数、振幅和持续时间。

``` python
channelmap = Array[UInt32]([1, 2])
syncoutmap = Array[UInt32]([1, 2])
repeat = Array[UInt32]([10, 0])
amplitude = Array[Int32]([-100, 100])
duration = Array[UInt64]([10000, 10000])

# channelmap：定义刺激的通道映射。
# syncoutmap：定义同步输出映射。
# repeat：定义每个通道的重复次数。
# amplitude：定义刺激的振幅（单位为 μV）。
# duration：定义刺激的持续时间（单位为 μs）。
```

4. **配置并发送刺激**：设置触发器，指定电压模式，准备并发送刺激数据，然后启动刺激。

``` python 
device.SetupTrigger(0, channelmap, syncoutmap, repeat)
device.SetVoltageMode()
device.PrepareAndSendData(0, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)
device.SendStart(1)
# SetupTrigger(0, channelmap, syncoutmap, repeat)：设置刺激触发器，指定通道映射、同步输出映射和重复次数。
# SetVoltageMode()：设置设备为电压模式。
# PrepareAndSendData(0, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)：准备并发送刺激数据，设置通道、振幅和持续时间，参数需要设置到对应的MEA电极通道上。
# SendStart(1)：启动刺激。 参数代表不同的触发器例如 1+2 表示触发1,2触发器
```

1. **等待并断开连接**：等待刺激完成（10秒），然后断开设备连接。

``` python 
device.Disconnect()
```



# ToDo

``` python
PS D:\ZY\McsUsbNet_Examples\Examples\Python> & D:/DownloadCache/Anaconda/envs/MetaBOC/python.exe d:/ZY/McsUsbNet_Examples/Examples/Python/DeviceList.py
found 1 devices
Device: MEA2100-Mini   Serial: 81083-A
        
        
PS D:\ZY\McsUsbNet_Examples\Examples\Python> & D:/DownloadCache/Anaconda/envs/MetaBOC/python.exe d:/ZY/McsUsbNet_Examples/Examples/Python/Recording.py
found 1 devices
Device: MEA2100-Mini   Serial: 81083-A
Traceback (most recent call last):
  File "d:\ZY\McsUsbNet_Examples\Examples\Python\Recording.py", line 66, in <module>
    voltageRanges = device.HWInfo().GetAvailableVoltageRangesInMicroVoltAndStringsInMilliVolt(miliGain);
TypeError: 'CHWInfo' object is not callable
PS D:\ZY\McsUsbNet_Examples\Examples\Python> 
```

```python 
    nchannels = self.device.GetNumberOfAnalogChannels()

​    nsync = self.device.GetNumberOfSyncoutChannels()

trigger_inputs = self.device.GetNumberOfTriggerInputs()
print("There is %f trigger inputs."%(trigger_inputs))
number_stg_source = self.device.GetNumberOfStimulationSourcesPerElectrode()
```

- MetaBOC 如何解析的软件设置的刺激参数？

- 刺激相关的对象是如何初始化的？

- **一个刺激又是如何施加的？触发器？发生器？到最终执行，到刺激恢复等等，这样一整套流程是如何进行的？**

  > 先梳理代码执行流，后梳理文档中逻辑硬件执行流，再结合平台奖惩等一般刺激的执行流，进行汇总归一融合

![image-20240924143115213](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240924143115213.png)

`reset_electrode_stimulating_state` 函数的作用是将所有电极的状态重置为安全和非刺激状态。通过设置电极模式、DAC多路复用器、禁用电极、启用信号消隐和放大器保护开关，确保设备在重新开始刺激之前处于安全状态。这是确保设备正常运行和避免意外刺激的关键步骤。

- 为什么要在初始化的时候setup一次触发器？

- intan刺激器 触发器 分别有几个，

- 两种刺激模式？是什么意思？

  > 两个刺激器，同时只能够触发两种刺激模式（不包括同步非同时的切换模式）



mea-2100-mini 包含的刺激模式

![b3761999c29f2ef95caef7406df64f9](D:\DownloadCache\WeChat\Cache\WeChat Files\wxid_9xsyqltvnmso22\FileStorage\Temp\b3761999c29f2ef95caef7406df64f9.png)

![image-20240926233646408](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240926233646408.png)

headstage 名称

![d2aaf23a6dceb044f92c713a6e8510d](D:\DownloadCache\WeChat\Cache\WeChat Files\wxid_9xsyqltvnmso22\FileStorage\Temp\d2aaf23a6dceb044f92c713a6e8510d.png)



#### MEA2100-Mini系统组件

1. **头部放大器 (MEA2100-Mini)**

   - 提供60或120个记录通道，适用于湿润的孵育器环境。
   - 直接在**头部放大器上集成了放大器、刺激器和A/D转换器**，以减少噪声。
   - 支持多种刺激模式，如单相或双相矩形脉冲、斜坡、正弦波和随机波形，可以独立控制每个电极的刺激。

   **刺激控制**

   - 每个**头部放大器中集成了两个刺激生成器**，可以提供多种刺激波形。
   - 刺激的控制和设置通过Multi Channel Suite软件进行，用户可以根据需要选择刺激的电极。
   - 可以为不同的头部放大器选择不同的刺激方案，而对一个头部放大器所做的任何更改不会影响到其他头部放大器的刺激设置。

2. **信号收集单元 (SCU)**

   - **可连接多达四个头部放大器。**
   - 通过连接多个信号收集单元，支持多达**480个电极**的同时记录。
   - 提供控制多达四个独立光学刺激设备的接口。
   - 通过iX电缆与接口板（IFB-C）连接。

3. **接口板 (IFB-C Multiboot)**

   - 支持整个2100放大器解决方案的操作，包括多个类型的头部放大器。
   - 允许通过SYNC输出和输入连接多个接口板，以实现同步录音。

![image-20240926232030846](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240926232030846.png)



### SCU Stimulator与Headstage Stimulator的区别

1. **位置**：
   - **Headstage Stimulator**：集成在每个头部放大器中，负责直接对特定电极施加刺激。每个头部放大器可以独立控制其刺激器，允许不同的刺激设置。
   - **SCU Stimulator**：位于信号收集单元中，可以为多个头部放大器提供刺激信号。它通常用于协调和管理多个头部放大器的刺激任务。
2. **控制方式**：
   - SCU的刺激器可以通过**Multi Channel Experimenter**软件进行设置和控制，可以为连接到同一SCU的所有头部放大器提供统一的刺激策略。
   - 头部放大器中的刺激器则可以针对每个头部放大器进行单独的调整和优化。





该软件支持所有可用的ME2100-Mini-System头部放大器变体。一个信号收集单元（SCU）及其所有连接的头部放大器始终由一实例软件独立操作。图标指示代表哪个SCU，1和2表示连接到接口板端口1或2的信号收集单元。一旦将MEA2100-Mini-System添加到仪器配置中，MEA2100-Mini刺激器和SCU刺激器也将作为仪器可用。



![image-20240926235742396](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240926235742396.png)





- 在使用ME2100刺激器时，每个电极只能被分配给一个刺激器单元。也就是说，如果你选择了一个电极作为某个刺激器单元的刺激电极，那么这个电极就不能同时被分配给其他的刺激器单元。
- 一次只能将一个电极分配给一个刺激器装置。
- One electrode can only be assigned to one stimulator unit at a time.
- 在压控刺激中，每个刺激器可以选择任意数量的电极。如果在电流控制刺激中每个刺激器选择一个以上的电极，则施加的电流将以不可预测的方式根据其阻抗在所有选择的电极之间分配。因此，在电流模式下，每个刺激器装置仅允许一个刺激电极。
- Blanking disconnects all electrodes for a short period of time during the stimulation pulse, to reduce stimulation artefacts. **Blanking** 在刺激脉冲期间会短暂断开所有电极，以减少刺激伪影。
- Dedicated Stimulation Electrodes 专用刺激电极，一旦选定某些电极作为刺激电极，这些电极将一直保持与相应的刺激器单元连接，不会在记录信号和施加刺激之间进行切换。也就是说，这些电极将专门用于刺激，而不用于记录，从而减少伪影的干扰。
- 这将进一步减少伪影，但刺激电极会丢失，无法再编码。激活此功能后，刺激电极将显示噪声水平增加，
- 每个刺激器单元都有独立的控制选项，可以选择启动条件、加载或保存刺激模式等功能。
- 在数据源中，数字输出位可以分配给每个刺激仪，您必须为三个刺激装置中的每一个选择不同的位（参见5.2.4.1，数字输出位选择）







SCU刺激器和头部放大器（headstage）刺激器的主要区别在于以下几点：

1. **功能**：SCU刺激器**专注于控制Opto Stim**输出，主要用于光刺激，而头部放大器刺激器可以同时进行电刺激和记录信号。
2. **连接性**：SCU刺激器可以与多个头部放大器连接（如ME2100或MEA2100-Mini），而每个头部放大器通常只配备两个独立的刺激器单元。
3. **编程与控制**：SCU刺激器的刺激单元数量和颜色编码不同，**SCU通常有四个刺激单元（9到12）**，而**头部放大器刺激器有两个刺激单元**（1到2）。
4. **操作界面**：虽然两者的操作方式类似，但SCU刺激器与特定的Opto Stim功能结合得更紧密，适用于特定的光刺激应用。





STG400x STimulus Generator



- 在施加刺激前对电极做一些预设的设置，以便60个电极可以接受新的刺激模式？

- **DAC（Digital-to-Analog Converter）** 是将数字信号转换为模拟信号的设备。在生物电刺激系统中，DAC用于将数字化的刺激信号转换为连续的模拟电压或电流，以便施加到电极上进行刺激。





``` c#
uint electrode = 3;

// ElectrodeMode: emManual: electrode is permanently selected for stimulation
cStgDevice.SetElectrodeMode(electrode, ElectrodeModeEnumNet.emManual);

// ElectrodeDacMux: DAC to use for stimulation
// 第二个参数为 listmodeIndex，第三个指定刺激器，有四种刺激器
cStgDevice.SetElectrodeDacMux(electrode, 0, ElectrodeDacMuxEnumNet.Stg1);

// ElectrodeEnable: enable electrode for stimulation
cStgDevice.SetElectrodeEnable(electrode, 0, true);

// BlankingEnable: false: do not blank the ADC signal while stimulation is running
cStgDevice.SetBlankingEnable(electrode, false);

// AmplifierProtectionSwitch: false: Keep ADC connected to electrode even while stimulation is running
cStgDevice.SetEnableAmplifierProtectionSwitch(electrode, false);
```

listmodeIndex - 什么意思？

![image-20240927190440799](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240927190440799.png)





``` python

channelmap = Array[UInt32]([1, 2])
        syncoutmap = Array[UInt32]([1, 2])
        repeat = Array[UInt32]([1, 1])
        self.device.SetVoltageMode()
        self.device.SetupTrigger(0,  channelmap, syncoutmap, repeat)
        # self.device.SetupTrigger(0, Array[UInt32]([255]), Array[UInt32]([255]), Array[UInt32]([10]))
        self.device.PrepareAndSendData(1, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)
        
# 这儿无论什么刺激 怎么参数都是一样的？
```





- 有没有可能 channelmap 不等同于 channel？
- channel 有可能就是刺激器

![image-20240927192548848](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240927192548848.png)

![image-20240927192554148](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240927192554148.png)

![image-20240927192759364](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240927192759364.png)

![image-20240929205849427](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240929205849427.png)



- channelmap 又有多少？
- 连续发送两次不影响吗？

> 因为是 append data，所以不会出现重写。

``` python 
self.device.PrepareAndSendData(0, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)
        
        # 记录刺激时刻
        amp = asNumpyArray(amplitude, ctypes.c_int32); sync = []
        for i in range(len(amp)):
            if amp[i] > 0:
                sync.append(5 << 2)    #  1 << 8  2 << 8
            elif amp[i] < 0:
                sync.append(6 << 2)
            else:
                sync.append(0)
        sync = Array[UInt32](sync)
        self.device.PrepareAndSendData(0, sync, duration, STG_DestinationEnumNet.syncoutdata)
```

- 去连实机测一下channel输出的有几个，两个则说明与刺激器相关

  > 不是一个东西，这个 channel 输出的是获取的 block 数据块中的通道数。
  >
  > 2100 mini 返回的值为 77，其中前 60 个是数据通道。

![image-20240927193406424](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240927193406424.png)

- channel - > stg1 or stg 2 -> MEA 电极.

  > 其实 channel 可以看做刺激数据的逻辑载体。
  >
  > stg 是刺激器单元，用于生成刺激波形。
  >
  > channel 是刺激器内部的输出通道，用于与 MEA 电极建立绑定，发送具体刺激数据。
  >
  > 通过配置刺激器的输出通道，

- 实机测试一下两个刺激器构建一个pattern，如何触发.

  > MCS 软件文档与 dll 去找触发接口。

- 看一下listmode又是如何协调作用的.

  > mini 没这儿玩意儿。





``` python
    def update_stimulation_stg2_punish(self, eles_key, amplitude, duration):
        """
        在出现撞击时，进行奖惩刺激
        """
        # self.device.SendStop(UInt32(1))
        if not self.has_set_sti_ele_stg2:
            if not self.has_set_sti_ele_stg1 and not self.has_set_sti_ele_stg2:
                self.reset_electrode_stimulating_state()    # 都未连接的情况下，断开所有刺激器的连接

            self.has_set_sti_ele_stg2 = True


            for i in range(len(eles_key)):
                electrode = self.channel_map[eles_key[i]] - 1
                self.device.SetElectrodeMode(UInt32(electrode), ElectrodeModeEnumNet.emManual)

                # ElectrodeDacMux: DAC to use for stimulation
                self.device.SetElectrodeDacMux(UInt32(electrode), UInt32(0), ElectrodeDacMuxEnumNet.Stg2)

                # ElectrodeEnable: enable electrode for stimulation
                self.device.SetElectrodeEnable(UInt32(electrode), UInt32(0), True)

                # BlankingEnable: false: do not blank the ADC signal while stimulation is running
                self.device.SetBlankingEnable(UInt32(electrode), False)

                # AmplifierProtectionSwitch: false: Keep ADC connected to electrode even while stimulation is running
                self.device.SetEnableAmplifierProtectionSwitch(UInt32(electrode), False)

        channelmap = Array[UInt32]([1, 2])
        syncoutmap = Array[UInt32]([1, 2])
        repeat = Array[UInt32]([1, 1])
        self.device.SetVoltageMode()
        self.device.SetupTrigger(0,  channelmap, syncoutmap, repeat)
        # self.device.SetupTrigger(0, Array[UInt32]([255]), Array[UInt32]([255]), Array[UInt32]([10]))
        self.device.PrepareAndSendData(1, amplitude, duration, STG_DestinationEnumNet.channeldata_voltage)
        

        amp = asNumpyArray(amplitude, ctypes.c_int32); sync = []
        for i in range(len(amp)):
            if amp[i] > 0:
                sync.append(7 << 2)
            elif amp[i] < 0:
                sync.append(8 << 2)
            else:
                sync.append(0)
        sync = Array[UInt32](sync)
        self.device.PrepareAndSendData(1, sync, duration, STG_DestinationEnumNet.syncoutdata)

        self.device.SendStart(2)
        print("Start time SendStart", time.time())

        # 清空缓存，并传入对应的信号
        self.ele_key_sti = eles_key
        self.sti_data_amp = amplitude
        self.sti_data_dur = duration
        self.recording.clear_recording_buffer_get_sti_time(eles_key, amplitude, duration)

```



-  self.sti_sig = signal 这是什么？
- MEA 电极默认状态是接地？

- 去判断MetaBOC刺激设置与MCS刺激设置如何建立的关联？

- TriggerInputs 干啥用的？

- 我更希望是有16种 刺激器1与刺激器2刺激模式的组合方式，然后从中选取某种组合
  