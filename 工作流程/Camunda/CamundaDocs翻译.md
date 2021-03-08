# Camunda Docs翻译

> 找了一圈没有找到中文版，决定自己看看英文文档，学习并翻译一下，供以后学习和参考使用,前期因为要快速使用和了解，概述方面的翻译会简略一下，后期有时间再完善

# 介绍

Camunda 是基于支持BPMN 的Java框架的工作流程和过程自动化，CMMN 用例管理和DNB 商业决策管理。也可以访问：[实现标准](https://docs.camunda.org/manual/7.14/introduction/implemented-standards/)

该文档包含有关Camunda平台提供的功能的信息

为了使您对 Camunda有所了解，下图显示最重要组件所属典型的用户角色

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gocuks6c23j30lo0cdaax.jpg)



## 流程引擎&基础架构

- `Process Engine`该流程引擎是
- `Spring FrameworkIntegration`
- `CDI/Java EE Integration`
- `Runtime Container Integration`

## 设计器

- `Camunda Modeler`:
- `bpmn.io`

## 网络应用

- `REST API`
- `Camunda TaskList`
- `Camunda Cockpit`
- `Camunda Admin`



## 架构概述





![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gocuugzrhcj30d50a8gly.jpg)

- 流程引擎公共Api
- BPMN 2.0 核心引擎
- 工作执行器
- 持久层

## 受支持的环境

### 容器

### 数据库

### 游览器

### Java

### Java运行时

### Camunda 设计器





## 第三方库

### 流程引擎

下面是流程引擎依赖的第三方库：

- MyBatis
- Joda Time
- Java Uuid Generator(JUG)
- SLF4J

另外，流程引擎可以集成的：

- Apache Commons Email
- 

## Telemetry 感知测试？



## Public API

Camunda 平台提供公共的 API. 

### 定义的公共API



# 用户手册

> 该文档针对使用 Camunda 流程引擎在它们应用的开发者

- 流程引擎
- 流程因果那个
- 运行时容器集成
- Camunda 平台运行
- Spring Framework 集成
- Spring Boot 集成
- CDI 和 Java 集成
- 测试
- Model API
- 数据格式(XML，JSON，OTher)
- 用户任务表单
- DMN 引擎
- 日志
- 安全集成
- Camunda 认证Key
- 额外的任务客户端
- Camunda BPM RPA Bridge



## 🌟流程引擎

### Process Engine Bootstrapping

> 流程引擎启动配置

### Process Engine API

#### Servicces API

Java API 是与引擎交互的最常见方式，中心起点是 PorcessEngine，可以按照配置部分中所述的几种方式创建它.从 ProcessEngine ，可以包含几种服务工作流/BPM 方法。ProcessEngine 和这些服务对象都是线程安全的

![image.png](http://ww1.sinaimg.cn/large/006rAlqhgy1gocvqmdozyj30ss0fr75l.jpg)



```java
    public static void main(String[] args) {
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

#### Query API

