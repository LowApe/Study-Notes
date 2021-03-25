# 流程引擎API

[官方原文地址](https://docs.camunda.org/manual/latest/user-guide/process-engine/process-engine-api/)

# 服务API

JavaAPI是最通用的方式交互引擎。中心起点是 Porcess Engine，可以按照配置部分中所述的几种方式来创建它。从 ProcessEngine，你可以获得几种 Services 包含流/BPM的方法。ProcessEngine 和这些 Services 对象是线程安全的。所以可以始终保持引用它们

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gow6s5h2vij30ss0fr75l.jpg)

具体代码：

```java
public static void main(String[] args){
        ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine();
        RepositoryService repositoryService = processEngine.getRepositoryService();
        RuntimeService runtimeService = processEngine.getRuntimeService();
        TaskService taskService = processEngine.getTaskService();
        IdentityService identityService = processEngine.getIdentityService();
        FormService formService = processEngine.getFormService();
        HistoryService historyService = processEngine.getHistoryService();
        ManagementService managementService = processEngine.getManagementService();
        FilterService filterService = processEngine.getFilterService();
        ExternalTaskService externalTaskService = processEngine.getExternalTaskService();
        CaseService caseService = processEngine.getCaseService();
        DecisionService decisionService = processEngine.getDecisionService();
    }
```

`ProcessEngines.getDefaultProcessEngine()`将会第一时间初始化和构建流程引擎并且之后总是返回同样的 ProcessEngine 对象。可以使用`ProcessEngines.init()`, `ProcessEngines.destroy()`关闭和创建所有的引擎

这个 ProcessEngines calss 将会扫描所有 `camunda.cfg.xml`和`activit.cfg.xml`文件。通过`camunda.cfg.xml`文件，创建 ProceeEngine

```java
ProcessEngineConfiguration
    .createProcessEngineConfigurationFromInputStream(inputStream)
    .buildProcessEngine()
```

对于所有`activiti.cfg.xml`文件，这个 process engine 将被创建通过 Spring 方式：启动 Spring application 上下文是创建和然后 process engine 从上下文中获取

**所有services是无状态的**。这意味着你可以轻松的运行 Camunda 平台在集群的多个节点上，每个节点都去同一个数据库，不必担心哪台计算机实际执行了先前的调用

## RepositoryService

**RepositoryService 可能是使用 Camunda 引擎需要的第一个服务**。这个服务提供操作管理和 **操作部署和流程定义**。这里没有太多详情，process definition 是 BPMN 2.0 流程的 Java 对应项。它是每一个方法的步骤的结构和行为的表示。**部署是引擎内包装的单位**。一个部署可以包含多样的 `BPMN 2.0 XML`文件和其他资源。**一个部署中包含哪些内容的选择取决于开发人员**。它的范围可以从单个流程 `BPMN 2.0 XML`文件到整个流程包和相关资源(例如，部署"hr-processes" 可能包含于 HR 流程相关的所有内容).这个 RepositoryService 运行部署类似的包。**部署意味着上传到引擎，所有流程被检查和解析在存储数据库之前**。从这之后，系统便知道该部署并且现在可以**启动**该部署中的**任意过程**

此外，RepositoryService 允许：

- 查询引擎已知的部署和流程定义
- 挂机和激活流程定义。挂起意味着无法操作它们，而激活是相反的操作
- 检索各种资源如包含部署或者流程图文件通过引擎自动生成

## RuntimeService

如果说 RepostoryService 是关于静态的信息，那 **RuntimeService** 恰恰相反。它处理启动流程定义的新流程实例。正如刚才提到的，一个流程定义，定义了结构和行为在不同流程步骤中。对于每个流程定义，通常多个类型的实例同时运行。这个 RuntimeService 还是用于检测和存储过程**变量**的服务。这个数据具体给流程实例的，并且可由流程中的各种构造使用 (专用网关经常使用流程变量去决定选择哪个路径以继续该过程)。RuntimeService 也允许查询**流程实例**和**执行记录**。执行是 BPMN 2.0 的“token” 概念的表示🤔，**基础上，执行是指向流程实例当前位置的指针**。最后，RuntimeService 被用来每当流程实例等待外部触发并且流程需要继续。一个流程实例可以多种等待状态并且这个Serivce 包含多种操作去标识这个状态在外部触发后接收并且流程实例继续执行

## TaskService

任务是通过实际用户操作流程引擎核心。**所有围绕任务的归类到 TaskService，**类似

- 查询分配到用户或者组的任务
- 创建新的独立的任务.这些是流程实例无关的任务
- 操作任务分配给哪个用户或用户使用一些方式涉及到任务
- 领取并且完成任务。认领意味着一些人决定分配这个任务，意味着分配用户将会完成这个任务。完成意味着"doing the work of the tasks". 通常这是一种形式的填充



