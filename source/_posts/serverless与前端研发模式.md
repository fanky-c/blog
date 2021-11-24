---
title: serverless与前端研发模式
date: 2021-11-18 21:52:57
tags:
 - serverless
 - BaaS
 - FaaS
 - IaaS
 - PaaS
 - SaaS
---

### 名称解释
> 1. IaaS:  IaaS：Infrastructure as a Service（基础设施即服务）;IaaS处于最底层，服务商提供底层/物理层基础设施资源（服务器，数据中心，环境控制，电源，服务器机房），客户自己部署和执行操作系统或应用程序等各种软件。 **例如：你提供他人厨房、煤气、炉子，他人使用基础设施， 来烤自己的披萨**
> 
> 2. PaaS:  PaaS：Platform as a Service（平台即服务）; PaaS处于中间层，服务商提供基础设施底层服务，提供操作系统（Windows，Linux）、数据库服务器、Web服务器、域控制器和其他中间件，以及服务模型中的备份服务等中件层服务。例如IIS，.NET，Apache，MySQL …，客户自己控制上层的应用程序部署与应用托管的环境。**例如：你提供他人披萨皮，他人只要设计披萨口味， 来烤自己的披萨**
> 
> 3. SaaS:  SaaS：Software as a Service（软件即服务）; SaaS处于最上层，服务商提供基于软件的解决方案，满足客户最终需求；如OA、CRM、MIS、ERP、HRM、CM、Office 365、iCloud、G Suite等应用，客户不需考虑任何形式的专业技术知识，获得完整的软件包，使他们的日常工作和生活变得更轻松。**例如：你提供完整的披萨，到手是一个成品；他人只要设计好自己logo， 来卖自己的披萨**
> 
> 4. BaaS:  BaaS：Backend as a Service（后端即服务）;  服务商为客户(开发者)提供整合云后端的服务，如提供文件存储、数据存储、推送服务、身份验证服务等功能，以帮助开发者快速开发应用。
> 
> 5. FaaS:  FaaS：Function as a service（函数即服务）; 服务商提供一个平台，允许客户开发、运行和管理应用程序功能，而无需构建和维护通常与开发和启动应用程序相关的基础架构的复杂性。 按照此模型构建应用程序是实现“无服务器”体系结构的一种方式，通常在构建微服务应用程序时使用。

### 前端发展历程和挑战
#### 1. 发展历程
##### pc时代
主要工作是切图、写样式、写交互、浏览器兼容； 还会做一些组件化、前后端分离的事情； ***总结就是专注于浏览器领域***

##### 移动互联网时代
跨端方向： h5、react native、 flutter；工程化方向： 基于nodejs前端工程 ***总结前端跨出了浏览器， 走向了android、ios和前端工程化*** 

##### serverless时代  
前端的工作更加丰富化，从客户端走向服务端，从单端走向多端： ssr、 bff、csr、微前端 ***总结前端整体向”前端全栈化“的方向发展*** 
#### 2. 挑战
##### 知识体系
除了传统的js、css、node、前端框架、打包工具、前端工程化。 还会涉及到后端领域：egg、redis、网关、mysql、rpc服务等。 运维领域： 容器、监控和告警
##### 工作内容
除了偏前端的工作：写页面、组件、性能优化、跨端、前端工程化。 还会涉及到后端和运维的工作：bff、ssr、网关、监控告警和集群化部署

### 前端面对日益复杂的业务场景
#### 1. CSR
CSR: 是一种目前流行的渲染方式，它依赖的是运行在客户端的JS，用户首次发送请求只能得到小部分的指引性HTML代码。第二次请求将会请求更多包含HTML字符串的JS文件。 ***使用场景：C端业务***
#### 2. SSR
SSR: 传统的渲染方式，由服务端把渲染的完整的页面吐给客户端。这样减少了一次客户端到服务端的一次http请求，加快相应速度，一般用于首屏的性能优化。 ***使用场景：对首屏要求高、有内容分享需求***

#### 3. BFF
BFF: Backend For Frontend (后端服务于前端) ，前后端胶水层。 ***使用场景：减少前后端沟通成本、多端应用适配、让后端更加专注于原子化服务***
#### 4. 微前端
微前端：微前端是一种多个团队通过独立发布功能的方式来共同构建现代化 web 应用的技术手段及方法策略。 ***使用场景：后端管理系统***

### 传统的研发模式和挑战
#### 1. 研发流程
<img src="/img/csr.webp" height = "auto" align=center />
如图可得： 
1. cdn服务
2. 静态server托管html。 含文件服务、登陆认证、AB 灰度等功能; 运维部署需要域名申请、lvs接入、机器资源
3. BFF服务。 需要运维强介入，域名申请、lvs接入、机器资源。（ 如果接入网关基本无需运维介入 ）

#### 2. 面临挑战
1. 运维成本高、流程长； 需要换种qps、cpu和内存等指标
2. 开发成本高； 不仅需要开发业务、更需要开发基础服务
3. 机器资源利用率的浪费，无法精确预估流量 （使用k8s动态扩容可以无视）
4. 对前端要求更高， 招聘成本更难
   
### 基于serverless的研发模式
#### 1. 策略和架构
##### 基础能力平台化
<img src="/img/serverless_2.webp" height = "auto" align=center />

> 业务再也不需要关注前端基础服务的开发，平台提供，开箱即用，简单配置即可

##### 友好的开发体验
<img src="/img/serverless.webp" height = "auto" align=center />

> 用户在开发 SSR 和 BFF 的过程中无需感知到差异性，和开发 CSR 一样轻松
   
##### 整体架构
<img src="/img/serverless_3.webp" height = "auto" align=center />

#### 2. cicd
<img src="/img/ci.webp" height = "auto" align=center />

#### 3. 实践
### serverless监控和运维
#### 1. 业务监控和指标大盘
1. grafana报表
   
#### 2. 日志系统
1. 用户行为打点日志
#### 3. 运行时的监控和调试
1. sentry错误监控
2. apm前端资源和接口性能监控
3. easy-monitor服务端性能


<br/>
[文章来源于](https://mp.weixin.qq.com/s/J2fHm_mR7UE65q1vSQ9xpA?st=4C7DB6F51F132A3368EFB533BCD7A842B49302A24C9477FEF33A439C6057A1F595541AEE2507F7D0CB08FD444CC58669B1141CEF1EFB448E84E1E825E66F504045EBF38276934F3E35D21BF73B76272A96E1B0AAFC936CBB482A7F390990EAE634528C4DFD88BD0F8B70FFCE7AC31F5AE9FDD6C618736BA66C81871C490D1388F6EF36CDB44ADD6259F52C61DD87E1B9BFAFBA47A642670BA4677C94EDD0D78F38F958F30B026FFF29B3CEA80EFD5E96C67F14EDB9F3EC31B62BB153DBEFC37B3A78E43240E76183903CD576EDF7A624A2FA101E6FC3CCE984436F5C7EA13572&vid=1688851884861646&cst=0B8E0646D00C493A9AF905E0F7F1AD4DB95E6574B50C74C6B2C1D12178C7839181D489E09EA6DDF4D3396322BD31131D&deviceid=fae71291-ae83-4e26-9054-666127dd2311&version=3.1.12.70057&platform=mac)