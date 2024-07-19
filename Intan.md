- In RHS systems, the Set Stim button allows you to configure stimulation settings for headstage
  channels, analog outputs, or digital outputs.

  > 在 RHS 系统中，“设置刺激”按钮允许您配置前置通道、模拟输出或数字输出的刺激设置。

- FPGA



- Intan 需要在连接成功后进行断点，目前只有连接失败的信息.
- 到底什么样的状态算做了连接成功，什么样的状态算作了初始化完成，初始化有哪工作.
- 做到什么样的程度，Qt构建对应模块，该模块称为Connect模块.
- 为什么Intan一开始需要设置采样率.
- 将一开始的采样率设置为了手动值，跳过GUI阶段.
- 明早带电脑过去看一下这些够不够把连通性跑通.



- open 与 上级 open 的接口文档.
- 单独导出该接口，并基于python成功调用该接口，则说明该部分已成功导出.
- 查看所以初始化工作，逐步解离所有初始化代码，直到整个设备初始化完成，该阶段完成.
- 再去与师兄探讨下一步所需基本dll模块，深入代码寻找.
- 一个方法一个方法的解离.

- 确定需要哪些部分的dll，这个dll又是如何使用的.
- MEA的dll是什么样的，它将什么样的功能进行封装导出.





- 我需要 Intan 地址
- 确定需要哪些部分的dll，这个dll又是如何使用的.
- MEA的dll是什么样的，它将什么样的功能进行封装导出.
- 我并不确定这个导出的dll是否可用
- 该项目同时也存在dll依赖dll的情况，我看python项目也存在，pyd于dll文件，检索过pyd 专用于python的动态链接库
- 时间问题，
- 整个连接控制流的梳理谁来，是不是需要我这边先梳理.



## 2024.1.3

- QString 该数据类型是否可以导入到py时仍可识别？
- createActions 接口.

```c++
createActions
// 用于创建菜单项和工具栏按钮的动作（QAction 对象），并将它们连接到相应的槽函数。
```

- initializeInterfaceBoard 接口.

```
initializeInterfaceBoard();
```

- changeSampleRate 接口.

```
changeSampleRate
```

- scanPorts();

```
scanPorts();
```

- MainWindow::MainWindow

