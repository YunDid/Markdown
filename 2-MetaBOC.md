# MetaBOC

### 功能概述：

这个代码实现了一个基于 PyQt5 的 GUI 应用程序，主要用于实时处理和可视化 MEA（微电极阵列）信号数据与 Intan 信号数据。主要功能包括：

- **系统设置**：支持两种设备类型（MEA2100 和 INTAN），可以配置数据保存路径和其他参数。
- **任务控制**：能够启动/停止任务记录，更新当前时间和记录时间戳。
- **数据分析与可视化**：实时分析并绘制 MEA 信号。
- **用户界面**：包含菜单栏、工具栏、状态栏以及各种对话框（如关于对话框）。



### 可能的扩展点：

1. **数据处理**：
   - 当前代码只展示了如何获取和绘制 MEA 信号，可能需要进一步实现数据预处理、滤波、特征提取等功能。
2. **用户界面增强**：
   - 添加更多的控件（如滑块、旋钮）来调整绘图参数或数据分析选项。
   - 改进状态栏或消息框的显示，提供更详细的反馈信息。
3. **文件管理**：
   - 实现数据文件的加载功能，以便进行回放分析。
   - 增加文件导出格式，如 HDF5 格式或其他自定义格式。
4. **多线程支持**：
   - 当前代码可能在主线程中处理所有任务，对于大数据量或实时性要求高的场景，可以考虑使用多线程或异步编程来提高性能。





### Recording.py

> 这段代码实现了一个用于记录和处理通道数据的基础类，主要是通过McsUsb设备与硬件进行交互，采集神经信号，并实现数据保存和处理。它包含了设备连接、数据采集、信号处理、spike检测等功能，并提供了一些暴露的接口供外部调用。

#### 1 主要方法

- **`__init__`**: 构造函数，初始化设备连接、数据模式、通道配置等，准备好数据采集相关的设置。
- **`device_setting`**: 设置设备的工作模式，包括采样率、数据模式、通道配置、触发阈值等。
- **`start_dacq` 和 `stop_dacq`**: 启动和停止数据采集。
- **`plot_channel_data` 和 `plot_channel_data_voltage`**: 数据可视化方法，显示某个通道的采集数据。
- **`get_recording`**: 主要的方法，返回左侧和右侧的spike检测结果。
- **`get_recording_signal_all_channel`**: 获取所有通道的信号数据。
- **`get_one_second_signal` 和 `get_target_time_signal`**: 获取特定时间段的数据。
- **`OnChannelData` 和 `OnError`**: 数据读取和错误回调函数。

#### 2 执行流程和暴露接口

1. **设备连接与配置**
   - 在`__init__`方法中，代码通过`CMcsUsbListNet`类查找所有连接的设备，并将第一个设备连接起来。
   - 设备连接后，配置采样率、数据模式和通道数量等，并根据硬件提供的电压范围设置增益和电压范围。
2. **数据采集**
   - `start_dacq`方法启动数据采集，`stop_dacq`方法停止数据采集。`OnChannelData`方法是设备每次返回新数据时的回调函数，负责获取数据并进行后续处理。
3. **数据处理与存储**
   - `get_recording`方法是获取数据并执行`spike detection`的核心方法，它将对每个通道的信号进行处理，分为左通道和右通道进行spike检测。
   - 数据采集的数据可以通过`ChannelDataSaving`线程类进行保存，`read_data_func`负责将数据保存到指定路径，并进行缓存。
4. **信号处理与Spike检测**
   - `get_spike_data_from_channel_data`方法会在特定的刺激时刻后10ms内剔除数据，从而去除不需要的信号干扰。
   - `spike_detection_para`和`spike_det_left/right`是负责进行spike检测的参数和对象，`spike_det_left`和`spike_det_right`分别用于检测左侧和右侧的spike。

#### 3 暴露的接口

