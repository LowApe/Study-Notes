# 目录
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [目录](#目录)
- [在 Spring 中配置 Web Flow](#在-spring-中配置-web-flow)
- [流程的组件](#流程的组件)
	- [状态](#状态)
		- [视图状态](#视图状态)
		- [行为状态](#行为状态)
		- [决策状态](#决策状态)
		- [子流程状态](#子流程状态)
		- [结束状态](#结束状态)
	- [转移](#转移)
		- [全局转移](#全局转移)
	- [流程数据](#流程数据)
		- [声明变量](#声明变量)
		- [定义流程数据的作用域](#定义流程数据的作用域)
- [组合起来:披萨流程](#组合起来披萨流程)
	- [定义基本流程](#定义基本流程)
	- [收集顾客信息](#收集顾客信息)
		- [询问电话号码](#询问电话号码)
		- [查找顾客](#查找顾客)
		- [注册新顾客](#注册新顾客)
		- [检查配送区域](#检查配送区域)
		- [存储顾客数据](#存储顾客数据)
		- [结束流程](#结束流程)
	- [构建订单](#构建订单)
	- [支付](#支付)
- [保护 Web 流程](#保护-web-流程)
- [小结](#小结)

<!-- /TOC -->
Spring Web Flow 是一个 Web 框架,它适用于元素按规定流程运行的程序.在本章中,我们将会探索 Spring Web Flow 并了解它如何应用于 Spring Web 框架平台。

Spring Web Flow 是 Spring MVC 的扩展,它支持开发基于流程的应用程序.它将流程的定义与流程行为的类和视图分离开。

下面使用 Spring Web Flow 来定义订单流程

# 在 Spring 中配置 Web Flow

> flag:第四版的说还不支持在 Java 中配置 Spring Web Flow,所有我们使用 xml 进行配置

1. xml声明 webflow 的命名空间
2. 装配流程执行器
3. 配置流程注册表
4. 处理流程请求


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:flow="http://www.springframework.org/schema/webflow-config"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/webflow-config http://www.springframework.org/schema/webflow-config/spring-webflow-config.xsd">
    <!--流程执行器-->
    <flow:flow-executor  id="flowExecutor" />
    <!--流程注册表-->
    <flow:flow-registry id="flowRegistry" base-path="/WEB-INF/flows">
        <flow:flow-location-pattern  value="*-flow.xml" />
    </flow:flow-registry>

    <!--处理流程请求-->
    <bean class="org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
        <property name="flowRegistry" ref="flowRegistry"/>
    </bean>
    <!--处理适配器-->
    <bean class="org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
        <property name="flowExecutor" ref="flowExecutor"/>
    </bean>
</beans>
```

# 流程的组件

1. 状态
2. 转移
3. 流程数据


## 状态

Spring Web Flow 可供选择的状态

|状态类型|描述|
|:--:|:--:|
|行为 Action|行为状态是流程逻辑发生的地方|
|决策 Decision |决策状态将流程分为两个方向,它会基于流程数据的评估结果确定流程方向|
|结束 End|结束状态是流程的最后一站,一旦进入 End 状态,流程就会终止|
|子流程 Subflow|子流程会在当前正在运行的流程上下文启动一个新的流程|
|视图 View|视图状态会暂停流程并邀请用户参与流程|

### 视图状态
视图状态用于为用户展现信息并使用户在流程中发挥作用。

```xml
<view-state id="welcome"/>
```
简单示例,id 属性有两个含义.
- 在流程内标示这个状态.
- 流程到达这个状态要展现的逻辑视图名为 welcome

```xml
<view-state id="welcome" view="greeting"/>
```
显示指定视图名可以使用 view 属性

如果流程为用户展现一个表单,你可能希望**指明表单所绑定的对象**。可以设置 model 属性:

```xml
<view-state id="takePayment" model="flowScope.paymentDetails"/>
```
这里我们指定 takePayment 视图中的表单将绑定**流程作用域**内的 paymentDetails 对象。

### 行为状态
行为状态则是应用程序自身在执行任务。行为状态一般会触发 Spring 所管理的 bean 的一些方法并根据方法调用的执行结果转移到另一个状态

```xml
<action-state id="saveOrder">
    <evaluate expression="pizzaFlowActions.savaOrder(order)"/>
    <transition to="thankYou"/>
</action-state>
```
`<evaluate>`元素给出了行为状态要做的事情。expression 属性指定了进入这个状态时要评估的表达式。给出的是 SpEL 表达式,它表明将会找到 ID 为 pizzaFlowActions 的 bean 并调用其 saveOrder() 方法。

### 决策状态

更常见的情况是流程在某一点根据流程的当前情况进入不同的分支。

```xml
<decision-state id="checkDeliveryArea">
       <if test="pizzaFlowActions.checkDeliveryArea(customer.zipCode)"
           then="addCustomer"
           else="deliveryWarning"/>
</decision-state>
```
`<if>`元素是决策状态的核心。如果表达式结果 true,流程将转移到 then 属性指定的状态中,如果结果为 false,流程将会转移到 else 属性指定的状态中。


### 子流程状态

`<subflow-state>`允许在一个正在执行的流程中调用另一个流程。这类似于在一个方法调用另一个方法

```xml
<subflow-state id="order" subflow="pizza/order">
        <input name="order" value="order"/>
        <transition on="orderCreated" on-exception="com.springinaction" to="payment"/>
</subflow-state>
```
在这里,`<input>`元素用于传递订单对象作为子流程的输入。如果子流程结束的 `<end-state>`状态 orderCreated,那么流程将会转移到名为 payment 的状态

### 结束状态
所有流程都要结束。当流程转移到结束状态时所做的 `<end-state>`元素指定了流程的结束

```xml
<end-state id="customerReady"/>
```
当到达结束状态,流程会结束。取决于几个因素:
- 如果结束的是子流程,那调用它的流程将会从 `<subflow-state>` 处继续执行。`<end-state>`的 ID 将会用作事件触发从 `<subflow-state>` 开始的转移
- 如果 `<end-state>`设置了 view 属性,指定的视图将会被渲染。视图可以是相对于流程路径的视图模板,如果添加"externalRedirect:"前缀的话,将会重定向到流程外部的页面,如果添加 "flowRedirect:"将重定向另一个流程中。
- 如果结束的不是子流程也没有指定 view 属性,那这个流程只是会结束而已。


## 转移
转移连接了流程中的状态。流程中除结束状态之外的每个状态,至少都需要一个转移,在这样就能够知道一旦这个状态完成时流程要去向哪里。状态可以有多个转移,分别对应于当前状态结束时可以执行的不同的路径。

转移使用 `<transition>`元素来进行定义,它会作为各种状态元素 `<action-state>`、`<view-state>`、'<subflow-state>'的子元素。
```xml
<transition to="customerReady" />
```

属性 to 用于指定流程的下一个状态。如果只使用了 to 属性,那么这个转移就会是当前状态的默认转移选项,如果没有其他可用转移的话,就会使用它。

> 更常见的转移定义是基于事件的触发来进行的。在视图状态,事件通常会是用户采取的动作.在行动状态,事件是评估表达式得到的结果.而在子流程状态,事件取决于子流程结束状态的 ID。 **在任何事件中使用 on 属性来指定触发转移的事件**

```xml
<transition on="phoneEntered" to="lookupCustomer"/>
```

如果触发 phoneEntered 事件,流程将会进入 lookupCustomer 状态

在抛出异常时,流程也可以进入另一个状态

```xml
<transition on-exception="com.xxx.CustomerNotFoundException" to="registrationForm"/>
```
on-exception 类似于 on 属性,只不过它指定要发生转移的异常而不是一个事件。在本示例中,CustomerNotFoundException 异常将会导致流程转移到 registrationForm 状态

### 全局转移
与其在多个状态中都重复的转移,我们将`<transition>`元素作为 `<global-transitions>` 的子元素,把它们定义为全局转移.

```xml
<global-transition>
    <transition on="cancel" to="endState"/>
</global-transitions>
```
> 定义完这个全局转移后,流程中的所有状态都会默认拥有这个 cancel 转移

## 流程数据

当流程从一个状态进行到另外一个状态时,它会带走一些数据。有时候,这些数据只需要很短的时间(可能只要展现页面给用户)。有时候,这些数据会在整个流程中传递并在流程结束的时候使用

### 声明变量

- `<var>`元素
    ```xml
        <var name="customer" class="com.xx.Customer"/>
    ```
- `<evaluate>`元素
    ```xml
        <evaluate result="viewScope.toppingsList"
            expression="T{com.xxx.Topping}.asList()"/>
    ```
    expression 结果方法哦 toppingsList 变量中,这个变量是视图作用域的

- `<set>`元素
    ```xml
        <set name="flowScope.pizza"
            value="new com.xxx.Pizza()"/>
    ```
    这里设置一个流程作用域内的 pizza 变量,它的值是 Pizza 对象的新实例

### 定义流程数据的作用域
流程中携带的数据会拥有不同的生命作用域和可见性,这取决于保存数据的变量本身的作用域。 Spring Web Flow 定义了五种作用域

|范围|生命作用域和可见性|
|:--:|:--:|
|Conversation|最高层级的流程开始时创建,在最高层级的流程结束时销毁。被最高层级的流程和其所有的子流程所共享|
|Flow|当流程开始时创建,在流程结束时销毁。只有在创建它的流程中是可见的|
|Request|当一个请求进入流程时创建,在流程返回时销毁|
|Flash|当流程开始时创建,在流程结束时销毁。只在视图状态内是可见的|
|View|当进入视图状态时创建,当这个状态退出时销毁。只在视图状态内是可见的|

- `<var>` 元素声明变量时,变量始终是流程作用域的,也就是在定义变量的流程内有效
- `<set>`或`<evaluate>`作用域通过 name 或 result 属性的前缀指定。

# 组合起来:披萨流程
## 定义基本流程

订购披萨流程

![](http://ww1.sinaimg.cn/large/006rAlqhly1g30wlnlgyej30hi0aa75s.jpg)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/webflow
       http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd">
    <var name="order"
            class="top.blogcode.pizza.domain.Order"/>
<!--顾客子流程-->
   <subflow-state id="identifyCustomer" subflow="pizza/customer">
       <output name="customer" value="order.customer"/>
       <transition on="customerReady" to="buildOrder" />
   </subflow-state>
<!--调用订单子流程-->
   <subflow-state id="buildOrder" subflow="pizza/order">
       <input name="order" value="order"/>
       <transition on="orderCreated" to="takePayment" />
   </subflow-state>
<!--调用支付子流程-->
   <subflow-state id="takePayment" subflow="pizza/payment">
       <input name="order" value="order"/>
       <transition on="paymentTaken" to="saveOrder"/>
   </subflow-state>
<!--保存订单-->
   <action-state id="saveOrder">
       <evaluate expression="pizzaFlowActions.saveOrder(order)" />
       <transition to="thankCustomer" />
   </action-state>
<!--感谢顾客-->
   <view-state id="thankCustomer">
       <transition to="endState" />
   </view-state>
   <end-state id="endState" />
<!--全局取消转移-->
   <global-transitions>
       <transition on="cancel" to="endState" />
   </global-transitions>
</flow>
```

在流程定义中,我们看到的第一件事就是 order 变量的声明。每次流程开始的时候,都会创建一个 Order 实例。Order 类会带有关于订单的所有信息,包含顾客信息、订购的披萨列表以及支付详情

```java
package com.springinaction.pizza.domain;
import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;
public class Order implements Serializable {
   private static final long serialVersionUID = 1L;
   private Customer customer;
   private List<Pizza> pizzas;
   private Payment payment;
      public Order() {
      pizzas = new ArrayList<Pizza>();
      customer = new Customer();
   }
   public Customer getCustomer() {
      return customer;
   }
   public void setCustomer(Customer customer) {
      this.customer = customer;
   }
   public List<Pizza> getPizzas() {
      return pizzas;
   }
   public void setPizzas(List<Pizza> pizzas) {
      this.pizzas = pizzas;
   }
   public void addPizza(Pizza pizza) {
      pizzas.add(pizza);
   }
   public float getTotal() {
      return 0.0f;
   }
   public Payment getPayment() {
      return payment;
   }
   public void setPayment(Payment payment) {
      this.payment = payment;
   }
}
```

流程定义的主要组成部分时流程的状态。默认情况下,流程定义文件中的第一个状态也会时流程访问中的第一个状态。在本例中,也就是 identifyCustomer 状态(一个子流程),也可以定义 `<flow>` 元素的 start-state 属性将状态指定为开始状态

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow
  http://www.springframework.org/schema/webflow/spring-webflow-2.3.xsd"
  start-state="identifyCustomer">
...
</flow>
```
识别顾客、构造披萨订单以及支付这样的活动太复杂了，并不适合将其强行塞入一个状态。这是我们为何在后面将其**单独定义为流程**的原因。但是为了更好地整体了解披萨流程，这些活动都是以`<subflow-state>`元素来进行展现的。

流程变量 order 将在前三个状态中进行填充并在第四个状态中进行保存。
identifyCustomer 子流程状态使用 `<output>`元素来填充 order 的 customer 属性,将其设置为顾客子流程收到的输出。buildOrder 和 takePayment 状态使用了不同的方式,它们使用 `<input>`将 order 流程变量作为输入,这些子流程就能在其内部填充 order 对象。在订单得到顾客、一些披萨以及支付细节后,就可以进行保存。saveOrder 是处理这个任务的行为状态。它使用 `<evaluate>`来调用 ID 为 pizzaFlowActions 的 bean 的 saveOrder() 方法,并将保存的订单对象传递进来.订单完成保存后,它会转移到 thankCustomer

thankCustomer 状态是一个简单的视图状态,后台使用了 `/WEB-INF/flows/pizza/thankCustomer.jsp`

```html
<html xmlns:jsp="http://java.sun.com/JSP/Page">
  <jsp:output omit-xml-declaration="yes"/>
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
  <head><title>Spizza</title></head>
  <body>
    <h2>Thank you for your order!</h2>
    <![CDATA[
    <a href='${flowExecutionUrl}&_eventId=finished'>Finish</a>
    ]]>
    </body>
</html>
```
SpringWebFlow为视图的用户提供了一个flowExecutionUrl变量，它包含了流程的 URL.结束链接将一个`"_eventId"`参数关联到 URL 上，以便回到 Web 流程时触发finished事件。这个事件将会让流程到达结束状态。

流程将会在结束状态完成。鉴于在流程结束后没有下一步做什么的具体信息，流程将会重新从 identifyCustomer 状态开始，以准备接受另一个披萨订单。

## 收集顾客信息

下订单前需要顾客信息,最重要的是手机号码和地址。下面是开始订单前的流程图:

![](http://ww1.sinaimg.cn/large/006rAlqhly1g30y5fu8qsj30ig0dwgoa.jpg)

这个流程不是线性的而是在好几个地方根据不同的条件有了分支。
例如,在查找顾客后,流程可能结束(如果找到了顾客),也有可能转移到注册表单(如果没有找到顾客)。同样,在 checkDeliveryArea 状态,顾客有可能被警告也有可能不被警告它们的地址在配送范围之外

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      xsi:schemaLocation="http://www.springframework.org/schema/webflow http://www.springframework.org/schema/webflow/spring-webflow-2.0.xsd">

        <var name="customer"
             class="top.blogcode.pizza.domain.Customer"/>
    <!--欢迎顾客-->
        <view-state id="welcome">
            <transition on="phoneEntered" to="lookupCustomer"/>
        </view-state>
        <!-- 查找用户 -->
        <action-state
                 id="lookupCustomer">
            <evaluate result="customer" expression=
                    "pizzaFlowActions.lookupCustomer(requestParameters.phoneNumber)" />
            <transition to="registrationForm" on-exception=
                    "top.blogcode.pizza.service.CustomerNotFoundException" />
            <transition to="customerReady" />
        </action-state>
        <!-- 注册新顾客 -->
        <view-state id="registrationForm" model="customer">
            <on-entry>
                <evaluate expression=
                                  "customer.phoneNumber = requestParameters.phoneNumber" />
            </on-entry>
            <transition on="submit" to="checkDeliveryArea" />
        </view-state>
        <!-- 检查配送区域 -->
        <decision-state id="checkDeliveryArea">
            <if test="pizzaFlowActions.checkDeliveryArea(customer.zipCode)"
                then="addCustomer"
                else="deliveryWarning"/>
        </decision-state>
        <!-- 显示配送警告 -->
        <view-state id="deliveryWarning">
            <transition on="accept" to="addCustomer" />
        </view-state>
        <!-- 添加顾客 -->
        <action-state id="addCustomer">
            <evaluate expression="pizzaFlowActions.addCustomer(customer)" />
            <transition to="customerReady" />
        </action-state>

        <end-state id="cancel" />
        <end-state id="customerReady">
            <output name="customer" />
        </end-state>
        <global-transitions>
            <transition on="cancel" to="cancel" />
        </global-transitions>
</flow>
```

这个流程包含新技巧,包括我们首次使用 `<decision-state>`元素

### 询问电话号码
weclome 状态是一个很简单的视图状态,它欢迎访问 Spizza 站点的顾客并要求输入电话号码。它有两个转移:
- 如果从视图触发 phoneEntered 事件的话,转移会将流程定向到 lookupCustomer,
- 另外一个就是在全局转移中定义的用来响应 cancel 事件的 cancel 转移

```html
<html xmlns:jsp="http://java.sun.com/JSP/Page"
     xmlns:form="http://www.springframework.org/tags/form">
  <jsp:output omit-xml-declaration="yes"/>
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
  <head><title>Spizza</title></head>
  <body>
    <h2>Welcome to Spizza!!!</h2>
    <form:form>
      <input type="hidden" name="_flowExecutionKey"
             value="${flowExecutionKey}"/>
       <input type="text" name="phoneNumber"/><br/>
      <input type="submit" name="_eventId_phoneEntered"
             value="Lookup Customer" />
     </form:form>
  </body>
</html>

```
表单有两个特殊的部分来驱动流程继续。

隐藏的 `"_flowExecutionKey"`输入域。当进入视图状态时,流程暂停并等待用户采取一些行为。赋予视图的流程执行 key (flow executation key)就是一种返回流程的 "回程票" (claim ticket)。当用户提交表单时,流程执行 key 会在 `"_flowExecutionKey"` 输入域中返回并在流程暂停的位置进行恢复

> flag:这块不太懂

提交按钮名字的 `"_eventId_"` 部分是提供给 Spring Web Flow 的一个线索,它表明了接下来要触发事件。当点击这个按钮提交表单时,会触发 phoneEntered 事件进而转移到 lookupCustomer

### 查找顾客

当欢迎表单提交后，顾客的电话号码将包含在请求参数中并准备用于查询顾客。lookupCustomer 状态的`<evaluate>`元素是查找发生的地方。它将电话号码从请求参数中抽取出来并传递到  pizzaFlowActionsbean 的 lookupCustomer() 方法中。

然后根据查找用户结果,是返回异常(说明用户没有注册,然后进入用户注册状态),默认的转移将把流程带到 customerReady 状态.

### 注册新顾客
registrationForm 状态是要求用户填写配送地址的。就像我们之前看到其他的视图状态,它将被渲染成 JSP

```html
<html xmlns:c="http://java.sun.com/jsp/jstl/core"
     xmlns:jsp="http://java.sun.com/JSP/Page"
     xmlns:spring="http://www.springframework.org/tags"
     xmlns:form="http://www.springframework.org/tags/form">
  <jsp:output omit-xml-declaration="yes"/>
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
  <head><title>Spizza</title></head>
  <body>
    <h2>Customer Registration</h2>
    <form:form commandName="customer">
      <input type="hidden" name="_flowExecutionKey"
             value="${flowExecutionKey}"/>
      <b>Phone number: </b><form:input path="phoneNumber"/><br/>
      <b>Name: </b><form:input path="name"/><br/>
      <b>Address: </b><form:input path="address"/><br/>
      <b>City: </b><form:input path="city"/><br/>
      <b>State: </b><form:input path="state"/><br/>
      <b>Zip Code: </b><form:input path="zipCode"/><br/>
      <input type="submit" name="_eventId_submit"
             value="Submit" />
      <input type="submit" name="_eventId_cancel"
             value="Cancel" />
    </form:form>
    </body>
</html>
```
这里不是通过请求参数一个个地处理输入域,而是以更好的方式将表单绑定到 Customer 对象上(让框架来做所有繁杂的工作)

### 检查配送区域

在顾客提供其地址后,我们需要确认他的住处在**配送范围内** 如果 Spizza 不能派送给他们,就需要提示顾客

为了做出这个判断,我们使用了决策状态。决策状态 checkDeliveryArea 有一个 `<if>` 元素,它将顾客的邮政编码传递到 pizzaFlowActions bean 的 checkDeliveryArea() 方法中。这个方法返回一个 Boolean 值:如果顾客在配送区域则为 true,反之 false

如果在配送区域流程转移到 addCustomer 状态。否则顾客被带入 deliveryWarning 背后的视图就是 `"/WEB-INF/flows/pizza/customer/deliveryWarning.jspx"`

```html
<html xmlns:jsp="http://java.sun.com/JSP/Page">
  <jsp:output omit-xml-declaration="yes"/>
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
  <head><title>Spizza</title></head>
  <body>
        <h2>Delivery Unavailable</h2>
        <p>The address is outside of our delivery area. You may
        still place the order, but you will need to pick it up
        yourself.</p>
        <![CDATA[
        <a href="${flowExecutionUrl}&_eventId=accept">
                                Continue, I'll pick up the order</a> |
        <a href="${flowExecutionUrl}&_eventId=cancel">Never mind</a>
        ]]>
  </body>
</html>
```

在 deliveryWarning.jsp 中与流程相关的两个关键点就是那两个链接,它们允许用户继续订单或者将其取消。通过使用与 welcome 状态相同的 flowExecurtionUrl 变量,这些链接分别触发流程中的 accept 或 cancel 事件。

如果发送的是 accept 事件,那么流程会转移到 addCustomer 状态.否则,接下来会是全局的取消转移,子流程将会转移到 cancel 结束状态

### 存储顾客数据
当流程抵达 addCustomer 状态时,用户已经输入了它们的地址.这个地址需要存储卡来。 addCustomer 状态有一个 `<evaluate>`元素,它会调用 pizzaFlowActions bean 的 addCustomer() 方法,并将 customer 流程参数传递进去

一旦完成这个过程,流程将会转移到 ID 为 customerReady 的结束状态

### 结束流程

一般来讲,流程的结束状态并不会那么有意思。但是这个流程中,它不仅仅只有一个结束状态,而是两个。当子流程完成时,它会触发一个与结束状态ID 相同的流程事件,流程能够影响到调用状态的执行方向。

当 customer 流程走完所有正常的路径后,它最终会到达 ID 为 customerReady 的结束状态。当调用它的披萨流程恢复时,它会接收到一个 customerReady 事件,这个事件将会使得转移到 buildOrder 状态

要注意的事 customerReady 结束状态包含了一个 `<output>`元素。在流程中这个元素等同于 Java 中的 return 语句。

- 这样在披萨流程中,能够将 identifyCustomer 子流程的状态指定给订单。
- 如果在识别顾客流程的任意地方触发了 cancel 事件,将会通过 ID 为 cancel 的结束状态退出流程,这也会在触发 全局转移到披萨流程的结束状态

## 构建订单

订单子流程用于提示用户创建披萨并将其放入订单中的。
![](http://ww1.sinaimg.cn/large/006rAlqhly1g311gzv36ij307g07jt9b.jpg)

showOrder 状态位于订单子流程的中心位置。这是用户进入这个子流程时第一个状态,它也是用户添加 pizza 到订单后要转移的状态。它展现了订单的当前状态并允许用户添加其他的披萨到订单中。

添加 pizza 到订单时,流程会转移到 createPizza 状态。这是另外一个视图状态,允许用户选择披萨的尺寸和面饼配料等。这里用户可以添加或取消 pizza,这两种事件都会使流程翳荟 showOrder 状态

从 showOrder 状态,用户可能创建订单也可能取消订单。两种选择都会结束子流程,但是主流程会根据选择的不同进入不同的执行路径。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow
  http://www.springframework.org/schema/webflow/spring-webflow-2.3.xsd">
  <!-- 接收 order 作为输入 -->
  <input name="order" required="true" />
<!-- 展现 order的状态 -->
  <view-state id="showOrder">
     <transition on="createPizza" to="createPizza" />
    <transition on="checkout" to="orderCreated" />
    <transition on="cancel" to="cancel" />
  </view-state>
<!-- 创建 pizza 状态 -->
  <view-state id="createPizza" model="flowScope.pizza">
     <on-entry>
      <set name="flowScope.pizza"
          value="new com.springinaction.pizza.domain.Pizza()" />
      <evaluate result="viewScope.toppingsList" expression=
           "T(com.springinaction.pizza.domain.Topping).asList()" />
    </on-entry>
    <transition on="addPizza" to="showOrder">
      <evaluate expression="order.addPizza(flowScope.pizza)" />
    </transition>
    <transition on="cancel" to="showOrder" />
  </view-state>

<!-- 取消的结束状态 -->
  <end-state id="cancel" />
<!-- 创建订单的结束状态 -->
  <end-state id="orderCreated" />
</flow>

```
这个子流程实际上会操作主流程创建的 Order 对象。因此,我们需要以某种方式将 Order 从主流程传到子流程。这里我们使用 `<input>` 元素来将 Order 传递进流程。

接下来看到 showOrder 状态,它是一个基本的视图状态并具有三个不同的转移,分别用于创建披萨、提交订单以及取消订单。

createPizza 状态的视图是一个表单,这个表单可以添加新的 Pizza 对象到订单中。`<on-entry>`元素添加了一个新的 Pizza 对象到流程作用域内,当表单提交时,表单的内容会填充到该对象中。 **需要注意的是,这个视图状态引用的 model 时流程作用域内的同一个 Pizza 对象。 Pizza 对象将傍顶到创建披萨的表单中**

```html
<div xmlns:form="http://www.springframework.org/tags/form"
     xmlns:jsp="http://java.sun.com/JSP/Page">
  <jsp:output omit-xml-declaration="yes"/>
  <jsp:directive.page contentType="text/html;charset=UTF-8" />
    <h2>Create Pizza</h2>
    <form:form commandName="pizza">
      <input type="hidden" name="_flowExecutionKey"
                value="${flowExecutionKey}"/>
      <b>Size: </b><br/>
  <form:radiobutton path="size"
                    label="Small (12-inch)" value="SMALL"/><br/>
  <form:radiobutton path="size"
                    label="Medium (14-inch)" value="MEDIUM"/><br/>
  <form:radiobutton path="size"
                    label="Large (16-inch)" value="LARGE"/><br/>
  <form:radiobutton path="size"
                    label="Ginormous (20-inch)" value="GINORMOUS"/>
      <br/>
      <br/>
      <b>Toppings: </b><br/>
      <form:checkboxes path="toppings" items="${toppingsList}"
                       delimiter="&lt;br/&gt;"/><br/><br/>
      <input type="submit" class="button"
          name="_eventId_addPizza" value="Continue"/>
      <input type="submit" class="button"
          name="_eventId_cancel" value="Cancel"/>
    </form:form>
</div>
```

当通过 Countinue 按钮提交订单时,尺寸和配料选择将会绑定到 Pizza 对象中并且触发 addPizza 转移.与这个转移关联的 `<evaluate>`元素表明在转移到 showOrder 状态之前,流程作用域内的 Pizza 对象将会传递给订单的 addPizza() 方法中。


主流程要么基于 cancel 事件要么基于 orderCreated 事件进行状态转移。 在前者情况下,外边的主流程会结束；在后者,它将转移到 takePayment 子流程


## 支付
支付流程

![](http://ww1.sinaimg.cn/large/006rAlqhly1g312cuwt4wj30cg09igm9.jpg)

想订单子流程一样,支付子流程也使用`<input>` 元素接收一个 Order 对象作为输入。

进入支付子流程的时候,用户会到达 takePayment 状态。这是一个视图状态,在这里用户可以选择使用信用卡、支票或现金进行支付。提交支付信息后,将进入 verifyPayment 状态,这是一个行为状态,它将校验支付信息是否可以接收。


```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow xmlns="http://www.springframework.org/schema/webflow"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/webflow
  http://www.springframework.org/schema/webflow/spring-webflow-2.3.xsd">
  <!-- 获取订单 -->
  <input name="order" required="true"/>
  <!-- 支付 -->
  <view-state id="takePayment" model="flowScope.paymentDetails">
    <on-entry>
      <set name="flowScope.paymentDetails"
 value="new com.springinaction.pizza.domain.PaymentDetails()" />
      <evaluate result="viewScope.paymentTypeList" expression=
"T(com.springinaction.pizza.domain.PaymentType).asList()" />
    </on-entry>
    <transition on="paymentSubmitted" to="verifyPayment" />
    <transition on="cancel" to="cancel" />
  </view-state>

  <!-- 验证支付 -->
  <action-state id="verifyPayment">
    <evaluate result="order.payment" expression=
        "pizzaFlowActions.verifyPayment(flowScope.paymentDetails)" />
    <transition to="paymentTaken" />
  </action-state>
  <end-state id="cancel" />
  <end-state id="paymentTaken" />
</flow>
```
在流程进入 takePayment 视图时,`<on-entry>` 元素将构建一个支付表单并使用 SpEL 表达式在流程作用域内创建一个 PaymentDetails 实例,这是支撑表单的对象。它也会创建视图作用域的 paymentTypeList 变量,这个变量是一个列表包含了 PaymentType 枚举的值。在这里,SpEL 的 T() 操作用于获得 PaymentType 类,这样就可以调用静态的 asList() 方法。


```java
package com.springinaction.pizza.domain;
import static org.apache.commons.lang.WordUtils.*;
import java.util.Arrays;
import java.util.List;
public enum PaymentType {
  CASH, CHECK, CREDIT_CARD;
  public static List<PaymentType> asList() {
    PaymentType[] all = PaymentType.values();
    return Arrays.asList(all);
  }
  @Override
  public String toString() {
    return capitalizeFully(name().replace('_', ' '));
  }
}
```
> flag: 目前还有一堆代码不完善,但是概念大部分理解了
# 保护 Web 流程

在下一章中,我们将会看到如何使用 Spring Security 来保护应用程序。块速地看一下 Spring Web Flow 是如何结合 Spring Security 支持流程级别的安全性的

Spring Web Flow 中的状态、转移甚至整个流程都可以借助 `<secured>` 元素实现安全性,该元素会作为这些元素的子元素。例如,为了保护对视图状态的访问,你可以这样使用 `<secured>`:

```xml
<view-state id="restricted">
    <secured attributes="ROLE_ADMIN" match="all"/>
</view-state>
```
> 按照这里的配置,只有授予 ROLE_ADMIN 访问权限的用户才能访问这个视图状态。 attributes 属性使用逗号分隔的权限列表来表明用户要访问指定状态、转移或流程所需要的权限。match 属性可以设置为 any 或 all。
- any,用户至少具有一个 attributes 属性所列的权限
- all 用户必须拥有所有的权限

# 小结
流程由多个状态和转移组成,它们定义了会话如何从一个状态到另一个状态。
状态:
- 行为状态执行**业务逻辑**
- 视图状态涉及到流程中的用户
- 决策状态动态地**引导流程执行**
- 结束状态表明流程的结束
- 子流程状态,它们自身通过流程来定义