```c++
MainWindow::MainWindow(int sampleRateIndex_, int stimStepIndex)
{
    // 采样率相关设置
    sampleRateIndex = sampleRateIndex_;
    stimStep = (Rhs2000Registers::StimStepSize)(stimStepIndex + 1);
	
    // 连通模式
    synthMode = false;
	
    // 刺激参数保存状态
    stimParamsHaveChanged = false;

    // Default reference source settings
    // 默认引用源设置
    referenceSource.channel = 0;
    referenceSource.stream = 0;
    referenceSource.softwareMode = false;

    // Default amplifier bandwidth settings
    // 默认放大器带宽设置
    desiredLowerBandwidth = 1.0;
    desiredLowerSettleBandwidth = 1000.0;
    desiredUpperBandwidth = 7500.0;
    desiredDspCutoffFreq = 1.0;
    dspEnabled = true;

    useFastSettle = false;
    headstageGlobalSettle = false;
    chargeRecoveryMode = false;
    chargeRecoveryCurrentLimit = Rhs2000Registers::ChargeRecoveryCurrentLimit::CurrentLimit10nA;
    chargeRecoveryTargetVoltage = 0.0;

    // Default electrode impedance measurement frequency
    // 默认电极阻抗测量频率设置
    desiredImpedanceFreq = 1000.0;

    actualImpedanceFreq = 0.0;
    impedanceFreqValid = false;

    // Set up vectors for 8 DACs on USB interface board
    // 为 USB 接口的数模转换设置向量
    dacEnabled.resize(8);
    dacEnabled.fill(false);
    dacSelectedChannel.resize(8);
    dacSelectedChannel.fill(0);

    cableLengthPortA = 1.0;
    cableLengthPortB = 1.0;
    cableLengthPortC = 1.0;
    cableLengthPortD = 1.0;

    chipId.resize(MAX_NUM_DATA_STREAMS);
    chipId.fill(-1);

    for (int i = 0; i < 16; ++i) {
        ttlOut[i] = 0;
    }
    evalBoardMode = 0;

    // ---------------------------------------------------------------------------------------------------------------------
    // 有一个槽机制需要解决，看是否可以一并导出dll，还是需要省略，还是别的方法代替
    numSpiPorts = openInterfaceBoard(expanderBoardConnected);

    int maxPossibleDataStreams = 2 * numSpiPorts;
    int usbBufferSize = MAX_NUM_BLOCKS_TO_READ * 2 * Rhs2000DataBlock::calculateDataBlockSizeInWords(maxPossibleDataStreams);
    cout << "MainWindow: Allocating " << usbBufferSize / 1.0e6 << " MBytes for USB read buffer." << endl;
    usbReadBuffer = new unsigned char [usbBufferSize];
    const unsigned int numSeconds = 10;  // size of RAM buffer, in seconds, assuming maximum sampling rate of...
    const unsigned int maxSamplingRate = 40000; // in Samples/s
    unsigned int fifoBufferSize =
            numSeconds * maxSamplingRate * 2 * (Rhs2000DataBlock::calculateDataBlockSizeInWords(maxPossibleDataStreams) / SAMPLES_PER_DATA_BLOCK);
    usbStreamFifo = new DataStreamFifo(fifoBufferSize);
    if (!synthMode) {
        usbDataThread = new UsbDataThread(evalBoard, usbStreamFifo, this);
        connect(usbDataThread, SIGNAL(finished()), usbDataThread, SLOT(deleteLater()));
    } else {
        usbDataThread = nullptr;
    }
    // ---------------------------------------------------------------------------------------------------------------------
    
    // Set dialog pointers to null.
    spikeScopeDialog = nullptr;
    stimParamDialog = nullptr;
    digOutDialog = nullptr;
    anOutDialog = nullptr;
    keyboardShortcutDialog = nullptr;
    helpDialogChipFilters = nullptr;
    helpDialogComparators = nullptr;
    helpDialogDacs = nullptr;
    helpDialogHighpassFilter = nullptr;
    helpDialogNotchFilter = nullptr;
    helpDialogFastSettle = nullptr;
    helpDialogReference = nullptr;
    helpDialogIOExpander = nullptr;
	
    // 事件触发器设置
    recordTriggerChannel = 0;
    recordTriggerPolarity = 0;
    recordTriggerBuffer = 1;
    postTriggerTime = 1;
    saveTriggerChannel = true;

    signalSources = new SignalSources(numSpiPorts);

    channelVisible.resize(MAX_NUM_DATA_STREAMS);
    for (int i = 0; i < MAX_NUM_DATA_STREAMS; ++i) {
        channelVisible[i].resize(CHANNELS_PER_STREAM);
        channelVisible[i].fill(false);
    }
	
    signalProcessor = new SignalProcessor();
    notchFilterFrequency = 60.0;
    notchFilterBandwidth = 10.0;
    notchFilterEnabled = false;
    signalProcessor->setNotchFilterEnabled(notchFilterEnabled);
    highpassFilterFrequency = 250.0;
    highpassFilterEnabled = false;
    signalProcessor->setHighpassFilterEnabled(highpassFilterEnabled);
	
    // 相关状态位设置
    running = false;
    recording = false;
    triggerSet = false;
    triggered = false;

    saveTtlOut = false;
    saveDcAmps = false;
    validFilename = false;

	// 波形绘制相关
    wavePlot = new WavePlot(signalProcessor, signalSources, this, this);

    connect(wavePlot, SIGNAL(selectedChannelChanged(SignalChannel*)),
            this, SLOT(newSelectedChannel(SignalChannel*)));
	// gui 初始化相关
    // 事件绑定 信号-槽机制
    createActions();
    createMenus();
    createStatusBar();
    createLayout();

    recordButton->setEnabled(validFilename);
    triggerButton->setEnabled(validFilename);
    stopButton->setEnabled(false);
	
    // 手动延迟设置
    manualDelayEnabled.resize(MAX_NUM_SPI_PORTS);
    manualDelayEnabled.fill(false);
    manualDelay.resize(MAX_NUM_SPI_PORTS);
    manualDelay.fill(0);

    if (!synthMode) {
        initializeInterfaceBoard();
    }

    changeSampleRate(sampleRateIndex, true);

    scanPorts();
    setStatusBarReady();

    if (!synthMode) {
        setDacThreshold1(0);
        setDacThreshold2(0);
        setDacThreshold3(0);
        setDacThreshold4(0);
        setDacThreshold5(0);
        setDacThreshold6(0);
        setDacThreshold7(0);
        setDacThreshold8(0);

        evalBoard->enableDacHighpassFilter(false);
        evalBoard->setDacHighpassFilter(250.0);
    }

    // Default data file format.
    setSaveFormat(SaveFormatIntan);
    newSaveFilePeriodMinutes = 1;

    // Default settings for display scale combo boxes.
    yScaleComboBox->setCurrentIndex(3);
    yScaleDcAmpComboBox->setCurrentIndex(3);
    yScaleAdcComboBox->setCurrentIndex(3);
    tScaleComboBox->setCurrentIndex(4);

    changeTScale(tScaleComboBox->currentIndex());
    changeYScale(yScaleComboBox->currentIndex());
    changeYScaleDcAmp(yScaleDcAmpComboBox->currentIndex());
    changeYScaleAdc(yScaleAdcComboBox->currentIndex());

    // QSound cannot play wav files directly from Qt resource files, so
    // this is a workaround:  Copy the wav file from the resource file to
    // a temporary directory, then play it from there when needed.
    QFile fileTemp(QDir::tempPath() + "/triggerbeep.wav");
    if (fileTemp.open(QIODevice::ReadWrite))
    {
        QFile resourceFile(":/sounds/triggerbeep.wav");
        if(resourceFile.open(QIODevice::ReadOnly))
        {
            fileTemp.write(resourceFile.readAll());
            resourceFile.close();
        }
        fileTemp.close();
    }

    QFile fileTemp2(QDir::tempPath() + "/triggerendbeep.wav");
    if (fileTemp2.open(QIODevice::ReadWrite))
    {
        QFile resourceFile2(":/sounds/triggerendbeep.wav");
        if(resourceFile2.open(QIODevice::ReadOnly))
        {
            fileTemp2.write(resourceFile2.readAll());
            resourceFile2.close();
        }
        fileTemp2.close();
    }
    adjustSize();
}
```