1. **`get_recording_signal`**: 允许外部获取某个通道的信号数据，并可以指定返回的数据类型（原始数据或已处理的数据）。
2. **`get_recording_signal_all_channel`**: 返回所有通道的数据。
3. **`set_save_state`**: 设置是否保存数据的状态。
4. **`set_record_para` 和 `set_spike_detection_para`**: 设置记录参数和spike检测的参数。
5. **`clear_recording_buffer`**: 清空缓存区的数据，并进行标记，控制缓存的清空和数据存储。





### stimulation.py

该类负责处理刺激设备的连接、设置刺激信号、施加刺激、以及管理刺激的状态。它通过硬件接口与设备进行通信，并根据不同的信号进行相应的电极刺激控制。

#### 类的主要方法：

1. **`__init__`**
   - 构造函数，初始化设备列表，配置电极通道映射，并设置刺激参数、记录状态等。它还初始化了一些标志变量来控制电极的设置状态。
2. **`initial_device`**
   - 初始化设备，连接硬件设备，并配置一些初始的参数（例如电压、电流、触发输入等）。它还调用设备的设置函数来配置触发器、输出模式等。
3. **`PollHandler`**
   - 该方法是设备的事件回调，用来处理设备状态变化的事件。它通过设备状态判断是否开始或结束刺激，并在刺激结束时发出`stimulate_finshed`信号。
4. **`set_sti_signal`**
   - 该方法接收一个信号对象，解析其中的刺激参数（如刺激幅度、时长等），并设置相应的刺激信号和刺激电极。该方法通过分析信号的参数，构建实际的刺激信号数组。
5. **`set_reward_sti`**
   - 设置奖励刺激的信号，它会定义一个固定的奖励刺激（例如：100Hz、75mV、100ms）并将其存储到相应的刺激信号中。
6. **`update_amplitude_duration`**
   - 用于动态修改刺激信号的幅度和时长。
7. **`update_record_stimulation`**
   - 用于更新并施加刺激信号到设备。该方法通过接收左侧和右侧的刺激频率，并根据比较的结果选择相应的刺激通道进行施加。
8. **`update_stimulation`**
   - 用于施加实际的刺激信号，调用设备接口准备并发送数据，包括设置电极、DAC通道、启用电极等。
9. **`update_stimulation_stg1` 和 `update_stimulation_stg2`**
   - 分别处理两种类型的刺激通道（Stg1和Stg2）。它们负责设置电极的模式、启用电极、设置DAC通道，并最终通过设备发送刺激信号。
10. **`reset_electrode_stimulating_state`**
    - 重置所有电极的刺激状态，断开所有电极的连接，并关闭刺激模式。
11. **`disconnect`**
    - 断开与设备的连接。

### 执行流与设备通信

1. **设备连接与初始化：**
   - 当`Stimulation`对象被创建时，`initial_device`方法会被调用来连接到刺激设备并进行初始化。初始化时，程序会设置一些基础参数，例如电压范围、电流范围、触发器等。设备连接成功后，它会准备好接收刺激信号。
2. **设置刺激信号：**
   - 通过调用`set_sti_signal`方法，程序将设置具体的刺激信号（幅度、时长等），并指定刺激电极。刺激信号会按照解析的参数（例如奖励刺激或其他特定刺激信号）来构建。
3. **发起刺激：**
   - 刺激的发起是通过调用`update_record_stimulation`方法完成的。根据传入的左侧和右侧刺激频率，系统会选择相应的刺激通道（`Stg1`或`Stg2`）来进行信号的施加。
   - 在`update_stimulation_stg1`和`update_stimulation_stg2`方法中，程序会为每个电极设置相应的模式、启用电极、设置DAC通道、禁用ADC信号等，之后通过`PrepareAndSendData`和`SendStart`方法将数据发送到硬件，启动刺激信号。
4. **记录刺激数据：**
   - 在每次刺激开始时，刺激的幅度、时长和电极数据会被记录并传输到设备，用于后续的分析。`recording.clear_recording_buffer_get_sti_time`方法会被调用来清空缓存并保存刺激时刻的数据。
5. **处理刺激结束：**
   - 设备会周期性地回调`PollHandler`方法，通过设备状态判断刺激是否结束。当刺激完成时，`stimulate_finshed`信号会被发出，表明刺激已成功完成。

