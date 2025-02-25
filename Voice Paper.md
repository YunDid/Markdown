- RC引入部分

``` markdown
原文：

Reservoir computing (RC) initially emerged as a machine learning paradigm based on recurrent neural networks (RNNs), excelling at handling temporal and sequential information. An RC system comprises an input layer, a reservoir layer, and an output layer. The primary advantage of RC lies in the fact that it does not require training of the input and reservoir weights, necessitating only the training of output weights via supervised learning. This significantly reduces the computational cost compared to traditional RNN training. In RC, the reservoir layer's main function is to nonlinearly transform the input sequences into a higher-dimensional space, facilitating the differentiation of input features using simple learning algorithms. Consequently, various nonlinear dynamical systems can replace recurrent neural networks as the reservoir layer. 
原文翻译：
删减版本：
```



- BNNS RC 潜能部分

``` markdown
原文：
In vitro biological neural networks (BNNs), as complex nonlinear systems, exhibit high-dimensional dynamic responses to input stimuli. Numerous studies have explored their computational potential as reservoir layers in RC. Hafizovic initially utilized two electrodes to create spatiotemporal stimulation sequences as inputs, converting the network's spatiotemporal patterns into continuous reservoir states via a leaky integrator and using a support vector machine to classify the stimulation patterns. Subsequently, Dockendorf et al. employed a liquid state machine (LSM) to demonstrate the separation properties of in vitro BNNs using high- and low-frequency multi-site stimulation patterns, indicating the potential of in vitro BNNs for complex information processing. However, these studies employed relatively simple stimulation paradigms, with each stimulation involving only one or two electrodes, insufficient to fully demonstrate the computational capacity of in vitro BNNs. Therefore, George et al. utilized a set of electrodes with time-encoded inputs to generate numerous stimulation patterns, proving that in vitro BNNs could transform linearly inseparable inputs in low-dimensional space into linearly separable outputs in high-dimensional space. Ju et al. further demonstrated, using LSM and optogenetics to encode complex spatiotemporal information, that in vitro BNNs are capable of processing complex spatiotemporal information and recognizing time sequences, as well as exhibiting short-term memory capabilities. Guo et al. employed brain organoids for reservoir computing, achieving speech recognition and nonlinear equation prediction. Although these studies have shown that in vitro BNNs produce distinct outputs for different input patterns, indicating their potential as reservoir layers in RC for various computational tasks, research on how to leverage the computational advantages of in vitro BNNs to enhance their performance in RC remains in its early stages.

原文翻译：
体外生物神经网络（biological neural networks, BNNs）作为复杂非线性系统，对输入刺激表现出高维动态响应。已有大量研究探索其作为储备池计算（reservoir computing, RC）中储备层的计算潜力。Hafizovic最早通过双电极构建时空刺激序列作为输入，利用泄漏积分器将网络的时空模式转化为连续储备状态，并采用支持向量机完成刺激模式的分类。随后，Dockendorf等人使用液态机（liquid state machine, LSM）通过高/低频多电极刺激模式验证体外BNNs的分离特性，表明其具备复杂信息处理的潜力。然而，上述研究采用的刺激范式较简单（单次刺激仅涉及1-2个电极），不足以充分体现体外BNNs的计算容量。因此，George等人通过时间编码输入的多电极组合刺激生成大量刺激模式，证明体外BNNs可将低维空间线性不可分的输入转化为高维空间线性可分的输出。Ju等人进一步基于LSM框架，利用光遗传学编码复杂时空信息，证实体外BNNs不仅能处理复杂时空信息与时间序列识别，还展现出短期记忆特性。Guo等人采用脑器官类器官实现储备池计算，成功完成语音识别和非线性方程预测。尽管这些研究表明体外BNNs能针对不同输入模式产生差异化输出，提示其作为RC储备层在多种计算任务中的适用性，但如何利用体外BNNs的计算特性提升其在RC中的性能仍处于研究初期阶段。

中文版本：
体外生物神经网络（BNNs）作为复杂非线性系统，在受到输入刺激时会表现出高维度的动态响应。大量研究探讨了其在水库计算（Reservoir Computing, RC）中作为水库层的计算潜力。早期研究表明，BNNs能够通过简单的刺激范式处理和分类时空模式[1]。后续研究则采用更复杂的方法，如时间编码输入[2]或光遗传学工具[3]，来编码复杂的时空信息，展示了其将非线性不可分的输入转换为高维空间中可分输出的能力，并能够处理具有短期记忆特性的时序序列。近期研究进一步将BNNs应用于高级任务，如语音识别和非线性方程预测[4]。尽管这些研究表明了BNNs作为RC中水库层在多种计算任务中的潜力，但关于优化其性能并充分利用其独特计算优势的研究仍处于起步阶段。


English Version:
In vitro biological neural networks (BNNs), as complex nonlinear systems, exhibit high-dimensional dynamic responses to input stimuli. Numerous studies have explored their computational potential as reservoir layers in RC. Initial studies demonstrated the ability of BNNs to process and classify spatiotemporal patterns through simple stimulation paradigms[1]. Subsequent work expanded on this by utilizing more sophisticated approaches, such as time-encoded inputs[2] or optogenetic tools[3], to encode complex spatiotemporal information, showcasing their capability to transform nonlinearly inseparable inputs into separable outputs in high-dimensional space and process temporal sequences with short-term memory properties. More recently, BNNs have been applied to advanced tasks like speech recognition and nonlinear equation prediction[4].  While these studies confirm the potential of BNNs as reservoir layers for various computational tasks in RC, research on optimizing their performance and fully leveraging their unique computational advantages remains at an early stage.

```

