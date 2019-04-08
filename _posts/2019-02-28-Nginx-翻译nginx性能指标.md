# Nginx 性能监控

标签（空格分隔）： 未分类

---

## 什么是NGINX？

NGINX（发音为“engine X”）是一种流行的HTTP服务器和反向代理服务器。作为HTTP服务器，NGINX使用相对较少的内存非常有效且可靠地提供静态内容。作为反向代理，它可以用作多个后端服务器的单个受控访问点，也可以用作缓存和负载平衡等其他应用程序。NGINX可作为免费的开源产品或更全功能的商业分销版本NGINX Plus提供。NGINX也可以用作邮件代理和通用TCP代理，但本文并没有直接针对这些用例的NGINX监控。


----------

## 关键的NGINX指标

通过监控NGINX，您可以发现两类问题：NGINX本身的资源问题，以及Web基础架构中其他地方的问题。大多数NGINX用户将从监控中受益的一些指标包括每秒请求数，它提供了综合最终用户活动的高级视图;服务器错误率，表示服务器无法处理看似有效的请求的频率;请求处理时间，它描述了服务器处理客户端请求所花费的时间（以及可能指向环境中的速度减慢或其他问题）。
更一般地说，至少要观察三个关键指标类别：
- 基本指标
- 错误指标
- 性能指标

下面我们将分解每个类别中一些最重要的NGINX指标，以及值得特别提及的相当常见用例的指标：使用NGINX Plus进行反向代理。我们还将介绍如何使用您选择的图形或监控工具监控所有这些指标。本文引用了Monitoring 101系列中引入的度量标准术语，该系列提供了度量标准收集和警报的框架。


----------

### 基本指标

无论您的NGINX用例如何，您无疑都希望监控服务器接收的客户端请求数以及这些请求的处理方式。NGINX Plus可以像开源NGINX一样报告基本动态指标，但它也提供了一个二级模块，可以略微区别地报告指标。我们首先讨论开源NGINX，然后讨论NGINX Plus提供的额外报告功能。

### NGINX