### 刺激施加的执行流：

1. **初始化与连接：** 在`__init__`和`initial_device`方法中，设备被初始化并准备好接收刺激信号。
2. **设置刺激信号：** 通过`set_sti_signal`方法设置刺激信号，包括幅度、时长、刺激电极等。
3. **施加刺激信号：** 通过`update_record_stimulation`方法和`update_stimulation_stg1` / `update_stimulation_stg2`方法，设备将信号施加到指定的电极上。
4. **记录刺激数据：** 刺激开始时会记录相关数据（如幅度、时长等）并保存。
5. **刺激完成回调：** 设备通过`PollHandler`回调通知刺激是否完成。





# 截至 2025/1/14 工作记录

- 左轮为什么要给两个刺激电极？而且还是两种触发器？

  > 因为左右轮需要区分高低频刺激，用于编码不同距离信息，目前就编码为两种，远or近。

- 可以一个 setting 绑定 4个电极。

  > intan 目前配置为 8个 setting，可以映射到对应的电极，用哪几个，设置哪几个就行，剩余的为默认配置。

- 有设置 有触发 需要把一开始的参数设置改了,也需要把触发的位置也改了。

  > 触发的时机没变，直接改底层调用接口。

- metaboc 昨天的工作部分需要汇总到对应的文档中。

  

**Todo：**

- metaboc 尾迹剔除位置在哪里改。

- 断点改刺激 list 为什么会超出维度。

- 奖励刺激会覆盖之前的刺激这个问题，后续用于一些任务的时候需要处理一下。

- 需要统计为什么只触发 f1 f3 高频刺激？

  > 设置的阈值 threshold = 100，**这个阈值的设置并不准确，需要后续看一下如何设置。**

- **借此机会对 metaboc 平台三维部分进行分析。  -----周四组会需要提及的**

  > **说明一下之前只是写接口，现在对三维的刺激部分也做了分析。**

- intan 新机型需要多做一些测试。

- 这里需要写活一些，使其能够同时适应多个刺激电极通道，3,4刺激通道能够发生切换

 ```python
  # 高频刺激
  self.stimulating.configure_stimulation([self.left_electrode_key[0],self.left_electrode_key[1]], amp[0], dur[0], trigger=self.trigger8[0])
  self.stimulating.configure_stimulation([self.right_electrode_key[0],self.right_electrode_key[1]], amp[2], dur[2], trigger=self.trigger8[2])
  # 低频刺激
  self.stimulating.configure_stimulation([self.left_electrode_key[2],self.left_electrode_key[3]], amp[1], dur[1], trigger=self.trigger8[1])
  self.stimulating.configure_stimulation([self.right_electrode_key[2],self.right_electrode_key[3]], amp[3], dur[3], trigger=self.trigger8[3])
  # self.stimulating.configure_stimulation([self.left_electrode_key[1]], amp[1], dur[1], trigger=self.trigger8[1])
 ```







# 截至 2025/2/7 工作记录

------

### 1. **`QActionGroup` 是什么？**

- **用途：** 它专门用来管理“操作项”（ QAction ）。

- **常见场景：** 这些操作项通常在菜单栏、工具栏或者快捷键中出现。

- **例子：** 假设你有一个文字处理软件，菜单栏里有三个选项：

  - 文字加粗（ QAction1）
  - 文字斜体（ QAction2）
  - 文字下划线（ QAction3）

  如果这些操作项属于同一个 `QActionGroup`，那么用户一次只能选择其中一种格式。当你点击“文字加粗”时，“文字斜体”和“文字下划线”就会自动取消选中。

- **通俗解释：** `QActionGroup` 是菜单栏或工具栏上的“选项组”，确保你一次只能选一个操作。

------

### 2. **`QButtonGroup` 是什么？**

- **用途：** 它用来管理按钮控件（比如 RadioButton、PushButton 等）。

- **常见场景：** 这些按钮通常在界面上排列在一起，供用户直接点击选择。

