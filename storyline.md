##### **问题描述**

**用一句话描述你想解决的问题，并添加要点作出解释**

- **问题的一句话描述**：随着 CPS 从相对封闭的控制系统演化为高度互联的 cyber-physical infrastructure，攻击也不再局限于单一设备、协议或物理量异常，而是可能跨越控制主机、工业通信和物理过程持续传播；在这一趋势下，如何低延迟识别由恶意程序触发的 CPS 攻击，并将其定位到具体异常进程及其系统级行为证据，仍是现有检测方案的结构性不足；这些细粒度证据进一步构成后续 CPS 节点影响分析和恢复决策支持的入口，而不是仅给出 attack/no-attack 判断。
- 解释1：攻击者可以通过恶意程序伪造传感器数据、修改控制程序或运行时参数、发送异常 command/reporting messages，或操纵 HMI/工程站与工业协议交互，从而进入 sensing/control path 并诱导错误控制决策。现有 CPS/ICS 防御一部分关注控制资产、控制逻辑、工业协议和运行环境是否异常，另一部分关注传感器读数、物理模型残差、控制不变量和 transduction attack；这些观测能够描述控制链路或物理后果，却难以及时捕捉恶意程序在控制主机 OS 层留下的 runtime 行为痕迹。
- 解释2：许多 CPS 检测方法主要报告 attack/no-attack、物理状态异常或受影响状态集合，并依赖这些结果进行恢复；即使部分 host provenance 方法能够恢复攻击足迹，其证据也通常缺少面向 CPS 影响分析的明确边界，难以直接支持 CPS 场景下的 process-level attribution 和 recovery-oriented impact analysis。因此，后续影响分析和恢复决策仍缺少明确证据入口，恢复过程可能只修复 sensor/control path 上的表面后果，而难以面向攻击来源做出准确响应。

##### **问题重要性**

**写出这是一个重要问题的理由，并给出论据。**

- **论据**：文献，博客，新闻报道，自己做的实证研究等

- **有文献**：一句话总结文献的论证

- 文献不要太少（尽量避免一篇论文的孤证），尽量选取 CCF-A 类会议/期刊

- **没有文献**：

- 在**原理上**论证重要性

- 针对性地设计**实证研究/实验/问卷调查**来获取支撑数据

- **理由 1：CPS 攻击具有物理后果。**
  - 论据1：CPS 将软件执行、传感器感知、控制决策和执行器动作连接到真实物理过程，并被广泛部署在 smart grid、industrial control systems、autonomous vehicles、medical devices 等安全关键场景中。因此，一旦攻击影响 sensing/control path，后果就可能从信息系统异常扩展到错误控制、服务中断、设备损坏甚至关键基础设施故障 [Humayed17; Mitchell14; Kayan22]。
  - 论据2：真实攻击事件进一步说明了这一风险。Stuxnet 利用工业控制环境中的软件和 PLC 弱点，目标是在关键基础设施中制造 physical anomalies；Ukraine power grid attack 则通过 BlackEnergy 等攻击活动造成大规模停电，影响约 225,000 名用户。这些案例表明，针对 CPS 的攻击并不只造成数字层面的异常，还可能直接破坏物理设备和关键服务 [Humayed17; Kayan22]。

- **理由 2：低延迟进程定位不足会削弱攻击响应。**
  - 论据1：已有 CPS IDS 研究指出，CPS 攻击既可能快速破坏物理过程，也可能像 Stuxnet 一样长期潜伏并逐步设置攻击条件，因此检测未知攻击和降低 detection latency 是 CPS IDS 的核心挑战 [Mitchell14; Giraldo18; Hu18]。在闭环控制场景中，错误观测、异常控制命令或被篡改的运行时参数可能快速转化为 controller decision 和 actuator action；如果系统只能在 residual、physical invariant 或受影响状态明显异常后报警，响应时机可能已经晚于恶意程序从控制主机进入 sensing/control path 的关键阶段。
  - 论据2：低延迟报警本身仍不足以支持准确响应。恢复或隔离攻击源需要知道异常是否由具体进程触发、该进程通过哪些 file、socket、device interface 或控制相关资源影响系统，以及这些行为是否与 CPS sensing/control path 有关。若告警只能指出物理状态、控制组件或异常时间窗口，响应者仍难以判断应该隔离哪个进程、保留哪些证据、以及把哪些系统实体交给后续 CPS 影响分析。
  - 论据3：现有 provenance 研究说明 attack reconstruction 可以为响应提供因果证据，但代表性工作通常只覆盖因果链的一侧。PLC-PROV 可以解释 sensor、actuator 或 PLC interaction 如何导致 CPS 后果；KAIROS 等 whole-system provenance 方法可以重建 process、file、socket 层面的恶意活动。然而，前者不直接定位控制主机上的恶意程序 runtime 行为，后者不直接组织这些行为与 CPS physical consequence 之间的入口边界。因此，仍需要一种 CPS-aware 的 process-level provenance evidence boundary，将主机侧攻击行为转化为后续影响分析和恢复决策可使用的证据入口 [PLCProv19; Kairos24]。
  - 论据4：这种证据入口不足还会影响 recovery-oriented decision support。现有恢复决策通常依赖告警结果来选择恢复受影响的 sensor/control state、回滚异常操作，或重置受影响组件。Taser 和 RETRO 等入侵恢复研究进一步说明，恢复不能只关注系统是否回到某个正常状态，还需要区分 attack-dependent operations 与合法操作 [Taser05; RETRO10]。因此，若 CPS 告警不能定位触发攻击的恶意程序及其因果证据，恢复过程可能只处理 sensor/control path 上的表面后果，而缺少面向攻击来源的处置依据。

