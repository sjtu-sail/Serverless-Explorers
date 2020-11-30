# Cloud Native Compution Foundation
### 介绍：致力于维护和集成开源技术，支持编排容器化微服务架构应用
CNCF 为开源项目提供了强大的服务支柱，除了代码的管理和技术上的决策，他还维护着这些开源项目的需求。
为了统一云计算接口和相关标准，2015 年 7 月隶属于 Linux 基金会的云原生计算基金会也即 CNCF 应运而生。谈到 CNCF，首先它是一个非营利组织，致力于通过技术优势和用户价值创造一套新的通用容器技术，推动本土云计算和服务的发展。CNCF 关注于容器如何管理而不是如何创建，因为如果没有一个成熟的平台去管理容器，那么大型企业无法真正放心接受并使用容器。

CNCF 吸收了多家会员，其中有铂金会员：
![avatar](http://blog.daocloud.io/wp-content/uploads/2017/05/Members-1.jpg)
黄金会员：
![avatar](http://blog.daocloud.io/wp-content/uploads/2017/05/Members02.png)
银牌会员：
![avatar](http://blog.daocloud.io/wp-content/uploads/2017/05/Members-Cloud-Native-Computing-Foundation.jpg)

CNCF自创立以来名下管理了多个云端原生技术项目，包括 Kuberenetes、Prometheus 和 gRPCs 等。
我的想法：跟我们的平台好像没啥卵关系。

# Cloud Zero
### 介绍：拉取数据，计算成本
    We pull data billions of times per second, any time an event occurs,
    from 74 different AWS service plug-ins, from workloads running in
    containers in Kubernetes, and from data streams like CloudTrail and CloudWatch
Cloud zero 通过将这些数据应用于机器学习，给予用户最有意义的成本数据（给予用户一种对这些服务的成本控制的指导）

# Dashbird
### 介绍：一个智能监控平台，可帮助用户去操作那些 AWS 中的复杂的 serverless 应用
    Easily understand and operate complex cloud environments. Get best
    practice guidance to improve your infrastructure. Receive instant 
    error and warning alerts. 
Dashbird 作为监控平台可以帮助用户易于理解和操作复杂的云环境，得到最好的技术指导，能够得到及时的报错信息和警告！可用于 debug！
可能有助于我们的云平台，因为亚纶说过注重 ui 和 debug 方面，Dashbird 监控能够一览所有serverless 应用 ( AWS 上的 )，同时得到及时的报错信息：
![avatar](https://mk0dashbirdiokg5ry66.kinstacdn.com/wp-content/uploads/2020/10/Screenshot-2020-10-12-at-15.58.27.png)

及时的报错信息：

![avatar](https://mk0dashbirdiokg5ry66.kinstacdn.com/wp-content/uploads/2020/10/Dashbird-insights.png)

架构建议：
Dashbird提供五十多次连续检查所有资源，这些资源涉及性能、成本、安全性和最佳实践等难以检测的问题，这可以对用户的无服务应用架构提出一些建议：
![avatar](https://mk0dashbirdiokg5ry66.kinstacdn.com/wp-content/uploads/2020/10/insights-.png)

# Hasura GraphQL
### 介绍：一个统一 API 管理您的所有数据
Hasura连接到您的数据库、 REST 服务器、 GraphQL 服务器和第三方 API，为所有数据源提供统一的实时 GraphQL API。
看了看，对我们可能没有什么用，主要是用于管理数据获取用的。

# lumigo
### 介绍：一个用于云和无服务应用程序的监控和故障排除平台
Lumigo是一个用于云和无服务器应用程序的监控和故障排除平台。使用智能监视、警报和端到端自动跟踪，快速识别和解决分布式环境中的关键问题。此外，您可以清楚地看到您的计算成本结构，并基于此做出设计决策。
与 Dashbird 有些相似，也可以帮助 debug：
![avatar](https://files.readme.io/09aca54-Dashboard.png)

# Node Lambda
### 介绍：命令行工具，帮助部署到 Amazon lambda
命令行工具，用于本地运行 node.js 应用程序并将其部署到 Amazon Lambda
github主页：
https://github.com/motdotla/node-lambda
帮助你了解其命令行中各种命令。

# SCAR
### 介绍：一个框架
打包自己的 docker 镜像，SCAR 可以将其镜像设置于 AWS Lambda 中运行，这样就不需要考虑各种语言了。
文档地址：https://scar.readthedocs.io/en/latest/intro.html

# Serverless Devs
### 介绍：一个开源开放的开发者平台
提供开发者工具以及 Serverless 应用中心

Serverless Devs Tool：一个无厂商绑定的、帮助开发者实现在 Serverless 架构下开发/运维效率翻倍的 开发者工具。开发者可以简单、快速的创建应用、项目开发、项目测试、发布部署 等，实现项目的全生命周期管理

Serverless Devs App Store：一个集 Serverless 应用在线搜索、一键部署于一体的 应用中心。分享、共建海量生产级项目模板、案例模板，供开发者自由选择，并支持一键部署到线上

# Sigma
### 介绍：一款IDE

SLAppForge Sigma IDE允许您通过简单地将组件拖放到web浏览器中的开发空间来开发无服务器应用程序。该编辑器通过允许您使用 Sigma 的富文本编辑器编辑代码，以及轻松地拖放元素。
Sigma 还负责所有平台级别的配置、资源管理、构建和部署，这意味着你可以完全专注于你的应用程序逻辑，忘掉其他一切。

![avatar](https://www.slappforge.com/docs/sigma/images/guide/function_developed.JPG)

在这款IDE上开发需要：
- 无服务计算的云平台证书：例如 AWS 或谷歌云平台。
- 一个 cloud version control 的账号：例如 GitHub 或 Bit Bucket。

# Stackery
### 介绍：一个无服务平台
帮助无服务应用的 Design，Develop 和 Delivery。

##### Design：
运行状况和指标汇总：
![avatar](https://media.graphcms.com/1ZSaMZ2JT7SmfLZccCbc)
提供了 serverless function 中的相当信息量，单击一下就能部署！

自动识别 AWS Automatic 角色：
![avatar](https://media.graphcms.com/Frck8qhS6SM3bVOrft3y)

CI/CD：
![avatar](https://media.graphcms.com/kVbO3EYnQSmk1afmaIGj)

于堆栈式快速了解变化
![avatar](https://media.graphcms.com/tImHKyQQRySg7TVb91vb)

    Know exactly which changes are being promoted between Stackery
    environments with an interactive git diff that summarizes changes and
    provides deep-links to your VCS provider to inspect specific app or 
    infrastructure changes.

设计您的架构：
![avatar](https://media.graphcms.com/aOaeeObpQIO8ul8EpKYO)


# Thundra
### 介绍：一站式服务
用于提交和操作云应用的一站式服务：
![avatar](https://www.thundra.io/hubfs/New%20Design%20Website%20-%202020/Features%20Page/SSs/BG-Features-01.svg)

快速启动：
用户启动一个程序，Thundra 利用 CloudWatch 日志加载该应用程序，用户则可以了解该应用的拓扑结构图，而不用花大量时间在通过一些细枝末节的地方去了解这个应用：
![avatar](https://www.thundra.io/hubfs/New%20Design%20Website%20-%202020/Features%20Page/SSs/BG-Features-02.svg)

分析：
Thundra 提供跨无服务器架构、容器和虚拟机的分布式应用程序的真正端到端管理。Thundra 用一种整体的方法解释了性能、成本和安全问题的根源：
![avatar](https://www.thundra.io/hubfs/New%20Design%20Website%20-%202020/Features%20Page/SSs/BG-Features-03-Update.svg)

检测异常行为：
使用数据科学和机器学习，识别出异常的应用程序行为模式可以被自动列入黑名单或采取其他自动操作。团队可以得到实时警报通知，也可以集成第三方工具，如 OpsGenie、PagerDuty、VictorOps、Slack 或 ServiceNow：
![avatar](https://www.thundra.io/hubfs/New%20Design%20Website%20-%202020/Features%20Page/SSs/BG-Features-04-Update.svg)

Debug：
本地 IDE 使用 Thundra 可以远程开启调试会话，帮助调试：
![avatar](https://www.thundra.io/hubfs/New%20Design%20Website%20-%202020/Features%20Page/SSs/BG-Features-05-Update.svg)