- 需要关注触发器的设置，观察是否与数据记录控制有关.
- 注释之后添加`.`作为末尾，否则中文注释容易爆C2065报错.
- 因为将 waveplot 部分的代码均注释掉了，因此存在的问题就是.
- 没有关灯.
- 熟练刺激流程.

![image-20240104161222573](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240104161222573.png)

![image-20240104164412817](C:\Users\Yundid\AppData\Roaming\Typora\typora-user-images\image-20240104164412817.png)

```
16:58:46: Starting E:\Intan-Link\LinkRHS\build-Intan_Yundid_StimRecordController-Desktop_Qt_5_12_12_MSVC2017_64bit-Debug\debug\Intan_Yundid_StimRecordController.exe ...
Rhs2000EvalBoard: Allocating 5.77536 MBytes for USB buffer.
---- Intan Technologies ---- Rhythm Stim RHS2000 Controller v1.0 ----


FrontPanel DLL loaded.  Built: Nov 16 2012  10:01:49

Scanning USB for Opal Kelly devices...

Found 1 Opal Kelly device connected:
  Device #1: Opal Kelly XEM6010LX45 with serial number 2118000W54

FPGA system clock: 100 MHz
Opal Kelly device firmware version: 3.1
Opal Kelly device serial number: 2118000W54
Opal Kelly device ID string: Opal Kelly XEM6010

Rhythm Stim configuration file successfully loaded.  Rhythm Stim version number: 1

Board mode: 14

MainWindow: Allocating 0.770048 MBytes for USB read buffer.
DataStreamFifo: Allocating 300.8 MBytes for FIFO buffer.
Scanning ports....
Port A cable delay: 0
Port B cable delay: 0
Port C cable delay: 5
Port D cable delay: 5
```





Dear Intan Technologies Support Team, 

I hope this message finds you well. My name is Ma Zhiyun, a first-year master's student at Tianjin University, currently engaged in a research project titled "Brain Organoid-Based External Device Control." Our team is dedicated to advancing neuroscientific research through innovative methodologies and technologies. 

As part of our project, we have developed a Python-based platform for controlling various external devices and are now aiming to integrate electrophysiological data acquisition and analysis capabilities. Given Intan's exemplary reputation in electrophysiological data recording and acquisition, we have chosen and purchased the RHS Stim/Recording System from your company for this integral part of our research. We are now hoping to receive some technical support from your esteemed team to further our endeavors.

To facilitate this integration, we kindly request your assistance in providing us with a Dynamic Link Library (DLL) that is compatible with Python. This DLL should ideally encompass basic functionalities for device control, data recording, and stimulation, which are essential for our research needs. 