##### **问题重要性：实证研究（选填）**

**设计实证研究/实验/问卷调查来获取支撑数据。**
- 收集到的数据尽量简单直观
- 收集数据的方法能够反映真实世界的普遍情况

- **本节状态：暂时跳过。**
  - 跳过原因：当前问题重要性主要由 CPS 安全关键性、真实攻击事件、CPS IDS 文献中的 detection latency 挑战，以及 provenance/recovery 文献中的因果证据需求共同支撑。本文暂时不在 storyline 阶段额外设计独立实证研究来证明问题重要性。
  - 后续可能补充的数据：如果需要进一步增强问题重要性，可以在实验阶段补充攻击 trace 中 host-side malicious behavior 与 physical anomaly alarm 之间的时间差、现有 CPS IDS 输出中是否包含 process-level attribution 信息，以及 recovery 所需证据项与现有告警输出之间的缺口。

##### **背景知识**

**给出相关技术的背景介绍。**

- 文章用到的理论、概念等、
- 区别于相关工作，背景知识主要是教科书上的内容

- **背景知识 1**：CPS 恶意程序攻击面与物理影响
  - 解释 1：CPS 通常由 sensor、controller、actuator 和 physical process 构成闭环：sensor 观测物理状态，controller 根据观测值计算控制决策，actuator 将控制命令作用到物理过程。针对 CPS 的恶意程序通常不会只破坏普通 IT 资源，而是通过控制主机、HMI、工程站、PLC 接口、工业协议、I/O driver、项目文件或配置文件进入 sensing/control path，进而影响控制逻辑、传感器读数、执行器命令或操作员看到的系统状态。
  - 解释 2：这类恶意程序的典型行为包括识别控制软件和 PLC 配置、读取或修改工程项目文件、篡改控制程序或运行时参数、伪造或重放 sensor data、发送异常 command/reporting messages、操纵 HMI 显示或告警信息、干扰设备通信、连接外部 C&C server，或通过持久化机制在控制主机上长期存在。这些行为可能导致 controller 基于错误状态做出决策、actuator 执行错误动作、operator 看到失真的系统视图，最终造成服务中断、设备损坏、loss of control、loss of view 或 unsafe physical state。

- **背景知识 1.1：本文关注的攻击路径与证据边界**
  - 解释：本文关注的是由控制主机、HMI、工程站、PLC 接口或 CPS 仿真/控制环境中的恶意程序触发的攻击。这类攻击通常需要通过 file、process、socket、device interface、控制软件接口或工业通信端点与 sensing/control path 发生交互，因此会在主机侧留下可观测的 syscall/provenance 行为证据。对于完全不经过主机软件路径的物理扰动或硬件故障，本文将其视为由物理冗余、故障诊断或 safety monitoring 互补处理的场景。
  - 证据边界：本文所说的 evidence boundary 指围绕一次可疑攻击行为划定的最小可解释证据范围，包括异常时间边界、可疑进程、关键 syscall/provenance 子图、相关 file/socket/device interface/control resource，以及这些系统实体与 CPS sensing/control path 的关联。该边界不是完整的 CPS impact analysis，也不是完整 recovery plan，而是后续影响分析和恢复决策可以使用的 process-level CPS-aware provenance evidence 入口。
  - CPS 节点映射入口：本文还需要维护或推断系统实体到 CPS 节点/组件的映射入口，例如某个 device interface 对应 sensor input，某个 socket endpoint 对应 PLC/HMI 通信，某个配置文件对应 controller runtime parameter，某个 control software interface 对应 actuator command path。该映射入口只说明 OS-level evidence 可以交给后续 CPS 节点影响分析的连接位置，不直接声称完成完整 CPS impact analysis；没有这种入口，OS-level provenance 只能说明主机侧行为异常，难以成为 CPS 节点影响分析的证据入口。
  - 证据可信假设：本文假设审计日志或 provenance logs 由受保护的采集机制产生，攻击者可以通过恶意程序影响控制软件、文件、socket、device interface 或工业通信端点，但不能完全关闭、伪造或回滚日志采集通道本身。若攻击者已经控制日志基础设施或能够系统性篡改 provenance records，则需要结合可信执行、远程 attestation、secure logging 或独立监控通道来保证证据可信性，这属于互补防护假设，而不是本文主要解决的问题。

