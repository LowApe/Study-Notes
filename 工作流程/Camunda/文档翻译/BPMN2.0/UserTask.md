# User Task

User Task 通常用于需要人操作的工作模型。当流程执行到达这样的用户任务时，将在分配给该任务的用户或组的任务列表中创建一个新的任务。

User Task 定义在 XML 如下。id 属性时必须的，当name 属性是可选的

```xml
<userTask id="theTask" name="Important task"/>
```

## 描述

User Task  可以有一个描述，实际上，任何 BPMN 2.0 元素都有一个描述。一个描述定义被加到文档元素中

```xml
<userTask id="theTask" name="Schedule meeting">
	<documention>
    	描述内容
    </documention>
</userTask>
```

描述文本可以从task对象获取，标准 Java 方式：

```java
task.getDescription();
```

## 属性

**Due Date 到期日**

每个任务有一个字段指向到期日。Query API 可以通过到期日查询，在这之前或者之后确切的日期。有一个活动扩展，它使您可以在任务定义中指定一个**表达式**，以设置任务创建时初始到期日期。**这个表达式应始终解析为 `java.util.Date,java.util.String(ISO 8601 formatted)`**或者 null.当使用`ISO8601`格式Strings，你也可以具体设置一个精确的时间点或者一段时间在任务创建时。例如，当你进入一个流程的前置表单或者计算一个之前服务任务。你可以使用这个时间

```xml
<userTask id="theTask" name="Important task" camunda:dueDate="${dateVariable}" />
```

也可以使用 TaskService 或在 TaskListeners 中使用传递的 Delegate Task 更改任务的截止日期

**Follow up Date 日期**

每个任务有一个指向跟进日期的字段。使用 Query API 可以查询跟进之前或者之后具体日期的任务。这有一个活动扩展，允许在创建任务时在你的任务定义中设置一个关于 follow up 日期

```xml
<userTask id="theTask" name="Important task" camunda:followUpDate="${dateVariable}" />
```

## 用户分配

一个用户任务可以直接分配**给单个用户，用户列表或组列表**