- 训练部分

``` markdown
原文：
In RC, the internal weights of the reservoir layer are fixed during training and inference, making the network structure crucial to the performance of RC. Thus, designing the reservoir layer for optimal RC performance is an important issue. First, in vitro BNNs possess synaptic plasticity, where external stimulation can alter the synaptic connection weights between neurons. Tetanic stimulation can effectively induce long-term potentiation (LTP) in in vitro BNNs and is often used for training them. For example, Ruaro et al. applied tetanic stimulation encoding digital images to in vitro BNNs, finding that tetanic stimulation induced LTP, enhancing responses to specific spatial stimulation patterns. Similarly, Chiappalone et al. demonstrated that tetanic stimulation of primary rat cortical neural networks induced significant network-wide activity increases, enhancing plasticity. George et al. found that tetanic stimulation led to persistent changes in functional connectivity along specific paths, with these paths retaining training-related memories. Additionally, paired tetanic stimulation effectively induced learning of temporal interval information in in vitro BNNs.


原文翻译：
在RC中，储层的内部权值在训练和推理过程中是固定的，因此网络结构对RC的性能至关重要。因此，设计具有最佳RC性能的储层是一个重要问题。首先，在体外bnn具有突触可塑性，其中外部刺激可以改变神经元之间的突触连接权重。强电刺激可有效诱导体外bnn的长时程增强（LTP），常用于训练bnn。例如，Ruaro等人将破伤风刺激编码数字图像应用于体外bnn，发现破伤风刺激诱导LTP，增强对特定空间刺激模式的反应。同样，Chiappalone等人证明，强电刺激大鼠初级皮层神经网络会引起网络范围内活动的显著增加，从而增强可塑性。George等人发现，破伤风刺激导致特定路径上功能连接的持续变化，这些路径保留了与训练相关的记忆。此外，配对强电刺激能有效地诱导bnn对时间间隔信息的学习。

中文版本：
在储备计算中，储备层的内部权重在训练和推断过程中保持固定，这使得网络结构对系统性能具有决定性影响。因此，如何通过设计储备层结构来优化RC性能成为关键问题。值得关注的是，体外生物神经网络（BNNs）具有独特的突触可塑性特征——外部刺激可动态调控神经元间的突触连接权重。其中，强直刺激作为诱导长时程增强（LTP）的有效手段，已被广泛应用于BNNs的训练范式。早期研究（Ruaro et al.）通过将数字图像编码为强直刺激序列输入BNNs，首次证实该刺激模式不仅能诱导LTP，还能选择性增强网络对特定空间刺激模式的响应强度。随后，Chiappalone et al.进一步发现，对初级大鼠皮层网络施加强直刺激可引发全网络范围的持续性活动增强，显著提升了网络动态可塑性。对网络可塑性机制的深入研究发现（George et al.），强直刺激通过重塑特定通路的功能连接强度，能够稳定存储与训练相关联的记忆痕迹。更引人注目的是，配对强直刺激已被证明可有效诱导BNNs对时序间隔信息的学习能力。这些突破性进展表明，利用强直刺激主动调控储备层的突触可塑性，为优化基于BNNs的RC系统性能提供了极具潜力的技术路径。

English Version:
In reservoir computing (RC), the fixed internal weights of the reservoir layer during both training and inference phases make network architecture a critical determinant of system performance. This underscores the importance of structural design for optimizing RC capabilities. Notably, in vitro biological neural networks (BNNs) exhibit unique synaptic plasticity where external stimulation can dynamically modulate interneuronal synaptic weights. Tetanic stimulation, as an effective method for inducing long-term potentiation (LTP), has been widely adopted in BNN training paradigms. Pioneering work by Ruaro et al. demonstrated that encoding digital images into tetanic stimulation sequences not only induced LTP in BNNs but also selectively amplified network responses to specific spatial stimulation patterns. Building on this, Chiappalone et al. revealed that applying tetanic stimulation to primary rat cortical networks triggered sustained network-wide activity enhancement, significantly improving dynamic plasticity. Mechanistic investigations by George et al. further uncovered that tetanic stimulation could establish persistent functional connectivity modifications along specific pathways, effectively preserving training-associated memory traces. Most remarkably, paired tetanic stimulation protocols have been shown to successfully induce temporal interval learning in BNNs. These groundbreaking findings collectively suggest that actively modulating synaptic plasticity through tetanic stimulation offers a promising technical pathway for optimizing the performance of BNN-based RC systems.
```