- **背景知识 2**：系统级行为证据与 provenance graph
  - 解释 1：上述攻击行为通常伴随程序与系统资源或控制资产之间的运行时交互，例如访问配置文件、启动子进程、调用控制软件接口、连接 socket、读写日志、操作 device file 或与 PLC/HMI 通信。这些交互可以作为刻画恶意程序运行时行为的底层行为证据，帮助说明攻击并非只体现为 sensor/control path 上的数值异常。
  - 解释 2：这类行为证据可以被组织成 syscall trace，也可以进一步抽象为 provenance graph。在 provenance graph 中，process、file、socket、device interface 等系统实体作为节点，read、write、execute、send、receive 等交互作为因果边，从而描述攻击行为如何在 cyber layer 内部传播，并如何接触与 sensing/control path 相关的系统实体或控制接口。它本身不直接完成 physical impact analysis，但可以为后续分析提供因果入口。

- **背景知识 3**：CPS attack attribution 与 recovery-oriented decision support
  - 解释 1：CPS attack attribution 不只是识别攻击类型，而是将异常与具体恶意程序、系统行为证据、受影响控制环节和可能的物理后果之间的因果链关联起来。对本文而言，attribution 关注的是“哪个进程通过哪些系统交互影响了哪些 CPS 节点或组件入口”，而不是仅给出 attack/no-attack 判断。
  - 解释 2：CPS recovery-oriented decision support 指在攻击发生后，为系统恢复到安全、可控且不再受同一攻击源影响的状态提供证据依据。本文不直接生成完整 recovery action，而是输出可疑进程、相关系统实体、控制相关资源和 CPS 节点映射入口，帮助后续恢复过程判断攻击源是否需要隔离、哪些组件需要检查、哪些状态恢复可能只处理了表面后果。
  

##### **解决问题的现有相关方法**

**列出解决这一问题的现有相关方法。**
- **直接方法：**     直接针对这一问题
- **间接方法：**     针对其它问题的方法稍作调整可以适应这一问题
- 其它问题和这一问题有共同性/相似性

- **第一类方法：CPS/ICS 控制资产与控制链路防御**
  - 直接方法：这类方法围绕控制系统资产、控制逻辑、工业协议和运行环境建立保护与检测机制，可进一步分为三种常见方向。第一种是控制资产保护与完整性验证，例如 defense-in-depth、网络分区与访问控制、工程站和 HMI 加固、白名单执行、补丁管理、PLC 程序完整性检查、远程 attestation，以及针对 PLC firmware、control logic、runtime memory 和 communication module 的完整性验证。第二种是工业通信和控制链路监控，例如工业协议监控、异常命令检测、HMI/PLC interaction 检查，以及 I/O 和 control command 异常检测。第三种是控制系统 provenance/forensics，例如 PLC-PROV 将 input/output、PLC interaction、sensor/actuator event 或 control-state change 组织成因果证据，用于 policy violation detection、attack reconstruction 和 impact understanding [Mitchell14; Kayan22; LopezMorales24; PLCProv19]。
  - 方法的局限：这类方法能保护或解释控制资产、控制链路和控制事件，但通常不直接建模恶意程序在控制主机 OS 层留下的细粒度 runtime 行为痕迹，也不一定能以低延迟把告警定位到具体进程及其系统级因果链。换言之，它们更擅长说明“控制资产、控制命令或控制事件是否异常”，但较难说明“哪个进程通过哪些系统行为触发了这次 CPS 攻击”。
  - 局限的根本原因：CPS 防御长期以工业资产和控制链路为中心，观测对象多是 PLC 程序、工业协议、控制命令、HMI 显示、网络流量或控制事件；恶意程序在 OS 层面的 process、file、socket、syscall 行为没有被纳入统一的 CPS-aware evidence boundary。因此，控制系统异常与恶意程序行为之间缺少可用于低延迟 attribution 和 recovery-oriented analysis 的因果连接。