下图显示了客户端连接的生命周期以及NGINX的开源版本如何在
连接期间收集指标。
![kTgu2q.png](https://s2.ax1x.com/2019/02/27/kTgu2q.png)

Accepts, handled, and requests 是不断增加的计数器，Active, waiting, reading, and writing 随着请求量的增长而缩小。

| Name |描述	|指标类型|
|--|--|--|
|accepts|	NGINX尝试接受的客户端连接数 |资源利用率|
|handled|	成功的客户端连接数 |	资源利用率|
|active|	当前活动的客户端连接|	资源利用率|
|dropped (calculated)|	丢弃连接数（接受 - 处理）|	错误|
|requests|	客户请求总数|	吞吐量|

当NGINX工作人员从OS获取连接请求时，**accepts**的计数器会递增。如果工作者未能获得请求的连接（通过建立新连接或重新使用打开的连接），则连接被丢弃并且**dropped**会增加。通常会删除连接，因为已达到资源限制，例如NGINX的worker_connections限制。

- Waiting: 如果此时没有活动请求，则活动连接也可以在等待子状态中。新连接可以绕过此状态并直接转到Reading，最常见的是使用“accept filter” 或 “deferred accept”，在这种情况下，NGINX在有足够的数据开始处理响应之前不会收到工作通知。如果连接设置为keep-alive，则在发送响应后，Connections也将处于Waiting状态。
- Reading: 收到新请求后，连接将退出等待状态，Reading计数器累加。在这种状态下，NGINX正在读取客户端请求标头。请求标头是轻量级的，因此这通常是一种快速操作。
- Writing: 读取请求后，Writing 计数器累加，并保持该状态，直到将响应返回给客户端。这意味着请求是Writing状态，而NGINX正在等待来自上游系统（“NGINX”后面的系统）的结果，而NGINX正在对响应进行操作。**请求通常会将大部分时间花在写作状态上**。
 通常，连接一次只能支持一个请求。在这种情况下，活动连接的数量==等待连接+读取请求+写入请求。但是，HTTP / 2允许多个并发请求/响应通过连接进行多路复用，因此Active可能小于等待，读取和写入的总和。


----------

### NGINX Plus

如上所述，所有开源NGINX的指标都可在NGINX Plus中获得，但Plus还可以报告其他指标。本节介绍仅可从NGINX Plus获得的指标。

![k7ZsB9.png](https://s2.ax1x.com/2019/02/28/k7ZsB9.png)

接受，丢弃和总数是不断增加的计数器。活动，空闲和当前跟踪每个状态中的当前连接或请求数，因此它们随请求量增长和缩小。

|Name|	Description|	Metric Type|
|--|--|--|
|accepted|	NGINX尝试的客户端连接数|	Resource: |Utilization
|dropped| 丢弃连接数|	Work: Errors
|active|	当前活动的客户端连接	|Resource: Utilization
|idle|	空闲连接|	Resource: Utilizations
|Total|	客户请求数	|Work: Throughput

当NGINX Plus工作人员从OS获取连接请求时，**accepted**的计数器会递增。如果工作者未能获得请求的连接（通过建立新连接或重新使用打开的连接），然后连接被丢弃并且丢弃递增。通常会删除连接，因为已达到资源限制，例如NGINX Plus的worker_connections限制

如上所述，Active和idle与开源NGINX中的“Active”和“Waiting”状态相同，但有一个关键例外：在开源NGINX中，“waiting” 包含于 “active”  ，而在NGINX Plus中，“idle”连接被排除在“Active”计数之外。**Current**与开源NGINX中的“Reading+Writing”状态相同。

**Total** 是客户端请求的累积计数。请注意，单个客户端连接可能涉及多个请求，因此此数量可能远远大于累积连接数。实际上，（总计/已接受）会产生每个连接的平均请求数。

|NGINX (open source)|	NGINX Plus
|--|--|
|accepts|	accepted
|dropped must be calculated	|dropped is reported directly
|reading + writing|	current
|waiting |	idle
|active (includes “waiting” states)|	active (excludes “idle” states)
|requests|	total


----------

## 重要指标：Dropped connections
Dropped connections 等于handled-accepts (NGINX)，或直接作为标准指标（NGINX Plus）公开。在正常情况下，丢弃的连接应为零。如果每单位时间的连接断开率开始上升，请查找可能的资源饱和度。
[![k7n0c8.md.png](https://s2.ax1x.com/2019/02/28/k7n0c8.md.png)](https://imgchr.com/i/k7n0c8)


----------

## 重要指标：Requests per second

以固定的时间间隔对您的请求数据（在requests开源版本中或Plus版本中）进行采样，可以为您提供每单位时间（通常为几分钟或几秒）的请求数。监控此度量标准可以提醒您传入的Web流量高峰，无论是合法的还是恶意的，或突然丢失，这通常表示存在问题。每秒请求的急剧变化可以提醒您在环境中的某个地方酝酿的问题，即使它无法准确地告诉您这些问题在哪里。请注意，无论其URL如何，所有请求都计为相同。

[![k7ny7j.md.png](https://s2.ax1x.com/2019/02/28/k7ny7j.md.png)](https://imgchr.com/i/k7ny7j)


----------

### 如何收集动态指标

开源NGINX在简单的状态页面上公开这些基本服务器指标。因为状态信息以标准化形式显示，几乎任何图形或监视工具都可以配置为解析相关数据以进行分析，可视化或警报，NGINX Plus提供了更丰富数据的JSON提要。阅读有关NGINX指标集的配套文章，了解有关启用指标收集的说明。


----------

### 错误指标

|Name |	Description|	Metric type|	Availability
|--|--|--|--|
|4xx codes|	Count of client errors such as "403 Forbidden" or "404 Not Found"|	Work: Errors|	NGINX logs, NGINX Plus
|5xx codes|	Count of server errors such as "500 Internal Server Error" or "502 Bad Gateway"|	Work: Errors|	NGINX logs, NGINX Plus

NGINX错误指标告诉您服务器返回错误的频率，而不是产生有用的工作。客户端错误由4xx状态代码，服务器错误和5xx状态代码表示。


----------

## 重要指标：Server error rate

您的服务器错误率等于5xx错误的数量，例如“502 Bad Gateway”除以每单位时间（通常为1到5）的状态代码总数（1xx，2xx，3xx，4xx，5xx）分钟）。如果您的错误率随着时间的推移开始攀升，则可能需要进行调查。如果突然出现高峰，则可能需要采取紧急措施，因为客户可能会向最终用户报告错误。

[![k7nW90.md.png](https://s2.ax1x.com/2019/02/28/k7nW90.md.png)](https://imgchr.com/i/k7nW90)

关于客户端错误的说明：虽然很容易监控4xx，但是您可以从该指标中获得有限的信息，因为它可以衡量客户端行为，而无需提供对特定URL的任何洞察。换句话说，4xx的变化可能是干扰，例如网络扫描仪盲目地寻找漏洞。


----------

### 收集错误指标

虽然开源NGINX不会立即使用错误率进行监控，但至少有两种方法可以捕获这些信息：

1、使用商业支持的NGINX Plus提供的扩展状态模块
2、配置NGINX的日志模块以在访问日志中写入响应代码

阅读有关NGINX指标集的配套文章，了解有关这两种方法的详细说明。

### 性能指标

|Name|	Description	|Metric type|	Availability|
|--|--|--|--|
|request time|	Time to process each request, in seconds|	Work: Performance	|NGINX logs|


----------


## 重要指标：Request processing time （请求处理时间）
NGINX记录的请求时间度量记录了每个请求的处理时间，从读取第一个客户端字节到完成请求。较长的响应时间可能指向上游问题。

### 收集处理时间指标

NGINX和NGINX Plus用户可以通过将$ request_time变量添加到访问日志格式来捕获处理时间数据。有关配置日志以进行监控的更多详细信息，请参阅我们的NGINX指标集合配套文章。

### 反向代理指标

|Name|	Description|	Metric type|	Availability|
|--|--|--|--|
|Active connections by upstream server|	Currently active client connections|	Resource: Utilization|	NGINX Plus|
|5xx codes by upstream server|	Server errors|	Work: Errors|	NGINX Plus|
|Available servers per upstream group|	Servers passing health checks|	Resource: Availability|	NGINX Plus|

使用NGINX的最常用方法之一是作为反向代理。商业支持的NGINX Plus公开了大量关于后端（或“上游”）服务器的指标，这些指标与反向代理设置相关。本节重点介绍NGINX Plus用户可以使用的一些关键上游指标。

NGINX Plus首先按组划分其上游指标，然后按个别服务器划分。因此，例如，如果您的反向代理正在向五个上游Web服务器分发请求，您可以一眼看出这些单个服务器是否负担过重，以及上游组中是否有足够的健康服务器以确保良好的响应时间。


----------

## 动态指标

每个上游服务器的活动连接数可以帮助您验证反向代理是否正确地在整个服务器组中分配工作。如果您使用NGINX作为负载平衡器，任何一台服务器处理的连接数量的显着偏差都表明服务器正在努力及时处理请求或者负载平衡方法（例如，循环或IP散列）您配置的流量模式不是最佳选择。

### 错误指标
回想一下上面的错误度量部分5xx（服务器错误）代码，例如“502 Bad Gateway”或“503 Service Temporarily Unavailable”，是一个值得监控的重要指标，尤其是作为总响应代码的一部分。NGINX Plus允许您轻松提取每个上游服务器的5xx代码数量以及响应总数，以确定特定服务器的错误率。


----------

## 可用性指标

有关Web服务器运行状况的另一种视图，NGINX还可以通过每组中当前可用的服务器总数来监控上游组的运行状况。在大型反向代理设置中，您可能不太关心任何一台服务器的当前状态，只要您的可用服务器池能够处理负载。但是，监视每个上游组中的服务器总数可以提供Web服务器总体运行状况的高级视图。


----------

## 收集上游指标

NGINX Plus上游指标在内部NGINX Plus监控仪表板上公开，也可通过JSON接口获得，该接口可将指标提供给几乎任何外部监控平台。请参阅我们的收集NGINX指标的配套文章中的示例。


----------


## 结论

在这篇文章中，我们已经介绍了一些可以监控的最有用的指标，以便密切关注NGINX服务器。如果您刚刚开始使用NGINX，则监控下面列表中的大部分或全部指标将提供对Web基础结构的运行状况和活动级别的良好可见性：
- Dropped connections
- Requests per second
- Server error rate
- Request processing time

最终，您将认识到与您自己的基础架构和用例特别相关的其他更专业的指标。当然，您监控的内容取决于您拥有的工具和可用的指标。有关度量标准收集的分步说明，请参阅配套文章，无论您使用的是NGINX还是NGINX Plus。

在Datadog，我们已经与NGINX和NGINX Plus进行了集成，因此您可以通过最少的设置开始收集和监控所有Web服务器的指标。
在这篇文章中使用Datadog监控NGINX，并立即开始免费试用Datadog。