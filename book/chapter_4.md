# chapter 4 监控

## 监测策略的理想特征

在选择监控系统时，了解并优先考虑对您而言重要的功能非常重要。如果您正在评估不同的监控系统，本节中的属性可以帮助您了解哪种解决方案最适合您的需求。如果您已有监控策略，则可以考虑使用当前解决方案的一些其他功能。根据您的需要，一个监控系统可能会解决您的所有用例，或者您可能希望使用系统的组合。

### 速度

在数据的新鲜度和数据检索的速度方面，不同的组织将有不同的需求。

数据应该在您需要时可用：新鲜度会影响您的监控系统在出现问题时向您寻呼的时间。此外，数据缓慢可能会导致您意外地对不正确的数据进行操作。例如，在事件响应期间，如果原因（采取操作）和效果（看到监视中反映的操作）之间的时间太长，您可能会认为更改没有效果或推断原因和结果之间的错误关联。过时四到五分钟的数据可能会显着影响您对事件的响应速度。

如果您根据此标准选择监控系统，则需要提前确定速度要求。 当您查询大量数据时，数据检索的速度主要是一个问题。如果图表必须从许多受监控系统中计算大量数据，则可能需要一些时间来加载图表。为了加快速度较慢的图形，监控系统可以根据传入的数据创建和存储新的时间序列，这很有用。 然后它可以预先计算常见查询的答案。

### 计算

对计算的支持可以跨越各种复杂程度的各种用例。至少，您可能希望系统在多个月的时间范围内保留数据。如果没有长期的数据视图，就无法分析系统增长等长期趋势。就粒度而言，摘要数据（即您无法深入研究的汇总数据）足以促进增长计划。保留所有详细的个人指标可能有助于回答诸如“这种异常行为之前发生过吗？”之类的问题。但是，数据存储起来可能很昂贵或者检索起来不切实际。

您保留的有关事件或资源消耗的指标理想情况下应该是单调递增计数器。使用计数器，您的监控系统可以随时间计算窗口函数 - 例如，报告该计数器每秒的请求率。通过较长的窗口（最多一个月）计算这些速率，可以实现 SLO 基于刻录的警报的构建块（请参阅第 5 章）。

最后，支持更完整的统计函数可能很有用，因为琐碎的操作可能会掩盖不良行为。在记录延迟时支持计算百分位数（即第 50, 95, 99 百分位数）的监控系统将让您查看 50％，5％ 或 1％ 的请求是否太慢，而算术平均值只能告诉您 - 没有细节 - 请求时间较慢。或者，如果您的系统不直接支持计算百分位数，您可以通过以下方式实现此目的：

- 通过将请求中花费的秒数相加并除以请求数来获得平均值
- 通过扫描或采样日志条目来记录每个请求并计算百分位数值

您可能希望将原始度量标准数据记录在单独的系统中以进行离线分析 - 例如，在每周或每月报告中使用，或执行在监视系统中难以计算的更复杂的计算。

### 接口

强大的监控系统应该允许您在图表中简明地显示时间序列数据，还可以在表格或一系列图表样式中构建数据。您的仪表板将是显示监控的主要界面，因此选择最清晰显示您关注的数据的格式非常重要。一些选项包括热图，直方图和对数比例图。

您可能需要根据受众提供相同数据的不同视图; 高级管理层可能希望查看与 SRE 完全不同的信息。特别注意创建对消费内容的人有意义的仪表板。对于每组仪表板，始终如一地显示相同类型的数据对于通信很有价值。

您可能需要跨度量的不同聚合（例如机器类型，服务器版本或请求类型）实时绘制信息。 您的团队可以轻松地对您的数据执行钻取工作。通过根据各种指标对数据进行切片，您可以在需要时查找数据中的相关性和模式。

### 警报

能够对警报进行分类是有帮助的：多类警报允许进行比例响应。为不同警报设置不同严重性级别的功能也很有用：您可以提交票证以调查持续时间超过一小时的低错误率，而 100％ 错误率是一种值得立即响应的紧急情况。