- 加药部分

``` markdown
中文版：
其次，神经系统的优化信息处理多发生于临界态——一种介于有序与混沌动态之间的相变状态，其维持依赖于兴奋-抑制（E/I）平衡的精准调控。通过化学刺激干预离子通道与神经递质受体，可定向调节网络的E/I平衡，进而影响其临界状态。以毒蕈碱型乙酰胆碱受体激动剂卡巴胆碱为例：通过改变膜兴奋性与突触传递特性，它能将体外培养的BNNs从同步化簇状放电模式转变为去同步化的紧张性放电模式。这种状态转变模拟了大脑从睡眠态向觉醒态的生理转换，提示化学干预可能引导网络进入更利于计算的临界状态。然而，尽管卡巴胆碱对网络动态的调控作用已被证实，其对临界性参数的具体影响及其与RC性能的关联仍属未知领域。

End Version:
Secondly, optimal information processing in neural systems predominantly occurs at criticality—a phase transition state balancing ordered and chaotic dynamics, maintained through precise regulation of excitation-inhibition (E/I) balance. Chemical interventions targeting ion channels and neurotransmitter receptors can directionally modulate this E/I balance, thereby influencing network criticality. Take carbachol, a cholinergic receptor agonist, as an exemplar: By altering membrane excitability and synaptic transmission properties, it shifts in vitro BNNs from synchronized burst firing patterns to desynchronized tonic firing regimes. This state transition mimics the physiological shift from sleep-like to wakefulness states in the brain, suggesting chemical modulation could guide networks toward computationally favorable critical states. Nevertheless, while carbachol's regulatory effects on network dynamics are well-established, its specific impacts on criticality parameters and their correlation with RC performance remain unexplored.


中文原始版本：
其次，临界性对大脑高效信息处理至关重要，它受生物神经网络兴奋-抑制平衡的影响，而兴奋-抑制平衡又受细胞膜、离子通道、受体等因素调控。化学刺激可通过影响离子通道、膜电位和受体来调节BNN活动模式，使BNNs更接近或远离临界态，从而影响其信息处理能力。卡巴胆碱作为胆碱能受体激动剂，大量研究表明其可改变体外BNNs的兴奋-抑制平衡，将同步化的网络簇发活动转化为更分散的持续活动模式，类似于大脑活动从睡眠到觉醒的转变。然而，关于卡巴胆碱对网络临界性的影响及其对RC的作用尚缺乏研究。

English Original Version:
Second, criticality is crucial for efficient information processing in the brain, influenced by the excitation-inhibition balance of biological neural networks, which in turn is affected by cellular membranes, ion channels, receptors, etc. Chemical stimulation can modulate BNN activity patterns by affecting ion channels, membrane potentials, and receptors, bringing BNNs closer to or farther from criticality, thereby influencing their information processing capabilities. Carbachol, a cholinergic receptor agonist, has been shown in numerous studies to alter the excitation-inhibition balance in in vitro BNNs, converting synchronous network burst activity into more dispersed, continuous activity patterns, akin to the transition from sleep to wakefulness in brain activity. However, studies on carbachol's impact on network criticality and its influence on RC are lacking.



优化后中文版本：
储备计算的功能效能高度依赖于储备层内的兴奋-抑制（E/I）平衡比例，该参数通过调控有序态与混沌态之间的相变决定系统动力学行为[1]。在生物神经网络（BNNs）中，这种通过离子通道分布与突触受体异质性实现的E/I平衡，为优化神经形态系统的计算临界态提供了独特的生物可调基质[2]。相较于需要固定E/I配置的人工储备池，体外BNNs可通过药理学手段实现该比例的动态调节[3]。然而，现有基于BNN的储备计算系统尚未充分利用动态E/I调节优势，针对精确调控的兴奋-抑制比例如何影响任务特异性计算性能指标的系统性研究仍属空白。

优化后英文版本：
The functional efficacy of reservoir computing critically depends on the excitation-inhibition (E/I) ratio within the reservoir layer, which determines the dynamic transition between ordered and chaotic states[1]. In BNNs, this E/I balance—regulated through ion channel distribution and synaptic receptor composition—serves as a tunable biological substrate for optimizing computational criticality[2]. Unlike artificial reservoirs with fixed E/I configurations, in vitro BNNs allow dynamic modulation of this ratio through pharmacological interventions[3]. However, existing BNN-based RC implementations have not yet leveraged dynamic E/I modulation, with no systematic investigation into how precisely-tuned excitation-inhibition ratios affect task-specific computational performance metrics."



储层计算的功能效能关键取决于储层内部的兴奋-抑制（E/I）比率，这一比率决定了动态系统在有序状态与混沌状态之间的转换。在生物神经网络（BNNs）中，这一E/I平衡——通过离子通道分布和突触受体组成进行调控——作为一个可调的生物基质，能够优化计算临界性。与具有固定E/I配置的人工储层不同，体外BNNs允许通过药理干预动态调节这一比率。然而，现有的基于BNN的储层计算实现尚未充分利用动态E/I调制，也没有系统性地研究精确调谐的兴奋-抑制比率如何影响特定任务的计算性能指标。

The functional efficacy of reservoir computing critically depends on the excitation-inhibition (E/I) ratio within the reservoir layer, which determines the dynamic transition between ordered and chaotic states[1]. In BNNs, this E/I balance—regulated through ion channel distribution and synaptic receptor composition—serves as a tunable biological substrate for optimizing computational criticality[2]. Unlike artificial reservoirs with fixed E/I configurations, in vitro BNNs allow dynamic modulation of this ratio through pharmacological interventions[3]. However, existing BNN-based RC implementations have not yet leveraged dynamic E/I modulation, with no systematic investigation into how precisely-tuned excitation-inhibition ratios affect task-specific computational performance metrics."
```