We believe that incorporating Intan's technology into our research will significantly enhance our capabilities and contribute to the broader field of neuroscience. Any additional information on how best to implement this integration with our existing Python platform would be greatly appreciated. 

Thank you in advance for your time and assistance. We eagerly await your guidance and are hopeful for a positive collaboration.

Best regards,

Ma Zhiyun

Master's Student

Under the supervision of Professor Li Xiaohong

Tianjin University

mzy_0326@tju.edu.cn







Dear Intan Technologies Support Team,

I am writing to express my deepest gratitude for your prompt and insightful response to my previous query. Your willingness to assist and provide guidance is truly appreciated. The suggestion to utilize TCP control for the next phase of our project development is indeed valuable, and we are looking forward to implementing it.

We are currently evaluating two potential approaches for this integration: the TCP control method, as suggested by your team, and another method involving the creation of a Dynamic Link Library (DLL) that is **compatible with Python**, based on the source code provided by Intan. While we recognize the merits of the TCP control method, we have some concerns regarding its **real-time data transmission capabilities** when integrated with our platform. However, it should be noted that we also have limited experience in developing DLLs, which adds another layer of complexity to our decision-making process. Therefore, we are contemplating which approach would best suit our needs. 

I would like to reiterate our project's requirements to ensure a clear understanding for a better assessment.Our project involves developing a Python-based platform for controlling various external devices, and we are now looking to integrate electrophysiological data acquisition and analysis capabilities. To this end, we have chosen and purchased the RHS Stim/Recording System from Intan, known for its exceptional capabilities in electrophysiological data recording and acquisition. Our basic requirement is to have control over the hardware for stimulation, real-time recording and display on all or selected channels, and the ability to save data efficiently.

We would greatly appreciate it if your expert team could assess the feasibility, difficulty level, and real-time data transmission capabilities of two potential approaches for hardware control in our project. Your insights will be invaluable in helping us choose the most suitable solution for our project's requirements.

Once again, I extend my sincere thanks to the Intan Technologies Support Team for your time, support, and invaluable expertise. We eagerly await your assessment and further advice on this matter. 

Best regards,

Ma Zhiyun





# 2024/1/14

- 对于多通道实时传输问题，这是一种什么样的替代方案？

# 2024/1/15

- 重复连接存在无法连接问题.

  > ```
  > firstCommandConnection
  > ```
  >
  > 该状态变量控制在第一次连接时奏效，其他连接不再读取缓冲区.

- 信号到底什么时候发生改变啊？

  > 明确信号的来源，可能存在多个对象之间同名信号的传递.

- Socket 中等待队列的变化.

- 信号-槽 机制是否存在多次调用的情况.

- 信号发不出来.

# 2024/1/22

- 保存的数据文件格式是否统一？
- 即每个通道一份数据，每个类型一份数据在保存之后是否格式相同，处理方式相同？

- 数据的格式又是怎么样的，为什么parse时进行的处理又是几种类型的数据

# 2024/1/29

- magic 头？
- 了解新的数据块是如何附加到文件的.
- 指定时间跨度时可以指定读多少样本.
- 根据文件大小来计算增量.
- 是否有自己的滤波.

# 2024/1/30

- 文件监控的具体实现.

- 引入数据读取策略，完善数据的实时读取.

  > 最大缓冲区大小计算.

- 明确需要什么数据，确定自己的开发方向，基于 amp 进行开发？

# 2024/1/31

- 新时间戳会干扰对于数据的读取.
- 时间戳无法同步于创建进行监测.

``` import threading
from queue import Queue

class RealTimeDataReader:
    def __init__(self, directory_to_monitor):
        # 省略之前的初始化代码...

        # 新增队列和线程管理
        self.data_queue = Queue(maxsize=10000)  # 设置合适的缓冲区大小
        self.producer_thread = threading.Thread(target=self.data_loading_task)
        self.running = threading.Event()
        self.running.set()

    def start(self):
        self.observer.start()
        self.producer_thread.start()

    def stop(self):
        self.running.clear()
        self.observer.stop()
        self.observer.join()
        self.producer_thread.join()

    def data_loading_task(self):
        """生产者线程任务：监控文件变化并加载数据到队列中。"""
        while self.running.is_set():
            # 这里实现具体的数据加载逻辑
            # 示例：模拟从文件中读取数据并放入队列
            if self.t_fid and self.d_fids:
                for fid in self.d_fids:
                    try:
                        # 假设每次从文件中读取一个数据点
                        data = np.fromfile(fid, dtype=np.int16, count=1)
                        if data.size > 0:
                            self.data_queue.put(data[0])  # 将数据点放入队列
                    except Exception as e:
                        print(f"Error loading data: {e}")
            time.sleep(0.01)  # 控制读取频率，防止CPU过载

    # 省略其他方法的实现...
```