- **例子：** 假设你有一个设置窗口，里面有三个RadioButton：

  - 深色主题
  - 浅色主题
  - 自动跟随系统

  如果这些RadioButton属于同一个 `QButtonGroup`，那么用户一次只能选一个主题。当你点击“深色主题”时，“浅色主题”和“自动跟随系统”就会自动取消选中。

- **通俗解释：** `QButtonGroup` 是界面上的“按钮组”，确保你一次只能选一个按钮。

------

### 3. **它们的区别在哪里？**

虽然 `QActionGroup` 和 `QButtonGroup` **都是用来实现互斥选择**，但它们管理的对象不同：

- **`QActionGroup` 管理的是菜单项或工具栏上的操作项（ QAction ）。**
  
  - 这些选项通常是隐藏的，不在界面上直接显示。
  - 例子：菜单栏里的“文件”、“编辑”等下拉菜单中的选项。
- **`QButtonGroup` 管理的是界面上的按钮控件（比如 RadioButton、PushButton 等）。**
  - 这些按钮是用户可以直接看到和点击的。
  - 例子：设置窗口中的RadioButton或单选按钮。

- 为什么pyqt下每一个UI文件会对应一个同名的py文件？

  > 为了将设计好的界面与业务逻辑分离，意思就是可视化设置UI后，自动生成py文件，用于python项目**执行时生成**ui界面。







# 截至 2025/2/11 工作记录





- 先基于 gpt 实现对某个类中主要方法和对象的概要总结
- 再去看执行流
- **设备相关类**：理解`CMcsUsbListNet`、`CMeaDeviceNet`等设备相关类的作用，特别是它们如何与硬件设备进行交互。
- 说实话要是再深入这几个接口去深入感觉没有必要，dll 提供的接口可以在后续有功能需要添加时再检索，目前就可以基于师兄对接口的调用方式进行梳理即可，出现问题时可以断点到问题所在，也可以在此基础上进行其他基于通道spike数据的操作。

- UI 逻辑也不会深入去过，没有必要。

- 框架式的明白刺激与记录的流，并能够对离线数据进行分析即可。

  > 对于数据的存储方式拓展性并不好，而且没有针对特定任务进行数据存储，待优化的地方其实有很多。
  >
  > 若后续在进行实验设计的时候需要限定数据存储跨度等功能时，此时需要对该模块进行优化，使平台本身具备方便可用的功能而不是从离线数据切入进行复杂分析。

- 已经来不及基于 dll 文档去深入看具体的初始化流程了

``` python 
def initial_device(self):
        self.connect_state = True

        self.device = CStg200xDownloadNet()
        # self.device = CStg200xStreamingNet()  # not recommend for stg

        self.device.Stg200xPollStatusEvent += self.PollHandler

        self.device.Connect(self.deviceList.GetUsbListEntry(0))

        memory = self.device.GetTotalMemory()
        # segment_num = 2
        # segmentmemory = np.uint16(memory / segment_num)
        # self.device.SegmentDefine(segmentmemory)

        nchannels = self.device.GetNumberOfAnalogChannels()
        nsync = self.device.GetNumberOfSyncoutChannels()
        channel_cap = []
        syncout_cap = []
        # for i in range(5):
        #     # self.device.SegmentSelect(i)
        #     # segment_mem = self.device.GetMemory()
        #
        #     for j in range(nchannels):
        #         channel_cap.append(segment_mem / (nchannels + nsync))
        #
        #     for m in range(nsync):
        #         syncout_cap.append(segment_mem / (nchannels + nsync))
        #
        #     self.device.SetCapacity(channel_cap, syncout_cap)

        self.voltageRange = self.device.GetVoltageRangeInMicroVolt(0)
        self.voltageResulution = self.device.GetVoltageResolutionInMicroVolt(0)
        self.currentRange = self.device.GetCurrentRangeInNanoAmp(0)
        self.currentResolution = self.device.GetCurrentResolutionInNanoAmp(0)

        print('Voltage Mode:  Range: %d mV  Resolution: %1.2f mV' % (
        self.voltageRange / 1000, self.voltageResulution / 1000.0))
        print('Current Mode:  Range: %d uA  Resolution: %1.2f uA' % (
        self.currentRange / 1000, self.currentResolution / 1000.0))

        # set up the trigger
        trigger_inputs = self.device.GetNumberOfTriggerInputs()
        print("There is %f trigger inputs."%(trigger_inputs))
        number_stg_source = self.device.GetNumberOfStimulationSourcesPerElectrode()

        channelmap = Array[UInt32]([1, 2])
        syncoutmap = Array[UInt32]([1, 2])
        repeat = Array[UInt32]([1, 1])

        # self.device.SetupTrigger(0,  Array[UInt32]([255]), Array[UInt32]([255]), Array[UInt32]([10]))
        self.device.SetupTrigger(0,  channelmap, syncoutmap, repeat)
        self.device.SetVoltageMode()
        self.reset_electrode_stimulating_state()
```