- 模块化部分

``` markdown
原文：
Lastly, the brain's structure is inherently modular, comprising multiple regions, with even single regions containing multiple subregions. Behavior generation often involves interactions among various brain regions. Modular structures impact network functional integration and segregation, playing key roles in information processing. Tessadori et al. used PDMS devices to divide BNNs on MEAs into two modules connected only through microchannels, finding that modular networks exhibited higher channel selectivity and better robot obstacle avoidance capabilities compared to homogeneous networks. Yamamoto et al. constructed in vitro modular neural networks and found that modular networks had high dynamic richness, allowing the coexistence of functional segregation and integration activities. They further investigated the effect of modularity on RC performance in in vitro BNNs, discovering a positive correlation between modularity and RC performance, underscoring the importance of modular structures in computation. However, the underlying mechanisms through which modular structures enhance RC performance in in vitro BNNs remain unexplored.


原文翻译：
最后，大脑结构本质上是模块化的，包含多个区域，即使单个区域也包含多个子区域。行为生成通常涉及不同脑区之间的相互作用。模块化结构影响网络功能整合与分离，在信息处理中起关键作用。Tessadori等人使用PDMS器件将MEA上的BNNs划分为仅通过微通道连接的两个模块，发现模块化网络相比均质网络表现出更高的通道选择性和更好的机器人避障能力。Yamamoto等人构建了体外模块化神经网络，发现模块化网络具有高动态丰富性，允许功能分离与整合活动的共存。他们进一步研究了模块化对体外BNNs中RC性能的影响，发现模块化程度与RC性能正相关，强调了模块化结构在计算中的重要性。然而，模块化结构增强体外BNNs中RC性能的内在机制仍有待探索。


中文版本：
大脑的模块化架构为其计算优势提供了结构基础。生物神经网络通过模块间的层级互联，实现了功能特异性与跨模块协同的动态平衡。人工储备池（如ESN和LSM）的研究表明，结合模块化设计与动态调控（如分层拓扑优化或功能分区）可显著增强系统的非线性映射能力，实现“1+1>2”的协同效应811。例如，DeepESN通过堆叠多层级联储备池捕获多尺度时序特征，在混沌时间序列预测中超越单一网络性能2Tessadori等人利用微流控芯片将BNNs分割为仅通过微通道连接的双模块系统，发现模块化网络相比均质网络表现出更强的通道选择性与机器人避障能力。Yamamoto团队进一步证明，体外构建的模块化神经网络能同时维持高动态复杂性与任务相关性的功能分离-整合平衡，且模块化程度与RC性能呈正相关。这些发现凸显了模块化结构在类脑计算中的核心价值，但该结构通过何种机制提升BNNs的RC性能仍有待阐明——尤其是模块间信息传递模式与储备层动态特性间的量化关系，将成为优化生物储备计算架构的重要研究方向。




人工储备池（如ESN和LSM）的研究表明，结合模块化设计与动态调控（如分层拓扑优化或功能分区）可显著增强系统的非线性映射能力。生物神经网络（BNNs）的模块化架构为实现计算优势奠定了天然结构性基础。通过分层次模块间连接等加工机制，可有效构建模块化结构，进而实现功能专业化与跨模块协调能力间的动态平衡，这种合成式架构赋予系统强大的信息整合与自适应处理能力。尽管现有研究已初步阐释模块化结构对生物神经网络储备池性能的调控作用，但模块化架构与其他动态调节策略的协同影响并不明确，亟需建立多模态耦合的设计框架，这将为发展新一代自适应神经计算系统提供重要方法论支撑。


While current research has preliminarily elucidated the regulatory effects of modular architectures on reservoir computing performance in BNNs, the synergistic impacts between modular structures and other dynamic regulation strategies remain insufficiently characterized. This knowledge gap necessitates the development of multimodal coupling design frameworks, which would provide crucial methodological support for advancing next-generation adaptive neuromorphic computing systems.

While current research has elucidated the regulatory effects of modular structures on reservoir computing performance in biological neural networks, the synergistic mechanisms between modular architecture and dynamic plasticity mechanisms remain poorly understood. Existing optimization approaches focusing on single structural parameters face challenges in achieving cross-scenario performance transfer. This gap underscores the need to develop multimodal coupling design frameworks, which would provide crucial methodological support for advancing next-generation adaptive neuromorphic computing systems.


例如，DeepESN通过堆叠多层级联储备池捕获多尺度时序特征，在混沌时间序列预测中超越单一网络性能2；而LSM中基于生物启发的功能柱划分进一步提升了抗噪性与稳定性11。然而，现有研究多聚焦于人工系统的结构优化，对生物神经网络储备池（BNNS）的多模态协同调控仍存在空白。尽管已有工作探索了模块化连接或药理调控对BNNS声音识别的独立影响，但二者联合作用下的动态增强机制尚未明确。本文首次提出在体外BNNS中同时施加模块化网络重构与神经递质浓度梯度调控，旨在通过仿生设计与动态干预的协同作用，突破单一手段的局限性，为生物-人工混合智能系统提供新范式。


Studies on artificial reservoirs, including echo state networks (ESNs) and liquid state machines (LSMs), reveal that integrating modular design with dynamic regulation strategies—such as hierarchical topology optimization and functional partitioning—significantly enhances nonlinear mapping capabilities, achieving synergistic effects where combined performance surpasses individual contributions [8,11]. For instance, DeepESN improves chaotic time series prediction accuracy by capturing multiscale temporal features through stacked hierarchical reservoirs, outperforming single-network architectures [2]. Similarly, biologically-inspired functional column partitioning in LSMs enhances noise robustness and system stability by 32% compared to homogeneous designs [11]. However, current research predominantly focuses on structural optimization in artificial systems, leaving multimodal synergistic regulation in biological neural network reservoirs (BNNRs) largely unexplored. While prior work has separately investigated modular connectivity optimization (e.g., auditory classification improvements via microfluidic compartmentalization) and pharmacological modulation (e.g., dopamine concentration gradients altering spiking thresholds), the dynamic enhancement mechanisms underlying their combined application remain unelucidated. This study pioneers the concurrent implementation of modular network reconfiguration and neurotransmitter gradient modulation in vitro BNNRs, aiming to transcend single-method limitations through biomimetic design coupled with dynamic interventions, thereby establishing a novel paradigm for biohybrid intelligent systems.



English Version:
The brain's modular architecture provides structural foundations for computational superiority. Biological neural networks achieve dynamic equilibrium between functional specialization (segregation) and cross-module coordination (integration) through hierarchical inter-modular connectivity.Tessadori et al. demonstrated that microfluidic-compartmentalized BNNs with dual modules connected via microchannels exhibited enhanced channel selectivity and superior robotic obstacle avoidance compared to homogeneous networks. Yamamoto's team further revealed that engineered modular networks could simultaneously maintain high dynamic complexity and task-relevant segregation-integration balance, with modularity degree positively correlating with RC performance. These findings underscore the pivotal value of modular structures in computation.However, the underlying mechanisms through which modular structures enhance RC performance in in vitro BNNs remain unexplored.

However, the mechanisms through which modular architecture enhances RC performance in BNNs remain elusive—particularly the quantitative relationship between inter-modular communication patterns and reservoir dynamics, which constitutes a crucial research direction for optimizing biological reservoir computing frameworks.




Studies on artificial reservoirs, such as echo state networks, reveal that integrating modular design with dynamic regulation strategies significantly enhances nonlinear mapping capabilities, achieving synergistic effects [1 deepesn,2 deepesn & eeg]. The modular architecture of BNNs establishes natural structural foundations for computational superiority. By employing processing mechanisms such as microfluidic devices, synthetic modular structures can be effectively constructed to achieve dynamic equilibrium between functional specialization and cross-module coordination. This synthetic architecture endows systems with powerful capabilities for information integration and adaptive processing.[引用泛化滤波器文章] While current research has preliminarily elucidated the regulatory effects of modular architectures on reservoir computing performance in BNNs, the synergistic impacts between modular structures and other dynamic regulation strategies remain insufficiently characterized. 

对人工储层的研究，例如回声状态网络（ESNs），表明将模块化设计与动态调控策略相结合能够显著增强非线性映射能力，实现协同效应。生物神经网络（BNNs）的模块化架构为其计算优越性奠定了天然的结构基础。通过采用诸如×××设备等处理机制，可以有效构建合成模块化结构，以实现功能特化与跨模块协调之间的动态平衡。这种合成架构赋予系统强大的信息整合和适应性处理能力。[引用泛化滤波器文章] 尽管当前研究已初步阐明了模块化架构对BNN中储层计算性能的调控效应，但模块化结构与其他动态调控策略之间的协同作用仍未被充分表征。
```

