### Rhs2000EvalBoard::open()

##### 功能描述： 

`open()` 函数用于查找并打开连接到USB端口的Opal Kelly XEM6010-LX45板。它负责加载必要的驱动程序、识别设备、并建立与设备的通信。

##### 参数：

无参数。

##### 返回值：

- `1`: 成功打开设备。
- `-1`: 无法加载FrontPanel DLL。
- `-2`: 未找到XEM6010-LX45设备。

##### 详细流程：

1. **加载FrontPanel DLL**: 首先尝试加载FrontPanel DLL。如果加载失败，返回错误代码 `-1`。
2. **设备扫描**: 扫描连接到USB的Opal Kelly设备，并打印设备信息。
3. **选择设备**: 查找类型为XEM6010LX45的设备，并获取其序列号。
4. **打开设备**: 使用获取的序列号尝试打开设备。如果打开失败，返回错误代码 `-2`。
5. **配置PLL**: 配置板载的PLL（相位锁定环）。
6. **获取设备信息**: 获取并打印FPGA系统时钟频率、固件版本、序列号和设备ID。



### DeviceManager::openInterfaceBoard(...)

##### 功能描述：

`openInterfaceBoard()` 函数是对 `open()` 函数的进一步封装，用于初始化与Intan RHS2000评估板的通信。它包括打开设备、加载配置文件、重置板卡、读取板卡模式等步骤。

##### 参数：

- `bool &expanderBoardDetected`: 用于指示是否检测到I/O扩展板的引用参数。

##### 返回值：

- `4`: 表示成功完成所有初始化步骤，并返回 SPI 端口数量，默认为4。

##### 详细流程：

1. **创建Rhs2000EvalBoard实例**: 实例化一个Rhs2000EvalBoard对象。
2. **打开设备**: 调用 `open()` 函数打开Opal Kelly XEM6310板。如果打开失败，将打印错误信息并退出应用程序。
3. **加载FPGA配置文件**: 加载FPGA配置文件 `main.bit`。如果加载失败，打印错误信息并退出应用程序。
4. **重置板卡**: 重置RHS2000评估板于初始态。
5. **读取板卡模式**: 读取并存储评估板的模式。
6. **检测I/O扩展板**: 读取数字输入，以检测是否连接了I/O扩展板。

### Rhs2000EvalBoard::initialize()

##### 功能：
初始化 Rhythm FPGA，设置其为默认起始状态。

##### 操作：

1. **复位 FPGA**：重置板卡，确保从已知状态开始。
2. **配置辅助命令**：启用所有数据流上的辅助命令。
3. **设置总线参数**：配置总线输出模式和全局静止策略。
4. **采样率设定**：设置默认的放大器采样率。
5. **数据流管理**：启动只有一个数据流，并关闭其他数据流。
6. **DAC 配置**：设置所有 DAC 为默认状态，包括数据流和通道选择。
7. **电缆长度设置**：为所有端口设置默认电缆长度。
8. **刺激器和序列器配置**：为模拟和数字输出配置刺激触发器和脉冲序列。

**注意**：该函数主要负责对 FPGA 进行软件层面的初始化和配置，确保 FPGA 能够按照预定的方式正常运行。



### MainWindow::initializeInterfaceBoard()

##### 功能描述: 
初始化连接到 USB 端口的 RHS2000 评估板接口，配置相关参数和设置。

##### 主要操作:

1. **初始化接口板**:
   - 初始化 RHS2000 评估板，将其设置为默认状态。
2. **配置 DC 放大器转换**:
   - 启用 DC 放大器转换功能。
3. **设置额外状态**:
   - 配置额外的状态参数。
4. **设置采样率和上传 SPI 命令**:
   - 根据采样率设置，上传辅助 SPI 命令序列。
5. **运行和处理 SPI 接口**:
   - 启动 SPI 接口，并在完成后读取单个数据块，用于配置放大器芯片和 ADC 校准。
6. **DAC 配置**:
   - 为接口板上所有 DAC 设置默认配置，包括启用/禁用、数据流和通道选择、手动值、增益和音频噪声抑制。
7. **电缆长度配置**:
   - 为每个端口设置电缆长度。



### Rhs2000EvalBoard::changeSampleRate(...)

#### 功能

此方法负责更改采样率并更新与之相关的设置。它主要在与 Rhs2000 评估板进行交互的上下文中使用，用于设置采样率、调整相关参数，并确保这些更改在硬件上生效。

#### 参数

- `int sampleRateIndex`: 用于选择采样率的索引。
- `bool updateStimParams`: 指示是否更新刺激参数的布尔标志。

#### 实现细节

1. **设置采样率**：
   - 根据传入的 `sampleRateIndex`，选择相应的采样率（如 20000Hz, 25000Hz, 30000Hz, 40000Hz）。
   - `numUsbBlocksToRead` 的设置旨在使帧率大致保持在 30Hz。
2. **配置波形绘图**：
   - 调用 `wavePlot->setNumUsbBlocksToPlot` 以更新绘图所需读取的 USB 块数量。
