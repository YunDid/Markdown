# MetaBOC

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



### 4 刺激执行流

1. **搜索并连接设备**：列举所有连接的 MCS USB 设备，并连接到第一个设备。

2. **获取设备参数**：获取设备的电压和电流范围及分辨率，并打印出来。

   > Voltage Mode:  Range: 10000 mV  Resolution: 0.50 mV
   > Current Mode:  Range: 1000 uA  Resolution: 0.05 uA

3. **设置刺激参数**：定义刺激的通道映射、同步输出映射、重复次数、振幅和持续时间。

4. **配置并发送刺激**：设置触发器，指定电压模式，准备并发送刺激数据，然后启动刺激。

   1. **等待并断开连接**：等待刺激完成（10秒），然后断开设备连接。



### 5 委托事件机制

> 它允许一个函数在某个事件发生或某个操作完成时通知另一个函数。

**委托** - 类型

事件 - 回调函数集合

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

