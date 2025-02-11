# MetaBOC

### 功能概述：

这个代码实现了一个基于 PyQt5 的 GUI 应用程序，主要用于处理和可视化 MEA（微电极阵列）信号数据。主要功能包括：

- **系统设置**：支持两种设备类型（MEA2100 和 INTAN），可以配置数据保存路径和其他参数。
- **任务控制**：能够启动/停止任务记录，更新当前时间和记录时间戳。
- **数据分析与可视化**：实时绘制 MEA 信号、时空矩阵图等，并将结果保存为图片或 CSV 文件。
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

# 2025/1/14

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







# 2025/2/7

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