- **第二类方法：sensor/control path 数值异常检测**
  - 直接方法：面向传感器数值攻击的 CPS 检测方法，通常把攻击建模为 sensor reading、control input 或 physical state 与正常物理模型之间的偏离。典型检测方法包括 threshold-based detection、residual-based detection、Kalman filter / observer-based detection、chi-square detector、CUSUM / EWMA 等 stateful statistics、physical invariant checking、cross-sensor redundancy checking、watermarking 和 moving target defense [Giraldo18; Cardenas11; Mo15; Mitchell14]。这些方法通常通过比较观测值与预测值、检查物理不变量、累积 residual 变化或引入主动扰动来判断 sensor/control path 是否被篡改。
  - 方法的局限：这类方法能够发现 sensor/control path 上的数值异常或物理不变量破坏，但检测对象主要是“物理状态是否异常”，而不是“哪个恶意程序造成异常”。当攻击者通过恶意程序缓慢、隐蔽或模型感知地修改数据，使 residual 保持在阈值以下时，这类方法可能漏报；即使报警，也往往只能报告某个传感器、控制信号或状态变量异常，缺少对恶意程序、攻击传播路径和系统级行为证据的定位。
  - 局限的根本原因：传感器数值防御把攻击后果映射到物理量空间，依赖物理模型、统计阈值或传感器冗余来判断异常。它缺少从物理异常反向连接到 OS 层恶意程序的因果证据，因此难以支持 process-level attribution 和面向攻击来源的 recovery-oriented evidence construction。

- **第三类方法：host syscall/provenance 入侵检测**
  - 间接方法：非 CPS 场景下，针对 syscall trace 的恶意行为检测通常把程序运行时行为表示为系统调用序列或系统实体交互图。基于学习的方法使用 n-gram、frequency profile、TF-IDF、HMM、Markov model、SVM、LSTM、CNN、Transformer 或 NLP-style embedding 来学习正常 syscall sequence 与恶意 syscall sequence 的差异；基于规则的方法使用签名、if-then rules、RIPPER、automaton、专家规则或 CTI-derived graph pattern 匹配已知攻击行为；基于溯源图的方法将 process、file、socket 等实体构造成 provenance graph，通过因果依赖、信息流、图相似度、图嵌入或 GNN 检测 APT，并支持 attack reconstruction，例如 HOLMES、POIROT、UNICORN、KAIROS、ANUBIS、PROGRAPHER 等 [Forrest96; Warrender99; Liu18; Sworna23; Zipperle22; Li21; Holmes19; Poirot19; Unicorn20; Kairos24]。
  - 方法的局限：这类方法能较好刻画恶意程序的 OS-level 行为，但大多数不是为 CPS 场景设计。syscall-sequence 方法通常能判断进程行为是否异常，但缺少可解释的因果链；rule-based provenance 方法能匹配已知攻击模式，但依赖专家规则和先验攻击知识；learning-based provenance 方法能发现未知异常，但输出常是 anomalous node、time window 或 subgraph。部分在线 provenance investigation 系统能够恢复较紧凑的 host-side attack footprint，但其输出通常面向通用主机入侵调查，不一定能识别哪些 file、socket、device interface 或 control resource 与 CPS sensing/control path 相关，也不一定能形成后续 CPS 影响分析可直接使用的 evidence boundary。
  - 方法局限的根本原因：syscall/provenance 方法的语义中心是计算系统中的 process-file-socket 交互，而 CPS 攻击的安全后果发生在 cyber-physical loop 中。如果缺少 CPS 节点映射入口和面向控制相关资源的证据边界组织，系统级行为证据就难以直接转化为 CPS attack diagnosis、attribution 和 recovery-oriented decision support 的入口信息。

##### **现有相关方法的共性缺陷**

**总结现有相关方法的共同的根本的缺陷。**