3. **寄存器设置**：
   - 使用所选的采样率初始化 `Rhs2000Registers` 对象。
   - 设置和调整各种寄存器配置，包括 MISO 采样延迟、辅助输出模式等。
4. **命令列表创建和上传**：
   - 根据需要创建和上传不同的命令列表到评估板的辅助命令槽位。
   - 包括设置 250 Hz 的零幅度正弦波、配置数字输出等。
5. **放大器带宽设置**：
   - 设置 DSP 截止频率、下限和上限带宽。
   - 根据是否启用了 DSP，更新 GUI 中相应的标签。
6. **滤波器设置**：
   - 设置陷波滤波器和高通滤波器的参数。
   - 如果不是合成模式，还会设置 DAC 的高通滤波器。
7. **界面更新**：
   - 更新 GUI 中显示的实际采样率和带宽设置。
   - 如果有启用的刺激参数，执行额外的配置步骤。
8. **其他操作**：
   - 在更改注册配置后，通过运行评估板的一次短暂运行来使更改生效。
   - 更新阻抗测量频率的有效性。
   - 将焦点返回到波形绘图。

#### 重要注释

- 此方法与评估板硬件紧密相关，涉及到底层寄存器的配置。

### MainWindow::findConnectedAmplifiers()

##### 功能概述：

- 此接口用于扫描连接到 RHS2000 接口板的 SPI 端口 A-D，以识别所有连接的 RHS2000 放大器芯片。
- 通过读取芯片上的 ROM 注册（寄存器 63）来确定每个端口上的放大器通道数量。
- 该过程在 FPGA 上所有可能的 MISO 延迟设置中重复进行，以便从中推断出每个端口的电缆长度。

##### 主要步骤：

1. 设置采样率以获得最大时间分辨率。
2. 为每个 SPI 端口启用数据流。
3. 运行 SPI 接口，并读取从 USB 接口返回的数据。
4. 读取并记录每个 RHS2000 芯片的 ID 号，以及与芯片良好通信的延迟设置。
5. 设置与每个 RHS2000 芯片良好通信的最佳电缆延迟设置。
6. 根据识别的芯片确定每个端口上的放大器通道总数，并据此配置 USB 数据流。
7. 更新 GUI 中的 SPI 端口显示，以反映连接的放大器芯片和通道。

##### 异常处理：

- 如果未检测到任何放大器芯片，则显示相应的消息，并提供重新扫描端口的选项。



### MainWindow::scanPorts()

#### 功能: 

此接口用于扫描连接到 RHS2000 接口板的 SPI 端口，以识别所有连接的 RHS2000 放大器芯片，并根据检测到的放大器配置相关系统组件。

#### 功能步骤:

1. **禁用按钮**:
   - 禁用运行（`runButton`）、记录（`recordButton`）和触发（`triggerButton`）按钮，防止在扫描过程中的操作。
2. **状态栏信息显示**:
   - 在状态栏显示“Scanning ports...”信息，告知用户正在进行端口扫描。
3. **扫描 SPI 端口**:
   - 调用 `findConnectedAmplifiers()` 方法扫描 SPI 端口，并识别连接的 RHS2000 放大器芯片。
4. **配置信号处理器**:
   - 根据检测到的数据流数量为 `signalProcessor` 分配内存。
   - 在合成模式（`synthMode`）下，分配固定数量的内存。
5. **LED 指示**:
   - 在非合成模式下，根据每个 SPI 端口的放大器连接状态，设置对应的 LED 指示灯。
6. **更新波形显示**:
   - 根据识别出的第一个有放大器连接的端口更新波形显示界面。
   - 如果没有检测到任何放大器，则弹出提示框并允许从模拟输入和数字输入端口进行记录。
7. **更新采样率和显示比例**:
   - 设置波形绘图组件的采样率。
   - 更新时间和电压显示比例。
8. **重新启用按钮**:
   - 根据是否有有效文件名（`validFilename`），重新启用运行、记录和触发按钮。
9. **清除状态栏信息**:
   - 扫描完成后，清除状态栏的消息。

#### 使用场景:

- 当需要检测和配置与 RHS2000 接口板连接的放大器设备时。
- 在启动设备或用户请求重新扫描端口时调用。











1. **核心组件**：
   - **FileSystemEventHandler**：用于处理各种文件系统事件的基类。
   - **Observer**：负责监控文件系统中的变化。
2. **事件处理方法**：
   - **on_created**：当文件或目录被创建时触发。
   - **on_deleted**：当文件或目录被删除时触发。
   - **on_modified**：当文件或目录被修改时触发。
   - **on_moved**：当文件或目录被移动时触发。
3. **功能**：
   - **递归目录监控**：能够监控指定目录及其所有子目录的变化。
   - **跨平台能力**：支持多个操作系统，如 Windows、macOS 和 Linux。
   - **实时文件系统监控**：能够实时响应文件系统的变化。
4. **使用案例**：
   - **文件创建检测**：监控特定目录，一旦有新文件创建，立即采取行动。
   - **目录监控**：监控整个目录树的变化，包括所有子目录。
   - **处理不同类型的文件系统事件**：针对文件或目录的创建、删除、修改和移动等事件编写特定的处理逻辑。