## IdentityService

IdentityService 非常简单。它总是管理组和用户(创建，更新，删除，查询)。重要的是了解，核心引擎实际上在运行时不对用户进行任何检查。例如，一个任务可能分配任何用户，**但是引擎不会验证系统是否知道用户**。这是因为引擎可以被LDAP,Active Directory 等服务结合使用。

## FormService

FormService 是可选服务。这意味着 Camunda 引擎在没有它的情况下完美使用，而不会牺牲任何功能。这个 service 介绍开启表单和表单任务的概念。**开启表单是在开启流程实例前给用户看，任务表单是当用户想要完成任务时展示**。这个Service暴露数据使用很简单的方式去工作。但是，这又是可选的，因为表单不需要嵌入流程定义中



## HistoryService

HistoryService 暴露所有历史数据收集通过引擎。当执行流程时，引擎可以保存很多数据(这是可配置的)，例如流程实例的启动时间，谁执行那些任务，完成任务所花的时间，每个流程实例遵循的路径等。这个服务暴露主要查询能力访问这些数据。

## ManagementService

ManagementService 通常不需要用在编写自定义应用。**它允许检索信息关于数据表和表元数据。此外，它暴露查询能力和管理操作jobs**。引擎中的作业用于各种事情，例如计时器，异步延迟，延迟暂停/激活。之后，这些主题将浸洗更详细的讨论

## FilterService

FilterService 允许去创建和管理过滤器。过滤器是存储的查询，例如任务查询。例如过滤器由任务清单使用去过滤用户任务

## ExternalTaskService

ExternalTaskService 提供访问外部任务实例。外部任务表示在外部独立于流程引擎处理的工作项

## CaseService

CaseService 类似 RuntimeService 但不适用 case 实例。它处理启动 case 定义(案例定义)的新案例实例并管理案例执行的生命周期。这个 service 也习惯检索和更新**案例实例**的流程变量

## DecisionService

DecisionService 允许评估部署到引擎的角色。业务规则的任务重评估独立于流程定义的决策是一种替代方法



# 查询 API

从引擎查询数据，有多种可能：

- Java Query ApI:
- REST Query API:
- Native Queries:
- Custom Queries:
- SQL Queries:



# 查询最大结果数限制

查询结果没有限制最大数量，或者查询大量结果可能造成大量内存消耗，甚至导致内存不足异常。随着`Query Maximum Results Limit` 帮助下，你可以限制最大查询结果。

仅在一下情况下强制执行：

- 经过身份验证的用户执行查询
- 这个查询 API 是直接调用 REST API

## 禁止的

- 使用`list()`方法操作一个无限制数量结果查询
- 执行分页查询超过已配置的最大结果限制
- 执行一个基本查询同步操作，该操作影响的实例超过最大结果数的限制(请改用批处理操作)

## 允许的

- 执行查询使用`unlimitedList`方法
- 执行分页查询低于最大数量结果或等于最大结果限制
- 执行一个原生查询因为无法使用 REST API 或者 Webapps ，因此不太可能被利用

## 局限性

- 通过 REST API 执行统计信息查询
- 通过 Webapps 调用实例查询

## 用户身份服务查询



# 分页查询

```java
List<Task> tasks = taskService.createTaskQuery()
    .taskAssignee("kermit")
    .processVariablevalueEuals("orderId","0815")
    .orderByDueDate().asc()
    .listPage(20,50)
```

# Or 查询



# REST Query API



# Native Queries

有时你需要更强大的查询，查询使用过 or 操作或者使用Query Api 限制你不能表达的查询。对于这些情况，我们介绍允许我们编写SQL查询的原生查询。返回结果定义通过 Query 对象你使用并且映射正确的对象，Task，ProcessInstance,Execution,等。**由于查询将在数据库中触发，你必须使用在数据库定义的表和字段名称**。这需要一些知识关于关于内部数据结构并且建议小心使用原始查询。可以通过 API 检索表名，以使依赖性尽可能小。

```java
List<Task> tasks = taskService.createNativeTaskQuery()
    .sql("SELECT count(*) FROM " + managementService.getTableName(Task.class) + " T WHERE T.NAME_ = #{taskName}") 
    .parameter("taskName","aOpenTask")
    .list();
long count = taskService.createNativeTaskQuery()
    .sql("SELECT count(*) FROM " + magementService.getTableName(Task.class)) + "T1, "
    	+ managementService.getTableName(VariableInstanceEntity.class) + "V1 WHERE V1.TASK_ID_ = T1.ID_")
    .count();
```

# 自定义查询

出于性能原因，有时可能不希望查询引擎对象，而是查询自己的值或者 DTO 对象，这些对象从不同表中收集数据-可能包括自己的域类

- [官方链接](https://camunda.com/blog/2017/12/custom-queries/)

# SQL 查询

?