- 最好是 1 个，不超过 3 个
- 导致缺陷的根本技术原因从**思想上**或者**大的原理上**讨论
- 避免讨论具体算法和工程实现
- 技术原因：如果这个缺陷不被解决，为什么这个问题就无论如何也解决不好
- 不是这个缺陷为什么困难
- **共性缺陷 1：** 现有方法缺少以低延迟进程定位为主目标、并能为后续 CPS 控制/物理影响分析提供细粒度证据入口的 provenance 模型。
  - 技术原因 1：现有 CPS/ICS 控制资产防御和 sensor/control path 数值异常检测主要把攻击投影到 PLC 程序、工业协议、控制命令、sensor reading、control signal、physical state 或 control invariant 等 CPS 侧观测空间中。因此，它们能够判断控制链路、控制资产或物理状态是否异常，却难以持续捕捉恶意程序 在控制主机 OS 层产生的 process、file、socket、device interface 和 syscall/provenance 行为，也难以及时回答“哪个进程通过哪些系统交互触发了这次 CPS 异常”。
  - 技术原因 2：现有 host syscall/provenance 方法虽然能够刻画程序的 OS-level 行为，但其语义中心通常停留在通用计算系统中的 process-file-socket 交互上。因此，它们可以报告 anomalous process、time window、node 或 subgraph；即使部分方法能够恢复较紧凑的 attack-related provenance graph，其解释对象仍主要是 host-side attack footprint，而不是面向 CPS 影响分析的 evidence boundary，因而难以判断哪些系统级证据应作为后续 CPS 影响分析的入口。
  - 技术原因 3：上述两类证据分别停留在 CPS 后果侧和 OS 行为侧，缺少统一的时间、实体和因果关联机制。结果是，CPS 侧告警往往低估攻击源定位问题，host 侧告警又难以说明其 CPS 相关性；二者都难以同时满足低延迟检测、process-level attribution，以及为后续 CPS 节点影响分析和恢复决策提供细粒度证据入口的需求。
  - 因此，问题的关键不只是增加一种新的异常检测器，而是建立一种低延迟、可用于 attribution 和后续 CPS 影响分析的 provenance evidence，使系统能够回答“谁触发了攻击”“通过什么系统行为触发”，并为后续判断“可能影响哪些 CPS 节点”提供明确的证据入口。

##### **解决共性缺陷的难点/挑战**

**列出解决现有相关方法的共性缺陷的难点/挑战。**
- 解决共性缺陷需要技术满足某些需求/达到一定指标，目前无法满足/达到：
- 技术需要资源太多/理论复杂度太高/存在 open problem
- 技术需求之间存在矛盾

- **技术需求：** 本文需要从连续产生的 syscall/provenance stream 中，低延迟捕捉可能短暂、局部且被正常行为稀释的恶意程序行为；同时，还需要把这些行为组织成 process-level CPS-aware provenance evidence boundary，使后续分析能够知道可疑进程是谁、相关系统实体是什么、这些实体如何连接到 sensor、actuator、controller node、control command 或 physical/simulation component 的入口。

- **技术需求矛盾 1：自适应粒度矛盾。**
  - 解释：provenance-based detection 需要足够大的上下文来保留 process、file、socket、device interface 和 syscall event 之间的因果关系；但 CPS 恶意程序可能只通过少量关键 syscall 或短步长行为触发 sensing/control path 上的影响，过大的窗口会把这些短攻击行为淹没在大量正常事件中，导致异常证据密度下降或定位粒度变粗。相反，过小的窗口虽然有利于低延迟捕捉局部异常，却可能截断因果链，使系统难以判断局部异常是否属于同一恶意程序行为。因此，核心挑战是如何自适应选择检测粒度，使系统既不丢失必要 causality，也不牺牲短攻击行为的可见性和响应时机。

- **技术需求矛盾 2：证据边界化矛盾。**
  - 解释：后续 CPS 影响分析和 recovery-oriented decision support 需要的是明确、可解释、控制相关的证据入口，而不是完整 host provenance graph 或单个异常节点。完整 provenance graph 通常过大，包含大量与 CPS 控制路径无关的系统交互，难以直接用于响应；单点告警或粗粒度异常窗口又会丢失攻击源、关键系统实体和控制相关资源边界。因而，系统需要在“保留足够细的 process-level 行为证据”和“输出足够紧凑、可交给后续 CPS 分析使用的 evidence boundary”之间取得平衡。

- **不作为本文主要挑战的问题：** 本文假设 provenance logs 由受保护机制采集，不主要解决日志基础设施被完全控制、审计记录被系统性伪造或回滚的问题。这类问题需要 secure logging、trusted execution、remote attestation 或独立监控通道等互补机制处理。

##### **Insights**

**针对现有相关方法的共性缺陷及其技术原因，提出新的 insights。**

- 1-3 条 insights，最好是 1 条，不要超过 3 条
- 避免算法/代码等技术细节

**Insight 1**：CPS 恶意程序的关键攻击行为可能在时间上短暂稀疏，但在因果上通常会集中表现为少数控制相关的 OS-level provenance evidence；因此，检测粒度应围绕 evidence density 与 causality preservation 自适应变化，而不是固定观察整段执行。

- 该 insight 能解决现有方法的共性缺陷并克服相关困难的原因：固定窗口在 CPS 场景中存在天然矛盾。窗口过大时，短攻击行为会被大量正常 syscall/provenance events 稀释，导致异常证据密度下降或定位粒度变粗；窗口过小时，又会丢失 process、file、socket、device interface 之间的因果上下文，难以判断局部异常是否属于同一恶意程序行为。围绕 evidence density 调整观察粒度，可以及时暴露可能进入 sensing/control path 的短攻击证据；同时保留必要的 provenance causality，可以避免把孤立事件误判为攻击，并为后续 attribution 提供更准确的证据边界。

