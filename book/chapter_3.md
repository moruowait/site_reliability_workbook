# chapter 3 SLO工程案例研究

尽管 SRE 的许多原则都是在 Google 墙内形成的，但它的原则早已存在于我们的大门之外。 许多标准的 Google SRE 实践已经被并行发现，或者被整个行业的许多其他组织采用。

SLO 是 SRE 模型的基础。 自从我们启动了客户可靠性工程（CRE）团队 - 一组经验丰富的SRE，帮助 Google Cloud Platform（GCP）客户构建更可靠的服务 - 几乎每个客户交互都以 SLOs 开始和结束。

在这里，我们介绍了两个非常不同的公司讲述的故事，概述了他们在与 Google CRE 团队合作时采用 SLO 和基于错误预算的方法的过程。 有关SLO和错误预算的更一般性讨论，请参阅本书中的第 2 章和第一本书中的第 3 章。

## Evernote 的 SLO 故事

*作者：Ben McCormack，Evernote*

Evernote 是一款跨平台应用程序，可帮助个人和团队创建，组装和共享信息。 我们在全球拥有超过 2.2 亿用户，我们在平台内存储了超过 120 亿条信息 - 混合了基于文本的笔记，文件和附件/图像。 在幕后，Evernote 服务由 750 多个 MySQL 实例支持。

我们向 Evernote 引入了 SLO 的概念，作为更广泛的技术改造的一部分，旨在提高工程速度，同时保持服务质量。 包括我们的目标：

- 将工程重点从数据中心中无差别的繁重工作转移到客户实际关心的产品工程工作上。 为此，我们停止运行物理数据中心并转移到公共云。
- 修改操作和软件工程师的工作模型，以支持提高特征速度，同时保持整体服务质量。
- 改进我们对 SLAs 的看法，以确保我们更加关注故障如何影响我们庞大的全球客户群。

许多行业的组织可能都熟悉这些目标。 虽然没有一种简单的方法来进行这些类型的更改将全面运作，但我们希望分享我们的经验将为面临类似挑战的其他人提供有价值的见解。

### 为什么 Evernote 采用 SRE 模型？

在此过渡开始时，Evernote 的特点是传统的运维/开发分离：运维团队保护生产环境的神圣性，而开发团队的任务是为客户开发新的产品功能。 这些目标通常是冲突的：开发团队感到受到冗长的操作要求的限制，而当新代码在生产中引入新问题时，运维团队感到沮丧。 当我们在这两个目标之间疯狂地摇摆时，运维组和开发团队发展了一种沮丧和紧张的关系。 我们希望达到一种更快乐的媒介，更好地平衡所涉及团队的不同需求。

我们试图在五年多的时间里以各种方式解决传统二分法中的差距。 在尝试了“你编写它，运行它”（开发）模型，以及“你编写它，我们为你运行它”（运维）模型后，我们转向了以 SLO 为中心的 SRE 方法。

那么是什么促使 Evernote 向这个方向发展呢？

在 Evernote，我们将运维和开发的核心学科视为工程师可以专业化的独立专业轨道。 其中一条轨道涉及近乎全天候向客户提供服务。另一方面关注该服务的扩展和发展，以满足未来客户的需求。 近年来，这两个学科相互走向，因为像 SRE 和 DevOps 这样的改变强调软件开发应用于运维。（数据中心自动化的发展和公共云的发展进一步推动了这种融合，这两者都为我们提供了一个可以完全由软件控制的数据中心。）另一方面，全栈所有权和持续部署越来越多地应用于软件开发。

我们被 SRE 模型所吸引，因为它完全接受并接受了运维和开发之间的差异，同时鼓励团队朝着共同的目标努力。 它不会尝试将运维工程师转变为应用程序开发人员，反之亦然。 相反，它给出了一个共同的参考框架。 根据我们的经验，错误预算/ SLO 方法导致两个团队在提出相同事实时做出类似的决定，因为它从对话中消除了大量的主观性。

### SLO 简介：正在进行的旅程

我们旅程的第一部分是从物理数据中心迁移到 Google Cloud Platform。<sup>1</sup> 一旦 Evernote 服务在 GCP 上运行并稳定运行，我们就引入了 SLO。 我们的目标有两个：

