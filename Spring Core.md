# Spring Core
#spring/core

这部分文档涵盖了 Spring Framework 绝对不可或缺的所有技术,其中最重要的是 Spring Framework 的控制反转容器. Spring 框架的 IoC 容器的thorough treatment紧随其后，全面覆盖了 Spring 中的面向切面编程技术。 Spring Framework 有自己的 AOP 框架. 此外,文档还提供了 Spring 与 AspectJ 集成(其实是目前最全面的切面编程库).
- - - -
# 1. The IoC Container
![](Spring%20Core/A99C07B7-443A-49B3-A4E7-D46F508AAAB8.png)

## 1.1. Introduction to the Spring IoC Container and Beans

本章介绍了控制反转（IoC）的 Spring Framework 实现。 IoC 也称为依赖注入（DI）。这是一个过程，通过这个过程，对象想要::定义::他们的依赖关系(引用的其他对象),只需要通过构造函数的参数，工厂方法的参数,或者是构造方法或从工厂方法返回后 在对象实例上set的properties,这就行了. 然后容器在创建 bean 时将注入这些依赖项。想象一下,直接调用类的构造函数来直接构造依赖对象,或诸如Service Locator pattern的机制来控制其依赖的对象的位置, IOC 就是这样的做法的一个 inversion。

*org.springframework.beans* 和 *org.springframework.context* 包是 Spring Framework 的 IoC 容器的基础。 BeanFactory 接口提供了一种能够管理任何类型对象的高级配置机制。 ApplicationContext 是 BeanFactory 的子接口,它更提供了了：
* 更容易与 Spring 的 AOP 集成
* Message资源处理（用于国际化）
* Event publication
* 特定于应用程序层的上下文，例如 WebApplicationContext，用于 Web 应用程序。

简而言之，BeanFactory 提供了配置框架和基本功能，而ApplicationContext在此基础之上则添加了更多企业specific 的功能。 ApplicationContext 是 BeanFactory 的完整超集，在本章中用于 Spring 的 IoC 容器的描述。 有关直接使用 BeanFactory 而不是 ApplicationContext 的更多信息，请参阅[TheBeanFactory](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-beanfactory)

在 Spring 中，构成应用程序主干并由 Spring IoC 容器管理的对象称为 bean。 bean 是一个由 Spring IoC 容器实例化，组装和管理的对象。 除此之外，bean 只是应用程序中许多对象之一。 Bean 及其之间的依赖关系提现在容器使用的configuration metadata中。

- - - -
## 1.2. Container Overview

::*org.springframework.context.ApplicationContext* 接口表示 Spring IoC 容器::，负责实例化，配置和组装 bean。容器通过读取configuration metadata来获取有关实例化，配置和组装的对象的指令。配置元数据可以以 XML，Java 注解或 Java 代码表示。它允许您阐述组成应用程序的对象,以及这些对象之间丰富的依赖关系。

Spring 提供了 ApplicationContext 接口的几个实现。在stand-alone applications中，通常会创建 *ClassPathXmlApplicationContext* 或 *FileSystemXmlApplicationContext* 的实例。虽然 XML 是定义configuration metadata的传统格式，但您可以通过提供少量 XML 配置,以声明方式来声明以让容器使用 Java 注解或Java代码来作为元数据格式.

在大多数应用程序方案中，不需要显式用户代码来实例化 Spring IoC 容器的一个或多个实例。例如，在 Web 应用程序场景中，应用程序的 web.xml 文件中的简单几行样板XML 通常就足够了。下图显示了 Spring 如何工作的高级视图。您的应用程序类与configuration metadata相结合，所以在创建和初始化 *ApplicationContext* 之后，您就拥有了一个完全配置且可执行的系统或应用程序。

![](Spring%20Core/container-magic.png)

### 1.2.1. Configuration Metadata

如上图所示，Spring IoC 容器使用一种配置元数据。 此配置元数据表示您作为应用程序开发人员如何告诉 Spring 容器在应用程序中实例化，配置和组装对象。 传统上，配置元数据以简单直观的 XML 格式提供，本章也用 XML 来阐述 Spring IoC 容器的关键概念和功能。

有关在 Spring 容器中使用其他形式的元数据的信息，请参阅：
*  [Annotation-based configuration](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-annotation-config) ：Spring 2.5 引入了对基于注解的配置元数据的支持。
*  [Java-based configuration](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java) ：从 Spring 3.0 开始，Spring JavaConfig 项目提供的许多功能成为核心 Spring Framework 的一部分。 因此，您可以使用 Java 而不是 XML 文件来定义 bean。 要使用这些新功能，请参阅 ::@Configuration，@ Bean，@ Import 和 @DependsOn 注解。::

Spring 配置包含了要交给容器管理的至少一个,通常不止一个 bean定义。基于 XML 的配置元数据将这些 bean 配置为顶级 <beans /> 元素内的 < bean /> 元素。而Java 配置通常在 @Configuration 类中使用 @Bean作为注解的方法。