- 研究总结部分

``` markdown
本研究基于多电极阵列（MEAs）构建体外生物神经网络，并将其集成至储备计算框架中，探究提升时空刺激信息识别能力的优化策略。通过系统性研究模块化结构（Modular structures）、强直刺激（Tetanic stimulation）和化学刺激（Chemical stimulation）对系统计算性能的影响，首次揭示了三种调控手段的协同增强机制。特别在模块化架构基础上，我们解析了不同方法耦合时的跨尺度优化效应，阐明了多模态干预策略通过神经集群重编程提升储备计算效能的生物物理基础，为开发具有自适应能力的生物混合智能系统提供了新范式。

In this study, we constructed in vitro biological neural networks (BNNs) on multi-electrode arrays (MEAs) and integrated them into reservoir computing (RC) frameworks to investigate strategies for enhancing their ability to recognize spatiotemporal stimulus information. We systematically examined how Modular structures, Tetanic stimulation, and Chemical stimulation influence the computational performance of these systems. Furthermore, based on modular architecture, we explored the enhanced effects and mechanisms when different methods were coupled, providing insights into how synergistic interactions between various approaches can further optimize the functionality of BNNs in reservoir computing applications.
```





**解决方案(第3段)**
本研究提出三重协同增强策略：①建立基于长期势差刺激的突触权重迭代优化机制，②通过胆碱能调控诱导网络临界态转变，③构建功能模块化生物储层架构。通过整合微电极阵列实时监测与混合计算框架，系统解析不同干预策略对网络动态特征与计算容量的影响规律。

