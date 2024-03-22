# go-ddd
<div>

![Go version](https://img.shields.io/badge/go-%3E%3Dv1.18-9cf)
![Release](https://img.shields.io/badge/release-1.0.0-green.svg)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
</div>

<p style="font-size: 15px">
  这是一个go的DDD（领域驱动设计）工程实践项目，项目展示了如何按照DDD的思想进行Go项目结构分层设计。
</p>

## MVC经典架构

对于传统面向过程的贫血模型而言，MVC 架构更加注重业务逻辑的封装和处理。

### MVC目录结构

如果你使用 Go Modules 来管理依赖，通常最理想的目录结构如下所示：
```
- cmd/
  - main.go
- internal/
  - app/
    - handlers/
    - middleware/
    - models/
    - routers/
    - services/
  - pkg/
    - utils/
- Dockerfile
- docker-compose.yaml
- go.mod
- README.md
```

* cmd/：包含应用程序的入口点，例如main.go文件。
* internal/：包含应用程序的内部代码，这些代码不会公开使用。该目录通常按照模块划分子目录，例如app/、pkg/等。
* internal/app/：包含应用程序的主要代码，这里应该包含所有的路由、处理函数、中间件、服务和模型等。
* internal/pkg/：包含应用程序的支持功能。例如，与日期和时间相关的工具和库等。
* Dockerfile：定义应用程序的容器化配置文件。
* docker-compose.yaml：定义多容器应用程序的容器编排配置文件。
* go.mod：定义应用程序的依赖项和版本。
* README.md：提供关于应用程序的信息和说明。

这样的目录结构非常清晰易懂，易于维护和扩展。

### 贫血模型与充血模型

目前，许多Web应用都采用前后端分离的方式，后端负责提供接口供前端调用。在这种情况下，我们通常将后端项目分为三层：M（Model）、V（View）和C（Controller），
目录命名可能不同，分层的思路大体都是相似的。

MVC 是基于 “贫血” 模型（Anemic Domain Model）进行开发的，将数据和业务逻辑分开到两个类中，这种基于贫血模型的设计思路本质上破坏了面向对象编程的封装特性，属于典型的*面向过程编程风格*。
而 DDD 是基于 “充血” 模型（Rich Domain Model） 进行开发的。数据和对应的业务逻辑被封装到同一个类中，属于典型的*面向对象编程风格*。

基于 “贫血” 模型的传统的开发模式，重Service层，轻BO；基于 “充血” 模型的DDD开发模式，轻Service层，重Demain层。

## 从MVC到DDD

单体架构大多是三层架构。三层架构解决了程序内代码间调用复杂、代码职责不清的问题，但这种分层是逻辑概念，在物理上它是中心化的集中式架构，并不适合分布式微服务架构。

DDD分层要类似三层架构，只是在DDD中，这些要素被重新划分了层，确定了层与层之间的交互规则和职责边界。

![i1](https://file.catnote.cn/com/05b765c9b2833f958c9ee6a4ea9e2e4a.png)

1. DDD分层架构相比MVC（只有API）在用户接口层新增了DTO，给前端提供了更多的可使用数据和更高的展示灵活性。

2. MVC架构向DDD分层架构演进，主要发生在业务逻辑层和数据访问层。

DDD分层架构将业务逻辑层的服务拆分到了应用层和领域层：
* 应用层快速响应前端的变化
* 领域层实现领域模型的能力

数据访问层和基础层之间：
* 三层架构数据访问采用DAO方式
* DDD分层架构的数据库等基础资源访问，采用了仓储（Repository）设计模式，通过依赖倒置实现各层对基础资源的解耦。

仓储又分为两部分：仓储接口和仓储实现。仓储接口放在领域层中，仓储实现放在基础层。原来三层架构通用的第三方工具包、驱动、Common、Utility、Config等通用的公共的资源类统一放到了基础层。

## DDD分层架构

### 分层架构

* 分层架构基本原则
  每层只与位于其下方的层发生耦合。
* 分层架构的分类
  1. 严格分层架构(Strict Layers Architecture)
  某层只能与其直接下层耦合，即我的奴隶的奴隶，不是我的奴隶。
  2. 松散分层架构(Relaxed Layers Architecture)
  允许任意上层与任意下层耦合。由于用户接口层和应用服务通常需要与基础设施打交道，许多系统都是该架构。
* 传统四层架构
  用户接口层->应用层->领域层->基础设施层，基础设施层位于底层，较高层与该层发生耦合以复用技术基础设施。
* 改良版四层架构
  使用依赖反转设计原则：低层服务（如基础设施层）应依赖高层组件（比如用户界面层、应用层和领域层）所提供的接口。
  依赖反转原则后的分层方式：基础设施层在最上方，可实现所有其他层中定义的接口

### DDD分层

![dddl](https://catnote-prod.oss-cn-beijing.aliyuncs.com/com/WX20240322-112250.png)

#### 一、Interfaces（用户接口层）

一般包括用户接口、Web 服务等。 只处理用户显示和用户请求，不应包含领域或业务逻辑。由于主要负责接入各种终端，所以该层有人也叫接入层。

**特点**
* 关心视图和对外的服务，Restful、页面渲染、websocket、XMPP 连接等
* 如果没有多种用户端接入方式，可以和应用层合并
* 对应到分布式系统中的网关、BFF、前台等概念
* 只产生接入异常，例如数据校验，对应 HTTP 状态码 400、415 等
* 一个应用可以有多个接入层
* 接入层做和业务规则无关的 bean validation 验证
* 准单体系统下，按照连接方式分包

**细分结构**：
* controller/facade：提供较粗粒度的调用接口，将用户请求委派给一个或多个应用服务进行处理。比如调用应用层创建用户的方法。
* dto：数据传输的载体，内部不存在任何业务逻辑，可以通过DTO把内部的领域对象与外界隔离。
比如接收请求传入的数据CustomerDTO：不同的对象在不同的层转换。用户接口层DTO和DO转换，应用层主要是DO，调外部微服务的服务的时候应用层有dto和do的转换。领域层与基础层之间，在基础层有DO和PO的转换。
在接口层定义DTO对象。数据可能来源于多个DO对象。
* assembler：实现DTO与DO间的相互转换和数据交换。
一般assembler与dto一同出现。比如创建用户时，将CustomerDTO转换为CustomerEntity。你可以在用户接口层创建DTO类和assembler类。在assembler类里完成映射。

#### 二、Application（应用层）

**负责**
* 服务的组合、编排、转发、转换和传递，处理业务用例的执行顺序以及结果的拼装，以粗粒度服务通过API网关发布到前端 
* 发送或订阅领域事件

开发设计时，不要将本该放在领域层的业务逻辑放到应用层。庞大的应用层会使领域模型失焦，时间一长，微服务就会退化为MVC。

**特点**
* 关心处理完一个完整的业务
* 该层只负责业务编排，对象转换，实际业务逻辑由领域层完成
* 不关心【请求从何处来】，但是关心【谁来、做什么、有没有权限做】 即复制安全认证、权限校验
* 集成不同的领域服务解决问题 应用层位于领域层之上，因为领域层包含多个聚合，所以它可协调多个聚合服务和领域对象完成服务编排和组合，协作完成业务。
* 最终一致性（最终一致性对业务有侵入）事务放到这层
* 对应到分布式系统中的中台等概念
* 方法级别的功能权限控制放到这层
* 只产应用异常，对应 HTTP 状态码 403、401
* 准单体系统下，按照应用划分模块

**细分结构**
* Service（应用服务）
  应用服务会对多个领域服务或外部应用服务进行封装、编排和组合，对外提供粗粒度的服务。应用服务主要实现服务组合和编排，是一段独立的业务逻辑。
* Event（事件）
  主要存放事件相关代码。包括两子目录：
  publish，主要存放事件发布相关代码。比如发布用户创建事件给其它微服务。 
  subscribe ，主要存放事件订阅相关代码（事件处理相关的核心业务逻辑在领域层实现）。
  虽然应用层和领域层都可进行事件的发布和处理，但为实现事件的统一管理，推荐将微服务内所有事件的发布和订阅处理都统一放到应用层，事件相关的核心业务逻辑实现放在领域层。通过应用层调用领域层服务，来实现完整的事件发布和订阅处理流程。

#### 三、Domain（领域层）

存放领域层核心业务逻辑相关的代码。 可包含多个聚合代码包，共同实现领域模型的核心业务逻辑。 一个聚合对应一个聚合代码目录，聚合之间在代码上完全隔离，聚合之间通过应用层协调。

**Aggregate（聚合）**
聚合内的代码模型是标准和统一的，包括：entity、event、repository、service 子目录。 

聚合软件包的根目录，可根据实际项目的聚合名称命名，比如权限聚合。 **聚合内实现高内聚的业务逻辑，它的代码可以独立拆分为微服务**。

**细分结构**
* Entity（实体）：存放聚合根、实体、值对象以及工厂模式（Factory，工厂模式主要是实现复杂聚合的实体的数据初始化。如果实体太多，聚合根处理起来会很复杂，通过工厂一次初始化）相关代码。
  实体类采用充血模型，同一实体相关的业务逻辑都在实体类代码中实现。跨实体的业务逻辑代码在领域服务中实现。
* Event（事件）：存放事件实体以及与事件活动相关的业务逻辑代码。
* Service（领域服务）：放领域服务代码。一个领域服务是多个实体组合出来的一段业务逻辑。领域服务封装多个实体或方法后向上层提供应用服务调用。
* Repository（仓储）：存放所在聚合的查询或持久化领域对象的代码，通常包括仓储接口和仓储实现方法。为了方便聚合的拆分和组合，设定原则：一个聚合对应一个仓储。比如将用户信息保存到数据库。
（按DDD分层架构，仓储实现本应属基础层代码，但为在微服务架构演进时，保证代码拆分和重组的便利性，把聚合仓储实现的代码放到聚合包内。这样，如果需求或设计发生变化导致聚合需要拆分或重组，就可将包括核心业务逻辑和仓储代码的聚合包整体迁移，轻松实现微服务架构演进。）

#### 四、Infrastructure（基础层）

基础层包含基础服务，它采用依赖反转，封装基础资源服务，实现应用层、领域层与基础层解耦。

为其它各层提供通用技术基础服务： 三方工具、驱动 、MQ 、API网关 、文件 、缓存 、DB、配置等

### DDD目录结构

```
server
├── interfaces              # 接口层
│   ├── controller/facade   # 控制器路由函数/提供较粗粒度的调用接口
│   ├── dto/vo              # 可包含多个领域对象的属性：DTO（Data Transfer Object）主要关注数据的传输，通常用于面向服务或接口设计，用于在系统的不同部分之间传递数据。VO（View Object）则更常用于用户界面 UI 层，用于呈现数据给用户
│   └── assembler           # 实现 DTO 数据传输对象与 Domain Entity 之间的相互转换和数据交换，从表示层（Presentation Layer）向领域层（Domain Layer）进行数据传递
├── application             # 业务调度层
│   ├── event               # 微服务事件推送或订阅
│   │   ├── publish         # 事件发布
│   │   └── subscribe       # 事件订阅
│   └── service             # 用于连接 Controller 和 Domain，进行三方接口调用等其他操作
├── domain                  # 领域服务层（领域逻辑和领域对象，主要的业务逻辑，采用充血模型）
│   ├── aggregate01         # Aggregate 聚合根目录
│   │   ├── entity          # entity 实体、VO 值对象以及工厂模式（Factory）相关
│   │   ├── event           # 事件实体以及与事件活动相关的业务逻辑代码
│   │   ├── service         # 领域服务代码，一个领域服务是多个实体组合出来的一段业务逻辑
│   │   └── repository      # 持久化领域对象，通常仅包括仓储接口；在微服务架构演进时保证代码拆分和重组的便利性，也可把聚合仓储实现的代码放到聚合包内
│   ├── aggregate02
│   └── ...
├── infrastructure          # 基础设施层
│   ├── api                 # 第三方 API/SDK
│   ├── configs             # 配置参数变量
│   ├── database            # 初始化数据库
│   ├── mq                  # 消息队列连接和配置
│   ├── persistence         # 数据持久化（Domain 层 repository 的具体实现，数据库 CRUD 操作） 
│   └── pkg                 # 工具函数
│       ├── common          # 与业务相关包
│       └── utils           # 公共基础包
├── README.md               # 项目文档
├── go.mod                  # 依赖文件
└── main.go                 # 主入口
```

### 涉及基础概念

|名词|英文|说明|
|-------|-------|-------|
|实体|Entity|在业务领域中具有唯一标识的对象。它们有自己的生命周期和状态，并通过标识属性进行区分。实体通常具有行为和属性，并可以包含其他值对象或实体。|
|值对象|Value Object|没有标识的对象，仅通过属性值来定义。通常用于描述特定概念或概念组合，在整个系统中广泛重用。与实体不同，值对象通常是不可变的。|
|领域服务|Domain Service|处理复杂业务逻辑、跨实体操作或无法由单个实体或值对象表达的功能的机制。领域服务的方法可能涉及多个实体或值对象，并可在整个领域模型中使用。|
|聚合及聚合根|Aggregate, Aggregate Root|相关实体和值对象的集合，以聚合根为核心。聚合根是聚合内部的一个实体，负责保护聚合的一致性和完整性。通过限制对聚合根之外实体的直接访问，维护聚合的内部一致性。|
|工厂|Factory|创建复杂对象或聚合的机制，封装了对象的创建过程。工厂可以处理对象的初始化逻辑、依赖注入和复杂的构建步骤，并返回创建的对象。降低对象创建的复杂性并提供可测试性。|
|仓储|Repository|持久化和检索领域对象的机制。封装了数据访问层的实现细节，为领域层提供简单的接口操作持久化状态的对象。负责管理对象的加载、保存和查询，并确保与持久化存储之间的交互。|
|领域事件|Domain Event|描述业务领域中重要事件的对象。用于在不同领域对象之间传递信息并触发相关行为。领域事件被广播给感兴趣的订阅者，进行后续处理或响应。|

### 何时使用DDD

1. 在大部分情况下，我们开发的系统的业务都比较简单，只涉及基于SQL的CRUD操作，因此我们不需要过于复杂的 “充血” 模型设计。对于这种简单的业务开发， “贫血” 模型已经足够应付。考虑到业务的简单性，我们选择了 “贫血” 模型进行设计，一个原因是领域模型相对较简单，另一个原因是考虑到ROI（投入产出比）的因素。
2. “充血” 模型的设计难度比 “贫血” 模型更大，因为 “充血” 模型更加贴近面向对象编程的风格。在开始设计时，我们需要考虑哪些操作需要对数据进行暴露，以及定义哪些业务逻辑。而不像 “贫血” 模型那样，最初只需定义数据，之后根据功能需求在Service层进行相关操作，无需事先进行过多的设计。
3. 这种基于 “贫血” 模型的传统开发模式已经存在多年，深入人心，是大多数开发人员习以为常的。如果在现有开发模式下没有遇到太大的问题，转向使用 “充血” 模型或领域驱动设计势必会增加学习成本和转型成本。在没有遇到紧迫的开发需求的情况下，许多开发人员可能不愿意进行这样的转变。

### DDD与MVC的优劣

#### DDD 的优势

* 高度关注业务领域：DDD 通过将重点放在核心业务领域上，提供了一种更好地理解和建模复杂领域的方法。它强调领域驱动的设计，使开发人员能够更好地与业务专家进行沟通，并更好地反映真实世界的业务逻辑。
* 模块化和可重用性：DDD 鼓励将代码组织成聚合和领域服务，以及其他相关概念。这种模块化和组件化的设计可以提高代码的可重用性，减少耦合，并使系统更加灵活和可扩展。
* 解决复杂性和变化：DDD 通过将复杂业务问题分解为可管理的领域对象和领域服务来应对系统的复杂性。它强调通过限界上下文和聚合根来处理业务规则和一致性，并尽可能地减少对外部依赖的影响。这使得系统更容易理解、演化和适应变化。

#### DDD 的劣势
DDD 只是一套先进的方法论，虽然已经诞生多年并随着微服务的兴起而变得热门，但它其实也有一些缺点需要注意：

* 性能方面：DDD 基于聚合来组织代码，对于高性能场景下，加载聚合中大量无用的字段可能会严重影响性能。
* 事务方面：DDD 中的事务被限定在限界上下文内，处理跨多个限界上下文的情况时，开发人员需要额外考虑分布式事务问题。
* 实现难度和推广成本：DDD 项目需要领域专家的参与，同时也要求开发者对业务、建模和面向对象编程有深入了解。

#### MVC 的优势

* 简单易上手：传统的 MVC 架构具有简单清晰的组织结构，易于理解和学习。它将应用程序分为模型、视图和控制器，提供了一种直观的方式来组织代码并处理用户交互。
* 快速开发和迭代：MVC 架构适用于快速开发和迭代的场景。它将前端展示和后端逻辑分离，使开发团队能够并行工作，并更容易实现界面的变化和改进。
* 广泛的支持和成熟的生态系统：MVC 架构已经存在多年，拥有广泛的支持和成熟的生态系统。有许多成熟的框架和工具可供选择，可以加快开发过程，并提供丰富的功能和插件。

简单来说，对于大多数项目而言，MVC 架构已经足够应对需求。然而，对于长期、持续投入的复杂业务系统，DDD 架构会更适合。在选择架构时，应根据具体项目需求、团队能力（学习成本/开发时间）和预期系统复杂性进行评估。

## 项目代码实践

本项目涉及：

### Go 工程化项目实践：

* DDD 项目分层设计
* GORM RBAC 权限控制表设计
* AuthMiddleware 与 JWT 的实践方案
* OPTION 函数式编程在业务代码中的场景
* KMS 密码管理的功能开发指导建议
* Swagger 接口文档
* Makefile 项目构建
* Wire 依赖注入

### 容器 CI/CD 服务部署：

* 包含 mysql-db、go-app、nginx-web
* 使用 Docker-compose 进行容器部署
* Container 容器之间网络端口相互访问
* Nginx 反向代理配置
* SQL 数据库脚本导入
* Dockerfile Go 应用程序镜像构建