这些 bean 定义对应了构成应用程序的实际对象。通常，您定义service层对象，数据访问对象（DAO），presentation对象（如 Struts Action 实例），infrastructure对象（如 Hibernate SessionFactories，JMS 队列等）。通常，不会在容器中配置细粒度domain objects，因为一般DAO 和业务逻辑通来负责创建和加载domain objects。 However，您可以使用 Spring 与 AspectJ 的集成来配置在 IoC 容器控制之外创建的对象。请参阅 [Using AspectJ to dependency-inject domain objects with Spring](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#aop-atconfigurable) .。

以下示例显示了基于 XML 的配置元数据的基本结构：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">   
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

id 属性的值指的是协作对象。 此示例中并未显示用于引用协作对象的 XML配置。 有关更多信息，请参阅 [Dependencies](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-dependencies) 。

### 1.2.2. Instantiating a Container

提供给 ApplicationContext 构造函数的位置路径是Resource Strings，它允许容器从各种外部资源（如本地文件系统，Java CLASSPATH 等）来加载配置元数据。
```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```
以下示例显示了services.xml配置文件和 daos.xml配置文件,就是上面 ApplicationContext 用到的配置文件内容,就不多解释了：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- services -->
    <bean id="petStore" class="org.springframework.samples.jpetstore.services.PetStoreServiceImpl">
        <property name="accountDao" ref="accountDao"/>
        <property name="itemDao" ref="itemDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions for services go here -->
</beans>
```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="accountDao"
        class="org.springframework.samples.jpetstore.dao.jpa.JpaAccountDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>

    <bean id="itemDao" class="org.springframework.samples.jpetstore.dao.jpa.JpaItemDao">
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>
    <!-- more bean definitions for data access objects go here -->
</beans>
```

让 bean 定义跨越多个 XML 文件会很有用。 通常，每个单独的 XML 配置文件都代表架构中的逻辑层或模块。您可以使用application context constructor来从所有这些 XML 片段加载 bean 定义。 此构造函数采用了多个 Resource 位置，如上一节中所示。 或者，使用一个或多个 <import /> 元素来从另一个或多个文件加载 bean 定义。 以下示例显示了如何执行此操作：
```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>
    <import resource="/resources/themeSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```
在这个示例中，外部 bean 定义从三个文件加载：services.xml，messageSource.xml 和 themeSource.xml。 所有位置路径都elative to执行import的这个文件.

namespace本身提供了import directive feature(前面配置的 namespace 是 beans)。 除了普通 bean 定义之外的其他配置功能在 Spring 提供的一系列 XML 命名空间中可用 - 例如，context 和 util 命名空间。

### 1.2.3. Using the Container

ApplicationContext 是一个**advanced factory**的接口，能够维护不同 bean 及其依赖项的registry。 通过使用方法 *T getBean（String name，Class <T> requiredType）*，您可以检索 bean 的实例。 ApplicationContext 允许您读取 bean 定义并访问它们，如以下示例所示：
```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("petStore", PetStoreService.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

然后，您可以使用 getBean 来检索 Bean 的实例。 ApplicationContext 接口有一些其他方法可以检索 bean，但理想情况下，应用程序代码永远不应该使用它们。 实际上，您的应用程序代码根本不应该调用 getBean（）方法，也就是不应该依赖于 Spring API。

- - - -
## 1.3. Bean Overview

Spring IoC 容器管理一个或多个 bean。这些 bean 是使用您提供给容器的配置元数据创建的（例如，以 XML <bean /> 定义的形式）。在container本身中，这些 bean 定义其实被表示为 *BeanDefinition* 对象，对象中包含了以下元数据：
* A package-qualified class name：通常是正在定义的 bean 的实际实现类。
* Bean behavioral configuration elements，说明 bean 在容器中的行为方式（范围，生命周期回调等）。
* References to other beans。这些引用也称为collaborators或依赖项。
* Other configuration settings to set in the newly created object. 例如，Pool的大小限制, 或者是管理连接池的 Bean 中所使用的连接数。

此元数据转换为构成每个 bean 定义的一组属性。下表描述了这些属性：
![](Spring%20Core/A8A8237E-5905-4B7C-A3B6-2A1817849799.png)

除了bean definitions包含了如何创建特定的 bean 的信息，ApplicationContext还允许用户在容器中注册现有的、已经在容器外创建的对象。 这是通过调用getBeanFactory()来访问ApplicationContext’s BeanFactory实现的,该方法的返回值为*DefaultListableBeanFactory*.

*DefaultListableBeanFactory*支持通过registerSingleton(..)和registerBeanDefinition(..)方法来直接把对象注册到容器。 然而，典型的应用程序只会通过元数据配置来定义 bean。

### 1.3.1. Naming Beans

每个 bean 都有一个或多个标识符。这些标识符在容器中必须是唯一的。bean 通常只有一个标识符，但如果它需要不止一个，可以使用额外的别名。

在基于 xml 的配置中，您可以使用 *id* 和 (或) *name*属性来指定 bean 标识符。你不需要提供一个 bean 的*name*或 *id*。如果没有显式地提供name或 id, 容器将生成一个唯一的名称给 bean 。然而，如果你想通过bean 的名字,使用 ref 元素来查找，你必须提供一个名称。不使用名称的动机和使用 [inner beans](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-inner-beans) and [autowiring collaborators](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-autowire) .相关。

Bean的命名也是有 convention 的. 也就是说，bean 名称开始以小写字母开头，后面采用 “骆峰式”。例如 “accountManager”、“accountService’,‘userDao’,‘loginController’, 等等。 一致的 beans 命名可以让您的配置容易阅读和理解，如果你正在使用 Spring AOP，当你通过 bean 名称来使用 advice 时，这会对你帮助很大。

在 bean 定义本身中，通过使用 id 属性指定的最多一个名称,和通过使用name属性来指定的任意数量的其他名称， 你可以为 bean 提供多个名称。 这些名称是同一个 bean 的等效别名，这对某些情况很有用，例如让应用程序中的每个组件通过使用component specific的 bean 名称来引用公共依赖项。

但是，指定实际定义 bean 的所有别名并不总是足够的。 有时需要为其他地方定义的 bean 引入别名。 在大型系统中通常就是这种情况，其中配置在每个子系统之间分配，每个子系统具有自己独立的一组对象定义。 在基于 XML 的配置元数据中，您可以使用 <alias /> 元素来完成此任务。 以下示例显示了如何执行此操作：
```xml
<alias name="fromName" alias="toName"/>
```
在这种情况下，在使用此别名定义之后，名为 fromName 的 bean（在同一容器中）也可以称为 toName。

例如，子系统 A 的配置元数据可以通过 subsystemA-dataSource 的名称来引用一个DataSource。 子系统 B 的配置元数据则可以通过 subsystemB-dataSource 的名称引用这个DataSource。 在编写使用这两个子系统的主应用时，主应用程序则通过 myApp-dataSource 的名称引用这个DataSource。 要使所有三个名称引用同一对象，可以将以下别名定义添加到配置元数据中：
```xml
<alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
<alias name="myApp-dataSource" alias="subsystemB-dataSource"/>
```

如果使用 Javaconfiguration，则 @Bean 注解可用于提供别名。 有关详细信息，请参阅 [Using the @Bean Annotation](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation) 。

### 1.3.2. Instantiating Beans

bean definition本质上是用于创建一个或多个对象的配方。 容器在被询问时查看命名 bean 的’配方’，并使用由该 bean 封装的配置元数据来创建（或获取）实际对象。

如果使用基于 XML 的配置元数据，你需要指定要在 <bean/> 元素的 class 属性中实例化的对象的类。 此Class属性（即BeanDefinition 实例上的 Class 属性）通常是必需的。 存在以下两种方法之一来使用 Class 属性：
* 通常，在容器通过反射调用其构造函数以直接创建 bean 的情况下,指定要构造的 bean的 class，效果等同于使用 new 的 Java 代码。
* 指定为创建对象而被调用的**static**工厂方法的实际class，在这种较不常见的情况下，容器在类上调用静态工厂方法来创建 bean。 从静态工厂方法的调用返回的对象类型可以完全是同一个类或另一个类。

#### Instantiation with a Constructor

当您通过构造方法创建 bean 时，所有普通类都可以使用,并与 Spring 兼容。也就是说，我们开发的类不需要实现任何特定接口或以特定方式编码。。但是，根据您为该bean 使用的 IoC 类型，可能需要为该类实现一个默认（空）构造函数。

Spring IoC 容器几乎可以管理您希望它管理的任何类。它不仅限于管理真正的 JavaBeans。大多数 Spring 用户更喜欢实际的 JavaBeans:只有一个默认（无参）构造函数，并包含合适的 setter 和 getter。您还可以在容器中拥有一些非 bean 样式类。例如，如果您需要使用不符合 JavaBean 规范的旧的连接池，Spring 也可以对其进行管理。

使用基于 XML 的配置元数据，您可以按如下方式指定 bean 类：
```xml
<bean id="exampleBean" class="examples.ExampleBean"/>

<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

#### Instantiation with a Static Factory Method

定义使用静态工厂方法创建的 bean 时，请使用 class 属性指定包含静态工厂方法的类,以及名为 factory-method 的属性，以指定工厂方法本身的名称。 您应该能够调用此方法并返回一个live对象，随后将其视为通过构造函数创建的对象。 这种 bean 定义的一个用途是在遗留代码中调用静态工厂。

以下 bean 定义指定通过调用工厂方法来创建 bean。 该定义未指定返回对象的类型（类），仅指定包含工厂方法的类。 在此示例中，createInstance（）方法必须是静态方法。 以下示例显示如何指定工厂方法：
```xml
<bean id="clientService"
    class="examples.ClientService"
    factory-method="createInstance"/>
```
以下示例显示了一个可以使用前面的 bean 定义的类：
```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```

### Instantiation by Using an Instance Factory Method

与通过静态工厂方法实例化类似，使用实例工厂方法进行实例化会从容器调用**现有bean**的非静态方法来创建新 bean。 要使用此机制，请将 class 属性保留为空，并在 factory-bean 属性中指定当前（或祖先）容器中 bean 的名称，该容器包含要被调用以创建对象的实例方法。 使用 factory-method 属性设置工厂方法本身的名称。 以下示例显示如何配置此类 bean：
```java
<!-- the factory bean, which contains a method called createInstance() -->
<bean id="serviceLocator" class="examples.DefaultServiceLocator">
    <!-- inject any dependencies required by this locator bean -->
</bean>

<!-- the bean to be created via the factory bean -->
<bean id="clientService"
    factory-bean="serviceLocator"
    factory-method="createClientServiceInstance"/>
```
以下示例显示了相应的 Java 类：
```java
public class DefaultServiceLocator {

    private static ClientService clientService = new ClientServiceImpl();

    public ClientService createClientServiceInstance() {
        return clientService;
    }
}
```

> In Spring documentation, “factory bean” refers to a bean that is configured in the Spring container and that creates objects through an [instance](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-class-instance-factory-method) or [static](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-class-static-factory-method) factory method. By contrast,FactoryBean(notice the capitalization) refers to a Spring-specific [FactoryBean](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-extension-factorybean) .  

[How to use the Spring FactoryBean?](bear://x-callback-url/open-note?id=2A2FFE48-4649-4FB4-A113-3E42FAB78884-6528-000003AC7C58F05E)

- - - -
## 1.4. Dependencies

典型的企业应用程序不是由单个对象组成的。 即使是最简单的应用程序也有一些对象可以协同工作，以呈现最终用户所看到的连贯的应用程序。 下一节的内容,将从如何定义多个独立的 bean definition，到对象如何协作以实现工作目标。

### 1.4.1. Dependency Injection

依赖注入（DI）是一个过程，通过这个过程，对象只能通过构造函数参数，工厂方法的参数,或构造对象实例后在对象实例上set的属性来::定义::它们的依赖关系。然后容器在创建 bean 时::注入::这些依赖项。这个过程基本上是 bean 本身直接使用构造函数或服务定位器模式来控制其依赖项的位置的逆转,所以也称作控制反转。

使用 DI 原则的代码更清晰，有利于解耦。::对象本身不查找其依赖项，也不知道依赖项的位置或actual类::。因此，您的类变得更容易测试，特别是当依赖关系是接口抽象类时，这允许单元测试使用使用stub或mock。

::DI 存在两个主要变体：基于构造函数的依赖注入和基于 Setter 的依赖注入。::

#### Constructor-based Dependency Injection

基于构造函数的 DI 由容器调用具有多个参数的构造函数来完成，每个参数表示一个依赖项。 调用具有特定参数的静态工厂方法来构造 bean 也能达到效果，以下示例显示了一个只能通过构造函数注入进行依赖注入的类：
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on a MovieFinder
    private MovieFinder movieFinder;

    // a constructor so that the Spring container can inject a MovieFinder
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```
请注意，这个类没有什么特别之处。 它是一个 POJO，它不依赖于容器特定的接口，基类或注解,后面将使用配置来为这个类配置 DI。

##### Constructor Argument Resolution

配置的时候一个很重要的事情就是让 Spring 能了解到配置中提供的依赖对象对应了构造函数里的哪个参数,下面就来看看 Spring 是怎么做的.

通过使用参数的类型来进行构造函数参数解析和匹配。 如果构造函数参数中不存在潜在的歧义，那么在 bean definition 中的构造函数参数的顺序,就是在实例化 bean 时将这些参数提供给适当的构造函数的顺序。 考虑以下class：
```java
package x.y;

public class ThingOne {
    public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
        // ...
    }
}
```
假设 ThingTwo 和 ThingThree 类没继承关系，则不存在潜在的歧义。 因此，以下配置工作正常，您无需在 <constructor-arg /> 元素中显式指定构造函数参数索引或类型。
```xml
<beans>
    <bean id="beanOne" class="x.y.ThingOne">
        <constructor-arg ref="beanTwo"/>
        <constructor-arg ref="beanThree"/>
    </bean>

    <bean id="beanTwo" class="x.y.ThingTwo"/>

    <bean id="beanThree" class="x.y.ThingThree"/>
</beans>
```

当使用简单类型时，例如 <value> true </value>，Spring 无法确定值的类型，因此无法在没有帮助的情况下按类型进行匹配。 考虑以下class：
```java
package examples;

public class ExampleBean {

    // Number of years to calculate the Ultimate Answer
    private int years;

    // The Answer to Life, the Universe, and Everything
    private String ultimateAnswer;

    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

在上面的场景中，可以在配置中使用 type 属性显式指定构造函数参数的类型，容器继而就可以使用简单类型的type matching。如下例所示：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg type="int" value="7500000"/>
    <constructor-arg type="java.lang.String" value="42"/>
</bean>
```

您也可以使用 index 属性显式指定构造函数参数的索引，如以下示例所示：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg index="0" value="7500000"/>
    <constructor-arg index="1" value="42"/>
</bean>
```
除了解决多个简单值的歧义之外，指定索引还可以解决构造函数具有相同类型的两个参数的歧义。

您还可以使用构造函数参数名称进行值消歧，如以下示例所示：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <constructor-arg name="years" value="7500000"/>
    <constructor-arg name="ultimateAnswer" value="42"/>
</bean>
```

请记住，为了使这个特性开箱即用，编译代码时必须with the debug flag enabled，以便 Spring 可以从构造函数中查找参数名称。 如果您不能或不想使用 debug 标志编译代码，则可以使用 @ConstructorProperties 注解来显式命名构造函数参数。 示例类如下所示：
```java
package examples;
public class ExampleBean {

    // Fields omitted

    @ConstructorProperties({"years", "ultimateAnswer"})
    public ExampleBean(int years, String ultimateAnswer) {
        this.years = years;
        this.ultimateAnswer = ultimateAnswer;
    }
}
```

#### Setter-based Dependency Injection

基于 setter 的 DI 是在调用无参构造函数或无参静态工厂方法来实例化 bean ::之后::，再调用 bean 上的 setter 方法来完成的。 以下示例显示了一个只能通过使用纯 setter 注入进行依赖注入的类。 这个类是传统的 Java。 它是一个 POJO，它不依赖于容器特定的接口，基类或注解。
```java
public class SimpleMovieLister {

    // the SimpleMovieLister has a dependency on the MovieFinder
    private MovieFinder movieFinder;

    // a setter method so that the Spring container can inject a MovieFinder
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // business logic that actually uses the injected MovieFinder is omitted...
}
```

ApplicationContext 支持它管理的 bean使用基于构造函数的 DI和基于 setter 的 DI。 甚至支持在通过构造函数注入了一些依赖项之后，再进行基于 setter 的 DI。 您使用 *BeanDefinition* 的形式来配置依赖项，并将其与 *PropertyEditor* 结合使用，以将属性从一种格式转换为另一种格式。 但是，大多数 Spring 用户不直接使用这些类，而是使用 XML bean 定义,或者注解（即使用 @ Component，@ Controller 等注解的类）或者在 基于 Java 的 @Configuration 类中使用 @Bean 方法。 然后，这些配置在内部转换为 BeanDefinition 的实例，并用于加载整个 Spring IoC 容器实例。

```
由于您可以混合使用基于构造函数的 DI和基于 setter 的 DI，因此将构造函数用于强制依赖项,使用 setter 方法配置可选依赖项的方法是一个很好的经验法则。请注意，在 setter 方法上使用 @Required 注解可用于使属性成为必需的依赖项；但是，最好使用构造函数注入。

Spring 团队提倡构造函数注入，因为它允许您将应用程序组件实现为不可变对象，并确保所需的依赖项不为 null。此外，使用构造函数注入的对象始终以完全初始化的状态被返回到客户端。注意，大量的构造函数参数是一个糟糕的代码风格，暗示该类可能有太多的责任，应该重构以更好地解决关注点的正确分离。

Setter 注入应主要仅用于可在类中指定默认值的可选依赖项。否则，必须在代码使用依赖项的任何位置执行非空检查。 setter 注入的一个好处是 setter 方法使该类的对象可以在以后reconfiguration or re-injection。因此，通过 JMX MBean 进行管理是setter注入的一个引人注目的用例。

使用对特定类最有意义的 DI 模式吧! 有时，在处理您没有源码的第三方类时，他们已经为你选好了。例如，当第三方类没有公开任何 setter 方法，那么构造函数注入可能是唯一可用的 DI 形式。
```

#### Dependency Resolution Process

容器采用下面的步骤进行 Bean 依赖解析
* *ApplicationContext* 是使用描述了所有 bean 的配置元数据来创建和初始化的。 配置元数据可以由 XML，Java 代码或注解指定。
* 对于每个 bean，它的依赖关系以properties, constructor arguments,或 static-factory 方法的参数的形式暴露出来,实际创建 bean 时，会将这些依赖项提供给 bean。
* 每个property或构造函数参数都是要设置的actual definition of the value，或者是对容器中另一个 bean 的引用。
* 是一个 value 的每个属性或构造函数参数,都从其指定的格式转换为该属性或构造函数参数的实际类型。 默认情况下，Spring 可以将以字符串格式提供的值转换为所有内置类型，例如 int，long，String，boolean 等(其他的得提供 PropertyEditor 吧)。

Spring 容器在创建容器时将验证每个 bean 的配置。 但是，在实际创建 bean 之前，不会提前设置好bean的属性本身。 创建容器时会创建单例Bean, 否则，仅在请求时才创建 bean。 创建 bean 可能会导致一系列的bean被创建，因为 bean 的依赖关系及其依赖关系也将被创建和分配。 请注意，这些依赖项之间的解决方案不匹配可能会显示的较晚 - 也就是说，首次创建受影响的 bean 时。

你通常可以相信 Spring 会做正确的事。它在容器加载时检测配置问题，例如对不存在的 bean 和循环依赖关系的引用。当实际创建 bean 时，::Spring 会尽可能晚地设置属性并解析依赖关系::。这意味着，如果在创建该对象或其中一个依赖项时出现问题，则在请求对象时，已正确加载的 Spring 容器可以在之后出问题时生成异常 - 例如，bean 因缺失或无效而抛出异常属性。这样的负面影响时可能会延迟一些配置问题的可见性，这就是默认情况下 ApplicationContext 实现预先实例化单例 bean 的原因。以在实际需要bean之前创建这些 bean 的一些前期时间和内存为代价，您会在创建 ApplicationContext 时发现配置问题，而不是以后。您仍然可以覆盖此默认行为，以便单例 bean 可以懒惰地初始化，而不是预先实例化。

如果不存在循环依赖关系，当一个或多个协作 bean 被注入依赖 bean 时，每个协作 bean 在被注入依赖 bean 之前将被完全配置。这意味着，如果 bean A 依赖于 bean B，则 Spring IoC 容器将在调用 bean A 上的 setter 方法之前完全配置好 bean B. 换句话说，bean 被实例化（如果它不是预先实例化的单例），设置其依赖项，并调用相关的生命周期方法（如配置的 init 方法或 InitializingBean 回调方法）。

#### Examples of Dependency Injection

以下示例将基于 XML 的配置元数据来使用基于 setter 的 DI。 Spring XML 配置文件的一小部分指定了一些 bean 定义，如下所示：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- setter injection using the nested ref element -->
    <property name="beanOne">
        <ref bean="anotherExampleBean"/>
    </property>

    <!-- setter injection using the neater ref attribute -->
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的 ExampleBean 类：
```java
public class ExampleBean {

    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public void setBeanOne(AnotherBean beanOne) {
        this.beanOne = beanOne;
    }

    public void setBeanTwo(YetAnotherBean beanTwo) {
        this.beanTwo = beanTwo;
    }

    public void setIntegerProperty(int i) {
        this.i = i;
    }
}
```

在前面的示例中，声明的setter 与 XML 文件中指定的属性匹配。 以下示例使用基于构造函数的 DI：
```xml
<bean id="exampleBean" class="examples.ExampleBean">
    <!-- constructor injection using the nested ref element -->
    <constructor-arg>
        <ref bean="anotherExampleBean"/>
    </constructor-arg>

    <!-- constructor injection using the neater ref attribute -->
    <constructor-arg ref="yetAnotherBean"/>

    <constructor-arg type="int" value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的 ExampleBean 类：
```java
public class ExampleBean {
    private AnotherBean beanOne;

    private YetAnotherBean beanTwo;

    private int i;

    public ExampleBean(
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
        this.beanOne = anotherBean;
        this.beanTwo = yetAnotherBean;
        this.i = i;
    }
}
```
bean definition中指定的构造函数参数用作 ExampleBean 的构造函数的参数。

现在考虑这个例子的变体，其中，不是使用构造函数，而是告诉 Spring 调用静态工厂方法来返回对象的实例：
```xml
<bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
    <constructor-arg ref="anotherExampleBean"/>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg value="1"/>
</bean>

<bean id="anotherExampleBean" class="examples.AnotherBean"/>
<bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
```

以下示例显示了相应的 ExampleBean 类：
```java
public class ExampleBean {

    // a private constructor
    private ExampleBean(...) {
        ...
    }

    // a static factory method; the arguments to this method can be
    // considered the dependencies of the bean that is returned,
    // regardless of how those arguments are actually used.
    public static ExampleBean createInstance (
        AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {

        ExampleBean eb = new ExampleBean (...);
        // some other operations...
        return eb;
    }
}
```

静态工厂方法的参数由 <constructor-arg /> 元素提供，与使用的构造函数完全相同。 工厂方法返回的类的类型不必与包含静态工厂方法的类相同（尽管在本例中，它是）。 非静态工厂方法可以以基本相同的方式使用DI,因此我们不在这里讨论这些细节。

### 1.4.2. Dependencies and Configuration in Detail

如上一节所述，您可以将 bean 属性和构造函数参数定义为对其他bean的引用，也可以将其定义为values defined inline。 为此，Spring 的基于 XML 的配置元数据支持其 <property /> 和 < constructor-arg /> 元素中的各种子元素类型。

#### Straight Values (Primitives, Strings, and so on)

<property /> 元素的 value 属性将property或构造函数参数指定为人类可读的字符串表示形式。 Spring 的 [conversion service](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#core-convert-ConversionService-API) 用于将这些值从 String 转换为属性或参数的实际类型。 以下示例显示了要设置的各种值：
```xml
<bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <!-- results in a setDriverClassName(String) call -->
    <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
    <property name="username" value="root"/>
    <property name="password" value="masterkaoli"/>
</bean>
```

您还可以配置 java.util.Properties 实例，如下所示：
```xml
<bean id="mappings"
    class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">

    <!-- typed as a java.util.Properties -->
    <property name="properties">
        <value>
            jdbc.driver.className=com.mysql.jdbc.Driver
            jdbc.url=jdbc:mysql://localhost:3306/mydb
        </value>
    </property>
</bean>
```
Spring 容器通过使用 JavaBeans PropertyEditor 机制将 <value /> 元素内的文本转换为 java.util.Properties 实例。 

另一种可用的配置元素时 idref. idref 元素只是一种防错方法，可以将容器中另一个 bean 的 id（字符串值 - 而不是引用）传递给 <constructor-arg /> 或 < property /> 元素。 以下示例显示了如何使用它：
```xml
<bean id="theTargetBean" class="..."/>

<bean id="theClientBean" class="...">
    <property name="targetName">
        <idref bean="theTargetBean"/>
    </property>
</bean>
```

前面的 bean 定义代码段与以下代码段完全等效（在运行时）：
```xml
<bean id="theTargetBean" class="..." />

<bean id="client" class="...">
    <property name="targetName" value="theTargetBean"/>
</bean>
```

第一种形式优于第二种形式，因为使用 idref 标记允许容器在部署时验证引用的bean 实际存在。 在第二个变体中，不对name 为targetName 的值执行验证。 当客户端 bean 实际被实例化时，才会发现错误。 特别当如果客户端 bean 是prototype bean，则只能在部署容器后很长时间才能发现此错误和产生的异常。

<idref /> 元素带来值的常见位置是在 ProxyFactoryBean bean 定义中的 AOP 拦截器的配置中。 指定拦截器名称时,使用 <idref /> 元素可以防止拼写错误的拦截器 ID。

#### References to Other Beans (Collaborators)

ref 元素是 <constructor-arg /> 或 < property /> 定义元素中的最后一个元素。您将 bean 的指定属性的值设置为对容器管理的另一个 bean的引用。引用的 bean 是要设置其property的 bean 的依赖项，并且在设置该property之前根据需要对其进行初始化。 （如果协作者是单例 bean，它可能已经被容器初始化。）所有引用最终都是对另一个对象的引用。Scoping和validation取决于您是否通过 bean，local 或 parent 属性指定其他对象的 ID或名称。

通过 <ref /> 标记指定目标 bean 是最常用的形式，并允许创建对同一容器**或父容器**中的任何 bean 的引用，而不管它是否在同一 XML 文件中。 bean 属性的值可以与目标 bean 的 id 属性相同，也可以与目标 bean 的 name 属性相同。以下示例显示如何使用 ref 元素：
```xml
<ref bean="someBean"/>
```

若通过 parent 标签指定目标 bean, 则会创建对当前容器的父容器中的 bean 的引用。 parent 属性的值可以与目标 bean 的 id 属性或目标 bean 的 name 属性中的值之一相同。 目标 bean 必须位于当前 bean 的父容器中。 当您有容器层次结构并且希望将父 bean包装在父容器中，让本容器中具有与父 bean 同名的代理, 此时您应该使用parent 标签。 以下一对列表显示了如何使用父属性：
```xml
<!-- in the parent context -->
<bean id="accountService" class="com.something.SimpleAccountService">
    <!-- insert dependencies as required as here -->
</bean>

<!-- in the child (descendant) context -->
<bean id="accountService" <!-- bean name is the same as the parent bean -->
    class="org.springframework.aop.framework.ProxyFactoryBean">
    <property name="target">
        <ref parent="accountService"/> <!-- notice how we refer to the parent bean -->
    </property>
    <!-- insert other configuration and dependencies as required here -->
</bean>
```

#### Inner Beans

<property /> 或 < constructor-arg /> 元素中的 < bean /> 元素定义了内部 bean(是定义依赖也是定义新 bean)，如下例所示：
```
<bean id="outer" class="...">
    <!-- instead of using a reference to a target bean, simply define the target bean inline -->
    <property name="target">
        <bean class="com.example.Person"> <!-- this is the inner bean -->
            <property name="name" value="Fiona Apple"/>
            <property name="age" value="25"/>
        </bean>
    </property>
</bean>
```

内部 bean 定义不需要定义的 ID 或名称(该 bean 只用在此处)。 如果指定，容器也不使用这个值作为标识符。 容器还会在创建时忽略 scope 标志，因为内部 bean 始终是匿名的，并且始终仅使用外部 bean 创建。 不可能独立访问内部 bean 或将它们注入其他bean。

作为一个极端情况，可以从自定义范围接收destruction callbacks - 例如，对于包含在单例 bean 中的request scope内部 bean。 内部 bean 的创建与其包含它的 bean 相关联，但是销毁回调允许它参与请求范围的生命周期。 这不是常见的情况。 内部 bean 通常只是共享包含它的 bean 的scope。

#### Collections

<list />，<set />，<map /> 和 < props /> 元素分别设置 Java Collection 类型 List，Set，Map 和 Properties 的属性和参数。 以下示例显示了如何使用它们：
```xml
<bean id="moreComplexObject" class="example.ComplexObject">
    <!-- results in a setAdminEmails(java.util.Properties) call -->
    <property name="adminEmails">
        <props>
            <prop key="administrator">administrator@example.org</prop>
            <prop key="support">support@example.org</prop>
            <prop key="development">development@example.org</prop>
        </props>
    </property>
    <!-- results in a setSomeList(java.util.List) call -->
    <property name="someList">
        <list> //两个元素
            <value>a list element followed by a reference</value>
            <ref bean="myDataSource" />
        </list>
    </property>
    <!-- results in a setSomeMap(java.util.Map) call -->
    <property name="someMap">
        <map> //两个元素
            <entry key="an entry" value="just some string"/>
            <entry key ="a ref" value-ref="myDataSource"/>
        </map>
    </property>
    <!-- results in a setSomeSet(java.util.Set) call -->
    <property name="someSet">
        <set>
            <value>just some string</value>
            <ref bean="myDataSource" />
        </set>
    </property>
</bean>
```

Map key或value的值或set 的值也可以是以下任何元素：
```
bean | ref | idref | list | set | map | props | value | null
```

Spring 容器还支持合并集合。 应用程序开发人员可以定义父 <list />，<map />，<set /> 或 < props /> 元素，并让子 < list />，<map />，<set /> 或 < props /> 元素 继承和覆盖父集合中的值。 也就是说，子集合的值是合并父集合和子集合的元素的结果，子集合的元素覆盖父集合中指定的值。 关于合并的这一部分讨论了父子 bean 机制。 不熟悉父母和子 bean 定义的读者可能希望在继续之前阅读 [relevant section](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-child-bean-definitions) 。

以下示例演示了集合合并：
```xml
<beans>
    <bean id="parent" abstract="true" class="example.ComplexObject">
        <property name="adminEmails">
            <props>
                <prop key="administrator">administrator@example.com</prop>
                <prop key="support">support@example.com</prop>
            </props>
        </property>
    </bean>
    <bean id="child" parent="parent">
        <property name="adminEmails">
            <!-- the merge is specified on the child collection definition -->
            <props merge="true">
                <prop key="sales">sales@example.com</prop>
                <prop key="support">support@example.co.uk</prop>
            </props>
        </property>
    </bean>
<beans>
```
 以下清单显示了结果:
```
administrator=administrator@example.com
sales=sales@example.com
support=support@example.co.uk
```

此合并行为同样适用于 <list />，<map /> 和 < set /> 集合类型。 在 <list /> 元素的特定情况下，保持与 List 集合类型相关联的语义（即，有序的值集合的概念）。 父bean集合的值位于所有子bean 集合的值之前。 对于 Map，Set 和 Properties 集合类型，不存在排序。

通过在 Java 5 中引入泛型类型，您可以使用强类型集合。 也就是说，可以声明 Collection 类型，使其只能包含String 元素这种。 如果使用 Spring 将强类型集合依赖注入到 bean 中，则可以利用 Spring 的类型转换支持，以便强类型 Collection 实例的元素在添加到类型之前转换为适当的类型。 以下 Java 类和 bean 定义显示了如何执行此操作：
```java
public class SomeClass {

    private Map<String, Float> accounts;

    public void setAccounts(Map<String, Float> accounts) {
        this.accounts = accounts;
    }
}
```

```xml
<beans>
    <bean id="something" class="x.y.SomeClass">
        <property name="accounts">
            <map>
                <entry key="one" value="9.99"/>
                <entry key="two" value="2.75"/>
                <entry key="six" value="3.99"/>
            </map>
        </property>
    </bean>
</beans>
```
当 bean 的 accounts 属性准备好进行注入时，可以通过反射获得有关强类型 Map <String，Float> 的元素类型的泛型信息。 因此，Spring 的类型转换基础结构将各种值元素识别为 Float 类型，并自动将字符串值（9.99,2.75 和 3.99）转换为实际的 Float 类型。

#### Null and Empty String Values

Spring 将属性等的空参数视为空字符串。 以下基于 XML 的配置元数据片段将 email 属性设置为空 String 值。
```xml
<bean class="ExampleBean">
    <property name="email" value=""/>
</bean>
```

<null /> 元素处理空值。 以下清单显示了一个示例：
```xml
<bean class="ExampleBean">
    <property name="email">
        <null/>
    </property>
</bean>
```

#### Compound Property Names

只要除最终属性名称之外的路径的所有组件都不为 null，您可以在设置 bean 属性时使用复合或嵌套属性名称。 考虑以下 bean 定义：
```xml
<bean id="something" class="things.ThingOne">
    <property name="fred.bob.sammy" value="123" />
</bean>
```
这个 bean 有一个 fred 属性，它有一个 bob 属性，它有一个 sammy 属性，最后的 sammy 属性被设置为 123. 为了使它工作，某事物的 fred 属性和 bob 属性 在构造 bean 之后，fred 不能为 null。 否则，抛出 NullPointerException。

### 1.4.3. Using depends-on

如果 bean 是另一个 bean 的依赖项，那通常意味着将一个 bean 被设置为另一个 bean 的属性。 通常，您可以使用基于 XML 的配置元数据中的 <ref /> 元素来完成此操作。 但是，有时 bean 之间的依赖关系不那么直接。 例如，需要触发类中的静态初始化程序，例如数据库驱动程序注册。 在初始化使用bean A的 bean B 之前，::depends-on 属性可以显式强制初始化bean B::。 以下示例使用 depends-on 属性表示对单个 bean 的依赖关系：
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager"/>
<bean id="manager" class="ManagerBean" />
```

要表示对多个 bean 的依赖关系，请提供 bean 名称的列表来作为 depends-on 属性的值（逗号，空格和分号是有效的分隔符）：
```xml
<bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="ManagerBean" />
<bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
```

### 1.4.4. Lazy-initialized Beans

默认情况下，ApplicationContext 实现会急切地创建和配置所有单例 bean，这是初始化过程的一部分。 通常，这种预先实例化是可取的，因为配置或环境中的错误是可以立即发现的，而不是几小时甚至几天后。 当不希望出现这种情况时，可以通过将 bean 定义标记为延迟初始化来阻止单例 bean 的预实例化。 延迟初始化的 bean 告诉 IoC 容器::在第一次请求时创建 bean 实例::，而不是在启动时。
```xml
<bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
<bean name="not.lazy" class="com.something.AnotherBean"/>
```

当 ApplicationContext 使用前面的配置时，在 ApplicationContext 启动时不会急切地实例化lazy bean，而是急切地预先实例化 not lazy bean。

**但是，当延迟初始化的 bean 是未进行延迟初始化的单例 bean 的依赖项时，ApplicationContext 会在启动时创建延迟初始化的 bean，因为它必须满足单例的依赖关系**。 惰性初始化的 bean 被注入到其他地方的单例非惰性初始化 bean 中.

您还可以使用 <beans /> 元素上的 default-lazy-init 属性在容器级别控制延迟初始化，以下示例显示：
```xml
<beans default-lazy-init="true">
    <!-- no beans will be pre-instantiated... -->
</beans>
```


### 1.4.5. Autowiring Collaborators

Spring 容器可以autowire协作 bean 之间的关系。 您可以让 Spring 自动为您的 bean 解析协作者（其他 bean）,这是通过检查 ApplicationContext 的内容实现的。 自动装配具有以下优点：
* 自动装配可以显着减少指定属性或构造函数参数的需要。
* 自动装配可以随着对象的发展来更新配置。 例如，如果需要向类添加依赖项，则可以自动满足该依赖项，而无需修改配置。 因此，自动装配在开发期间尤其有用，without negating the option of switching to explicit wiring when the code base becomes more stable.

使用基于 XML 的配置元数据（请参阅依赖注入）时，可以使用 <bean /> 元素的 autowire 属性为 bean 定义指定 autowire 模式。 自动装配功能有四种模式。 您指定每个 bean 的自动装配，因此可以自己选择模式。 下表描述了四种自动装配模式：
* ::no:: （默认）无自动装配。 Bean 引用必须由 ref 元素定义。 不建议对较大的部署更改默认设置，因为明确指定依赖对象可以提供更好的控制和清晰度。 在某种程度上，显示配置记录了系统的结构。
* ::byName::  按属性名称自动装配。 Spring 查找与需要自动装配的属性同名的 bean。 例如，如果 bean 定义为按名称autowire 并且它包含 master 属性（即，它具有 setMaster（..）方法），则 Spring 会查找名为 master 的 bean 定义并使用它来设置属性。
* ::byType::  ::如果容器中只存在一个属性对应类的 bean，则允许属性自动装配::。 如果存在多个，则抛出致命异常，这表示您不能对该 bean 使用 byType 自动装配。 如果没有匹配的 bean，则不会发生任何事情（该属性将是未设置）。
* ::constructor:: 类似于 byType 但适用于构造函数参数。 如果容器中没有匹配构造函数参数类型的exactly one bean，则会引发致命错误。

使用 byType 或constructor自动装配模式，您可以连接数组和类型集合。 在这种情况下，容器内与预期类型匹配的所有 autowire 候选者都将被提供以满足依赖性。 如果键类型是 String，则可以自动装配强类型的 Map 实例。 自动装配的 Map 的内容由与预期类型匹配的所有 Bean 实例的内容组成，Map 实例的键包含相应的 bean 名称。

当在整个项目中一致地使用自动装配时，自动装配效果最佳。 如果一般不使用自动装配，那么开发人员仅仅使用它来连接一个或两个 bean 可能会让人感到困惑。

考虑自动装配的局限和缺点：
* property 和 constructor-arg 设置中的显式依赖项始终会把自动装配覆盖。您不能自动装配简单属性，例如primitives，字符串和类（以及此类简单属性的数组）。
* 自动装配不如显式装配精确。虽然如前面的表中所述，但 Spring 会谨慎地避免在模糊的情况下进行猜测。您不再明确记录 Spring 管理对象之间的关系。
* 可能无法为可能从 Spring 容器生成文档的工具提供装配信息。
* 可能存在容器中的多个 bean definition可以匹配 setter 方法或构造函数参数来进行自动装配。对于数组，集合或 Map 实例，这不一定是个问题。但是，对于期望single value的依赖关系，这种模糊性不好解决.

在后一种情况下，您有几种选择：
* 放弃自动装配，使用显式装配。
* 通过将 autowire-candidate 属性设置为 false，避免对 bean 作为被自动装配的对象，如下一节所述。
* 通过将其 <bean /> 元素的 primary 属性设置为 true，将单个 bean 定义指定为主要的自动装配候选者。
* 使用基于注解的配置,来实现更细粒度的控制， [Annotation-based Container Configuration](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-annotation-config) .

您可以从自动装配中排除 bean。 在 Spring 的 XML 格式中，将 <bean /> 元素的 autowire-candidate 属性设置为 false。 容器使特定的 bean 对自动装配基础不可用（包括注解样式配置，如 @Autowired）。

您还可以根据与 bean 名称的模式匹配来限制 autowire 候选者。 顶级 <beans /> 元素在其 default-autowire-candidates 属性中接受一个或多个模式。 例如，要将 autowire 候选状态限制为名称以 Repository 结尾的任何 bean，请提供值 * Repository。 要提供多个模式，请在逗号分隔的列表中定义它们。 bean 定义的 autowire-candidate 属性的显式值 true 或 false 始终优先。 对于此类 bean，模式匹配规则不适用。

这些技术对于您永远不希望通过自动装配被注入其他 bean 的 bean 非常有用。 这并不意味着排除的 bean 本身不能使用自动装配进行配置。

### 1.4.6. Method Injection

在大多数应用程序场景中，容器中的大多数 bean 都是单例的。 当单例 bean 需要与另一个单例 bean 协作或非单例 bean 需要与另一个非单例 bean 协作时，通常通过将一个 bean 定义为另一个 bean 的属性来处理依赖关系。 实际上,当 bean 生命周期不同时会出现问题。 假设单例 bean A 需要使用非单例（原型）bean B，可能是在 A 上的每个方法调用上。容器只创建一次单例 bean A，因此只有一次机会来设置属性。 容器不能每次需要时都为 bean A 提供 bean B 的新实例。

解决方案是放弃一些控制反转。 您可以通过实现 *ApplicationContextAware* 接口使 [make bean A aware of the container](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-aware) ，并通过对容器进行 getBean（“B”）调用，每次 bean A 需要时都要求（通常是新的）bean B 实例。 以下示例显示了此方法：
```java
public class CommandManager implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Object process(Map commandState) {
        // grab a new instance of the appropriate Command
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    protected Command createCommand() {
        // notice the Spring API dependency!
        return this.applicationContext.getBean("command", Command.class);
    }

    public void setApplicationContext(
            ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
前面的内容是不太好的，因为业务代码耦合到 Spring Framework API。 **方法注入**是 Spring IoC 容器的一个高级功能，可以让您干净地处理这个用例。

Lookup Method Injection是容器去override所管理的 bean 上的方法,并返回容器中另一个命名 bean 的查找结果的能力。 查找通常涉及原型 bean，如上一节中描述的场景。 Spring Framework 通过使用 CGLIB 库中的字节码生成来动态生成覆盖该方法的子类来实现此方法注入。

对于前面代码片段中的 CommandManager 类，Spring 容器动态地覆盖 createCommand（）方法的实现。 CommandManager 类没有任何 Spring 依赖项，下面为重写的示例：
```java
// no more Spring imports!

public abstract class CommandManager {

    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```
现在这个createCommand()就是要注入的方法了. 在包含要注入的方法的client类（在本例中为 CommandManager）中，要注入的方法需要以下形式的签名：
```
<public|protected> [abstract] <return-type> theMethodName(no-arguments);
```

如果方法是抽象的，则动态生成的子类将实现该方法。 否则，动态生成的子类将覆盖原始类中定义的具体方法。 请考虑以下示例：
```xml
<!-- a stateful bean deployed as a prototype (non-singleton) -->
<bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
    <!-- inject dependencies here as required -->
</bean>

<!-- commandProcessor uses statefulCommandHelper -->
<bean id="commandManager" class="fiona.apple.CommandManager">
    <lookup-method name="createCommand" bean="myCommand"/>
</bean>
```
这里做了配置,意思是commandManager中的myCommand属性是通过createCommand()这个lookup-method进行实例化的,这使得每次单例需要 prototype 的时候,通过调用lookup-method都能得到一个新的原型 bean.

标识为 commandManager 的 bean 在需要 myCommand bean 的新实例时调用自己的 createCommand（）方法就好。 如果实际需要，您须将 myCommand bean 部署为原型。 如果它是单例，则每次都返回的都是 myCommand bean 相同实例。

或者，在基于注解的模型中，您可以通过 @Lookup 注解声明查找方法，如以下示例所示：
```java
public abstract class CommandManager {

    public Object process(Object commandState) {
        Command command = createCommand();
        command.setState(commandState);
        return command.execute();
    }

    @Lookup("myCommand")
    protected abstract Command createCommand();
}
```

请注意，您通常应该使用concrete stub implementation来实现一下这种带注解的查找方法，以使它们与 Spring 的component scan规则兼容(默认情况下抽象类被忽略)。 此限制不适用于显式注册或显式imported的 bean。
- - - -
## 1.5. Bean Scopes

创建 bean definition时，你其实创建了一个可以用于创建对应 bean的配方。 bean 定义是一个配方的想法很重要，因为它意味着，您可以从一个配方创建::许多::对象实例。

您不仅可以控制从bean 定义创建的对象的各种依赖项和配置值，还可以控制从特定的bean 定义创建的对象的scope。这种方法功能强大且灵活，因为您可以选择通过配置创建的对象的scope，而不必在 Java 类级别bake in the scope of an object。可以将 Bean 定义为部署在多个范围之一中。 Spring Framework 支持六个scope，其中有四个只有在使用支持 Web 的 ApplicationContext 时才可用。您还可以创建自定义的 scope。

下表描述了支持的范围：
* ::Singleton:: 将单个 bean definition限定为一个Spring IoC 容器仅存在单个对象实例。
* ::prototype:: 单个 bean definition 可以存在任何数量的对象实例
* ::request:: 将单个 bean definition 范围限定为单个 HTTP 请求的生命周期。 也就是说，每个 HTTP 请求都有自己的 bean 实例，仅在 Web 感知 Spring ApplicationContext 的上下文中有效。
* ::session::  将单个 bean definition 范围限定为一个HTTP 会话的生命周期。 仅在 Web 感知 Spring ApplicationContext 的上下文中有效。
* ::application:: 将单个 bean definition范围限定为 ServletContext 的生命周期。 仅在 Web 感知 Spring ApplicationContext 的上下文中有效。
* ::websocket:: 将单个 bean definition范围限定为 WebSocket 的生命周期。 仅在 Web 感知 Spring ApplicationContext 的上下文中有效。

### 1.5.1. The Singleton Scope

只管理单个 bean 的唯一的一个共享的实例，并且对与对应bean definition匹配的 ID 的 bean 的所有请求都会导致 Spring 容器返回这个bean 实例。

换句话说，当您定义 bean definition并将其作为sigleton作用域时，Spring IoC 容器只创建该 bean definition 所定义的对象的唯一一个实例。 此单例存储在这些sigleton bean 的缓存中，并且对该命名 Bean 的所有后续请求和引用都将返回缓存对象。 下图显示了单例范围的工作原理：

![](Spring%20Core/singleton.png)
Spring 单例的范围最好描述为being per-container and per-bean。 这意味着，如果在单个 Spring 容器中为特定类定义一个 bean，则 Spring 容器将根据bean definition创建一个且仅一个实例(::一个 ApplicationContext 一个::)。 单例范围是 Spring 中的默认范围。 要将 bean 定义为 XML 中的单例，您可以定义一个 bean，如以下示例所示
```xml
<bean id="accountService" class="com.something.DefaultAccountService"/>

<!-- the following is equivalent, though redundant (singleton scope is the default) -->
<bean id="accountService" class="com.something.DefaultAccountService" scope="singleton"/>
```

### 1.5.2. The Prototype Scope(原型范围)

非单例的prototype scope导致每次发出对该类bean 的请求时都创建新的 bean 实例。 也就是说，bean 被注入另一个 bean，或者通过container上的 getBean（）方法调用来请求它。 通常，::您应该对所有有状态 bean 使用prototype，对无状态 bean 使用singleton。::

下图说明了 Spring 原型范围：
![](Spring%20Core/prototype.png)
数据访问对象（DAO）通常不配置为原型，因为典型的 DAO 不会保持任何会话状态。重用单例设计方法会更简单一些。以下示例将 bean 定义为 XML 中的原型：
```xml
<bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
```

与其他范围相比，Spring 不管理原型范围 bean 的完整生命周期。容器实例化，配置和组装prototype对象,然后将其交给客户端，而不会有该原型实例的进一步记录。因此，尽管无论范围如何,都会在所有对象上调用初始化生命周期的回调方法，但在原型的情况下，不会调用销毁生命周期的回调方法。客户端代码必须清理原型范围的对象,并释放原型 bean 所拥有的昂贵资源。要使 Spring 容器释放原型范围的 bean 所拥有的资源，请尝试使用自定义的 [bean post-processor](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-extension-bpp) ，它包含对需要清理的 bean 的引用。

在某些方面，Spring 容器关于原型范围 bean 的角色是 Java new 运算符的替代品。超过该点的所有生命周期管理必须由客户端处理。 （有关 Spring 容器中 bean 的生命周期的详细信息，请参阅 Lifecycle Callbacks。）

### 1.5.3. Singleton Beans with Prototype-bean Dependencies

当您使用具有依赖于原型 bean 的单例 bean 时，请注意解析依赖项发生在实例化的时候。 因此，如果原型bean注入到单例bean 中，则会实例化一个新的原型 bean，然后将依赖注入到单例 bean 中。 之后使用单例 bean 的时候,应用到的都是这个原型 bean,并不是每次使用单例 bean 的时候都得到新的原型 bean.(但是若不同的单例 bean 都用到这个原型 bean,那么在不同的单例 bean中,原型 bean 实例并不是同一个的)

但是，假设您希望单例bean 在运行时,重复获取原型bean 的新实例。 您不能将原型范围的 bean 依赖注入到您的单例 bean 中，因为当 Spring 容器实例化单例 bean 并解析并注入其依赖项时，该注入只发生一次。 如果您需要在运行时多次使用原型 bean 的新实例，请参阅 [Method Injection](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection) 

### 1.5.4. Request, Session, Application, and WebSocket Scopes

仅当您使用 Web 感知的 Spring ApplicationContext 实现（例如 XmlWebApplicationContext）时，别的几个Scope才可用。 如果将这些范围与常规的 Spring IoC 容器（例如 ClassPathXmlApplicationContext）一起使用，则会由于未知 bean scope 引发 IllegalStateException。

要在请求，会话，应用程序和 websocket 级别（Web 范围的 bean）支持 bean 的scope，**在定义 bean 之前需要一些小的初始配置**。 （标准的单例和原型scope不需要此初始设置）如何完成此初始设置取决于您的特定 Servlet 环境。 如果您在 Spring Web MVC 中访问 scoped bean(实际上是在 Spring DispatcherServlet 处理的请求中使用这些 bean),则无需进行特殊设置。 DispatcherServlet 已经暴露了所有相关状态。

如果您在 Spring 的 DispatcherServlet 之外处理请求（例如，使用 JSF 或 Struts 时），则需要注册 org.springframework.web.context.request.RequestContextListener ServletRequestListener。对于 Servlet 3.0+，这可以利用 WebApplicationInitializer 接口以编程方式完成此操作。

#### Request scope

考虑以下 XML 配置：
```xml
<bean id="loginAction" class="com.something.LoginAction" scope="request"/>
```

Spring 容器通过对每个 HTTP 请求使用 loginAction bean definition 来创建 LoginAction bean 的新实例。 也就是说，loginAction bean 的范围是 HTTP 请求级别。 您可以根据需要更改所创建的实例的内部状态，因为从同一 loginAction bean denifition创建的其他实例看不到这些更改。当请求完成处理时，将丢弃这些作用于请求的 bean。

使用注解驱动的组件或 Java 配置时，可以使用 ::@RequestScope:: 注解将组件分配给request scope。 以下示例显示了如何执行此操作：
```java
@RequestScope
@Component
public class LoginAction {
    // ...
}
```

#### Session Scope

考虑以下 XML 配置：
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>
```

userPreferences bean 在 HTTP 会话级别有效地作用域。 与request scope的 bean 一样，您可以根据需要更改创建的实例的内部状态，因为同样使用从同一 userPreferences bean definition来创建bean实例的其他 HTTP Session,不会看到这些更改 ，因为它们特定于单个 HTTP 会话。 最终丢弃 HTTP 会话时，也会丢弃作用于该特定 HTTP 会话的 bean。

使用注解驱动的组件或 Java 配置时，可以使用 @SessionScope 注解将组件分配给会话范围。
```java
@SessionScope
@Component
public class UserPreferences {
    // ...
}
```

#### Application Scope

考虑一下 XML 配置
```xml
<bean id="appPreferences" class="com.something.AppPreferences" scope="application"/>
```

appPreferences bean 的作用域是 ServletContext 级别，并被存储为一般的 ServletContext 属性。 这有点类似于 Spring 单例 bean(::整个 application 只有一个::)，但在两个重要方面有所不同：它是每个 ServletContext 的单例，而不是每个 Spring 的 'ApplicationContext'（在一个 Web 应用程序中可能有好几个），它实际上是暴露的，因此 作为 ServletContext 属性可见。

使用注解驱动的组件或 Java 配置时，可以使用 @ApplicationScope 注解将组件分配给应用程序范围。 以下示例显示了如何执行此操作：
```java
@ApplicationScope
@Component
public class AppPreferences {
    // ...
}
```

#### Scoped Beans as Dependencies

Spring IoC 容器不仅管理对象（bean）的实例化，还管理协作者（或依赖关系）的连接。 举个例子,如果要将HTTP request范围的 bean 注入到寿命较长范围的另一个 bean 中，您可以选择注入 AOP 代理来代替这个 request scope bean。 也就是说，您需要注入一个代理对象，该对象公开与scoped对象相同的公共接口，也可以从相关scope（例如 HTTP 请求）中检索真实目标对象，并将方法调用委托给真实对象。

以下示例中的配置只有一行，但了解 “为什么” 以及它背后的 “如何” 非常重要：
```xml
    <!-- an HTTP Session-scoped bean exposed as a proxy -->
    <bean id="userPreferences" class="com.something.UserPreferences" scope="session">
        <!-- instructs the container to proxy the surrounding bean -->
        <aop:scoped-proxy/> 
    </bean>

    <!-- a singleton-scoped bean injected with a proxy to the above bean -->
    <bean id="userService" class="com.something.SimpleUserService">
        <!-- a reference to the proxied userPreferences bean -->
        <property name="userPreferences" ref="userPreferences"/>
    </bean>
```

要创建这样的代理，请将子 <aop：scoped-proxy /> 元素插入到作用域 bean 定义中。 为什么在请求，会话和自定义范围级别定义 bean 的定义会需要 <aop：scoped-proxy /> 元素呢？ 考虑以下单例 bean 定义，并将其与您需要为上述范围定义的内容进行对比（请注意，以下 userPreferences bean 定义不完整）：
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session"/>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

在前面的示例中，单例 bean（userManager）注入了对 HTTP 会话范围的 bean（userPreferences）的引用。 这里的重点是 userManager bean 是一个单例：它在每个容器只实例化一次，它的::依赖项也只注入一次::。 这意味着 userManager bean 仅在完全相同的 userPreferences bean 上（即最初注入它的对象）上运行。

这不是你想要的行为。相反，您需要一个 userManager 对象，并且，对于 HTTP 会话的生命周期，您需要userManeger中存在一个特定于 HTTP 会话的 userPreferences 对象。因此，容器创建一个对象，该对象公开与 UserPreferences 类完全相同的公共接口，该对象可以从scoping mechanism（HTTP 请求，会话等）获取真实的 UserPreferences 对象。容器将此代理对象注入 userManager bean，userManager并不知道此 UserPreferences 引用实际上是代理。

在此示例中，当 UserManager 实例在依赖注入的 UserPreferences 对象上调用方法时，它实际上是在代理上调用方法。然后，代理从HTTP Session中获取真实的 UserPreferences 对象，并将方法调用委托给检索到的真实 UserPreferences 对象。因此，在将request- and Session-scoped beans 注入协作对象时，您需要以下（正确和完整）配置，如以下示例所示：
```xml
<bean id="userPreferences" class="com.something.UserPreferences" scope="session">
    <aop:scoped-proxy/>
</bean>

<bean id="userManager" class="com.something.UserManager">
    <property name="userPreferences" ref="userPreferences"/>
</bean>
```

默认情况下，当 Spring 容器为使用 <aop：scoped-proxy /> 元素标记的 bean 创建代理时，将创建基于 CGLIB 的类代理。
其实，您可以通过把 <aop：scoped-proxy /> 元素的 proxy-target-class 属性的值指定为 false，来创建基于 JDK 接口的标准代理。 使用基于 JDK 接口的代理意味着您不需要在classpth中使用其他库来affect此类代理。 但是，这也意味着作用域 bean 的类必须至少实现一个接口(interface-based proxy),参见 [Proxying Mechanisms](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#aop-proxying) .

- - - -
## 1.6. Customizing the Nature of a Bean

Spring Framework 提供了许多可用于自定义 bean 特性的接口。 本节将它们分组如下：
*  [Lifecycle Callbacks](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-lifecycle) 
*  [ApplicationContextAware and BeanNameAware](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-aware) 
*  [OtherAwareInterfaces](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#aware-list) 

### 1.6.1. Lifecycle Callbacks

要与容器管理的 bean 生命周期进行交互，可以实现 Spring InitializingBean 和 DisposableBean 接口。 容器为前者调用 afterPropertiesSet（），为后者调用 destroy（），让 bean 在初始化之后和销毁 bean之前执行某些操作。

而JSR-250 @PostConstruct 和 @PreDestroy 注解通常被认为是在现代 Spring 应用程序中接收Lifecycle Callbacks的最佳实践。 使用这些注解意味着您的 bean 不会耦合到特定于 Spring 的接口。 有关详细信息，请参阅使用[Using @PostConstructand and @PreDestroy](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-postconstruct-and-predestroy-annotations)

::在内部，Spring Framework 使用 *BeanPostProcessor* 实现来处理它可以找到的任何回调接口,并调用适当的回掉方法。:: 如果您需要 Spring 默认提供的自定义功能或其他生命周期行为，您可以自己实现 BeanPostProcessor。 有关更多信息，请参阅 [Container Extension Points](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-extension) 。

除了初始化和销毁回调之外，Spring 管理的对象还可以实现 *Lifecycle* 接口，以便这些对象可以参与启动和关闭过程，这是由容器自身的生命周期驱动的。

#### Initialization Callbacks

org.springframework.beans.factory.InitializingBean 接口允许 bean 在容器为 bean 设置所有必需属性后执行初始化工作。 InitializingBean 接口包含一个方法：
```java
void afterPropertiesSet() throws Exception;
```

我们建议您不要使用 InitializingBean 接口，因为它会将代码耦合到 Spring。 我们建议使用 @PostConstruct 注解或在 POJP 内指定初始化方法。 对于使用 Java 配置的应用，您可以使用 @Bean 的 initMethod 属性。 请参阅接收生命周期回调。 请考虑以下示例：
```xml
<bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
```
```java
public class ExampleBean {
    public void init() {
        // do some initialization work
    }
}
```

前面的示例与以下示例几乎完全相同：
```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```
```java
public class AnotherExampleBean implements InitializingBean {

    public void afterPropertiesSet() {
        // do some initialization work
    }
}
```

#### Destruction Callbacks

实现 org.springframework.beans.factory.DisposableBean 接口允许 bean 在::包含它的容器被销毁时::获得回调。 DisposableBean 接口指定一个方法：
```java
void destroy() throws Exception;
```

我们建议您不要使用 DisposableBean 回调接口，因为它会将代码耦合到 Spring。 或者，我们建议使用 @PreDestroy 注解或指定 bean 定义支持的泛型方法。使用 Java 配置，您可以使用 @Bean 的 destroyMethod 属性。 请参阅接收生命周期回调。 考虑以下定义：
```xml
<bean id="exampleInitBean" class="examples.ExampleBean" destroy-method="cleanup"/>
```
```java
public class ExampleBean {

    public void cleanup() {
        // do some destruction work (like releasing pooled connections)
    }
}
```
前面的定义与以下定义几乎完全相同：
```xml
<bean id="exampleInitBean" class="examples.AnotherExampleBean"/>
```
```java
public class AnotherExampleBean implements DisposableBean {
    public void destroy() {
        // do some destruction work (like releasing pooled connections)
    }
}
```

#### Default Initialization and Destroy Methods

当您编写初始化和销毁​时​不使用InitializingBean 和 DisposableBean 接口时，通常会编写名称为 init（），initialize（），dispose（）等的方法。理想情况下，此类生命周期回调方法的名称在项目中是标准化的，以便所有开发人员使用相同的方法名称并确保一致性。

您可以将 Spring 容器配置为 “look for” 在每个 bean 上符合名字规范的回调方法来做会掉。这意味着，作为应用程序开发人员，您可以编写应用程序类并使用名为 init（）的初始化回调，而无需为每个 bean 定义配置 init-method =“init” 属性。 Spring IoC 容器在创建 bean 时调用该方法。此功能还强制执行初始化和销毁​​方法回调的一致命名约定。

假设您的初始化回调方法名为 init（），而您的 destroy 回调方法名为 destroy（）。然后，您的类将类似于以下示例中的类：
```java
public class DefaultBlogService implements BlogService {

    private BlogDao blogDao;

    public void setBlogDao(BlogDao blogDao) {
        this.blogDao = blogDao;
    }

    // this is (unsurprisingly) the initialization callback method
    public void init() {
        if (this.blogDao == null) {
            throw new IllegalStateException("The [blogDao] property must be set.");
        }
    }
}
```
然后，您需要做一下配置:
```xml
<beans default-init-method="init">

    <bean id="blogService" class="com.something.DefaultBlogService">
        <property name="blogDao" ref="blogDao" />
    </bean>

</beans>
```
上面配置了所有 bean 的默认 init 方法名为 init, 这会导致 Spring IoC 容器将 bean 类上的 init 方法识别为initialization方法回调。当 bean 被创建和组装时，如果 bean 类具有这样的方法，则会在适当的时候调用它。

您还可以使用顶级 <beans /> 元素上的 default-destroy-method 属性，以类似方式配置 destroy 方法回调。

Spring 容器保证在为 bean 提供所有依赖项后,就会立即调用配置好的初始化回调方法。因此，在原始 bean 引用上调用初始化回调，这意味着 AOP 拦截器等组件尚未应用于该 bean。目标 bean首先会被完全创建，然后再应用比如带有拦截器链的 AOP 代理. 因此，将拦截器应用于 init 方法是不一致的，因为这样做会将目标 bean 的生命周期耦合到其代理或拦截器，并在代码直接与目标 bean 交互时留下奇怪的语义。

#### Combining Lifecycle Mechanisms

若为同一个 bean 配置的多个生命周期机制,并具有不同的初始化方法，这些初始化方法将按下面顺序被调用：
1. methods annotated with @PostConstruct
2. afterPropertiesSet() as defined by the InitializingBean callback interface
3. A custom configured init() method
Destroy 方法以相同的顺序调用.

#### Startup and Shutdown Callbacks

Lifecycle 接口为任何具有自己特定的生命周期需求的对象（例如启动和停止某些后台进程）定义了基本方法：
```java
public interface Lifecycle {

    void start();

    void stop();

    boolean isRunning();
}
```

任何 Spring 管理的对象都可以实现 *Lifecycle* 接口。 然后，当 *ApplicationContext* 本身接收到启动和停止信号时（例如，对于运行时的停止 / 重启场景），它会将信号级联到该context中定义的所有*Lifecycle*实现,以调用方法中制定的逻辑。 它通过委托 *LifecycleProcessor* 完成此操作，如下面所示：
```java
public interface LifecycleProcessor extends Lifecycle {

    void onRefresh();

    void onClose();
}
```
请注意，LifecycleProcessor 本身是 Lifecycle 接口的扩展。 它还添加了另外两种方法来响应刷新和关闭的上下文。

启动和关闭调用的顺序非常重要。 如果任何两个对象之间存在 “depends-on” 关系，则依赖方在其依赖之后开始，并且在其依赖之前停止。 但是，有时，直接依赖性是未知的。 您可能只知道某种类型的对象应该在另一种类型的对象之前开始。 在这些情况下，*SmartLifecycle* 接口定义了另一个选项，即在其父接口上定义的 getPhase（）方法。 以下清单显示了 Phased 接口的定义：
```java
public interface Phased {
    int getPhase();
}
```
以下清单显示了 SmartLifecycle 接口的定义：
```java
public interface SmartLifecycle extends Lifecycle, Phased {

    boolean isAutoStartup();

    void stop(Runnable callback);
}
```
启动时，回调从具有最低phase的对象首先开始。停止时，遵循相反的顺序。因此，实现 SmartLifecycle 并且其 getPhase（）方法返回 Integer.MIN_VALUE 的对象将是第一个开始和最后一个停止的对象。在另一端，Integer.MAX_VALUE 的阶段值将指示对象应该最后启动并首先停止（可能是因为它依赖于正在运行的其他进程）。在考虑phase值时，要知道任何未实现 SmartLifecycle 的 “正常” Lifecycle 对象的默认阶段为 0. 因此，任何负相位值都表示对象应该在这些标准组件之前启动（并停止在他们之后）。任何正相值都是相反的。

*SmartLifecycle* 定义的 stop 方法接受回调。在任何SmartLifecycle的实现类的关闭过程完成之后，该实现都必须调用该回调的 run（）方法。这使得可以在必要时启用异步关闭，因为 LifecycleProcessor 接口的默认实现 DefaultLifecycleProcessor, 等待超过每个phase的对象组的超时值,来调用该回调。默认的每阶段超时为 30 秒。您可以通过在上下文中定义名为 lifecycleProcessor 的 bean 来覆盖缺省生命周期处理器实例。

如前所述，*LifecycleProcessor* 接口还定义了用于refreshing和closing上下文的回调方法。 后者驱动关闭过程，就像显式调用了 stop（）一样，但是其是当上下文关闭时发生。 另一方面，'refresh' 回调启用了 SmartLifecycle bean 的另一个功能。 刷新上下文时（在实例化并初始化所有对象之后），将调用该回调。 此时，默认 的lifecycle processor检查每个 SmartLifecycle 对象的 isAutoStartup（）方法返回的布尔值。 如果为 true，则在该点启动该对象，而不是waiting for an explicit invocation of the context’s or its ownstart() method。 如前所述，phase值和任何 “depend-on” 关系确定启动顺序。

#### Shutting Down the Spring IoC Container Gracefully in Non-Web Applications

首先,本节仅适用于非 Web 应用程序。 Spring 的基于 Web 的 ApplicationContext 实现已经具有代码，可以在相关 Web 应用程序关闭时优雅地关闭 Spring IoC 容器。

如果在非 Web 应用程序环境中使用 Spring 的 IoC 容器，请给JVM注册一个 shutdown hook。 这样做可确保优雅关闭并在单例 bean 上调用相关的 destroy 方法，以便释放所有资源。 您仍然必须正确配置和实现这些 destroy 回调。

要注册shutdown hook，请调用 ConfigurableApplicationContext 接口上声明的 registerShutdownHook（）方法，如以下示例所示：
```java
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");

        // add a shutdown hook for the above context...
        ctx.registerShutdownHook();

        // app runs here...

        // main method exits, hook is called prior to the app shutting down...
    }
}
```

### 1.6.2.ApplicationContextAware and BeanNameAware

当 ApplicationContext 创建实现 org.springframework.context.ApplicationContextAware 接口的对象时，将为该实例提供一个对该 ApplicationContext 的引用。 以下清单显示了 ApplicationContextAware 接口的定义：
```
public interface ApplicationContextAware {

    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}
```

因此，bean 可以通过 ApplicationContext 接口以编程方式操作创建它们的 ApplicationContext。一个使用场景是对其他 bean 进行编程检索。有时这很有用。但是，一般情况下，您应该避免使用它，因为它将代码耦合到 Spring 并且不遵循IOC模式(::其中协作者作为属性提供给 bean::)。 ApplicationContext 的其他方法可以提供对文件资源的访问，发布应用程序事件和访问 MessageSource。这些附加功能在 [Additional Capabilities of theApplicationContext](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#context-introduction) 中描述。

从 Spring 2.5 开始，自动装配是另一种获取 ApplicationContext 引用的替代方法。 “传统” 构造函数和 byType 自动装配模式可以分别为构造函数参数或 setter 方法参数提供 ApplicationContext 类型的依赖关系。如果这样做，对带有 @Autowired 的字段，构造函数参数或方法参数,ApplicationContext 将被自动装入。有关更多信息，请参阅[Using@Autowired](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-autowired-annotation)。

当 ApplicationContext 创建实现 org.springframework.beans.factory.BeanNameAware 接口的类时，该类将提供对其关联对象的名称的引用。以下清单显示了 BeanNameAware 接口的定义：
```java
public interface BeanNameAware {
    void setBeanName(String name) throws BeansException;
}
```

### 1.6.3. Other Aware Interfaces

除了 ApplicationContextAware 和 BeanNameAware（前面讨论过）之外，Spring 还提供了大量的 Aware 回调接口，::让 bean 向容器指示它们需要某种基础结构依赖性::。 作为一般规则，名称表示依赖类型。 下表总结了最重要的一些 Aware 接口：
![](Spring%20Core/DFB35E25-8DB3-49F8-8337-660181E69858.png)
请再次注意，使用这些接口会将您的代码绑定到 Spring API，而不遵循 Inversion of Control 样式。 因此，我们建议仅将它们用于需要以编程方式访问容器的基础架构 bean。

- - - -
## 1.7. Bean Definition Inheritance

bean definitons 中以包含许多配置信息，包括构造函数参数，属性值和容器相关的信息，例如初始化方法，静态工厂方法名称等。::子 bean 定义从父定义继承这些配置数据。子定义可以覆盖某些值或根据需要添加其他值::。使用父 bean 和子 bean 定义可以节省大量的输入。实际上，这是一种templating形式。

如果以编程方式使用 ApplicationContext 接口，则子 bean 定义将由 ChildBeanDefinition 类表示。大多数用户不在此级别上使用它们。相反，它们在类（如 ClassPathXmlApplicationContext）中以声明式地配置 bean definition。使用基于 XML 的配置元数据时，可以使用 parent 属性指定子 bean definition，将父 bean 指定为此属性的值。以下示例显示了如何执行此操作：
```xml
<bean id="inheritedTestBean" abstract="true"
        class="org.springframework.beans.TestBean">
    <property name="name" value="parent"/>
    <property name="age" value="1"/>
</bean>

<bean id="inheritsWithDifferentClass"
        class="org.springframework.beans.DerivedTestBean"
        parent="inheritedTestBean" 
		   init-method="initialize">  
    <property name="name" value="override"/>
    <!-- the age property value of 1 will be inherited from parent -->
</bean>
```

如果没有指定，则子bean 定义使用父bean的定义 ，但也可以覆盖它。 在后一种情况下，子 bean 类必须与父类兼容（即，它必须接受父类的属性值）。

子 bean 定义从父级继承Scopr，构造函数参数值，属性值,并可以覆盖方法以添加新值。 您指定的任何scope, initialization method, destroy method, orstaticfactory method settings都会覆盖父bean 中的设置。

有些设置始终取自子定义：depends on, autowire mode, dependency check, singleton, and lazy init.

- - - -
## 1.8. Container Extension Points
通常，应用程序开发人员不需要继承ApplicationContext实现类。相反，可以通过插入特殊集成接口的实现来扩展Spring IoC容器。接下来的几节将介绍这些集成接口。

### 1.8.1. Customizing Beans by Using a BeanPostProcessor

BeanPostProcessor也称为 Bean 后置处理器，它是 Spring 中定义的接口，在 Spring 容器的创建过程中（具体为 Bean 初始化前后）会回调BeanPostProcessor中定义的两个方法。
```java
public interface BeanPostProcessor {

    Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
   
    Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
}
```

其中,postProcessBeforeInitialization## 方法会在每一个 bean 对象的初始化方法调用之前回调；
postProcessAfterInitialization## 方法会在每个 bean 对象的初始化方法调用之后被回调

BeanPostProcessor接口定义了可以实现的回调方法，以提供您自己的实例化逻辑，依赖关系解析逻辑等。如果像要在Spring容器完成实例化，配置和初始化bean之后实现某些自定义逻辑，则可以插入一个或多个自定义BeanPostProcessor实现。

您可以配置多个BeanPostProcessor实例，并且可以通过设置order属性来控制这些BeanPostProcessor实例的执行顺序。仅当BeanPostProcessor实现Ordered接口时，才能设置此属性。食谱哦如果编写自己的BeanPostProcessor，则应考虑实现Ordered接口。有关更多详细信息，请参阅 [programmatic registration of BeanPostProcessor instances](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-programmatically-registering-beanpostprocessors) .。

::我们可以看到BeanPostProcessor中的postProcessBeforeInitialization 方法是在所有的 bean 的 InitializingBean 的 afterPropertiesSet 方法之前执行而 postProcessAfterInitialization 方法则是在所有的 bean 的 InitializingBean 的 afterPropertiesSet 方法之后执行的。::

后处理器可以对bean实例执行任何操作，包括完全忽略回调。 后处理器通常会检查回调接口，或者把 bean 使用代理包装一下。一些Spring AOP基础结构类就实现为bean后处理器，以便提供代理包装逻辑。 

举个例子,*ApplicationContextAwareProcessor* 后置处理器的作用是，当应用程序定义的 Bean 实现ApplicationContextAware 接口时,为其注入ApplicationContext对象。

ApplicationContext自动检测配置元数据中实现BeanPostProcessor接口的任何bean。 ApplicationContext将这些bean注册为 后处理器，以便在创建bean时可以稍后调用它们。 Bean后处理器可以以与任何其他bean相同的方式部署在容器中。

以下示例显示如何在ApplicationContext中编写，注册和使用BeanPostProcessor实例。该示例显示了一个自定义的BeanPostProcessor实现，该实现在容器创建时调用每个bean的toString（）方法，并将生成的字符串输出到系统控制台。以下清单显示了自定义BeanPostProcessor实现类定义：
```java
package scripting;

import org.springframework.beans.factory.config.BeanPostProcessor;

public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {

    // simply return the instantiated bean as-is
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        return bean; // we could potentially return any object reference here...
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) {
        System.out.println("Bean '" + beanName + "' created : " + bean.toString());
        return bean;
    }
}
```

The following Java application runs the preceding code and configuration:
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.scripting.Messenger;

public final class Boot {

    public static void main(final String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("scripting/beans.xml");
        Messenger messenger = (Messenger) ctx.getBean("messenger");
        System.out.println(messenger);
    }

}
```
上面的方法获取了一个Messenger bean 并打印. 上述应用程序的输出类似于以下内容：
```
Bean 'messenger' created : org.springframework.scripting.groovy.GroovyMessenger@272961
org.springframework.scripting.groovy.GroovyMessenger@272961
```

### 1.8.2. Customizing Configuration Metadata with a BeanFactoryPostProcessor

我们看到的下一个扩展点是BeanFactoryPostProcessor。此接口的语义类似于BeanPostProcessor的语义，但有一个主要区别：BeanFactoryPostProcessor是对bean配置元数据进行操作。也就是说，Spring IoC容器允许BeanFactoryPostProcessor读取配置元数据，并可能在容器实例化任何bean之前更改元数据。

postProcessBeanFactory 可以在 BeanFactory 完成实例化后,修改容器内部的 BeanFactory(即配置)。这时候所有的 bean 都被加载，但是还没有 bean 被初始化。这就允许 BeanFactoryPostProcessor 重写或者添加配置，甚至可以提前初始化 bean。

您可以配置多个BeanFactoryPostProcessor实例，并且可以通过设置order属性来控制这些BeanFactoryPostProcessor实例的运行顺序。但是，只有当BeanFactoryPostProcessor实现Ordered接口，你才能设置此属性。如果编写自己的BeanFactoryPostProcessor，则应考虑实现Ordered接口。

bean factory post-processor在ApplicationContext中声明时,就 能自动执行，以便将更改应用于配置元数据。 Spring包含许多预定义的bean factory post-processor，例如 PropertyOverrideConfigurer 和 PropertyPlaceholderConfigurer。您还可以使用自定义BeanFactoryPostProcessor, 例如register custom property editors。下面说一个例子:

您可以使用*PropertyPlaceholderConfigurer*来通过使用标准Java Properties格式,从单独文件中来externalize property values from a bean definition。这样做可以使部署应用程序的人员可以定制特定于环境的一些属性，例如数据库URL和密码.

请考虑以下基于XML的配置元数据片段，其中定义了具有placeholder的DataSource：
```xml
<bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations" value="classpath:com/something/jdbc.properties"/>
</bean>

<bean id="dataSource" destroy-method="close"
        class="org.apache.commons.dbcp.BasicDataSource">
    <property name="driverClassName" value="${jdbc.driverClassName}"/>
    <property name="url" value="${jdbc.url}"/>// 这里有占位符的值
    <property name="username" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
</bean>
```

该示例显示了从外部Properties文件配置的属性。在运行时，PropertyPlaceholderConfigurer将被应用,以替换DataSource的某些属性的元数据。要替换的值的格式是${property-name}形式。

实际值来自标准Java Properties格式的另一个文件：
```JAVA
jdbc.driverClassName=org.hsqldb.jdbcDriver
jdbc.url=jdbc:hsqldb:hsql://production:9002
jdbc.username=sa
jdbc.password=root
```

因此，${jdbc.username}字符串在运行时将被替换为值'sa’.  *PropertyPlaceholderConfigurer*检查bean定义中大多数properties和attributes中的占位符。此外，您还可以自定义占位符前缀和后缀。

*PropertyPlaceholderConfigurer*不仅在您指定的Properties文件中查找属性。默认情况下，如果它在指定的属性文件中找不到属性，它还会检查Java System属性。您可以配置configurer的*systemPropertiesMode*属性的不同 value来自定义此行为：
* never（0）：从不检查系统属性。
* fallback（1）：如果在指定的属性文件中无法解析，则检查系统属性。这是默认值。
* override（2）：在尝试指定的属性文件之前，首先检查系统属性。这使系统属性可以覆盖任何其他属性源。


### 1.8.3. Customizing Instantiation Logic with a FactoryBean

FactoryBean：是一个 Java Bean，但是它是一个能生产对象的工厂 Bean. 一般情况下，Spring 通过反射机制利用 <bean> 的 class 属性指定实现类实例化 Bean，::在某些情况下，实例化 Bean 过程比较复杂::，如果按照传统的方式，则需要在 < bean > 中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring 为此提供了一个 FactoryBean 的工厂类接口，通常是用来创建比较复杂的 bean,用户可以通过实现该接口定制实例化 Bean 的逻辑。

您可以为本身为工厂的对象实现FactoryBean接口。FactoryBean接口为Spring IoC容器实例化逻辑提供了pluggability。如果您有更复杂的初始化代码，这些代码在Java中更好地表达，而不是（可能）冗长的XML，您可以创建自己的FactoryBean，在该类中编写复杂的初始化逻辑，然后将自定义FactoryBean插入容器中。

FactoryBean接口提供了三种方法：
* Object getObject（）：返回此工厂创建的对象的实例。可以是共享实例，具体取决于此工厂是返回单例还是原型。
* boolean isSingleton（）：如果此FactoryBean返回单例，则返回true，否则返回false。
* Class getObjectType（）：返回getObject（）方法返回的对象类型，如果事先不知道类型，则返回null。

FactoryBean概念和接口用于Spring Framework中的许多地方。 FactoryBean接口包含了大约50多个实现。

当您需要向容器索要实际的FactoryBean实例本身,而不是它生成的bean时，在调用ApplicationContext的getBean（）方法时，使用＆符号（＆）作为bean的id前缀。因此，对于id为myBean的FactoryBean，::在容器上调用getBean（“myBean”）将返回FactoryBean的产品::，而调用getBean（“＆myBean”）则返回FactoryBean实例本身。

参见[How to use the Spring FactoryBean? | Baeldung](https://www.baeldung.com/spring-factorybean)
- - - -
## 1.9. Annotation-based Container Configuration
基于注解的配置提供了基于XML配置的替代方案，该配置依赖于字节码元数据来连接组件而不是角括号声明。开发人员不是使用XML来描述bean连接，而是通过在相关的类，方法或字段声明上使用注解将配置移动到组件类本身。

例如，从本质上讲，@Autowired注解提供的功能与 [Autowiring Collaborators](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-factory-autowire) 中描述的相同，但具有更细粒度的控制和更广泛的适用性。 Spring 2.5还增加了对JSR-250注解的支持,比如 @PostConstruct和@PreDestroy。 Spring 3.0增加了对javax.inject包中包含的JSR-330（Java的依赖注入）注解的支持，例如@Inject和@Named。

与往常一样，您可以将它们注册为单独的bean definition，但也可以通过在基于XML的Spring配置中包含以下标记来隐式注册它们
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

</beans>
```
在此隐式注册的post-processors包括 *AutowiredAnnotationBeanPostProcessor*，*CommonAnnotationBeanPostProcessor*，*PersistenceAnnotationBeanPostProcessor*和前面提到的*RequiredAnnotationBeanPostProcessor*。）

### 1.9.1. @Required

@Required注解适用于bean属性的setter方法，如下例所示：
```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

此注解表示必须这个 bean property(这里即movieFinder)必须在配置时通过bean definition中的显式属性值,或通过自动装配来填充。

如果没有填充这个bean property，::则容器将抛出异常::。这允许eager and explicit failure，避免了之后出现NullPointerException。我们仍然建议您将断言放入bean类本身（例如，转换为init方法）。当使用@Required,即使您在容器外部使用类，这也会enforces那些必需的引用和值。其实,@Required注解从Spring Framework 5.1开始正式过期，这有利于使用构造函数注入所需的配置。

### 1.9.2. Using@Autowired

在本节所包含的示例中，也可以使用JSR 330的@Inject注解代替Spring的@Autowired注解。有关详细信息，请参见 [here](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-standard-annotations) 

您可以将@Autowired注解应用于构造函数，如以下示例所示：
```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

从Spring Framework 4.3开始，如果目标bean只定义了一个构造函数，则不再需要在这样的构造函数上使用@Autowired注解。但是，如果有几个构造器可用，则必须注解至少一个构造器以告诉容器使用哪一个。

您还可以将@Autowired注解应用于“传统”setter方法，如以下示例所示：
```JAVA
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

您还可以将注解应用于具有任意名称和任意bean 作为参数的方法，如以下示例所示：
```JAVA
public class MovieRecommender {

    private MovieCatalog movieCatalog;

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您也可以将@Autowired应用于字段，甚至将其与在构造函数上使用的方式进行混合，如以下示例所示：
```java
public class MovieRecommender {

    private final CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    private MovieCatalog movieCatalog;

    @Autowired
    public MovieRecommender(CustomerPreferenceDao customerPreferenceDao) {
        this.customerPreferenceDao = customerPreferenceDao;
    }

    // ...
}
```

您还可以通过将注解添加到需要该类型数组的filed或method，这将从ApplicationContext获取特定类型的所有bean，如以下示例所示：
```java
public class MovieRecommender {
    @Autowired
    private MovieCatalog[] movieCatalogs;

    // ...
}
```

这同样适用于泛型集合，如以下示例所示：
```java
public class MovieRecommender {

    private Set<MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Set<MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

只要预期的key类型是String，即使是类型化的Map实例也可以自动装配。 Map值包含所有期望类型的bean，并且键包含相应的bean名称，如以下示例所示：
```java
public class MovieRecommender {

    private Map<String, MovieCatalog> movieCatalogs;

    @Autowired
    public void setMovieCatalogs(Map<String, MovieCatalog> movieCatalogs) {
        this.movieCatalogs = movieCatalogs;
    }

    // ...
}
```

默认情况下，当给定的注入点并没有匹配的候选bean时，自动装配就失败了。对于声明的数组，集合或映射，至少需要一个匹配元素。

默认行为是将带了这个注解的方法和字段里面的依赖视为required 的依赖项。您可以更改此行为，如以下示例所示，能够将其标记为非必需来**跳过**无法满足的注入点：
```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Autowired(required = false)
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

如果依赖对象不可用,则标记了non-required的方法根本不会被调用。在这种情况下，也就根本不会拿依赖对象去填充这个非必须的字段，保留其默认值(对于 int 就是0这样)。

然而,注入构造函数和工厂方法参数是一种特殊情况，因为@Autowired上的'required'标志,因为Spring的构造函数解析可能涉及多个构造函数,而具有一些不同的含义。默认行为下,构造函数和工厂方法参数实际上都是 required 的，但在single-constructor scenario中有一些特殊规则，例如，如果没有匹配的bean可用，则解析为空实例的多元素注入点（数组，集合，映射）。这允许一种通用的实现模式，其中所有依赖关系都可以在唯一的多参数构造函数中声明，例如，声明为没有@Autowired注解的单个公共构造函数。

> 任何给定bean类中,只能有一个构造函数可以声明@Autowired，并将required属性设置为true，表示构造函数在用作Spring bean时要自动装配。  
>   
> 此外，如果required属性设置为true，也只有一个构造函数能使用@Autowired注解。如果多个non-reuqired构造函数声明了注解，则它们将被视为自动装配的候选者。具有最大数量的依赖项的构造函数， 并且依赖可以通过匹配 bean来满足的那个将被选择。如果不能满足任何候选者，则将使用primary/default的构造函数。如果一个类只声明一个构造函数开头，它将始终被使用，即使没有注解。带注解的构造函数不必须是公共的。  

从Spring Framework 5.0开始，您还可以使用@Nullable注解还表示 non-required 的语义:
```java
public class SimpleMovieLister {
    @Autowired
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

您还可以将@Autowired用于众所周知的某些的接口：*BeanFactory，ApplicationContext，Environment，ResourceLoader，ApplicationEventPublisher和MessageSource*。这些接口及其扩展接口（如ConfigurableApplicationContext或ResourcePatternResolver）不需要你在app 里创建 bean 什么的,spring 会自动解析给你注入进来。以下示例自动装配ApplicationContext对象：
```java
public class MovieRecommender {
    @Autowired
    private ApplicationContext context;

    public MovieRecommender() {
    }

    // ...
}
```

注意, @Autowired，@Inject，@ Value和@Resource注解由Spring BeanPostProcessor来处理。这意味着您无法在自己写的的BeanPostProcessor或BeanFactoryPostProcessor实现中应用这些注解。只能使用XML或Spring @Bean方法显式地“连接”这些类型。

### 1.9.3. Fine-tuning Annotation-based Autowiring with @Primary

由于按type自动装配可能会导致多个候选人，因此通常需要对选择过程进行更多控制。实现这一目标的一种方法是使用Spring的@Primary注解。 @Primary指示当多个bean可以自动装配到单值依赖项时，应该优先选择特定的bean。如果候选者中只存在一个主bean，则它将成为自动装配的值。

请考虑以下配置，将firstMovieCatalog定义为主要MovieCatalog：
```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...
}
```

使用上述配置，以下MovieRecommender将与firstMovieCatalog一起自动装配：
```java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...
}
```

### 1.9.4. Fine-tuning Annotation-based Autowiring with Qualifiers

@Primary是一种有效的方法，可以在确定一个主要候选者的情况下按类型使用自动装配。单当您需要更多控制的选择时，可以使用Spring的@Qualifier注解。您可以将qualifier values与特定参数相关联，缩小类型匹配集，以便为每个参数选择特定的bean。在最简单的情况下，这可以是一个简单的描述性值，如以下示例所示：
```JAVA
public class MovieRecommender {
    @Autowired
    @Qualifier("main")
    private MovieCatalog movieCatalog;

    // ...
}
```

您还可以在各个构造函数参数或方法参数上指定@Qualifier注解，如以下示例所示：
```java
public class MovieRecommender {

    private MovieCatalog movieCatalog;//定义 bean 的时候为他指定了 Qualifier

    private CustomerPreferenceDao customerPreferenceDao;

    @Autowired
    public void prepare(@Qualifier("main")MovieCatalog movieCatalog,
            CustomerPreferenceDao customerPreferenceDao) {
        this.movieCatalog = movieCatalog;
        this.customerPreferenceDao = customerPreferenceDao;
    }
    // ...
}
```

bean名称被视为默认的Qualifier。因此，您可以使用id而不是嵌套的限定符元素定义bean，也可以得到相同的匹配结果。但是，虽然您可以使用此约定来按名称引用特定bean，但@Autowired基本上是一个”具有可选qualifier”的基于type的注入。这意味着即使可以使用bean名称来匹配，qualifier values在type matches中也总是具有比较窄语义。它们在语义上不表示对唯一bean id的引用。良好的限定符值的例子是main或EMEA或persistent这种，表示了不同于bean id的特定组件的行为特征。

限定符也适用于类型化集合，如前所述 - 例如，Set <MovieCatalog>。在这种情况下，根据声明的限定符，所有匹配的bean都作为集合注入。::这意味着限定符不必是Unique的::。相反，它们是构成了过滤标准。例如，您可以使用相同的限定符值“action”来定义多个MovieCatalog bean，所有这些bean都注入到使用@Qualifier（“action”）注解的Set <MovieCatalog>中。

另外,如果您真打算按bean name表达注入，请不要主要使用 @Autowired，即使它能够在类型匹配的中通过 bean 名称进行选择。相反，使用 JSR-250 @Resource 注解，该注解在语义上定义为通过其唯一名称标识特定目标组件，声明的类型与匹配过程无关。

您可以创建自己定制的qualifier注解。 为此，请定义注解,并在定义中加上 @Qualifier 注解，如以下示例所示：
```java
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Genre {
    String value();
}
```

然后，您可以在自动装配的字段和参数上提供自定义的qualifier，如以下示例所示：
```java
public class MovieRecommender {

    @Autowired
    @Genre("Action")
    private MovieCatalog actionCatalog;

    private MovieCatalog comedyCatalog;

    @Autowired
    public void setComedyCatalog(@Genre("Comedy") MovieCatalog comedyCatalog) {
        this.comedyCatalog = comedyCatalog;
    }

    // ...
}
```

接下来，您定义一下提供候选bean的信息。您可以将<qualifier />标记添加为<bean />标记的子元素，然后指定type和value与自定义的qualifier匹配。type与注解的fully-qualified class name匹配。或者，为方便起见，如果不存在冲突名称的风险，您可以使用短类名称。以下示例演示了这两种方法：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:annotation-config/>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="Genre" value="Action"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean class="example.SimpleMovieCatalog">
        <qualifier type="example.Genre" value="Comedy"/>
        <!-- inject any dependencies required by this bean -->
    </bean>

    <bean id="movieRecommender" class="example.MovieRecommender"/>

</beans>
```

在 [Classpath Scanning and Managed Components](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-classpath-scanning) 中，您可以看到基于注解的替代方法来在XML中提供qualifier的元数据。具体来说，请参阅 [Providing Qualifier Metadata with Annotations](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-scanning-qualifiers) .。

### 1.9.5. Using Generics as Autowiring Qualifiers

除了@Qualifier注解之外，您还可以使用Java泛型类型作为隐式的qualification。例如，假设您具有以下配置：
```java
@Configuration
public class MyConfiguration {

    @Bean
    public StringStore stringStore() {
        return new StringStore();
    }

    @Bean
    public IntegerStore integerStore() {
        return new IntegerStore();
    }
}
```

假设前面的bean分别实现了泛型接口（即Store <String>和Store <Integer>），您可以@Autowire Store接口，泛型将被用作限定符，如下例所示：
```JAVA
@Autowired
private Store<String> s1; // <String> qualifier, injects the stringStore bean

@Autowired
private Store<Integer> s2; // <Integer> qualifier, injects the integerStore bean
```

通用限定符也适用于自动装配List，Map实例和Array。以下示例自动装配泛型List：
```java
// Inject all Store beans as long as they have an <Integer> generic
// Store<String> beans will not appear in this list
@Autowired
private List<Store<Integer>> s;
```

### 1.9.6. Using CustomAutowireConfigurer

*CustomAutowireConfigurer*是一个*BeanFactoryPostProcessor*(操作配置元数据用的)，它允许您注册自己的自定义的qualifier注解，即使这个注解的定义中没有使用Spring的@Qualifier注解进行注解。以下示例显示如何使用CustomAutowireConfigurer：
```java
<bean id="customAutowireConfigurer"
        class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
    <property name="customQualifierTypes">
        <set>
            <value>example.CustomQualifier</value>
        </set>
    </property>
</bean>
```
你只需要注册一个CustomAutowireConfigurer的 bean,然后在customQualifierTypes这个 filed 中加入自定义的 Qualifier

AutowireCandidateResolver通过以下方式确定autowire候选者：
* 每个bean定义中的autowire-candidate值
* <beans />元素上可用的任何default-autowire-candidates模式
* 存在@Qualifier注解,以及使用CustomAutowireConfigurer注册的任何自定义注解.

当多个bean有资格作为autowire候选者时，“primary”的确定如下：如果候选者中只有一个bean定义的primary属性值设置为true，则选择它。

### 1.9.7. Injection with @Resource

Spring还支持通过对字段或setter方法使用@Resource注解（javax.annotation.Resource）来做DI。 @Resource有一个name属性。默认情况下，Spring认为这个值是要注入的bean名称。换句话说，它遵循by-name语义，如以下示例所示：
```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource(name="myMovieFinder") 
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

如果未明确指定名称，则默认名称是从字段名称或setter方法派生的。如果是字段，则采用字段名称。在setter方法的情况下，它采用bean属性名称。下面的例子将把名为movieFinder的bean注入setter方法：
```java
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

在没有指定显式name的特殊情况下，类似于@Autowired, @Resource会去找primary type matchinstead of a specific named bean，并可以解析众所周知的可解析依赖项：BeanFactory，ApplicationContext，ResourceLoader，ApplicationEventPublisher和MessageSource接口。

因此，在以下示例中，customerPreferenceDao field首先查找名为“customerPreferenceDao”的bean，如果没找到,泽辉找CustomerPreferenceDao类型的bean：
```JAVA
public class MovieRecommender {

    @Resource
    private CustomerPreferenceDao customerPreferenceDao;

    @Resource
    private ApplicationContext context; 

    public MovieRecommender() {
    }

    // ...
}
```

### 1.9.8. Using @PostConstruct and @PreDestroy

*CommonAnnotationBeanPostProcessor*不仅识别@Resource注解，还识别JSR-250中的生命周期注解：@PostConstruct和@PreDestroy。在Spring 2.5中引入，对这些注解的支持提供了 [initialization callbacks](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) 中方法的替代方法。如果CommonAnnotationBeanPostProcessor在Spring ApplicationContext中注册了，做了这两个注解的方法,会在与Spring lifecycle interface method or explicitly declared callback method的同一点调用点被调用。在以下示例中，缓存在初始化时预填充并在销毁时清除：
```JAVA
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // populates the movie cache upon initialization...
    }

    @PreDestroy
    public void clearMovieCache() {
        // clears the movie cache upon destruction...
    }
}
```

- - - -
## 1.10. Classpath Scanning and Managed Components

本章中的大多数示例都使用XML来指定用来在Spring容器中生成BeanDefinition的配置元数据。上一节演示了如何通过注解提供大量配置元数据。但是，即使在这些示例中，并未提到“基本”bean定义，注解仅用于了DI。

本节介绍通过扫描类路径来隐式检测component candidate。候选组件是与筛选条件匹配的类，并且向容器注册了相应的bean definition。这消除了使用XML执行bean注册的必要。您可以使用注解（例如，@Component），AspectJ类型表达式或您的自定义筛选条件来选择哪些类向容器注册bean定义。

### 1.10.1.@Component and Further Stereotype Annotations

@Repository注解用于任何作为存储库角色（也称为数据访问对象或DAO）的类。此标记的用法之一是异常的自动转换，如 [Exception Translation](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/data-access.html#orm-exception-translation) .中所述。 

Spring提供了进一步的构造型注解：@Component，@Service和@Controller。 
* @Component是任何Spring管理组件的通用构造型。 
* @Repository，@ Service和@Controller是@Component的特殊化，用于更具体的用例（分别在持久性，服务和表示层中）。

您可以使用@Component注解组件类，但是，通过使用@ Repository，@ Service或@Controller注解，您的类会更适合通过工具进行处理或与AOP关联。例如，these stereotype annotations make ideal targets for pointcuts。 @Repository，@Service和@Controller也可以在以后的版本中携带额外的语义。因此，如果您选择在服务层使用@Component或@Service，@Service显然是更好的选择。同样，::如前所述，@Repository已经支持在持久层中做自动异常转换。:: 

### 1.10.2. Using Meta-annotations and Composed Annotations

Spring提供的许多注解都可以在您自己的代码中用作元注解。元注解是可以应用于另一个注解的注解。例如，前面提到的@Service注解是使用@Component进行元注解的.

此外，组合注解可以选择从元注解重新声明属性以进行自定义。当您只想公开元注解属性的子集时，这可能特别有用。例如，Spring的@SessionScope注解将scope name硬编码为”Session”，但仍允许自定义Scope 中的proxyMode。以下清单显示了SessionScope注解的定义：
```JAVA
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Scope(WebApplicationContext.SCOPE_SESSION)
public @interface SessionScope {

    /**
     * Alias for {@link Scope#proxyMode}.
     * <p>Defaults to {@link ScopedProxyMode#TARGET_CLASS}.
     */
    @AliasFor(annotation = Scope.class)
    ScopedProxyMode proxyMode() default ScopedProxyMode.TARGET_CLASS;

}
```

然后，您可以使用@SessionScope而不声明proxyMode，如下所示：
```java
@Service
@SessionScope
public class SessionScopedService {
    // ...
}
```

### 1.10.3. Automatically Detecting Classes and Registering Bean Definitions

Spring可以自动检测并使用ApplicationContext注册相应的BeanDefinition实例。例如，以下两个类符合此类自动检测的条件：
```java
@Service
public class SimpleMovieLister {
    private MovieFinder movieFinder;
    @Autowired
    public SimpleMovieLister(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}

@Repository
public class JpaMovieFinder implements MovieFinder {
    // implementation elided for clarity
}
```

要自动检测这些类并注册相应的bean，需要将@ComponentScan添加到@Configuration的类(@Configuration本身也是一个 @Component)上，其中basePackages属性是两个类的公共父包。
```java
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

> @Comfiguration Indicates that the class declares one or more  [@Bean](file:///Users/zhengpeiwei/Library/Application%20Support/Dash/DocSets/Spring_Framework/Spring%20Framework.docset/Contents/Resources/Documents/org/springframework/context/annotation/Bean.html)  methods and may be processed by the Spring container to generate bean definitions and service requests for those beans at runtime, for example:  

此外，使用component-scan元素时，将隐式让*AutowiredAnnotationBeanPostProcessor*和*CommonAnnotationBeanPostProcessor*这两个 bean 出现在上下文中。这意味着,这两个组件是自动检测并wired 的 - 无需提供任何bean配置元数据。

### 1.10.4. Using Filters to Customize Scanning

默认情况下，使用@Component，@ Repository，@ Service，@ Controller注解的类或自身使用@Component注解的自定义注解是唯一检测到的候选组件。但是，您可以通过应用自定义filter来修改和扩展此行为。可以为@ComponentScan注解添加includeFilters或excludeFilters参数。每个filter元素都具有type和expression属性。下表介绍了筛选选项：
![](Spring%20Core/B5321F77-304C-4B18-A453-58AD95CBDEE3.png)
以下示例显示忽略所有@Repository注解并使用“stub”存储库的配置：
```java
@Configuration
@ComponentScan(basePackages = "org.example",
        includeFilters = @Filter(type = FilterType.REGEX, pattern = ".*Stub.*Repository"),
        excludeFilters = @Filter(Repository.class))
public class AppConfig {
    ...
}
```

### 1.10.5. Defining Bean Metadata within Components

Spring组件还可以向容器提供bean definition元数据。您可以使用用于在@Configuration注解的类中定义bean元数据,你只需要使用@Bean。以下示例显示了如何执行此操作：
```JAVA
@Configuration
public class FactoryMethodComponent {

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    public void doWork() {
        // Component method implementation omitted
    }
}
```

上面的类是一个Spring组件，在其doWork（）方法中具有特定于应用程序的代码。但是，它还提供了一个bean definition，也就是publicInstance的产出会被当做一个名为”public”的 bean。 @Bean注解识别工厂方法和其他bean definition属性，例如通过@Qualifier注解的限定符值。可以指定的其他方法级注解包括@Scope，@Lazy和自定义限定符注解。

> @Bean 的作用就是把方法的返回值注册为一个 bean.  

如前所述，支持自动装配的字段和方法，以及对@Bean方法的自动装配的额外支持。以下示例显示了如何执行此操作：
```JAVA
@Component
public class FactoryMethodComponent {

    private static int i;

    @Bean
    @Qualifier("public")
    public TestBean publicInstance() {
        return new TestBean("publicInstance");
    }

    // use of a custom qualifier and autowiring of method parameters
    @Bean
    protected TestBean protectedInstance(
            @Qualifier("public") TestBean spouse,
            @Value("#{privateInstance.age}") String country) {
        TestBean tb = new TestBean("protectedInstance", 1);
        tb.setSpouse(spouse);
        tb.setCountry(country);
        return tb;
    }

    @Bean
    private TestBean privateInstance() {
        return new TestBean("privateInstance", i++);
    }

    @Bean
    @RequestScope
    public TestBean requestScopedInstance() {
        return new TestBean("requestScopedInstance", 3);
    }
}
```

上面这个示例将方法中String参数: country自动装配为另一个名为privateInstance的bean上的age属性的值。 Spring Expression Language元素通过符号＃{<expression>}定义属性的值。对于@Value注解，表达式解析器已经预先配置,以在解析表达式文本时查找bean。

从Spring Framework 4.3开始，您还可以声明一个类型为InjectionPoint的工厂方法参数来访问触发当前bean构造的requesting injection point。请注意，这仅适用于actual creation of bean instances，而不适用于注入已存在的实例。因此，此功能对原型bean最有意义。在这种情况下，您可以使用提供的注入点元数据with seamatic care。以下示例显示了如何使用InjectionPoint:
```java
@Component
public class FactoryMethodComponent {

    @Bean @Scope("prototype")
    public TestBean prototypeInstance(InjectionPoint injectionPoint) {
        return new TestBean("prototypeInstance for " + injectionPoint.getMember());
    }
}
```

### 1.10.6. Naming Autodetected Components

当component作为扫描过程的一部分自动检测时，其bean名称由扫描程序的BeanNameGenerator策略生成。默认情况下，任何包含名称值的Spring构造型注解（@ Component，@ Repository，@ Service和@Controller）都会将该名称提供给相应的bean定义。

如果此类注解不包含name值或任何其他检测到的组件(such as those discovered by custom filters)，则默认bean生成为uncapitalized non-qualified class name。例如，如果检测到以下组件类，则名称将为myMovieLister和movieFinderImpl：
```java
@Service("myMovieLister")
public class SimpleMovieLister {
    // ...
}
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

作为一般规则，考虑其他component可能对其进行引用时,显式使用注解指定名称。另一方面，只要容器负责wiring，自动生成的名称也是足够的了。

### 1.10.7. Providing a Scope for Autodetected Components

与Spring管理的组件一样，自动检测组件的默认和最常见的范围是单例。但是，有时您需要一个可由@Scope注解指定的不同scope。您可以在注解中提供scope的名称，如以下示例所示：
```java
@Scope("prototype")
@Repository
public class MovieFinderImpl implements MovieFinder {
    // ...
}
```

有关只能Web使用的Scope（如Spring上下文中的“request”或“session”）的详细信息，请参阅 [Request, Session, Application, and WebSocket Scopes](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other) 。使用某些非sigleton作用域时，可能需要为作用域对象生成proxy,原因在[Scoped Beans as Dependencies](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection)进行了说明。为此，component-scan元素上提供了scoped-proxy属性。三个可能的值是：no，interfaces和targetClass。例如，以下配置将会导致使用标准JDK动态代理：
```java
@Configuration
@ComponentScan(basePackages = "org.example", scopedProxy = ScopedProxyMode.INTERFACES)
public class AppConfig {
    ...
}
```

### 1.10.8. Providing Qualifier Metadata with Annotations

在 [Fine-tuning Annotation-based Autowiring with Qualifiers](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-autowired-annotation-qualifiers) 已经讨论了@Qualifier注解。该部分中的示例演示了在解析auwowire candidates时使用@Qualifier注解和自定义限定符注解来提供细粒度控制。当依靠类路径扫描来自动检测组件时，可以在候选类上为class 级注解提供qualifier元数据。以下三个示例演示了此技术(在 class 上使用 Qualifier 来指定 component 的限定符值)：
```java
@Component
@Qualifier("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
@Component
@Genre("Action")
public class ActionMovieCatalog implements MovieCatalog {
    // ...
}
@Component
@Offline
public class CachingMovieCatalog implements MovieCatalog {
    // ...
}
```

### 1.10.9. Generating an Index of Candidate Components

虽然classpath scanning速度非常快，但是也是要扫描去发现有哪些 bean 的. 你可以通过在编译时创建静态的 candidate列表来提高大型应用程序的启动性能。在此模式下，所有作为组件扫描目标的模块都必须使用此机制。

要生成索引，请为包含组件扫描指令目标的每个模块都添加一个依赖项。以下示例显示了如何使用Maven执行此操作：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context-indexer</artifactId>
        <version>5.1.5.RELEASE</version>
        <optional>true</optional>
    </dependency>
</dependencies>
```
该过程生成包含在jar文件中的META-INF/spring.components文件。

- - - -
## 1.11. Using JSR 330 Standard Annotations

从Spring 3.0开始，Spring提供对JSR-330标准注解的支持。这些注解的扫描方式与Spring注解相同。要使用它们，您需要在类路径中包含相关的jar(当然,你可以不使用,使用 Spring 的那些就够了)。
```xml
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```

### 1.11.1. Dependency Injection with @Inject and @Named

您可以使用@ javax.inject.Inject代替@Autowired，如下所示：
```java
import javax.inject.Inject;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.findMovies(...);
        ...
    }
}
```

与@Autowired一样，您可以在字段级别，方法级别和构造函数参数级别使用@Inject。此外，您可以将injection point声明为Provider，这让你可以通过Provider.get（）进行on-demand access to beans of shorter scopes或对其他bean的延迟访问。以下示例提供了上述示例的变体：
```JAVA
public class SimpleMovieLister {

    private Provider<MovieFinder> movieFinder;

    @Inject
    public void setMovieFinder(Provider<MovieFinder> movieFinder) {
        this.movieFinder = movieFinder;
    }

    public void listMovies() {
        this.movieFinder.get().findMovies(...);
        ...
    }
}
```

如果要为应注入的依赖项使用限定名称，则可以使用@Named注解，如以下示例所示：
```JAVA
import javax.inject.Inject;
import javax.inject.Named;

public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(@Named("main") MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

与@Autowired一样，@Inject也可以与java.util.Optional或@Nullable一起使用。甚至这在这里更适用，因为@Inject没有required属性可以去设置。以下一对示例显示了如何使用@Inject和@Nullable：
```java
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(Optional<MovieFinder> movieFinder) {
        ...
    }
}
public class SimpleMovieLister {

    @Inject
    public void setMovieFinder(@Nullable MovieFinder movieFinder) {
        ...
    }
}
```

### 1.11.2.@Named and @ManagedBean: Standard Equivalents to the@ComponentAnnotation

您可以使用@javax.inject.Named或@javax.annotation.ManagedBean代替@Component，如以下示例所示：
```java
@Named("movieListener")  // @ManagedBean("movieListener") could be used as well
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

在不指定组件名称的情况下使用@Component是很常见的。 @Named可以以类似的方式使用，如下例所示：
```java
import javax.inject.Inject;
import javax.inject.Named;

@Named
public class SimpleMovieLister {

    private MovieFinder movieFinder;

    @Inject
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }

    // ...
}
```

使用@Named或@ManagedBean时，可以使用与使用Spring注解时完全相同的方式使用component scanning，如以下示例所示：
```
@Configuration
@ComponentScan(basePackages = "org.example")
public class AppConfig  {
    ...
}
```

### 1.11.3. Limitations of JSR-330 Standard Annotations

使用JSR-330注解时，您应该知道某些重要功能不可用，如下表所示：
![](Spring%20Core/7B5A3FD8-A348-4C87-9D9E-9396D31C1FEB.png)
![](Spring%20Core/92E41B2A-D6C4-4709-B654-3F18EAAB7763.png)

- - - -
## 1.12. Java-based Container Configuration

本节介绍如何在Java代码中使用注解来配置Spring容器。它包括以下主题：
*  [Basic Concepts:@Beanand@Configuration](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java-basic-concepts) 
*  [Instantiating the Spring Container by UsingAnnotationConfigApplicationContext](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java-instantiating-container) 
*  [Using the@BeanAnnotation](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java-bean-annotation) 
*  [Using the@Configurationannotation](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java-configuration-annotation) 
*  [Composing Java-based Configurations](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-java-composing-configuration-classes) 
*  [Bean Definition Profiles](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles) 
*  [PropertySourceAbstraction](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction) 
*  [Using@PropertySource](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-using-propertysource) 
*  [Placeholder Resolution in Statements](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-placeholder-resolution-in-statements) 

### 1.12.1. Basic Concepts:@Bean and @Configuration

Spring的Java配置的核心组件是@Configuration-annotated类和@ Bean-annotated方法。

@Bean注解用于指示方法是用来实例化，配置和初始化由Spring IoC容器管理的bean。@Bean注解扮演的角色与<bean />元素相同。您可以将@Bean-annotated方法与任何Spring @Component一起使用。但是，它们最常用于@Configuration bean之中。

使用@Configuration注解类表示其主要目的是作为bean定义的源。此外，@Configuration类允许通过调用同一个类中的其他@Bean方法来定义bean间依赖关系。最简单的@Configuration类如下所示：
```java
@Configuration
public class AppConfig {

    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}
```

@Bean和@Configuration注解将在以下部分中进行深入讨论。首先，我们将介绍使用基于Java的配置创建容器的各种方法。

### 1.12.2. Instantiating the Spring Container by Using AnnotationConfigApplicationContext

以下部分介绍了Spring的*AnnotationConfigApplicationContext*，它是在Spring 3.0中引入的。这个多功能的ApplicationContext实现不仅能够接受@Configuration类作为输入，还能接受使用JSR-330元数据注解的普通@Component类和类。

* 当@Configuration类作为输入提供时，@Consfiguration类本身也被注册为bean definition，并且类中所有声明的@Bean方法也被注册为bean定义。
* 当提供@Component和JSR-330类时，它们被注册为bean definition，并且假设在必要时在这些类中使用诸如@Autowired或@Inject之类的DI元数据。

与实例化ClassPathXmlApplicationContext时Spring XML文件用作输入的方式非常相似，在实例化*AnnotationConfigApplicationContext*时可以使用@Configuration类作为输入。这允许完全无XML地使用Spring容器，如以下示例所示：
```JAVA
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

如前所述，*AnnotationConfigApplicationContext*不仅限于使用@Configuration类。任何@Component或JSR-330带注解的类都可以作为输入提供给构造函数，如以下示例所示：
```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(MyServiceImpl.class, Dependency1.class, Dependency2.class);
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```
当然,一般还是提供一个@Configuration 类,并且这个类里有@ComponentScan 去扫描别的 bean.

您还可以使用无参构造函数来 new AnnotationConfigApplicationContext，然后使用register（）方法对其进行配置。在以编程方式构建AnnotationConfigApplicationContext时，此方法特别有用。以下示例显示了如何执行此操作：
```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class, OtherConfig.class);
    ctx.register(AdditionalConfig.class);
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
    myService.doStuff();
}
```

要启用组件扫描，可以按如下方式注解@Configuration类：
```java
@Configuration
@ComponentScan(basePackages = "com.acme") 
public class AppConfig  {
    ...
}
```

在前面的示例中，将扫描com.acme包以查找任何@Component-annotated类，并将这些类注册为容器中的Spring bean定义。 AnnotationConfigApplicationContext暴露了scan(String ...）方法来让用户显式实现组件扫描功能，如以下示例所示：
```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.scan("com.acme");
    ctx.refresh();
    MyService myService = ctx.getBean(MyService.class);
}
```

另外,*AnnotationConfigApplicationContext*结合*WebApplicationContext*的变体*AnnotationConfigWebApplicationContext*也是可用的。 你可以用这个实现来配置Spring的 *ContextLoaderListener* servlet侦听器，Spring MVC DispatcherServlet等等。以下web.xml代码段配置典型的Spring MVC Web应用程序（请注意contextClass context-param和init-param的使用）：
```xml
<web-app>
    <!-- Configure ContextLoaderListener to use AnnotationConfigWebApplicationContext
        instead of the default XmlWebApplicationContext -->
    <context-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </context-param>

    <!-- Configuration locations must consist of one or more comma- or space-delimited
        fully-qualified @Configuration classes. Fully-qualified packages may also be
        specified for component-scanning -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.acme.AppConfig</param-value>
    </context-param>

    <!-- Bootstrap the root application context as usual using ContextLoaderListener -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!-- Declare a Spring MVC DispatcherServlet as usual -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- Configure DispatcherServlet to use AnnotationConfigWebApplicationContext
            instead of the default XmlWebApplicationContext -->
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>
                org.springframework.web.context.support.AnnotationConfigWebApplicationContext
            </param-value>
        </init-param>
        <!-- Again, config locations must consist of one or more comma- or space-delimited
            and fully-qualified @Configuration classes -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.acme.web.MvcConfig</param-value>
        </init-param>
    </servlet>

    <!-- map all requests for /app/* to the dispatcher servlet -->
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

### 1.12.3. Using the @BeanAnnotation

@Bean是方法级注解，是XML <bean />元素的直接对应。注解支持了<bean />提供的一些属性，例如：
*  [init-method](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-initializingbean) 
*  [destroy-method](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-lifecycle-disposablebean) 
*  [autowiring](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-autowire) 
*  name

**您可以在@Configuration-annotated或@Component-annotated类中使用@Bean注解**。要声明bean，可以使用@Bean注解对方法进行注解,方法返回值类型就是 Bean 类型。默认情况下，**bean名称与方法名称相同**。以下示例显示了@Bean方法声明：
```java
@Configuration
public class AppConfig {

    @Bean
    public TransferServiceImpl transferService() {
        return new TransferServiceImpl();
    }
}
```

您还可以使用接口（或基类）返回类型声明您的@Bean方法(这更加普遍使用)，如以下示例所示：
```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

@ Bean-annotated方法可以有任意数量的参数来描述构建该bean所需的依赖项。例如，如果我们的TransferService需要AccountRepository，我们可以使用方法参数来实现该依赖关系，如以下示例所示：
```java
@Configuration
public class AppConfig {

    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}
```
解析机制与基于构造函数的依赖注入非常相似。有关详细信息，请参阅 [the relevant section](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-constructor-injection) 。

被@Bean注解实例化的任何类(比如@Bean 了一个TransferService)都支持正常的生命周期回调，并且可以使用JSR-250中的@PostConstruct和@PreDestroy注解。常规的Spring生命周期回调,比如InitializingBean，DisposableBean或Lifecycle也是支持的,只要 bean 的实现类实现了这些接口。同时,还完全支持标准的* Aware接口集（例如BeanFactoryAware，BeanNameAware，MessageSourceAware，ApplicationContextAware等）。

@Bean注解本身支持指定任意初始化和销毁​​回调方法，就像bean元素上的Spring XML的init-method和destroy-method属性一样，如下例所示：
```java
public class BeanOne {

    public void init() {
        // initialization logic
    }
}

public class BeanTwo {

    public void cleanup() {
        // destruction logic
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

对于前面示例中的BeanOne，在构造期间直接调用init（）方法同样有效，如下例所示：
```JAVA
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        BeanOne beanOne = new BeanOne();
        beanOne.init();
        return beanOne;
    }

    // …
}
```

另外,你可以在使用@Bean的同时使用使用@Scope 来指定 bean 的 Scope.您可以指定使用@Bean注解定义的bean应具有特定范围。您可以使用 [Bean Scopes](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes) 部分中指定的任何标准作用域。

默认范围是单例，但您可以使用@Scope注解覆盖它，如以下示例所示：
```JAVA
@Configuration
public class MyConfiguration {

    @Bean
    @Scope("prototype")
    public Encryptor encryptor() {
        // ...
    }
}
```

Spring提供了一种通过 [scoped proxies](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection) 处理拥有特殊作用域的依赖项的便捷方法。使用@Scope注解在Java中配置bean提供了与proxyMode属性的等效支持。默认值为无代理（ScopedProxyMode.NO），但您可以指定ScopedProxyMode.TARGET_CLASS或ScopedProxyMode.INTERFACES。

如果将XML参考文档（请参阅 [scoped proxies](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-other-injection) ）的作用域代理,移植到我们使用Java的@Bean，它类似于以下内容：
```JAVA
// an HTTP Session-scoped bean exposed as a proxy
@Bean
@SessionScope
public UserPreferences userPreferences() {
    return new UserPreferences();
}

@Bean
public Service userService() {
    UserService service = new SimpleUserService();
    // a reference to the proxied userPreferences bean
    service.setUserPreferences(userPreferences());
    return service;
}
```

默认情况下，配置类使用@Bean方法的名称作为结果bean的名称。但是，可以使用name属性覆盖此特征. 并且,正如Naming Beans中所讨论的，有时需要为单个bean提供多个名称，也称为bean别名。 @Bean注解的name属性为此接受String数组。以下示例显示如何为bean设置多个别名：
```
@Configuration
public class AppConfig {

    @Bean({"dataSource", "subsystemA-dataSource", "subsystemB-dataSource"})
    public DataSource dataSource() {
        // instantiate, configure and return DataSource bean...
    }
}
```

有时，提供更详细的bean文本描述会很有帮助。比如当bean被暴露出来（可能通过JMX）进行监视时，这可能特别有用。要向@Bean添加描述，可以使用@Description注解，如以下示例所示：
```java
@Configuration
public class AppConfig {
    @Bean
    @Description("Provides a basic example of a bean")
    public Thing thing() {
        return new Thing();
    }
}
```

### 1.12.4. Using the @Configuration annotation

@Configuration是一个class级别注解，指示注解的 bean包含一些bean定义。 @Configuration类通过public 的@Bean注解方法声明bean。在@Configuration类中调用@Bean方法也可用于定义bean间依赖关系。

当bean间彼此依赖时，表达该依赖关系只需要让一个bean方法调用另一个bean方法，如下例所示：
```java
@Configuration
public class AppConfig {

    @Bean
    public BeanOne beanOne() {
        return new BeanOne(beanTwo());
    }

    @Bean
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```
在前面的示例中，beanOne通过构造函数注入接收对beanTwo的引用。

如前所述， [lookup method injection](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection) 是一项很少使用的高级功能。在单例范围的bean依赖于原型范围的bean的情况下，它很有用。Java提供了实现此模式的自然方法。以下示例显示如何使用查找方法注入：
```java
public abstract class CommandManager {
    public Object process(Object commandState) {
        // grab a new instance of the appropriate Command interface
        Command command = createCommand();
        // set the state on the (hopefully brand new) Command instance
        command.setState(commandState);
        return command.execute();
    }

    // okay... but where is the implementation of this method?
    protected abstract Command createCommand();
}
```

Command 是一个原型 bean. 通过使用Java配置，您可以创建CommandManager的子类，其中抽象的createCommand（）方法被覆盖，以便查找新的Command对象。以下示例显示了如何执行此操作：
```java
@Bean
@Scope("prototype")
public AsyncCommand asyncCommand() {
    AsyncCommand command = new AsyncCommand();
    // inject dependencies here as required
    return command;
}

@Bean
public CommandManager commandManager() {
    // return new anonymous implementation of CommandManager with createCommand()
    // overridden to return a new prototype Command object
    return new CommandManager() {
        protected Command createCommand() {
            return asyncCommand();
        }
    }
}
```

##### Further Information About How Java-based Configuration Works Internally

请考虑以下示例，该示例显示了两次调用的@Bean注解方法：
```JAVA
@Configuration
public class AppConfig {

    @Bean
    public ClientService clientService1() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientService clientService2() {
        ClientServiceImpl clientService = new ClientServiceImpl();
        clientService.setClientDao(clientDao());
        return clientService;
    }

    @Bean
    public ClientDao clientDao() {
        return new ClientDaoImpl();
    }
}
```
clientDao（）在clientService1（）中调用一次，在clientService2（）中又被调用一次。此方法创建ClientDaoImpl的新实例并将其返回，通常我们会希望有两个ClientDaoImpl实例（每个service一个）。**实际上,根据测试,上面的例子两个 Service 中拥有的 Dao 是同一个**. 这肯定会有问题：在Spring中，实例化的bean默认具有单例范围。所有@Configuration类在启动时都会在内部使用CGLIB进行subclassed 。在subclass中，子方法在调用父方法并创建新实例之前， 将首先检查容器是否有任何cached（作用域）bean。

### 1.12.5. Composing Java-based Configurations

Spring的基于Java的配置功能允许您Compose注解，这可以降低配置的复杂性。

#### Using the @Import Annotation

就像在Spring XML文件中使用<import />元素来帮助模块化配置一样，@Immort注解允许从另一个configuration class加载@Bean定义，如下例所示：
```java
@Configuration
public class ConfigA {
    @Bean
    public A a() {
        return new A();
    }
}

@Configuration
@Import(ConfigA.class)
public class ConfigB {
    @Bean
    public B b() {
        return new B();
    }
}
```
现在，在实例化上下文时，不需要同时指定ConfigA.class和ConfigB.class，只需要显式提供ConfigB，就也把 ConfigA 的配置包括进来了.如下例所示：
```JAVA
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(ConfigB.class);

    // now both beans A and B will be available...
    A a = ctx.getBean(A.class);
    B b = ctx.getBean(B.class);
}
```
这种方法简化了容器实例化，因为只需要处理一个类，而不是要求您在构造期间记住你配置的大量的@Configuration类。

前面的例子有效，但很简单。在大多数实际情况中，bean跨配置类地彼此依赖。使用XML时，这不是问题，因为不涉及编译器，您可以声明ref =“someBean”并信任Spring在容器初始化期间解决它。但使用@Configuration类时，Java编译器会对配置模型施加约束，因为对其他bean的引用必须是有效的Java语法。

幸运的是，解决这个问题**很简单**。正如我们已经讨论过的，@Bean方法可以有任意数量的参数来描述bean的依赖关系。考虑以下更真实的场景​​，其中包含几个@Configuration类，每个类都依赖于在其他类中声明的bean：
```JAVA
@Configuration
public class ServiceConfig {
    @Bean
    public TransferService transferService(AccountRepository accountRepository) {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {
    @Bean
    public AccountRepository accountRepository(DataSource dataSource) {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {
    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```
可以看见, RepositoryConfig需要SystemTestConfig配置的 Datasource,而ServiceConfig又需要RepositoryConfig配置的accountRepository.这就是跨配置文件的相互依赖. *还有另一种方法可以达到相同的效果*,这需要你记得: @Configuration类最终只是容器中的另一个bean：这意味着它们可以利用@Autowired和@Value注入, 以及同其他bean相同的别的功能。

以下示例显示了如何将一个bean Autowire到另一个bean：
```JAVA
@Configuration
public class ServiceConfig {
    @Autowired
    private AccountRepository accountRepository;
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(accountRepository);
    }
}

@Configuration
public class RepositoryConfig {
    private final DataSource dataSource;
    @Autowired
    public RepositoryConfig(DataSource dataSource) {
        this.dataSource = dataSource;
    }

    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(dataSource);
    }
}

@Configuration
@Import({ServiceConfig.class, RepositoryConfig.class})
public class SystemTestConfig {
    @Bean
    public DataSource dataSource() {
        // return new DataSource
    }
}

public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext(SystemTestConfig.class);
    // everything wires up across configuration classes...
    TransferService transferService = ctx.getBean(TransferService.class);
    transferService.transfer(100.00, "A123", "C456");
}
```

在前面的场景中，使用@Autowired可以很好地工作，但确定autowired的bean的位置is still somewhat ambiguous。例如，作为一名查看ServiceConfig的开发人员，您如何确切地知道@Autowired AccountRepository bean的声明位置？它在代码中并不明确，一般来说这可能也还好。 如果这种ambiguous不可接受,并且您希望从IDE中从一个@Configuration类直接导航到另一个@Configuration类，请考虑自行装配配置类本身。以下示例显示了如何执行此操作：
```java
@Configuration
public class ServiceConfig {

		//注意这里没有引用accountRepository,而是装配了RepositoryConfig
    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        // 调用RepositoryConfig来获取需要的accountRepository
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}
```

在前面的例子中，定义AccountRepository是完全明确的。但是，ServiceConfig现在与RepositoryConfig的实现紧密耦合。这需要你进行权衡。 进而,通过使用基于接口的或基于抽象类的@Configuration类，可以在某种程度上减轻这种紧密耦合。请考虑以下示例：
```JAVA
@Configuration
public class ServiceConfig {
    @Autowired
    private RepositoryConfig repositoryConfig;

    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl(repositoryConfig.accountRepository());
    }
}

@Configuration
public interface RepositoryConfig { //这里是个接口
    @Bean
    AccountRepository accountRepository();
}

@Configuration
public class DefaultRepositoryConfig implements RepositoryConfig { //config 接口的具体实现
    @Bean
    public AccountRepository accountRepository() {
        return new JdbcAccountRepository(...);
    }
}

@Configuration
@Import({ServiceConfig.class, DefaultRepositoryConfig.class})  // import 实现类DefaultRepositoryConfig
public class SystemTestConfig {
    @Bean
    public DataSource dataSource() {
        // return DataSource
    }
}
```
现在，ServiceConfig与具体的DefaultRepositoryConfig松散耦合.

#### Conditionally Include @Configuration Classes or @Bean Methods

你可以基于某些系统状态，有条件地启用或禁用完整的@Configuration类甚至单个@Bean方法。一个常见的例子是，只有在Spring环境中启用了特定的profile文件时才使用@Profile注解来激活bean。(see [Bean Definition Profiles](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles) for details).

@Profile注解实际上是通过使用@Conditional实现的。 @Conditional注解表示在注册@Bean之前应该满足的特定org.springframework.context.annotation.Condition实现。

Condition接口的实现提供了一个返回true或false的matches（...）方法。例如，以下列表显示了用于@Profile的实际Condition实现：
```java
@Override
public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    if (context.getEnvironment() != null) {
        // Read the @Profile annotation attributes
        MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
        if (attrs != null) {
            for (Object value : attrs.get("value")) {
                if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                    return true;
                }
            }
            return false;
        }
    }
    return true;
}
```
See the [@Conditional](https://docs.spring.io/spring-framework/docs/5.1.5.RELEASE/javadoc-api/org/springframework/context/annotation/Conditional.html) javadoc for more detail.

另外,可以把 XML 配置也结合到 Java 配置中去,这需要使用@ImportResource 注解,如下面的例子
```JAVA
@Configuration
@ImportResource("classpath:/com/acme/properties-config.xml")
public class AppConfig {

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;

    @Bean
    public DataSource dataSource() {
        return new DriverManagerDataSource(url, username, password);
    }
}
```

- - - -
## 1.13. Environment Abstraction

*Environment*接口是集成在容器中的抽象，它模拟了应用程序环境的两个关键点： [profiles](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-definition-profiles) and [properties](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#beans-property-source-abstraction) 。

Profile是仅在给定的 profile 是is active时,才向容器注册的,包含Bean定义的命名逻辑组。可以将Bean分配给配置文件。与配置文件相关的Environment对象的作用, 是确定哪些profile 文件件当前处于active状态，以及哪些配置文件应默认处于活动状态。

Properties在几乎所有应用程序中都发挥着重要作用，可能源自各种源：Properties文件，JVM系统属性，系统环境变量，JNDI，servlet上下文参数，ad-hoc属性对象，Map对象等。与属性相关的Environment对象的作用是为用户提供方便的服务接口，用于配置property sources和从中解析属性。 

### 1.13.1. Bean Definition Profiles

Bean定义profiles在核心容器中提供了一种机制，允许在不同environment中注册不同的bean。 “environment”这个词对不同的用户来说意味着不同的东西，这个功能可以帮助到许多case，包括：
* 在QA或生产环境中，针对开发过程中使用内存数据源而不是从JNDI查找相同的数据源。
* 仅在将应用程序部署到性能环境时注册monitoring infrastructure。
* 为客户A和客户B注册不同的注册bean方案。

考虑第一个用例,实际应用程序中需要一个DataSource的。在测试环境中，配置可能类似于以下内容：
```JAVA
@Bean
public DataSource dataSource() {
    return new EmbeddedDatabaseBuilder()
        .setType(EmbeddedDatabaseType.HSQL)
        .addScript("my-schema.sql")
        .addScript("my-test-data.sql")
        .build();
}
```

现在考虑如何将此应用程序部署到QA或生产环境中: 假设应用程序的数据源已在生产应用程序服务器的JNDI目录中注册。我们的dataSource bean现在看起来如下：
```java
@Bean(destroyMethod="")
public DataSource dataSource() throws Exception {
    Context ctx = new InitialContext();
    return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
}
```

问题是如何根据当前环境,在使用这两种方式之间进行切换。::Bean定义profiles是一个核心容器功能，可提供此问题的解决方案。::如果我们推广一下前面的用例，那么最终需求是在某些上下文中而不在其他上下文中注册某些Bean定义。您可能会说您要在情况A中注册一个特定的bean定义配置文件，在情况B中注册一个不同的配置文件。我们首先更新配置以反映这种需求。

@Profile注释允许您指示当一个或多个指定的配置文件处于活动状态时，组件的注册条件。使用前面的示例，我们可以重写dataSource配置，如下所示：
```JAVA
@Configuration
@Profile("development")
public class StandaloneDataConfig {
    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }
}


@Configuration
@Profile("production")
public class JndiDataConfig {

    @Bean(destroyMethod="")
    public DataSource dataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

profile字符串可以包含简单的配置文件名称（例如，生产）或配置文件表达式。概要表达式允许表达更复杂的逻辑。配置文件表达式支持以下运算符：
* !: A logical “not” of the profile
* &: A logical “and” of the profiles
* |: A logical “or” of the profiles

您可以使用@Profile作为元注释，以创建自定义组合注释。以下示例定义了一个自定义@Production批注，您可以将其用作@Profile（“production”）的替代品：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Profile("production")
public @interface Production {
}
```

@Profile也可以在方法级别声明，以仅仅包含配置类的某个特定bean，如以下示例所示：
```JAVA
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

现在我们已经更新了配置，我们需要指示Spring哪个profile处于活动状态。如果我们现在开始我们的示例应用程序，我们会看到抛出NoSuchBeanDefinitionException，因为容器找不到名为dataSource的Spring bean。

激活profile可以通过多种方式完成，但最直接的方法是以编程方式对ApplicationContext提供的Environment API进行操作。以下示例显示了如何执行此操作：
```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
ctx.getEnvironment().setActiveProfiles("development");//设定活跃的 profile
ctx.register(SomeConfig.class, StandaloneDataConfig.class, JndiDataConfig.class);
ctx.refresh();
```

此外，您还可以通过spring.profiles.active property声明性地激活Profile，该属性可以通过系统环境变量，JVM系统属性，web.xml中的servlet上下文参数或。在集成测试中，可以使用spring-test模块中的@ActiveProfiles批注声明活动配置文件（请参阅使用环境配置文件的上下文配置）。

请注意，配置文件不是“either - or”命题。您可以一次激活多个配置文件。以编程方式，您可以为setActiveProfiles（）方法提供多个配置文件名称，该方法接受String ... varargs。以下示例激活多个配置文件：
```java
ctx.getEnvironment().setActiveProfiles("profile1", "profile2");
```

声明式地，spring.profiles.active可以接受以逗号分隔的概要文件名列表，如以下示例所示：
```java
-Dspring.profiles.active="profile1,profile2"
```

比如我们的项目里,就用了spring.profiles.active来指定活动的 profile 名,进而指定 Property 的来源
![](Spring%20Core/111866E1-B9BE-491C-980F-865AF9BF6378.png)

默认profile表示默认启用的配置文件。请考虑以下示例：
```java
@Configuration
@Profile("default")
public class DefaultDataConfig {

    @Bean
    public DataSource dataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .build();
    }
}
```

如果没有激活的profile，则创建里面的dataSource。您可以将此视为提供一个或多个bean的默认定义的方法。如果启用了任何配置文件，则默认配置文件不适用。

您可以使用环境上的setDefaultProfiles（）或声明性地使用spring.profiles.default属性更改默认配置文件的名称。

### 1.13.2.PropertySource Abstraction

Spring的Environment通过可配置的property sources层次结构提供搜索操作。请考虑以下列表：
```java
ApplicationContext ctx = new GenericApplicationContext();
Environment env = ctx.getEnvironment();
boolean containsMyProperty = env.containsProperty("my-property");
System.out.println("Does my environment contain the 'my-property' property? " + containsMyProperty);
```

在前面的代码片段中，我们看到了一种向Spring询问my-property属性是否在当前环境中定义的方法。要回答此问题，Environment对象会对一组PropertySource对象执行搜索。 ::PropertySource是对任何kv pair的源的抽象::，Spring的StandardEnvironment配置有两个固有的PropertySource对象 - 一个表示JVM系统属性集（System.getProperties（）），另一个表示系统环境变量集（ System.getenv（））。

具体来说，当您使用StandardEnvironment时，如果运行时存在my-property系统属性或my-property环境变量，则env.containsProperty（“my-property”）将返回true。

最重要的是，整个机制是可配置的。您可能希望将自定义的property集成到此搜索中。为此，请实现并实例化您自己的*PropertySource*，并将其添加到当前Environment的PropertySource集合中。以下示例显示了如何执行此操作： 
```JAVA
ConfigurableApplicationContext ctx = new GenericApplicationContext();
MutablePropertySources sources = ctx.getEnvironment().getPropertySources();
sources.addFirst(new MyPropertySource());//添加一个 PropertySource
```

### 1.13.3. Using @PropertySource

@PropertySource注解提供了一种方便的声明式机制(而不像上面例子的编程式)，用于向Spring的环境添加PropertySource。

给定一个名为app.properties的文件，其中包含键值对testbean.name = myTestBean，以下@Configuration类使用@PropertySource，以使得调用testBean.getName（）返回myTestBean：
```java
@Configuration
@PropertySource("classpath:/com/myco/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
````

@PropertySource资源位置中存在的任何$ {...}占位符,都将**根据已注册的property source集进行解析**，如以下示例所示：
```java
@Configuration
@PropertySource("classpath:/com/${my.placeholder:default/path}/app.properties")
public class AppConfig {

    @Autowired
    Environment env;

    @Bean
    public TestBean testBean() {
        TestBean testBean = new TestBean();
        testBean.setName(env.getProperty("testbean.name"));
        return testBean;
    }
}
```
假设my.placeholder存在于已注册的其中一个属性源中（例如，系统属性或环境变量），则占位符将解析为相应的值。如果不是，则default/path用作默认值。如果未指定缺省值且无法解析属性，则抛出IllegalArgumentException。

### 1.13.4. Placeholder Resolution in Statements

历史上，元素中占位符的值只能针对JVM系统属性或环境变量进行解析。现在已不再是这种情况。因为Environment抽象集成在整个容器中，所以很容易通过它来路由占位符的解析。这意味着您可以以任何您喜欢的方式配置解析过程。您可以更改搜索系统属性,或者环境变量的优先级，或完全删除它们。您也可以根据需要将自己的属性源添加到the mix中。

具体而言，只要在Environment中可用，无论customer属性在何处定义，以下语句都有效：
```xml
<beans>
    <import resource="com/bank/service/${customer}-config.xml"/>
</beans>
```

- - - -
## 1.14. Registering a LoadTimeWeaver
Spring使用LoadTimeWeaver在将类加载到Java虚拟机（JVM）时动态转换类。要启用load-time weaving(织入切面)，可以将@EnableLoadTimeWeaving添加到其中一个@Configuration类中，如以下示例所示：
```JAVA
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```

一旦为ApplicationContext做了这个配置，该ApplicationContext中的任何bean都可以实现LoadTimeWeaverAware，从而接收对load-time weaver实例的引用。这与 [Spring’s JPA support](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/data-access.html#orm-jpa) 结合使用特别有用，其中JPA类转换可能需要load-time weaver。有关更多详细信息，请参阅LocalContainerEntityManagerFactoryBean的javadoc。For more on AspectJ load-time weaving, see [Load-time Weaving with AspectJ in the Spring Framework](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#aop-aj-ltw) .

```
在 Java 语言中，从织入切面的方式上来看，存在三种织入方式：编译期织入、类加载期织入和运行期织入。编译期织入是指在 Java 编译期，采用特殊的编译器，将切面织入到 Java 类中；而类加载期织入则指通过特殊的类加载器，在类字节码加载到 JVM 时，织入切面；运行期织入则是采用 CGLib 工具或 JDK 动态代理进行切面的织入。

AspectJ 采用编译期织入和类加载期织入的方式织入切面，是语言级的 AOP 实现，提供了完备的 AOP 支持。它用 AspectJ 语言定义切面，在编译期或类加载期将切面织入到 Java 类中。
```

- - - -
## 1.15. Additional Capabilities of theApplicationContext

这部分选择性学习了,跳过的章节有
* 1.15.1. Internationalization using MessageSource
* 1.15.5. Deploying a Spring ApplicationContext as a Java EE RAR File

### 1.15.2. Standard and Custom Events

ApplicationContext中的事件处理是通过*ApplicationEvent*类和*ApplicationListener*接口提供的。::如果将实现*ApplicationListener*接口的bean部署到上下文中，则每次有ApplicationEvent发布到ApplicationContext时，都会通知到该bean。::从本质上讲，这是标准的Observer设计模式。

下表描述了Spring提供的标准事件：
![](Spring%20Core/179C4BAF-684C-4BDD-BAC1-2BC0068D89DF.png)

您还可以创建和发布自己的自定义事件。以下示例显示了一个扩展了Spring的ApplicationEvent基类的简单类：
```java
public class BlackListEvent extends ApplicationEvent {

    private final String address;
    private final String content;

    public BlackListEvent(Object source, String address, String content) {
        super(source);
        this.address = address;
        this.content = content;
    }

    // accessor and other methods...
}
```

要发布自定义的*ApplicationEvent*，请在*ApplicationEventPublisher*上调用publishEvent（）方法。通常，这是通过创建一个实现*ApplicationEventPublisherAware*并将其注册为Spring bean的类来完成的。以下示例显示了这样一个类：
```java
public class EmailService implements ApplicationEventPublisherAware {

    private List<String> blackList;
    private ApplicationEventPublisher publisher;

    public void setBlackList(List<String> blackList) {
        this.blackList = blackList;
    }

    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }

    public void sendEmail(String address, String content) {
        if (blackList.contains(address)) {
				//发布一个时间给监听器去听
            publisher.publishEvent(new BlackListEvent(this, address, content));
            return;
        }
        // send email...
    }
}
```

在配置时，Spring容器检测到EmailService实现了ApplicationEventPublisherAware,将自动调用setApplicationEventPublisher（）。实际上，传入的参数是Spring容器本身。您实际上是通过ApplicationEventPublisher接口在与application context进行交互。

要接收自定义的ApplicationEvent，您可以创建一个实现ApplicationListener的类并将其注册为Spring bean。以下示例显示了这样一个类：
```JAVA
public class BlackListNotifier implements ApplicationListener<BlackListEvent> {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

	  // 处理event 的方法	
    public void onApplicationEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
}
```

请注意，ApplicationListener使用你自定义的event class 做了泛型（前面示例中为BlackListEvent）。这意味着onApplicationEvent（）方法是类型安全的，从而避免任何downcasting的需要。您可以根据需要注册任意数量的侦听器，但请注意，默认情况下，**事件侦听器会同步接收事件。这意味着事件发布器的publishEvent()方法将阻塞，直到所有侦听器都已完成对事件的处理**。这种同步和单线程方法的一个优点是，当侦听器接收到事件时，如果事务上下文可用，它将在publisher的事务上下文内运行(参考事务传播)。如果需要另一个事件发布策略，请参阅Spring的*ApplicationEventMulticaster*接口的javadoc。

#### Annotation-based Event Listeners

从Spring 4.2开始，您可以使用@EventListener批注在bean的任何公共方法上注册事件监听器。 BlackListNotifier可以重写如下：
```JAVA
public class BlackListNotifier {

    private String notificationAddress;

    public void setNotificationAddress(String notificationAddress) {
        this.notificationAddress = notificationAddress;
    }

    @EventListener
    public void processBlackListEvent(BlackListEvent event) {
        // notify appropriate parties via notificationAddress...
    }
```

方法签名中的参数可以声明它侦听的事件类型，但这次使用灵活的名称并且没有实现特定的侦听器接口。如果您的方法应该监听多个事件，或者您想要根据任何参数进行定义，那么也可以在注释本身上指定事件类型。以下示例显示了如何执行此操作：
```java
@EventListener({ContextStartedEvent.class, ContextRefreshedEvent.class})
public void handleContextStart() {
    ...
}
```

还可以通过定义SpEL表达式,使用注解中的condition属性来添加额外的运行时过滤。以下示例显示了仅当事件的content属性等于my-event时，我们的通知程序才会进行调用：
``` java
@EventListener(condition = "#blEvent.content == 'my-event'")
public void processBlackListEvent(BlackListEvent blEvent) {
    // notify appropriate parties via notificationAddress...
}
```
每个SpEL表达式都针对上下文进行评估。下表列出了上下文可以用的项目，以便您可以将它们用于Conditional事件处理：
![](Spring%20Core/AB21685C-5F67-4EAB-AD20-94D76DBA07D8.png)

如果您发布事件是为了发布一个其他事件的结果，则可以更改方法返回值来发布事件，如以下示例所示：
```JAVA
@EventListener
public ListUpdateEvent handleBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress and
    // then publish a ListUpdateEvent...
}
```
这个新方法为上面方法处理的每个BlackListEvent发布一个新的ListUpdateEvent。如果需要发布多个事件，则可以返回事件集合。

如果希望特定侦听器异步处理事件(默认是同步的阻塞的,像之前说过的)，则可以重用常规@Async支持。以下示例显示了如何执行此操作：
```java
@EventListener
@Async
public void processBlackListEvent(BlackListEvent event) {
    // BlackListEvent is processed in a separate thread
}
```
使用异步事件时请注意以下限制：
* 如果侦听器抛出异常，则不会将其传播给调用者。有关详细信息，请参阅*AsyncUncaughtExceptionHandler*。
* 此类事件监听器无法发送reply。如果您需要作为处理结果发送另一个事件，请注入*ApplicationEventPublisher*以手动发送事件。

如果需要在一个侦听器之前调用另一个侦听器，则可以将@Order注释添加到方法声明中，如以下示例所示：
```java
@EventListener
@Order(42)
public void processBlackListEvent(BlackListEvent event) {
    // notify appropriate parties via notificationAddress...
}
```

您还可以使用泛型来进一步定义事件的结构。考虑这样一个事件类型:EntityCreatedEvent<T>，其中T是创建的实际实体的类型。例如，您可以创建以下侦听器定义以仅接收Person的EntityCreatedEvent：
```Java
@EventListener
public void onPersonCreated(EntityCreatedEvent<Person> event) {
    ...
}
```

### 1.15.3. Convenient Access to Low-level Resources

为了最佳地使用和理解应用程序上下文，您应该熟悉Spring的资源抽象，如 [Resources](https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/core.html#resources) .中所述。

应用程序上下文也是一个ResourceLoader，可用于加载Resource对象。 Resource本质上是JDK java.net.URL类的功能更丰富的版本。实际上，在适当的情况下，Resource的实现其实包装了java.net.URL的实例。资源可以以透明的方式从几乎任何位置获取低级资源，包括从类路径，文件系统位置，任何可用的标准URL描述位置，以及一些其他变体。如果资源位置字符串是没有任何特殊前缀的简单路径，where those resources come from is specific and appropriate to the actual application context type.

您可以配置部署到应用程序上下文中的bean来实现特殊的回调接口: *ResourceLoaderAware*，以便在初始化时获取ResourceLoader。
您还可以暴露Resource类型的属性，以用于访问静态资源。它们像任何其他属性一样被注入其中。您可以将这些Resource属性指定为简单的String路径，并依赖特殊的JavaBean PropertyEditor,以便在部署bean时将这些String转换为实际的Resource对象。

提供给ApplicationContext构造函数的位置路径实际上是资源字符串，并且以简单的形式根据特定的上下文实现进行适当处理。例如，ClassPathXmlApplicationContext将简单的位置路径视为类路径位置。您还可以使用具有特殊前缀的位置路径（资源字符串）来强制从类路径或URL加载定义，而不管实际的上下文类型如何。 

### 1.15.4. Convenient ApplicationContext Instantiation for Web Applications

您可以使用例如*ContextLoader*来声明式地创建ApplicationContext实例。当然，您也可以使用一个ApplicationContext实现以编程方式创建ApplicationContext实例。

您可以使用*ContextLoaderListener*注册ApplicationContext，如以下示例所示：
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-value>
</context-param>

<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>
```
侦听器会检查contextConfigLocation参数。如果参数不存在，则侦听器将/WEB-INF/applicationContext.xml用作默认值。当参数确实存在时，侦听器使用预定义的分隔符（逗号，分号和空格）分隔String，并将值用作搜索应用程序上下文的位置。还支持Ant样式的路径模式。示例是/WEB-INF/*Context.xml（对于名称以Context.xml结尾且位于WEB-INF目录中的所有文件）和/WEB-INF/**/*Context.xml（对于所有此类WEB-INF的任何子目录中的文件。

- - - -
## 1.16. The BeanFactory

BeanFactory API为Spring的IoC功能提供了基础。它主要用于与Spring的其他部分和相关的第三方框架集成，其DefaultListableBeanFactory实现是更高级别地GenericApplicationContext容器中的key delegate。 

BeanFactory和相关接口（例如BeanFactoryAware，InitializingBean，DisposableBean）是其他框架组件与 Spring集成地的重要集成点。不需要任何注解或甚至反射，它们允许容器与其中组件之间的非常有效的交互。应用程序级bean可以使用相同的回调接口，但通常更喜欢通过注解或通过编程方式进行声明式的 DI。

请注意，核心BeanFactory API级别及其DefaultListableBeanFactory实现不会对配置的格式或要使用的任何annotation做出假设。 所有这些风格都通过扩展（例如XmlBeanDefinitionReader和AutowiredAnnotationBeanPostProcessor）实现，并作为核心元数据表示在共享的BeanDefinition对象上运行。这是使Spring的容器如此灵活和可扩展的本质。 

### 1.16.1.BeanFactory or ApplicationContext?

本节在容器级别介绍BeanFactory和ApplicationContext之间的差异以及对bootstraping的影响。

您应该使用ApplicationContext，除非您有充分的理由不这样做，应该使用GenericApplicationContext及其子类AnnotationConfigApplicationContext来做自定义的bootstrapping。这些是spring核心容器用于所有常见的目的的的主要入口:比如加载配置文件，触发类路径扫描，以编程方式注册bean定义和带注释的类，以及（从5.0开始）registering functional bean definitions.

因为ApplicationContext包含BeanFactory的所有功能，所以通常建议使用ApplicationContext，除了需要完全控制bean处理的场景之外。在ApplicationContext中，将按惯例检测几种bean（that is, by bean name or by bean type — in particular, post-processors），而普通的DefaultListableBeanFactory对任何其实是应用程序中特殊的bean都是不可知的。

对于许多扩展容器功能，例如注释处理和AOP代理，BeanPostProcessor扩展点是必不可少的。如果仅使用普通的DefaultListableBeanFactory，则默认情况下不会检测到并激活此类post-processor。这令人困惑，因为您的bean配置实际上没有任何问题。相反，在这种情况下，容器需要通过额外的设置来bootstrapped。

下表列出了BeanFactory和ApplicationContext接口和实现提供的功能。 
![](Spring%20Core/A19EC22D-1844-4421-BF90-82124F1C2E20.png)

要使用DefaultListableBeanFactory显式注册bean后处理器，您需要以编程方式调用addBeanPostProcessor，如以下示例所示：
```java
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
// populate the factory with bean definitions

// now register any needed BeanPostProcessor instances
factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
factory.addBeanPostProcessor(new MyBeanPostProcessor());

// now start using the factory
```

显式注册并不方便，这就是为什么使用各种ApplicationContext要优先于普通DefaultListableBeanFactory，尤其是当在典型的企业配置中依赖BeanFactoryPostProcessor和BeanPostProcessor实例for extended container functionality。


