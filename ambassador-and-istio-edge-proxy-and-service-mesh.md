> 原文地址：<https://dzone.com/articles/the-importance-of-control-planes-with-service-mesh>
>
> 作者：[Daniel Bryant](https://dzone.com/users/1161205/daniel-bryant-uk.html)
>
> 译者：[navy](https://github.com/meua)
>
> 校对：[宋净超](http://jimmysong.io)

# 具有服务网格和边缘代理的控制平面的重要性

了解为什么服务网格和前端代理如此重要以及它们与持续交付的关系。

了解现代云架构如何使用微服务具有的许多优势，使开发人员能够以CI/CD方式交付业务软件。

去年，Matt Klein写了一篇精彩博客“[服务网格数据平面与控制平面](https://blog.envoyproxy.io/service-mesh-data-plane-vs-control-plane-2774e720f7fc)”。尽管我已经很熟悉“控制面板”这个术语。Matt让我完全理解我过去对这个概念的体验，以及与软件持续交付有关的重要性-特别是在入口/边缘网关和服务网格周围的部署控制（和细微差别）方面。

我之前写过关于边缘代理或API网关在软件交付中可以发挥的作用，持续交付：API网关如何帮助（或阻碍）？关于像Envoy这样的现代代理在“云原生”应用程序操作中所产生的影响，我们进行了几次很好的讨论。通过这一点，我得出的结论是，尽管微服务，具有动态编排的容器和云技术的使用提供了新的机会，剩下的核心挑战之一是我们的控制平面必须进行调整才能跟上变化的步伐。

## 控制平面和角色

在Matt的文章中，他指出服务网格控制平面“为网格中所有正在运行的数据平面提供策略和配置”，并且“控制平面将所有数据平面转变为分布式系统。”最终，控制平面的目标是设置将由数据平面制定的策略。控制平面可以通过配置文件，API调用和用户界面来实现。选择的实现方法通常取决于用户的角色，以及他们的目标和技术能力。例如，产品所有者可能想要在应用程序中发布新功能，这里UI通常是最合适的控制平面，因为这可以显示系统的可理解视图并且还提供一些导轨。但是，对于想要配置一系列低级防火墙规则的网络运维人员，使用CLI或配置文件将提供更细粒度（高级用户风格）控制，并且还便于自动化。

控制平面的选择也可能受所需控制范围的影响。我的同事[Rafi之前在QCon SF讨论过这个问题](https://www.infoq.com/news/2017/11/service-oriented-development)，集中或分散运营的要求肯定会影响控制平面的实施。这也直接关系到控制影响应该是本地的还是全局的。例如，运营团队可能希望指定全局合理的默认值和安全措施。但是，在前线工作的开发团队需要对其本地服务进行细粒度控制，并且可能（如果他们正在接受“自由和责任”模式）覆盖安全措施的能力。Matt还在最近的[QCon纽约演讲](https://www.infoq.com/news/2018/07/qcon-klein-service-mesh)中谈到了本地/全局互动，并展示了Lyft团队为服务到服务和边缘/入口代理创建的仪表板：

 ![](https://cdn-images-1.medium.com/max/800/1*QjLNa1Wh0Y_F87JghLpPUA.png) 
 
 ## 东西向流量与南北向流量
 
 在软件应用程序中流量的两种典型分类，其中之一是南北向流量，通常称为入口流量，流量流向外部系统或者外部服务调用内部系统。另外一个是南北向流量，通常称为数据中心内部流量，这是在（可能是虚拟化的）内部网络边界内流动的流量
 
 所谓东西向，大家能理解吧？东西向指服务间通讯，也就是A服务调用B服务。对应的还有南北向，南北向通常是指从外部网络进来调用服务，如走API Gateway调用服务。在东西向通讯中，我们有时会需要一个比较特殊的途径，比如说在这个图中，我们有两个集群，两个集群各有各自的服务注册中心。我们通过增强Pilot的方式打通两个注册中心，可以知道对方有什么服务。
 
  ![](https://ws1.sinaimg.cn/large/00704eQkgy1fsy0kakg35j30qo0f0dpi.jpg)
 
 在现代云原生应用程序中，两个独立的组件通常控制这些流量：API网关或前端代理处理南北流量，相对的service mesh处理东西向流量。在Kubernetes域内，[]( Ambassador )开源API网关可以处理入口流量，而[Istio](https://istio.io/)开放平台可以处理跨服务流量。
 
对于南北向和东西向代理组件，底层网络技术可以是相同的（[Envoy](https://www.envoyproxy.io/)）。但是，控制平面通常是不同的，基于与系统交互的角色。

Ambassador控制面板的主要目标是开发人员，并允许将简单的注释添加到Kubernetes配置中以控制核心部署功能，如路由、金丝雀发布、速率限制。

Istio关注的主要角色是运维人员，并且控制平面允许指定额外的Kubernetes资源以促进流量管理（包括失败注入），安全（基于角色的访问控制和认证安全）和遥测（包括分布式追踪和各监控指标）。

## 结论：分歧或趋同

Lyft使用Envoy作为边缘代理和service mesh，我还听到有关工程师使用Ambassador 来管理服务间（东西向）通信的报道，以及Istio处理入口流量（甚至在[v1.0发布](https://www.infoq.com/news/2018/08/istio-1.0-service-mesh)的新网关功能之前），然而，目前Ambassador 和Istio所代表的代理技术控制平面的两种方法似乎为开发商和运营商各自的角色提供了好处。鉴于我们对现代容器网络的整体知识和经验状况，我还不确信有一个简单的一刀切解决方案。因此，我认为用于管理南北和东西交通的控制平面解决方案可能会在最终收敛于终极解决方案之前出现分歧。

---

使用[Instana APM](https://dzone.com/go?i=290421&u=https%3A%2F%2Fwww.instana.com%2Ftrial%3Futm_source%3DdZone%26utm_medium%3Dpre_post_article_text_ad%26utm_campaign%3Dinstana_trial%26utm_content%3Dgot_cloud_get_instana) | [自动管理容器和微服务](https://dzone.com/go?i=290421&u=https%3A%2F%2Fwaw.instana.com%2Ftrial%3Futm_source%3DdZone%26utm_medium%3Dpre_post_article_text_ad%26utm_campaign%3Dinstana_trial%26utm_content%3Dgot_cloud_get_instana)，具有更好的控制和性能。[今天](https://www.instana.com/trial/)就试试吧。