- 使团队内部围绕 Evernote SLO，确保所有团队都在新框架内工作。
- 将 Evernote 的 SLO 纳入我们与 Google Cloud 团队合作的方式，他们现在负责我们的底层基础架构。 由于我们现在在整体模型中有了新的合作伙伴，因此我们需要确保迁移到 GCP 不会稀释或掩盖我们对用户的承诺。

在积极使用 SLO 约 9 个月后，Evernote 已经开始使用其 SLO 实践的第 3 版了！

在深入了解 SLO 的技术细节之前，从客户的角度开始转换是非常重要的：您想要坚持哪些承诺？ 与大多数服务类似，Evernote 具有许多功能和选项，我们的用户可以通过各种创造性方式使用这些功能和选项。我们希望确保我们最初关注最重要和最常见的客户需求：Evernote 服务的可用性，以便用户访问和同步多个客户端的内容。我们的 SLO 之旅从这个目标开始。通过关注正常运行时间，我们保持了第一次通过。使用这种简单的第一种方法，我们可以清楚地表达我们测量的内容以及测量方法。

> 1 但这是另一本书的故事 - 请参阅http://bit.ly/2spqgcl了解更多细节。

我们的第一份 SLO 文件包含以下内容：

*SLO的定义*

这是一个正常运行时间测量：在每月窗口中测量 99.95％ 的正常运行时间，为某些服务和方法设置。 我们根据与内部客户支持和产品团队的讨论以及更重要的用户反馈来选择此数字。 我们故意选择将我们的 SLO 绑定到一个日历月而不是滚动期，以便在运行服务评估时保持我们的专注和有条理。

*衡量什么，以及如何衡量它*

> 衡量什么
>> 我们指定了一个服务端点，我们可以调用它来测试服务是否按预期运行。在我们的例子中，我们的服务中内置了一个状态页面，它可以运行我们的大部分堆栈并返回 200 状态代码（如果一切正常）。
    