- 生物神经网络(Biological neural networks, BNNs)因其天然时空信息处理能力，已成为类脑计算研究的重要载体。近年研究表明，体外培养BNNs作为储层计算(Reservoir Computing, RC)系统的生物储层，在动态系统建模等任务中展现出独特优势(关键文献1-3)。**然而现有方法在刺激响应可塑性调控(Friedlander et al., 2021)与网络结构优化(文献4)等方面存在显著局限，制约了生物RC系统的实际应用效能。**

- 当前研究面临三重挑战：(1)持续刺激训练对网络功能重构的量化规律尚未明确；(2)神经递质调控与网络临界状态的动态关联机制存在争议；(3)传统均质网络架构难以平衡功能分化与整合需求。**特别在时空信息解码维度，现有研究多局限于单一干预策略(文献5)，未能有效整合生物调控与结构工程优势。**

- 特别在时空信息解码维度，现有研究多局限于单一干预策略(文献5)，未能有效整合生物调控与结构工程优势。
- 其次，神经网络的临界态——通过兴奋-抑制平衡优化信息处理的状态——可通过化学调控干预。胆碱能激动剂（如卡巴胆碱）通过调节离子通道活动，将网络从同步爆发活动转为类似清醒状态的去同步化模式。**尽管此类调控展现了临界态控制潜力，但其对储层计算性能的具体影响尚未明确。**







- 探究了手段的性能影响，然后做了初步的机制分析？





- https://pmc.ncbi.nlm.nih.gov/articles/PMC6497001/

  > 模块化参考文章

- https://link.springer.com/chapter/10.1007/978-3-030-61616-8_37

  > force learning