- 要不总结一下是基于什么数据结构实现的对刺激参数的设置 存储 与应用的？

- 有个问题，将数据记录与刺激控制置于主线程中，继承自object，其他任务置于其他线程中，这合理吗？



`QThread` 类为线程管理提供了多种功能，包括：

- **启动和管理线程**：通过继承 `QThread`，我们可以轻松启动、控制和停止后台线程。
- **后台任务执行**：通过重载 `run` 方法，子类可以定义线程要执行的具体任务。例如，在 `ChannelDataSaving` 中，`run` 方法定义了数据保存的操作，它将在后台线程中异步执行。
- **信号与槽机制**：`QThread` 支持 Qt 的信号和槽机制，这使得在后台线程中执行任务时，可以方便地与主线程进行通信（例如，执行完任务后通知主线程）。
- **不阻塞主线程**：由于任务在后台线程中运行，主线程（例如GUI线程）可以继续响应用户交互，不会因为数据保存等耗时任务而卡顿。

### 创建线程的过程

在创建 `ChannelDataSaving` 实例时，确实会为其创建一个新的线程，但是该线程的启动并不会立即执行，而是需要显式调用 `start()` 方法来启动线程。

- **线程启动**：调用 `start()` 方法时，Qt 会启动一个新的线程并调用 `run()` 方法。
- **`run()` 方法的执行**：`run()` 方法包含了线程要执行的具体操作（例如数据保存）。`run()` 方法是被子类重载的，用于执行具体的任务。
- **线程结束**：当 `run()` 方法执行完成后，线程会自动退出。



# 2025/2/17

- `__pycache__` 文件夹的存在是为了优化 Python 程序的执行，确保 Python 不需要每次都重新编译源代码，从而提升性能。你不需要手动管理这些文件，Python 会自动处理它们。你可以安全地删除这些 `__pycache__` 文件夹，它们会在下次运行代码时自动重新生成。

- MetaBOC 的目录结构一定要调整，在将对应部分的代码拆解完毕后，目前的目录结构过于冗余。

  > out,out_grasping,out_spikes,out_tracking。
  >
  > map.npz 电极映射数据。
  >
  > encode_decode 待加载的数据。
  >
  > 思路：
  >
  > 区分为 资源文件 resources（待加载的） 以及输出文件 outputs（生成的）
  
- 尽量按 MVM（Model-View-Controller）拆解控制逻辑，单独一个任务作为一个单独的继承对象。

``` markdown
E:\METABOC\BRAIN\SRC
├── core/                  # 核心业务逻辑
├── widgets/               # 自定义控件
├── dialogs/               # 对话框
├── system_device/         # 硬件相关
├── models/                # 数据模型
├── viewmodels/            # 界面逻辑
├── resources/             # 资源文件
│   ├── ui/               
│   │   ├── original/      # 原始.ui文件
│   │   └── generated/     # 生成的Ui_*.py
│   ├── qrc/              # QRC资源
│   └── styles/           # QSS样式表
├── tests/                 # 测试代码
├── docs/                 # 文档
├── scripts/              # 构建脚本
├── main.py               # 入口文件
└── requirements.txt      # 依赖列表
```

- 最上级以类型划分。

- 侧重点在于 robot 下的com,encode,robot,task 了。