# 记录方案

找到记录按钮以及停止按钮触发

等待1s

找到存储文件调整样例

完善刺激参数调整样例，将所有可调的参数罗列出来


数据存储可实现流：

指令控制文件存储格式，开始记录数据，等待一段时间，停止记录数据。

# 刺激方案

调整刺激参数的可选模式

循环所有通道，实现对所有通道的刺激，随后取消刺激模式，等待1s，对下一通道进行相同修改，直到对所有通道实现了刺激的施加

添加记录逻辑，记录时是否可以施加刺激。



# 2024/3/5

- 可以先尝试测试一下延时.

- 文件连续读取问题.

- 文件监控系统逻辑还得改.

  > 多次存储的数据在进行绘制时是否需要精确到对应文件直接读取，而不是直接压入缓冲池缓冲处理.

希望是确定实验流程，确定需要什么样的逻辑后修改.

- 是否需要考虑读取速率与存储速率的影响？
- 得从指令发送端当记录停止时停止传输.
- 缓冲池中的数据需要区分开，是第几次记录的数据.



- 实验指令传输.
- 讨论一下下一步计划.
- 采样率的获取没有基于文件.
- 能完善的，需要完善的是文件监控和缓冲池数据写入逻辑.
- 考虑不全面，希望后续遇到问题是再去完善架构.



- 如果不设置每5min存储一个新的文件夹，那么一批次的数据是连续的，新文件夹的监控，只发生在手动控制停止开始记录操作阶段.
- Added file: E:/TCP/Data/1\tetts_240307_105415\stim-A-002.dat 这是什么数据？
- 多线程问题未解决



- 触发器映射

- TCP 触发器问题

- **Intan 刺激器.**

  > **同时刺激左右轮.**

- Intan 高频刺激如何实现.

- 同时给定刺激.

- **指令构建的时延 0.1s.**

  > 连续指令的构建.

- Intan 是否只针对于三维的需求.

- 三维脑类器官的刺激是否只能单点刺激，高频刺激如何实现.

- 长序列高频刺激如何实现. 

  > 不局限于单点刺激.

- 奖励刺激同时刺激至少32电极 

  > 刺激器连接多电极实现多电极实现多通道同时刺激.

- 刺激信号与触发之间

- f1 - 0-32通道

  > 是否会有问题.

- Intan 电极排列方式

  > 8*8 点选电极







# 2024/5/5

- read data 读取不到数据的原因以及解决方案.

  > 因为初始 read data 接口并不可直接用于由外部发起调用读取文件中的数据，而只能由load线程发起，将数据读取缓冲区.
  >
  > 如若我们直接通过 read data 读取，将导致读取到空数据问题，因为load 现成率先发起调用，导致文件描述符发生移动，使得外部读取不到数据.
  >
  > 解决方案： 
  > 重新构建了一个 read 接口读取100ms，1000ms 的数据，该数据来源是缓冲区，而不是直接读取文件.

- 其次，缓冲区还存在一个问题，就是每次存入缓冲区的数据是最大可读的文件数据，即每个元素的大小将包含超过 30000 个样本，而我们每次只读 2500个样本，因此在将数据存入缓冲区之间，进行了重组切割，将数据大块切割为小块存储，避免了一次读取过多数据导致丢失问题.

- 采样率问题.

  > 添加采样率获取接口，在对象创建前获取采样率即可，目前写死.

- 监控不到文件的问题.

  > 对象的创建时机.
  
- 缓冲池大小不受限问题.

  > 给个定时器，每隔一段时间执行即可.

- 检查了文件指针的指向问题，表明数据读取没有问题.



# 2024/5/27

- 不上传设置无法生效

- 记录期间修改任何参数都会导致usb连接中断，需要重新连接intan设备。

  > 因此不允许在 run mode 模式下修改任何刺激参数。

- 可以提前设置好所有映射关系

- 采样率可以在运行时获取





# 2024/6/29