**Insight 2**：对 CPS recovery-oriented analysis 有用的不是完整 host provenance graph，也不是单个异常告警，而是围绕控制相关资源切出的最小、可解释、process-level CPS-aware evidence boundary。

- 该 insight 能解决现有方法的共性缺陷并克服相关困难的原因：通用 host provenance 方法可以恢复 host-side attack footprint，但完整 provenance graph 往往包含大量与 CPS 控制路径无关的系统交互，难以直接用于 CPS 响应；相反，attack/no-attack、单个 anomalous node 或粗粒度 time window 又过于简化，无法说明异常进程通过哪些系统实体接触了 sensing/control path。本文所需的 evidence boundary 至少包括异常时间边界、异常进程节点、关键 syscall/provenance 子图、相关 file/socket/device/control resource，以及这些资源与 sensor、actuator、controller node 或 simulation component 的映射入口。这样的边界既比完整 provenance graph 更紧凑，又比单点告警更可解释；它不直接声称完成完整 CPS impact analysis 或 recovery action，而是为后续判断哪些 CPS 节点可能受到影响、以及如何制定恢复决策提供证据入口。

##### **新的 Insights 的本质区别**

**归纳出新的 insights 和现有方法的思路的本质区别。**
- 1-3 点区别，最好是 1 点，不要超过 3 点
- 避免具体的算法设计
- **强调本质区别**
- 采用不同的机器学习模型/方法不算本质区别
- **区别 1：**     TODO
  - 解释 1：TODO
  - 解释 2：TODO

##### **新的 Insights 的技术难点**

**归纳出新的 insights 在实现时需要解决的技术难点。**
- 技术难点：整体上的困难，如效率、准确性等

- **技术难点 1**：  TODO
  - 解释：TODO
  - 本工作技术方案：TODO

- **技术难点 2**：  TODO
  - 解释：TODO
  - 本工作方案：TODO


##### **新的 Insights 的正确性：Insight 1**

**从概念上解释新的 insight 1 的正确性。**
- 避免算法/代码等技术细节
- **理由 1**：TODO
  - 解释 1：TODO
  - 解释 2：TODO

##### **Motivating Example**

**给出一个简单示例。**

- 示例能说明现有技术的共性缺陷
- 示例能体现新的 Insights 能解决共性缺陷
- 示意图/代码：一个简单 CPS 控制回路：sensor 将物理状态发送给 controller，controller 根据读数产生控制命令，actuator 作用于 physical process；同时，控制主机上存在一个恶意程序，它通过短暂的 file/device/socket/syscall 操作对 sensor reading 或 control input 注入很小扰动。示意图应同时画出两条观测路径：一条是 physical-state path，只看到传感器读数的轻微偏移；另一条是 OS-level provenance path，可以看到恶意程序与相关 file、socket、device interface 或控制程序之间的交互。
- 具体地说明现有技术的共性缺陷：当恶意程序对 CPS 造成的数值扰动很小时，单纯依赖 threshold、residual 或 physical invariant 的检测方法会陷入两难：如果阈值设置得较高，小扰动会被当作正常噪声而漏报；如果阈值设置得很低，CPS 本身由于传感器噪声、执行器抖动、环境变化或控制误差产生的正常扰动也可能被识别为攻击，使系统过于敏感并产生大量误报。因此，这类方法即使能发现 physical-state anomaly，也难以判断该异常是正常系统抖动还是恶意程序触发的攻击，更难定位攻击来源和受影响节点。
- 具体地解释新的 Insights 能解决共性缺陷：新的 insights 不把轻微物理扰动本身作为唯一攻击证据，而是关注恶意程序在 CPS 主机上留下的 syscall/provenance trace。正常系统抖动可能改变 sensor reading，但不会伴随可疑进程访问控制接口、写入配置或日志、连接异常 socket、修改控制程序或操作 device file 等 OS-level 行为；相反，恶意程序即使只造成很小的 physical disturbance，也需要通过短暂、局部的系统调用与关键系统实体交互。围绕这些异常证据动态调整检测窗口，可以避免大窗口稀释短攻击行为，也避免小窗口完全丢失因果上下文；进一步将可疑进程、相关系统实体和受影响 CPS 节点连接起来，可以输出面向 recovery 的定位信息，而不只是报告“传感器读数异常”。


##### 新的 Insights 下解决难点/挑战的思路
**基于新的 Insights，针对前面指出的难点/挑战，提出解决思路。**
- 列出关键技术点
- 是 Insights 的细化