警报抑制功能可以避免分散呼叫工程师的不必要噪音。 例如：

- 当所有节点都遇到相同的高错误率时，您只需警告一次全局错误率，而不是为每个节点发送单独的警报。
- 当您的某个服务依赖项具有触发警报（例如，慢速后端）时，您无需提醒您服务的错误率。

您还需要确保在事件结束后不再抑制警报。

您对系统的控制级别将决定您是使用第三方监控服务还是部署并运行您自己的监控系统。谷歌内部开发了自己的监控系统，但有大量的开源和商业监控系统可供使用。

### 监测数据的来源

您将选择的监控系统将通过您将使用的监控数据的特定来源。 本节讨论监视数据的两个常见来源：日志和指标。还有其他有价值的监视源，我们在这里不会介绍，例如[分布式跟踪](https://microservices.io/patterns/observability/distributed-tracing.html)和运行时内省。

度量是表示属性和事件的数值测量，通常以规则的时间间隔通过许多数据点来收获。日志是仅附加事件的记录。本章的讨论重点是结构化日志，它们支持丰富的查询和聚合工具，而不是纯文本日志。

Google 的基于日志的系统处理大量高度精细的数据。事件发生时和日志中可见之间存在一些固有的延迟。对于不是时间敏感的分析，可以使用批处理系统处理这些日志，使用即席查询进行查询，并使用仪表板进行可视化。此工作流程的一个示例是使用 [Cloud Dataflow](https://cloud.google.com/dataflow/) 处理日志，使用 [BigQuery](https://cloud.google.com/bigquery/) 进行[即席查询](https://zh.wikipedia.org/wiki/Ad_hoc#Ad_hoc_%E6%9F%A5%E8%AF%A2%EF%BC%88%E5%8F%88%E4%BD%9C%E2%80%9C%E5%8D%B3%E5%B8%AD%E6%9F%A5%E8%AF%A2%E2%80%9D%EF%BC%89)，使用 [Data Studio](https://datastudio.google.com/navigation/reporting) 进行仪表板处理。

相比之下，我们基于指标的监控系统从 Google 的每项服务中收集大量指标，提供的信息要少得多，但几乎是实时的。这些特性是其他基于日志和度量的监控系统的典型特征，但也有例外，例如实时日志系统或高基数度量。

我们的警报和仪表板通常使用指标。基于指标的监控系统的实时性意味着工程师可以非常快速地收到问题通知。我们倾向于使用日志来查找问题的根本原因，因为我们需要的信息通常不作为度量标准提供。

当报告不是时间敏感的时候，我们经常使用日志处理系统生成详细的报告，因为日志几乎总能产生比指标更准确的数据。

如果您根据指标发出警报，则可能很容易根据日志添加更多警报 - 例如，如果您需要在发生单个异常事件时收到通知。在这种情况下，我们仍建议使用基于指标的警报：您可以在特定事件发生时递增计数器指标，并根据该指标的值配置警报。此策略将所有警报配置保存在一个位置，使其更易于管理（请参阅第 67 页的“管理监控系统”）。

### 例子

以下实际示例说明了如何推断监控系统之间的选择过程。

#### 将信息从日志移动到指标

**问题。** HTTP 状态代码是 App Engine 客户调试错误的重要信号。此信息在日志中可用，但在度量标准中不可用。度量标准仪表板只能提供所有错误的全局速率，并且不包含有关确切错误代码或错误原因的任何信息。因此，调试涉及的问题的工作流程：

1. 查看全局错误图以查找发生错误的时间。
2. 读取日志文件以查找包含错误的行。
3. 尝试将日志文件中的错误与图表相关联。

日志记录工具没有给出规模感，因此很难知道在一个日志行中是否经常出现错误。日志还包含许多其他不相关的行，因此很难找到根本原因。

**提出的解决方案。** App Engine 开发团队选择将 HTTP 状态代码导出为度量上的标签(例如，requests_total{status=404}与requests_total{status= 500})。由于不同 HTTP 状态码的数量是相对有限的，这并没有将度量数据的容量增加到不切实际的大小，但确实为绘图和警报提供了最相关的数据。

**结果。** 这个新标签意味着团队可以升级图表，以显示不同错误类别和类型的单独行。客户现在可以根据暴露出来的错误代码，对可能出现的问题迅速形成猜测。我们现在还可以为客户机和服务器错误设置不同的警报阈值，从而使警报触发更准确。

#### 改进日志和指标

**问题。** One Ads SRE 团队维护了约 50 项服务，这些服务使用多种不同的语言和框架编写。该团队使用日志作为 SLO 合规性的规范来源。要计算错误预算，每个服务都使用日志处理脚本，其中包含许多特定于服务的特殊情况。这是一个处理单个服务的日志条目的示例脚本：

```
    If the HTTP status code was in the range (500, 599)
    AND the 'SERVER ERROR' field of the log is populated
    AND DEBUG cookie was not set as part of the request
    AND the url did not contain '/reports'
    AND the 'exception' field did not contain 'com.google.ads.PasswordException'
    THEN increment the error counter by 1
```

这些脚本很难维护，并且还使用了基于度量的监视系统无法使用的数据。由于指标会引发警报，因此有时警报不会与面向用户的错误相对应。每个警报都需要一个明确的分类步骤来确定它是否面向用户，这会减慢响应时间。

**提出的解决方案。** 该团队创建了一个库，该库连接到每个应用程序的框架语言的逻辑。该库确定错误是否在请求时影响用户。仪器在日志中写下了这个决定，并同时将其作为度量标准导出，从而提高了一致性。如果度量标准显示服务已返回错误，则日志包含确切的错误，以及请求相关的数据，以帮助重现和调试问题。相应地，日志中出现的任何影响 SLO 的错误也会改变 SLI 指标，然后团队可以提醒。

**结果。** 通过在多个服务之间引入统一的控制界面，团队重用了工具和警报逻辑，而不是实现多个自定义解决方案。所有服务都从删除复杂的特定于服务的日志处理代码中受益，从而提高了可伸缩性。 一旦警报直接与 SLO 相关联，它们就更明显可行，因此误报率显着下降。

#### 将日志保留为数据源

问题。 在调查生产问题时，一个 SRE 团队通常会查看受影响的实体ID，以确定用户影响和根本原因。与早期的 App Engine 示例一样，此调查需要仅在日志中可用的数据。在响应事件时，团队必须为此执行一次性日志查询。此步骤增加了事件恢复的时间：正确组合查询几分钟，加上查询日志的时间。

*提出的解决方案。* 该团队最初讨论了一个指标是否应该取代他们的日志工具。 与 App Engine 示例不同，实体 ID 可能具有数百万个不同的值，因此作为度量标签不实用。

最终，团队决定编写一个脚本来执行他们需要的一次性日志查询，并记录在警报电子邮件中运行的脚本。然后，如有必要，他们可以将命令直接复制到终端中。

*结果。* 团队不再具有管理正确的一次性日志查询的认知负担。因此，他们可以更快地获得他们需要的结果（尽管不如基于指标的方法那么快）。他们还有一个备份计划：他们可以在警报触发后自动运行脚本，并使用小型服务器定期查询日志以不断检索半新鲜的数据(产生不久的数据)。

## 管理您的监控系统

您的监控系统与您运行的任何其他服务一样重要。因此，应该给予适当的关注和关注。

### 将您的配置视为代码

将系统配置视为代码并将其存储在修订控制系统中是常见的做法，它们提供了一些明显的好处：更改历史记录，从特定更改到任务跟踪系统的链接，更简单的回滚和 linting 检查，以及强制执行的代码审查过程。

我们强烈建议您将监控配置视为代码（有关配置的更多信息，请参阅第 14 章）。支持基于意图的配置的监控系统优于仅提供 Web UI 或 [CRUD 样式](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) API 的系统。此配置方法是许多只读取配置文件的开源二进制文件的标准方法。一些第三方解决方案（如 [grafanalib](https://github.com/weaveworks/grafanalib)）为传统上使用 UI 配置的组件启用了此方法。

### 鼓励一致性

拥有多个工程团队并使用监控的大型公司需要达到良好的平衡：集中式方法提供一致性，但另一方面，各个团队可能希望完全控制其配置的设计。

正确的解决方案取决于您的组织。谷歌的方法随着时间的推移逐渐发展为在集中作为服务运行的单一框架上的融合。由于一些原因，此解决方案适用于我们。单一框架使工程师在切换团队时能够更快地提升，并使调试过程中的协作变得更加容易。我们还提供集中式仪表板服务，每个团队的仪表板都是可发现和可访问的。如果您轻松了解其他团队的仪表板，则可以更快地调试您的问题和他们的问题。

如果可能，请轻松进行基本监控。 如果您的所有服务<sup>2</sup>都导出一组一致的基本指标，则可以在整个组织中自动收集这些指标，并提供一组一致的仪表板。 此方法意味着您自动启动的任何新组件都具有基本监视功能。 贵公司的许多团队 - 甚至是非工程团队 - 都可以使用这些监控数据。

> 2 您可以通过公共库导出基本指标：像 OpenCensus 这样的检测框架，或者像 Istio 这样的服务网格。

### 喜欢松耦合

业务需求发生变化，一年后您的生产系统看起来会有所不同。同样，您的监控系统需要随着时间的推移而发展，因为它监控的服务通过不同的故障模式发展。

我们建议您将监控系统的组件松散耦合。您应该有稳定的接口来配置每个组件并传递监视数据。单独的组件应负责收集，存储，警告和可视化您的监控。稳定的接口使得更换任何给定组件更容易，以获得更好的替代方案。

将功能拆分为单个组件在开源世界中变得越来越流行。十年前，像 [Zabbix](https://www.zabbix.com/) 这样的监控系统将所有功能组合到一个组件中。现代设计通常涉及分离收集和规则评估（使用 [Prometheus 服务器](https://prometheus.io/)之类的解决方案），长期时间序列存储（[InfluxDB](https://www.influxdata.com/)），警报聚合（[Alertmanager](https://prometheus.io/docs/alerting/alertmanager/)）和仪表板（[Grafana](https://grafana.com/)）。

在撰写本文时，至少有两种流行的开放标准可用于检测软件和公开指标：

*[statsd](https://github.com/etsy/statsd)*

> 度量聚合守护进程最初由 Etsy 编写，现在移植到大多数编程语言。

*Prometheus*

一种开源监控解决方案，具有灵活的数据模型，支持元标签和强大的直方图功能。其他系统现在采用 Prometheus 格式，并且正在标准化为 [OpenMetrics](https://openmetrics.io/)。

可以使用多个数据源的单独仪表板系统可提供对服务的集中统一概述。谷歌最近在实践中看到了这一好处：我们的传统监控系统（Borgmon<sup>3</sup>）将仪表板与警报规则结合在同一配置中。在迁移到新系统（[Monarch](https://www.youtube.com/watch?v=LlvJdK1xsl4&feature=youtu.be)）时，我们决定将仪表板移动到单独的服务（[Viceroy](https://landing.google.com/sre/sre-book/chapters/communication-and-collaboration/)）中。 由于 Viceroy 不是 Borgmon 或 Monarch 的组成部分，因此 Monarch 的功能要求较少。由于用户可以使用 Viceroy 基于来自两个监控系统的数据显示图表，因此他们可以逐渐从 Borgmon 迁移到 Monarch。

> 3 有关 Borgmon 的概念和结构，请参见现场可靠性工程的第 10 章

## 有目的的度量标准

第 5 章介绍了在系统的错误预算受到威胁时如何使用 SLI 指标进行监控和警报。SLI 指标是您在基于 SLO 的警报触发时要检查的第一个指标。这些指标应显示在服务的信息中心中，最好位于其着陆页上。

在调查 SLO 违规的原因时，您很可能无法从 SLO 仪表板获得足够的信息。这些仪表板显示您违反了 SLO，但不一定是为什么。 监控仪表板应显示哪些其他数据？

我们发现以下指南有助于实施指标。这些指标应提供合理的监控，使您可以调查生产问题，并提供有关您的服务的广泛信息。

### 预期的变化

在诊断基于 SLO 的警报时，您需要能够从通知您的用户影响问题的警报指标转移到告知您导致这些问题的指标。最近对您的服务进行的预期更改可能是错误的。添加监视，通知您生产中的任何更改。<sup>4</sup>要确定触发器，我们建议执行以下操作：

- 监视二进制文件的版本。
- 监视命令行标志，尤其是在使用这些标志启用和禁用服务功能时。
- 如果配置数据动态推送到您的服务，请监视此动态配置的版本。

如果系统中的任何部分未进行版本控制，您应该能够监视上次构建或打包的时间戳。

当您尝试将停机与发布关联起来时，查看从警报链接的图表/仪表板要比在事后通过 CI / CD（持续集成/持续交付）系统日志更容易。

### 依赖

即使您的服务没有更改，其任何依赖项都可能会更改或出现问题，因此您还应该监视来自直接依赖项的响应。

以每个依赖项的字节，延迟和响应代码导出请求和响应大小是合理的。在选择要绘制的指标时，请牢记四个[黄金信号](https://landing.google.com/sre/sre-book/chapters/monitoring-distributed-systems/#xref_monitoring_golden-signals)。 您可以在度量标准上使用其他标签，以通过响应代码，RPC（远程过程调用）方法名称和对等作业名称对其进行细分。

理想情况下，您可以检测较低级别的 RPC 客户端库以导出这些度量标准，而不是要求每个 RPC 客户端库导出它们。<sup>5</sup>检测客户端库提供更高的一致性，并允许您免费监视新的依赖关系。

> 4 这是一种通过日志监控很有吸引力的情况，特别是因为生产变化相对较少。无论您使用日志还是指标，都应在仪表板中显示这些更改，以便轻松调试生产问题。

> 5 有关提供此功能的一组库，请参阅https://opencensus.io/

您有时会遇到提供非常狭窄的 API 的依赖项，其中所有功能都可通过名为 Get，Query 或同样无用的单个 RPC 获得，并且实际命令被指定为此 RPC 的参数。客户端库中的单个检测点与此类依赖关系不符：您将观察到延迟的高度变化和一些百分比的错误，这些错误可能会或可能不会表明此不透明 API 的某些部分完全失败。如果这种依赖关系很重要，那么您有几个选项可以很好地监控它：

- 导出单独的度量标准以定制依赖关系，以便度量标准可以解压缩它们收到的请求以获取实际信号。
- 要求依赖项所有者执行重写以导出更广泛的 API，该 API 支持跨单独的 RPC 服务和方法拆分的单独功能。

### 饱和

目的是监视和跟踪服务所依赖的每种资源的使用情况。某些资源具有您不能超过的硬限制，例如分配给您的应用程序的 RAM，磁盘或 CPU 配额。其他资源（如打开文件描述符，任何线程池中的活动线程，队列中的等待时间或写入的日志量）可能没有明确的硬限制，但仍需要管理。

根据使用的编程语言，您应该监视其他资源：

- 在 Java 中：堆和[元空间](https://plumbr.io/outofmemoryerror/metaspace)大小，以及更具体的度量标准，具体取决于您使用的垃圾收集类型
- 在Go：goroutine 的数量

语言本身为跟踪这些资源提供了不同的支持。

除了如第 5 章所述警告重大事件之外，您可能还需要设置在您为特定资源接近耗尽时触发的警报，例如：

- 当资源有硬限制时
- 超过使用阈值会导致性能下降

您应该拥有监控指标来跟踪所有资源 - 甚至是服务管理良好的资源。这些指标在容量和资源规划中至关重要。

### 服务流量情况

最好添加指标或指标标签，以便仪表板按状态代码细分服务流量（除非您的服务用于 SLI 目的的指标已经包含此信息）。以下是一些建议：

- 对于 HTTP 流量，监视所有响应代码，即使它们没有提供足够的警报信号，因为某些响应代码可能由不正确的客户端行为触发。
- 如果对用户应用速率限制或配额限制，请监视由于缺少配额而拒绝了多少请求的聚合。

### 实施有目的的度量标准

每个公开的指标都应该有用。仅仅因为它们易于生成而抵制导出少量指标的诱惑。相反，请考虑如何使用这些指标。公制设计或缺乏设计具有影响。

理想情况下，用于警报的度量标准值仅在系统进入问题状态时发生显着变化，并且在系统正常运行时不会发生变化。另一方面，调试指标没有这些要求 - 它们旨在提供有关触发警报时发生的情况的见解。良好的调试指标将指向可能导致问题的系统的某些方面。编写事后调查时，请考虑哪些其他指标可以让您更快地诊断问题。

## 测试警报逻辑

在理想情况下，监控和警报代码应遵循与代码开发相同的测试标准。 虽然 Prometheus 开发人员正在[讨论开发用于监控的单元测试](https://github.com/prometheus/prometheus/issues/1695)，但目前还没有广泛采用的系统允许您这样做。

在 Google，我们使用特定于域的语言测试我们的监控和警报，该语言允许我们创建合成时间序列。然后，我们根据派生时间序列中的值或特定警报的触发状态和标签存在来编写断言。

监控和警报通常是一个多阶段过程，因此需要多个单元测试系列。 虽然这个领域仍然很不发达，但如果你想在某个时候实施监控测试，我们建议采用三层方法，如图 4-1 所示。

![image](/book/figure/Figure4-1.png)

*图 4-1 监控测试环境层*

1. 二进制报告：检查导出的度量标准变量在特定条件下是否按预期变化。
2. 监控配置：确保规则评估产生预期结果，并且特定条件产生预期警报。
3. 警报配置：根据警报标签值测试生成的警报是否路由到预定目的地。

如果您无法通过合成方式测试您的监控，或者您的监控阶段无法进行测试，请考虑创建一个运行系统来导出众所周知的指标，例如请求数和错误数。您可以使用此系统来验证派生的时间序列和警报。配置后，您的警报规则很可能不会持续数月或数年，您需要确信当指标超过某个阈值时，正确的工程师会收到有意义的通知警报。

## 结论

由于 SRE 角色负责生产系统的可靠性，因此 SRE 通常需要非常熟悉服务的监控系统及其功能。如果没有这方面的知识，SRE 可能不知道在哪里查看，如何识别异常行为，或者如何在紧急情况下找到所需的信息。

我们希望通过指出我们认为有用的监控系统功能及其原因，我们可以帮助您评估监控策略与您的需求的匹配程度，探索您可能能够利用的一些其他功能，并考虑您可能想要做出的更改。您可能会发现将一些指标源和登录监控策略结合起来很有用；您需要的确切混合是高度依赖于上下文的。确保收集用于特定目的的指标。这样做的目的可能是更好地进行容量规划，协助调试或直接通知您有关问题的信息。

一旦您进行了监控，它就需要可见且有用。 为此，我们还建议您测试您的监控设置。 良好的监控系统可带来好处。 对于最能满足您需求的解决方案进行实质性思考并进行迭代直到您做对，这是非常值得的投资。