- 确认监控到的文件类型，避免之后再抛异常。

  > 之前监控文件的逻辑处理不当，导致一些其他以.dat结尾的文件被识别为数据文件。

- 刺激文件初始化数组维度需要处理，以适应对应的文件设置与数量。

  > 为什么会额外创建一次。
  >
  > 因为刺激文件的filename并未添加于列表中，导致每次重新进入。

- 刺激数据可以记录到，但是注意刺激参数在设置的时候需要非0电流才能检测。

  > 目前仅能识别刺激时刻，但是刺激的具体数据并不知道具体意义。
  >
  > 因为乘步长，该步长并未提到什么含义。

- stim.dat 文件格式。

```python
15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  +-- bit 0: 电流幅值 (最低有效位)
|  |  |  |  |  |  |  |  |  |  |  |  |  |  +----- bit 1: 电流幅值
|  |  |  |  |  |  |  |  |  |  |  |  |  +-------- bit 2: 电流幅值
|  |  |  |  |  |  |  |  |  |  |  |  +----------- bit 3: 电流幅值
|  |  |  |  |  |  |  |  |  |  |  +-------------- bit 4: 电流幅值
|  |  |  |  |  |  |  |  |  |  +----------------- bit 5: 电流幅值
|  |  |  |  |  |  |  |  |  +-------------------- bit 6: 电流幅值
|  |  |  |  |  |  |  |  +----------------------- bit 7: 电流幅值 (最高有效位)-后八位最终转换为刺激数据
|  |  |  |  |  |  |  +-------------------------- bit 8: 符号位 (1 = 负电流)-即如果为1，则表示电流为负。
|  |  |  |  |  |  +----------------------------- bit 9: 未使用 (总是0)
|  |  |  |  |  +-------------------------------- bit 10: 未使用 (总是0)
|  |  |  |  +----------------------------------- bit 11: 未使用 (总是0)
|  |  |  +-------------------------------------- bit 12: 未使用 (总是0)
|  |  +----------------------------------------- bit 13: 未使用 (总是0)
|  +-------------------------------------------- bit 14: 状态位：放大器稳定激活
+----------------------------------------------- bit 15: 状态位：充电恢复激活
+----------------------------------------------- bit 16: 状态位：达到合规限制

# 刺激数据文件中每个刺激数据样本（一个时间戳对应一个刺激样本）是一个是一个16位无符号整数，和数据文件一样，一个样本2字节.
# 目前可根据 read_data 返回的刺激数据样本值是否非0来判断是否施加了刺激，（六个为一组，数值含义不明，但是可判断刺激是否施加）.
# read_data 返回三个数据，时间戳+放大器值+刺激样本，两者已经基于时间戳对齐.
```

- read_data 解析数据格式。

``` python
        stim_data = {
            'Stimdata': [],
            'compliance_limit': [],
            'charge_recovery': [],
            'amplifier_settle': []
        }
# 只需要使用 Stimdata 即可，其余三个状态位不涉及.
# Stimdata 为多维数组 {已设置刺激参数通道数，样本数}，零表示该时刻下无电流（无刺激），非零则代表有电流刺激，第一个非零时刻为刺激开始时刻.
# 刺激是针对通道而言，需要遍历到每个通道来看某个时刻的刺激，因为同一时刻001通道的刺激不是在002上记录到.
```

- 改进了文件监控逻辑。

  > 区分数据文件与刺激文件，防止混淆存储导致维度不匹配，无法存储数据。

- 完善data_load接口与read_data接口。

  > 使得实现对刺激数据的解析存储以及基于read_data返回基于时间戳对齐的刺激数据。

- 整体代码待测试是否可行。

  > 目前逻辑针对开启记录前必须设置好从参数，因此从data_load中直接默认有刺激参数列表，直接读取刺激于缓冲区中。
  >
  > read_data直接从队列中获取即可.



# 2024/6/30

- ~~通道数据标识问题.~~
- 数据延时问题.
- ~~剔除尾迹可视化问题.~~
- ~~64通道 C - D 不一定对应到 A - B~~
- ~~返回一个名称列表即可.~~

# 2024/7/1

- 添加通道与刺激名称列表，以便对应到数据.

- 主要问题是在监控时无法预先直到需要监控的数据文件的维度，或者说个数.

  > 目前通过状态变量设置，基本保证能够在监控到所有的新文件后再启动数据加载，并且维度与文件个数自适应.

- 可视化