- 难点/挑战 1：TODO
  - 解决思路 1/关键技术点 1：TODO
  - 解释：TODO
- 难点/挑战 2：TODO
  - 解决思路 2/关键技术点 2：TODO
  - 解释：TODO

##### 设计方案：Overview
- 架构图：系统由四个主要部分组成：日志采集与规范化模块、自适应窗口控制模块、异常检测与证据汇总模块、recovery-oriented 输出模块。整体流程是：CPS 主机或仿真环境产生 syscall/provenance logs，日志经过规范化后进入检测系统；检测系统在当前窗口内计算异常分数，并估计扩大或缩小检测窗口带来的实时性变化；自适应窗口控制模块根据异常分数和实时性代价决定是否调整窗口大小以及调整幅度；最后，系统输出当前检测窗口、异常分数、可疑进程、相关系统实体、受影响 CPS 节点/组件等详细信息。
  **体现新的 Insights 的组件**：自适应窗口控制模块和 recovery-oriented 输出模块。前者体现 Insight 1：检测窗口由异常证据强度和实时性代价共同驱动，而不是固定观察整段执行；后者体现 Insight 2：告警输出不仅报告异常，还提供能够支持 recovery 的 process-level provenance evidence 和受影响节点定位。
  **架构输入**：syscall/provenance logs、当前检测窗口内的异常分数、窗口调整对检测延迟或实时性开销的影响估计，以及 CPS 环境中 process、file、socket、device interface 与 sensor、actuator、controller node 或 simulation component 之间的映射信息。
  **架构输出**：当前窗口是否异常、窗口调整决策、调整后的窗口大小、异常分数、可疑恶意程序、相关 file/socket/device interface、可能受影响的 CPS 节点或组件，以及用于 recovery 的定位提示。
  - 组件 1：日志采集与规范化模块
    - 功能：从 CPS 主机或仿真环境中收集 syscall/provenance logs，并将不同来源、不同顺序或可能存在轻微错乱的日志整理成检测系统可处理的事件流。该模块不直接判断攻击，而是为后续窗口调整和异常检测提供稳定输入。
  - 组件 2：自适应窗口控制模块
    - 功能：根据检测系统报告的 anomaly score 和修改窗口带来的 latency overhead，判断当前窗口是否过大或过小，并决定是否调整检测窗口以及调整幅度。其目标是在不过度牺牲实时性的前提下，保留足够 provenance causality，并避免短攻击行为被正常事件稀释。
  - 组件 3：异常检测与证据汇总模块
    - 功能：在当前窗口内分析 process、file、socket、device interface 等实体之间的 syscall/provenance 交互，计算当前状态的异常程度，并汇总可疑进程及其相关系统实体。该模块输出的不只是异常分数，还包括能够解释异常来源的行为证据。
  - 组件 4：鲁棒日志处理模块
    - 功能：提高检测系统对日志传输错乱的鲁棒性。当日志在传输到检测系统的过程中出现轻微乱序或局部错乱时，该模块尽量恢复事件的相对因果关系或降低乱序对检测窗口和异常分数的影响，使系统仍能保持较稳定的检测结果。
  - 组件 5：Recovery-oriented 输出模块
    - 功能：将异常检测结果转化为 CPS recovery 可使用的信息，包括可疑进程、相关系统实体、可能受影响的 sensor、actuator、controller node 或 simulation component。该模块帮助后续恢复过程执行隔离可疑进程、重置受影响节点、切换安全控制逻辑或触发局部状态恢复等动作。

##### **设计方案：组件1**

- 输入：TODO
- 输出：TODO
- 技术挑战 1：TODO
  - 解释： TODO
- 解决挑战的重要性（不解决挑战的影响）：TODO

- 现有相关技术方案（每种方案用一句话总结）：
  - 方案 1：TODO
  - 不能解决这一挑战：TODO
- 本工作技术方案：TODO

##### **实验：架构设计有效性**

实验验证新设计方案的有效性。
- 指标能够正确度量新设计方案的优势与额外代价
- 请对每个指标的合理性进行讨论，尽量采用其他顶会论文用过的指标

- 预期新架构有优势的方面以及度量优势的指标
  - 优势 1：指标 1
  - 优势 2：指标 2
  - 优势 3：指标 3
- 优势指标正确性：
  - 优势指标 1：原因 1
  - 优势指标 2：原因 2

- 预期新架构会付出额外代价的方面以及度量代价的指标：
  - 代价 1：指标 1
  - 代价 2：指标 2

- 代价指标正确性：
  - 代价指标 1：原因 1
  - 代价指标 2：原因 2

##### 实验：架构设计有效性

实验验证新设计方案的有效性。