> 怎么衡量
>> 我们想要一个定期调用状态页面的探测器。 我们希望探测器完全位于我们的环境之外并独立于我们的环境，因此我们可以测试所有组件，包括我们的负载平衡堆栈。 我们的目标是确保我们测量 GCP 服务和 Evernote 应用程序的任何和所有故障。 但是，我们不希望随机互联网问题触发误报。 我们选择使用专门建立和运行此类探测器的第三方公司。 我们选择了 [Pingdom](https://www.pingdom.com/)，但市场上还有很多其他产品。 我们按如下方式进行测量：
>>- **探测频率**：我们每分钟轮询一次前端节点。
>>- **探测器的位置**：此设置是可配置的; 我们目前在北美和欧洲使用多个探头。
>>- **“down”的定义**：如果探测器检查失败，则节点标记为 Unconfirmed Down，然后第二个地理位置独立的探测器执行检查。 如果第二次检查失败，则将节点标记为 down 以进行 SLO 计算。 只要连续探测请求注册错误，节点将继续标记为 down。

*如何从监控数据计算 SLO*

最后，我们仔细记录了我们如何根据从 Pingdom 收到的原始数据计算 SLO。例如，我们指定了如何考虑维护窗口：我们无法假设我们所有的数亿用户都知道我们发布的维护窗口。 因此，不知情的用户会将这些窗口视为通用和无法解释的停机时间，因此我们的 SLO 计算将维护视为停机时间。

一旦我们定义了 SLO，我们就必须对它们做些什么。 我们希望 SLO 能够推动软件和运营方面的变革，让我们的客户更快乐并让他们满意。 怎么做到最好？

我们使用 SLO/错误预算 概念作为分配资源的方法。 例如，如果我们错过了上个月的 SLO，那么这种行为有助于我们优先考虑相关的修复，改进和错误修复。 我们保持简单：来自 Evernote 和 Google 的团队每月都会对 SLO 性能进行审核。 在本次会议上，我们将审查上个月的 SLO 绩效，并对任何停机进行深入研究。 基于对过去一个月的分析，我们设置了可能未通过常规根本原因分析过程捕获的改进的行动项目。

在整个过程中，我们的指导原则是“完美是善的敌人。”即使 SLO 不完美，它们也足以指导长期改进。 “完美” SLO 将衡量每个可能的用户与我们的服务的交互，并解决所有边缘情况。 虽然这在纸面上是个好主意，但要实现完美（如果达到完美程度）还需要几个月的时间 - 我们可以用来改善服务的时间。 相反，我们选择了一个初始 SLO，它涵盖了大多数（但不是全部）用户交互，这是服务质量的良好代理。

自从我们开始以来，我们根据内部服务评估和响应客户影响停机的信号，对我们的 SLO 进行了两次修改。 因为我们一开始并不是为了完美的 SLO，所以我们很乐意进行更改以更好地与业务保持一致。 除了我们每月的 Evernote / Google 对 SLO 性能的评估之外，我们还确定了一个为期六个月的 SLO 审核周期，它在经常更换 SLO 并让它们变得陈旧之间取得了适当的平衡。 在修改我们的 SLO 时，我们还了解到，平衡您想要衡量的内容与可能的衡量标准非常重要。

自引入 SLO 以来，我们的运维和开发团队之间的关系有了微妙但显着的改善。 这些团队现在有一个共同的成功衡量标准：消除人类对服务质量（QoS）的解释使得两个团队都能够保持相同的观点和标准。 举一个例子，SLO 提供了一个共同点，当时我们必须在 2017 年的压缩时间线中促进多个版本。虽然我们追查了一个复杂的错误，但产品开发要求我们在多个单独的窗口中分配我们正常的每周版本，每个窗口都会对客户产生影响。 通过对问题应用 SLO 计算并从场景中消除人体主观性，我们能够更好地量化客户影响并将我们的发布窗口从五个减少到两个，以最大限度地减少客户的痛苦。

### 打破客户与云提供商之间的 SLO 墙

客户和云提供商关注点之间的虚拟墙可能看似自然或不可避免。 虽然谷歌为 GCP 平台提供了 SLO 和 SLA（服务水平协议），但我们运行 Evernote，但 Evernote 拥有自己的 SLOs 和 SLAs。 并不总是期望两个这样的工程团队会被告知彼此的 SLAs。

Evernote 永远不会想要这样的墙。 当然，我们可以设计一个分隔墙，将我们的 SLO 和 SLA 建立在基础 GCP 指标上。 相反，从一开始，我们就希望 Google 了解哪些性能特征对我们最重要，以及为什么。 我们希望将 Google 的目标与我们的目标保持一致，并让两家公司将 Evernote 的可靠性成功和失败视为共同责任。 为实现这一目标，我们需要一种方法：

- 协调目标

- 确保我们的合作伙伴（在本例中为Google）真正了解对我们重要的内容

- 分享成功和失败

大多数服务提供商为其云服务管理已发布的 SLO/SLAs。 在此上下文中工作很重要，但它无法全面表示我们的服务在云提供商的环境中运行得如何。

例如，给定的云提供商可能在全球运行数十万个虚拟机，它们可以管理正常运行时间和可用性。GCP 承诺计算引擎（即其虚拟机）的可用性为 99.95％。 即使 GCP SLO 图表为绿色（即高于 99.95％），Evernote 对同一 SLO 的看法可能也大不相同：因为我们的虚拟机占用空间仅占全球 GCP 数量的一小部分，因此我们所在地区的隔离（或隔离） 由于其他原因）可能会在整体汇总到全局级别时“丢失”。

为了纠正这样的情况，我们与 Google 分享 SLO 和 SLO 的实时性能。 因此，Google CRE 团队和Evernote 都使用相同的性能仪表板。 这似乎是一个非常简单的观点，但已被证明是一种非常强大的方式来推动真正以客户为中心的行为。 因此，Google 不会收到通用的“Service X 运行缓慢”类型的通知，而是向我们提供更具体的环境通知。 例如，除了通用的“ GCP 负载平衡环境今天运行缓慢”之外，我们还会被告知此问题对 Evernote 的 SLO 造成 5％ 的影响。 这种关系还可以帮助 Google 内部的团队，他们可以了解他们的行为和决策如何影响客户。

这种双向关系也为我们提供了一个非常有效的框架来支持重大事件。 大多数情况下，P1-P5 [Tickets](https://en.wikipedia.org/wiki/Ticket_(IT_security)) 和常规支持渠道的通常模式运作良好，使我们能够保持良好的服务并与 Google 建立良好的关系。 但我们都知道，有时P1 Tickets（“对我们的业务产生重大影响”）是不够的 - 整个服务上线的时间和您面临的扩展业务影响。

在这些时候，我们共享的 SLO 以及与 CRE 团队的关系得以实现。 我们有一个共同的理解，即如果 SLO 影响足够高，双方都会将问题视为具有特殊处理的 P1 Tickets。 很多时候，这意味着 Evernote 和 Google 的 CRE 团队在共享会议桥上迅速动员起来。 Google CRE 团队监控我们共同定义和商定的 SLO，使我们能够在优先级和适当响应方面保持同步。

### 当前状态

在积极使用 SLO 大约九个月之后，Evernote 已经在其SLO实践的第 3 版中。 下一版本的 SLO 将从我们简单的正常运行时间 SLO 开始。 我们计划开始探测单个 API 调用并考虑客户端的指标/性能视图，以便我们更好地表示用户 QoS。

通过提供标准和定义的 QoS 测量方法，SLO 允许 Evernote 更好地关注我们的服务运行方式。 我们现在可以在内部和谷歌之间进行数据驱动的对话，了解停机的影响，这使我们能够推动服务改进，最终建立更有效的支持团队和更快乐的客户。

## Home Depot 的 SLO 故事

作者：William Bonnell, The Home Depot

Home Depot（THD）是全球最大的家居装饰零售商：我们在北美拥有 2,200 多家商店，每家商店都拥有超过 35,000 种独特产品（并在线补充了超过 150 万种产品）。 我们的基础架构托管各种软件应用程序，每年支持近 400,000 名员工，处理超过 15 亿的客户交易。 这些商店与全球供应链和电子商务网站紧密集成，每年访问量超过 20 亿次。

在最近对我们旨在提高软件开发速度和质量的操作方法的更新中，THD 转向敏捷软件开发并改变了我们设计和管理软件的方式。 我们从支持大型单片软件包的集中支持团队转变为由小型独立运营的软件开发团队领导的微服务架构。 因此，我们的系统现在拥有更小的不断变化的软件块，这些软件也需要在堆栈中集成。

我们向微服务的转变得到了全栈所有权的新“自由和责任文化”的补充。 这种方法使开发人员可以自由地在需要时推送代码，同时也使他们共同负责其服务的运营。 对于这种共同所有权工作模式，运营和开发团队需要说一种促进问责制和跨越复杂性的共同语言：服务水平目标（SLO）。 相互依赖的服务需要知道如下信息：

- 您的服务有多可靠？ 它是为 3 个 9s，3 个半个 9s，还是 4 个 9s（或更好）构建的？ 有计划的停机时间吗？
- 在上限我可以期待什么样的延迟？
- 你能处理我要发送的请求量吗？ 你怎么处理超载？ 您的服务是否随着时间的推移实现了 SLO？

如果每项服务都能为这些问题提供透明和一致的答案，那么团队就可以清楚地了解他们的依赖关系，从而实现更好的沟通，增强团队之间的信任和责任感。

### SLO 文化项目

在我们的服务模式开始转变之前，Home Depot 没有 SLO 的文化。 监控工具和仪表板很多，但分布在各处，并且不会随着时间的推移跟踪数据。 我们并不总是能够在给定停机的根源上查明服务。 通常，我们开始在面向用户的服务中进行故障排除，然后向后工作直到我们发现问题，浪费了无数个小时。 如果服务需要计划停机时间，其依赖服务会感到惊讶。 如果一个团队需要建立一个三年半的 9s 服务，他们就不会知道他们有严格依赖的服务能否以更好的正常运行时间（4 个 9）来支持他们。 这些断开连接导致我们的软件开发和运维团队之间的混乱和失望。

我们需要通过建立 SLO 的共同文化来解决这些问题。 这样做需要一个影响人员，流程和技术的总体战略。 我们的努力跨越了四个方面：

*常见的白话*

在 THD 的上下文中定义 SLO。 定义如何以一致的方式测量它们。


*福音主义*

> 在整个公司传播这个词。
> - 创建培训材料以销售 SLO 的重要性，整个公司的路演，内部博客以及T恤和贴纸等宣传材料。
> - 争取一些早期采用者来实施 SLO 并向其他人展示其价值。
> - 建立一个吸引人的首字母缩略词（VALET;稍后讨论）以帮助传播这个想法。
> - 创建培训计划（FiRE 学院：可靠性工程基础），培训开发人员了解 SLO 和其他可靠性概念。<sup>2</sup>
>>  2 培训选项包括一小时的入门培训，半天的研讨会，以及成熟的 SRE 团队为期四周的沉浸式培训，以及毕业典礼和 FiRE 徽章。

*自动化*

> 为了减少采用的摩擦，实施度量收集平台以自动收集部署到生产的任何服务的服务级别指示器。 这些 SLIs 以后可以更容易地转换为 SLOs。

*激励*

> 为所有开发经理制定年度目标，为其服务设置和衡量 SLO。

建立一个共同的白话是让每个人都在同一页面上的关键。 我们还希望保持这个框架尽可能简单，以帮助这个想法更快地传播。 为了开始，我们仔细研究了我们在各种服务中监控的指标，并发现了一些模式。 每项服务都会监控某种形式的流量，延迟，错误和利用率指标，这些指标与 Google SRE 的[四个黄金信号](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals)密切相关。 此外，许多服务都可以从错误中明显监控正常运行时间或可用性。 不幸的是，所有指标类别都受到不一致的监控，命名不同或数据不足。

我们的服务都没有 SLO。 我们的生产系统与面向客户的 SLO 最接近的指标是支持票据。 我们测量部署到我们商店的应用程序可靠性的主要（通常也是唯一）方式是跟踪内部支持台收到的支持呼叫数量。

### 我们的第一套 SLO

我们无法为可以测量的系统的每个方面创建 SLO，因此我们必须决定哪些指标或 SLI 也应该具有 SLO。

#### API 调用的可用性和延迟

我们决定每个微服务必须具有其他微服务调用的 API 调用的可用性和延迟 SLO。 例如，Cart 微服务称为 库存微服务。对于那些API调用，库存微服务发布了 SLO，Cart 微服务（以及需要库存的其他微服务）可以查询以确定库存微服务是否能满足其可靠性要求

#### 基础设施利用

THD 团队以不同方式衡量基础设施利用率，但最典型的衡量标准是一分钟粒度的实时基础设施利用率。出于几个原因，我们决定不设置利用率 SLO。首先，微服务并不过分关注这个指标 - 你的用户并不真正关心利用率，只要你可以处理流量，你的微服务上升，它快速响应，它不会抛出错误 ，你没有失去容量的危险。此外，我们即将迁移到云计算意味着利用率不会受到关注，因此成本计划会使容量规划蒙上阴影。（我们仍然需要监控利用率并执行容量规划，但我们不需要将其包含在我们的 SLO 框架中。）

#### 流量

由于 THD 还没有容量规划文化，我们需要软件和运维团队的机制来传达他们的服务可以处理的数量。流量很容易定义为对服务的请求，但我们需要决定是否应该跟踪每秒平均请求数，每秒峰值请求数或报告时间段内的请求数。我们决定跟踪所有这三项，并让每项服务选择最合适的指标。我们讨论是否为流量设置 SLO，因为此度量标准由用户行为决定，而不是我们可以控制的内部因素。最终，我们决定作为零售商，我们需要为黑色星期五这样的峰值调整服务规模，因此我们根据预期的峰值容量设置 SLO。

#### 延迟

我们让每个服务定义其 SLO 以确定延迟并确定最佳测量位置。我们唯一的要求是服务应该通过黑盒监控来补充我们常见的白盒性能监控，以捕获由网络或其他层（如在微服务之外失效的缓存和代理）引起的问题。我们还确定百分位数比算术平均值更合适。 至少，服务需要达到 90％ 的目标; 面向用户的服务的首选目标是第 95 百分位数和第 99 百分位数或者第 99 百分位数。

#### 错误

错误有点复杂。 由于我们主要处理 Web 服务，因此我们必须标准化构成错误的内容以及如何返回错误。 如果 Web 服务遇到错误，我们自然会对HTTP响应代码进行标准化：

- 服务不应表明 2xx 响应正文中的错误; 相反，它应该抛出 4xx 或 5xx。
- 由服务问题（例如，内存不足）引起的错误应该引发 5xx 错误。
- 客户端引起的错误（例如，发送格式错误的请求）应该抛出 4xx 错误。

经过深思熟虑，我们决定跟踪 4xx 和 5xx 错误，但仅使用 5xx 错误来设置 SLO。 与我们针对其他 SLO 相关元素的方法类似，我们将此维度保持为通用，以便不同的应用程序可以将其用于不同的上下文。 例如，除 HTTP 错误外，批处理服务的错误可能是未能处理的记录数。

#### Tickets

如前所述，[Tickets](https://en.wikipedia.org/wiki/Ticket_(IT_security)) 最初是我们评估大多数生产软件的主要方式。 由于历史原因，我们决定继续跟踪其他 SLOs 的 Tickets。 您可以将此指标视为类似于“软件操作级别”之类的内容。

#### VALET

我们将新的 SLO 概括为一个方便的首字母缩略词：VALET。

*量（流量）*

> 我的服务可以处理多少业务量？

*可用性*

> 我需要的时候是服务吗？

*延迟*

> 我使用它时服务是否快速响应？

*错误*

> 我使用它时服务是否会抛出错误？

*Tickets*

> 该服务是否需要手动干预才能完成我的请求？

### SLOs 传福音化

凭借易于记忆的首字母缩略词，我们开始向企业传播 SLO:

- 为什么 SLO 很重要
- SLO 如何支持我们的“自由和责任”文化
- 应该测量什么
- 如何处理结果

由于开发人员现在负责其软件的运营，他们需要建立 SLO 以展示他们构建和支持可靠软件的能力，并且还需要与其服务和产品经理的消费者进行沟通，以获得面向客户的服务。 然而，大多数受众都对 SLA 和 SLO 等概念不满意，因此需要对这个新的 VALET 框架进行培训。

由于我们需要获得执行 SLO 的行政支持，我们的教育活动始于高层领导。 然后，我们逐一与开发团队会面，以支持 SLO 的价值观。 我们鼓励团队从他们的自定义度量跟踪机制（通常是手动）迁移到 VALET 框架。 为了保持这种势头，我们每周发送一份 VALET 格式的 SLO 报告，该报告与一般可靠性概念和从内部事件中吸取的经验教训相结合，与高级领导层合作。这也有助于构建业务指标，例如创建的采购订单（数量）或未能按 VALET 处理的采购订单（错误）。

我们还以多种方式扩展了我们的[传福音](https://en.wikipedia.org/wiki/Evangelism)：

- 我们建立了一个内部 WordPress 网站来托管有关 VALET 和可靠性的博客，与有用的资源相关联。
- 我们进行了内部技术讲座（包括 Google SRE 演讲嘉宾），讨论了一般可靠性概念以及如何使用 VALET 进行衡量。
- 我们举办了一系列 VALET 培训研讨会（后来演变为 FiRE 学院），并向所有想参加的人开放了邀请函。 这些研讨会的参加人数持续了好几个月。
- 我们甚至创建了 VALET 笔记本电脑贴纸和T恤，以支持全面的内部营销活动。

很快，公司里的每个人都知道 VALET，我们新的 SLO 文化开始占据上风。SLO 的实施甚至开始正式影响 THD 对开发经理的年度绩效评估。 虽然大约有 50 个服务每周定期捕获和报告其 SLO，但我们将这些指标临时存储在电子表格中。 虽然VALET 的想法像野火一样流行，但我们需要自动化数据收集以促进广泛采用。

### 自动化 VALET 数据收集

虽然我们的 SLO 文化现在有了强大的立足点，但 VALET 数据收集的自动化将加速 SLO 的采用。

#### TPS 报告

我们构建了一个框架来自动捕获部署到新 GCP 环境的任何服务的 VALET 数据。 我们将此框架称为 TPS 报告，这是我们用于数量和性能测试（每秒交易次数）的术语，当然，也是为了[嘲笑](https://static1.squarespace.com/static/52c1bc04e4b0410823c3b233/t/546eb062e4b06b70fedb2d0a/1416540259711/)<sup>3</sup>多个管理者可能想要查看这些数据的想法。 我们在 GCP 的 BigQuery 数据库平台之上构建了 TPS Reports 框架。 我们的 Web 服务前端生成的所有日志都被输入 BigQuery 以供 TPS Reports 处理。 我们最终还包括来自各种其他监控系统的指标，例如 Stackdriver 的可用性探针。

> 3 由于在 1999 年的电影 Office Space 中出名。

TPS 报告将这些数据转换为任何人都可以查询的每小时 VALET 指标。 新创建的服务自动注册到 TPS 报告中，因此可以立即查询。 由于数据全部存储在 BigQuery 中，因此我们可以跨时间帧有效地报告 VALET 指标。 我们使用此数据构建了各种自动报告和警报。 最有趣的集成是一个聊天机器人，让我们直接在商业聊天平台上报告服务的价值。 例如，任何服务都可以显示过去一小时的 VALET，VALET 与前一周，SLO 之外的服务以及聊天频道内的各种其他有趣数据。

#### VALET 服务

我们的下一步是创建一个 VALET 应用程序来存储和报告 SLO 数据。 由于 SLO 最适合用作趋势工具，因此该服务以每日，每周和每月粒度跟踪 SLO。请注意，我们的 SLO 是一种趋势工具，我们可以将其用于错误预算，但不直接连接到我们的监控系统。相反，我们有各种不同的监控平台，每个平台都有自己的警报。这些监控系统每天汇总其 SLO 并发布到 VALET 服务以进行趋势分析。 这种设置的缺点是监控系统中设置的警报阈值未与 SLO 集成; 但是，我们可以根据需要灵活地更换监控系统。

预计需要将 VALET 与未在 GCP 中运行的其他应用程序集成，我们创建了一个 VALET 集成层，该层提供 API 以每天为服务收集聚合的 VALET 数据。 TPS Reports 是第一个与 VALET 服务集成的系统，我们最终集成了各种内部部署应用平台（VALET 中注册的服务的一半以上）。

#### VALET 仪表板

VALET仪表板（如图 3-1 所示）是我们用于可视化和报告此数据的UI，并且相对简单。 它允许用户：

- 注册新服务。 这通常意味着将服务分配给一个或多个 URL，这些 URL 可能已经收集了 VALET 数据。
- 为五个 VALET 类别中的任何一个设置 SLO 目标。
- 在每个 VALET 类别下添加新的指标类型。 例如，一个服务可以跟踪第 99 百分位的延迟，而另一个服务跟踪第 90 百分位（或两者）的延迟。或者，后端处理系统可以在每日级别跟踪量（在一天中创建的购买订单），而客户服务前端可以跟踪每秒的峰值事务。

![image](/book/figure/Figure3-1.png)

*图 3-1 VALET 仪表板*

VALET 仪表板允许用户一次报告许多服务的 SLO，并以多种方式对数据进行切片和切块。例如，团队可以查看过去一周错过 SLO 的所有服务的统计信息。寻求审查服务性能的团队可以查看其所有服务及其所依赖的服务的延迟。VALET 仪表板将数据存储在一个简单的 Cloud SQL 数据库中，开发人员使用流行的商业报告工具来构建报告。

这些报告成为开发人员新的最佳实践的基础：定期对其服务进行 SLO 审核（通常是每周或每月）。基于这些评论，开发人员可以创建操作项以将服务返回到其 SLO，或者可能决定需要调整不切实际的 SLO。

### SLOs 的扩散

一旦 SLO 牢牢地巩固在组织的集体思想中，并且有效的自动化和报告已经到位，新的 SLO 迅速扩散。 在年初跟踪 SLO 约 50 项服务之后，截至今年年底，我们正在跟踪 SLO 的 800 项服务，每月约有 50 项服务使用 VALET 进行注册。

由于 VALET 允许我们在 THD 中扩展 SLO 的采用，因此开发自动化所需的时间非常值得。 但是，如果不能开发类似的复杂自动化，其他公司也不应该害怕采用基于 SLO 的方法。 虽然自动化提供了额外的好处，但首先编写 SLO 也是有好处的。

### 将 VALET 应用于批处理应用程序

当我们围绕 SLO 开发强大的报告时，我们发现了 VALET 的一些其他用途。 通过一些调整，批处理应用程序可以适用于此框架，如下所示：

*量*

> 处理的记录量

*可用性*

> 在一定时间内完成工作的频率（百分比）

*延迟*

> 作业运行所需的时间

*错误*

> 无法处理的记录

*Tickets*

> 操作员必须手动修复数据和重新处理作业的次数

### 在测试中使用 VALET

由于我们同时开发了 SRE 文化，我们发现 VALET 在我们的临时环境中支持我们的破坏性测试（混沌工程）自动化。 有了 TPS Reports 框架，我们就可以自动运行破坏性测试并记录服务的 VALET 数据的影响（或者希望没有影响）。

### 未来的愿望

通过 800 个服务（并且不断增长）收集 VALET 数据，我们可以使用大量有用的运维数据。我们对未来有几个愿望。

现在我们正在有效地收集 SLO，我们希望使用这些数据来采取行动。 我们的下一步是与 Google 类似的错误预算文化，当服务脱离 SLO 时，团队会停止推送新功能（除了提高可靠性）。为了保护我们业务的速度需求，我们必须努力在 SLO 报告时间范围（每周或每月）与违反 SLO 的频率之间找到一个良好的平衡点。 像许多采用错误预算的公司一样，我们正在权衡滚动窗口与固定窗口的优缺点。

我们希望进一步优化 VALET 以跟踪详细的端点和服务的使用者。目前，即使特定服务具有多个端点，我们也只在整个服务中跟踪 VALET。因此，很难区分不同的操作（例如，对目录的写入与对目录的读取;虽然我们分别监视和警告这些操作，但我们不跟踪 SLO）。同样，我们也希望为服务的不同消费者区分 VALET 结果。

虽然我们目前在 Web 服务层跟踪延迟 SLO，但我们还希望跟踪最终用户的延迟 SLO。此度量将捕获第三方标记，网络延迟和 CDN 缓存等因素如何影响页面开始呈现和完成呈现所需的时间。

我们还想将 VALET 数据扩展到应用程序部署。具体来说，在将更改推广到下一个服务器，区域或区域之前，我们希望使用自动化来验证 VALET 是否在容差范围内。

我们已经开始收集有关服务依赖性的信息，并且已经制作了一个可视化图表原型，该图表显示了我们未在调用树中访问 VALET 指标的位置。 新兴的服务网格平台将使这种类型的分析变得更加容易。

最后，我们坚信服务的 SLO 应该由服务的业务所有者（通常称为产品经理）根据其对业务的关键性来设置。至少，我们希望业务所有者设置服务正常运行时间的要求，并将 SLO 用作产品管理和开发之间的共享目标。虽然技术人员发现 VALET 很直观，但对于产品经理来说，这个概念并不那么直观。我们正在努力使用与它们相关的术语来简化 VALET 的概念：我们既简化了正常运行时间的选择数量，又提供了示例指标。 我们还强调从一个级别转移到另一个级别所需的大量投资。以下是我们可能提供的简化 VALET 指标的示例：

- 99.5％：商店员工未使用的应用程序或新服务的 MVP
- 99.9％：适用于 THD 的大多数非销售系统
- 99.95％：销售系统（或支持销售系统的服务）
- 99.99％：共享基础架构服务

以商业术语来衡量指标并在产品和开发之间共享可见目标（SLO！）将减少许多对大型公司常见的可靠性的错位预期。

### 总结

向大公司介绍一个新流程，更不用说新文化，需要一个好的策略，高管的支持，强大的传福音，简单的采用模式，以及最重要的耐心。像 SLO 这样的重大变革可能需要数年才能在公司中牢固地建立起来。我们想强调的是，Home Depot 是一家传统企业; 如果我们能够成功地引入如此大的变化，你也可以。您也不必一次完成此任务。虽然我们逐步实施 SLO，但制定全面的传福音策略和明确的激励结构促进了快速转型：我们在不到一年的时间内从 0 到 800 获得了 SLO 支持的服务。

## 结论

SLO 和错误预算是有助于解决许多不同问题集的强大概念。 这些来自 Evernote 和 Home Depot 的案例研究提供了非常真实的例子，说明如何实施SLO文化可以使产品开发和运维更紧密地结合在一起。这样做可以促进沟通并更好地为制定决策提供信息它最终将为您的客户带来更好的体验 - 无论这些客户是内部，外部，人类还是其他服务。

这两个案例研究强调 SLO 文化是一个持续的过程，而不是一次性修复或解决方案。 虽然它们共享哲学基础，但 THD 和 Evernote 的测量风格，SLI，SLO和实现细节明显不同。 这两个故事都通过证明 SLO 实施不需要特定于 Google 来补充 Google 对 SLO 的看法。 正如这些公司将 SLO 定制在自己独特的环境中一样，其他公司和组织也是如此。