- Baselines 能够代表现有相关方法的普遍情况
  - 每一类现有相关方法都需要有至少一个 baseline
- Baseline 开源：配置
- Baseline 不开源：复现

- 选取的 baselines：
  - Baseline 1：原因
  - Baseline 2：原因
- Baselines 的实现与正确性：
  - Baseline 1：实现与正确性
  - Baseline 2：实现与正确性
- 排除的 baselines 与原因
  - 排除的 Baseline 1：原因


##### 实验：架构设计有效性

实验验证新设计方案的有效性。
- Dataset/Benchmark 能够公平地评测 baselines
- Dataset/Benchmark 公开：如何使用
- Dataset/Benchmark 不公开：如何构造，为什么合理
- 选取的 datasets/benchmark：

- datasets/benchmark 1：描述/原因
- datasets/benchmark 2：描述/原因
- datasets/benchmark 的使用：
  - datasets/benchmark 1：配置
  - datasets/benchmark 2：配置
- 排除的 datasets/benchmark 与原因
  - 排除的 dataset/benchmark 1：原因

##### 实验：架构设计有效性

实验验证新设计方案的有效性。

- 实验方法能够反映真实世界的普遍情况
- 典型偏差 1：没有多次实验计算平均值和方差，与baseline比较没有计算统计显著性
- 典型偏差 2：baseline没有覆盖主要典型相关方法
- 典型偏差 3：实现未开源的baseline，没有讨论如何保证实现的正确性

- 实验 1：TODO
  - 实验方法为何能够反映普遍情况：TODO
  - 可能的偏见/误差：TODO
  - 避免偏见/误差的方法：TODO
  - 对于 baselines 的公平性：TODO

- 实验 2：...

##### 实验：架构设计有效性

实验验证新设计方案的有效性。
列出实验数据
- 优势：
  - 优势 1：指标数据 1，图或表，一句话总结实验结果
  - 优势 2：指标数据 2，图或表，一句话总结实验结果
  - 优势 3：指标数据 3，图或表，一句话总结实验结果
- 代价：
  - 代价 1：指标数据 1，图或表，一句话总结实验结果
  - 代价 2：指标数据 2，图或表，一句话总结实验结果
  - 代价 3：指标数据 3，图或表，一句话总结实验结果

##### 实验：讨论现有方法性能不够好的技术原因

现有方法的实验可能和其论文中的表述有出入，请详细分析解释原因
讨论现有方法效果不好的原因，为什么不好，你的实验中和他论文的实验中有什么不同，这些不同怎么导致了现有方法性能不好，这些不同是否是由于你的insight导致的

- baseline 1（SOTA） 性能不好：
  技术理由1：一句话总结
- baseline 2（Pruned） 性能不好：
  技术理由1：一句话总结

##### 实验：Insights 正确性

实验验证 Insights 的正确性。

- 分析优势 1：是由 insights 导致的某种现象引起的
  - 现象：TODO
  - 现象观察实验：TODO
    - 实验方法：TODO
    - 实验方法普适性：TODO
    - 实验方法正确性：TODO
  - 现象观察结果：观察到的数据，图或表，一句话总结

##### 实验：Insights 正确性

实验验证 Insights 的正确性。
证明 insights 的风险/局限能够被克服，或虽然存在但在实际情况中影响不大

- Insights 风险/局限 1：TODO
  - 能够被克服/不能被克服但对实际效果影响不大

  - 实验证明：TODO
    - 实验方法：TODO
    - 实验方法普适性：TODO
    - 实验方法正确性：TODO

  - 实验结果：图或表，一句话总结

##### 实验：重要模块对实验结果的影响

实验测量重要模块对实验结果的影响，证明各个模块的重要性。

- 重要模块 1：TODO
  - 实验方法：TODO
  - 实验方法普适性：TODO
  - 实验方法正确性：TODO
  - 实验结果：图或表，一句话总结

##### 关于实验部分的典型拒稿理由

- Does not follow accepted evaluation standard. The evaluation standard in the fuzzing community (Klees et al.'s "Evaluating Fuzz Testing") recommends statistical significance tests to compare competing tools, yet the authors did not perform any such tests in their evaluation. Moroever, the metric of "unique crashes" is widely known to be over-counted and unreliable, yet the authors list it as a key evaluation metric in comparing their approach to others.
- Unclear experimental setup. The paper claims that competing state-of-the-art tools fail on a large percentage of benchmarks, yet the authors provide only vague explanations of the supposed causes of failures. Recent literature shows that these tools do in fact support many of these benchmarks, suggesting there are discrepancies in the authors' experimental setup. Moreover, at least one of the competing tools has been obsolete for several years, and the authors fail to include the state-of-the-art successor in their evaluation.
