## Spring6

### 核心特性

#### 1. **IoC 容器（控制反转）**

- **依赖注入（DI）**：将对象的创建和依赖关系管理交给 Spring 容器，降低组件间耦合度。
- **Bean 管理**：通过配置文件或注解定义和管理对象（Bean）的生命周期。

#### 2. **AOP（面向切面编程）**

- 分离核心业务逻辑和横切关注点（如日志、事务、权限），提高代码复用性。

#### 3. **数据访问**

- 简化 JDBC 操作，支持 ORM 框架（如 Hibernate、MyBatis）。
- 提供事务管理抽象层，支持声明式事务。

#### 4. **Web 框架**

- Spring MVC：基于 Servlet 的轻量级 Web 框架，支持 RESTful。
- Spring WebFlux：响应式 Web 框架，支持非阻塞 IO。

#### 5. **企业集成**

- 支持消息队列（JMS、RabbitMQ）、远程调用（RMI、HTTP）等。

------

注入bean的demo：

​	

```java
<!--spring 3以下主要用xml,让spring帮我们管理bean,也就是说找到要注入的Java实现类-->
<!--bean class="com.xs.service.UserService">
    <!-注入userDao这个对象-->
    <!--property name="userDao" ref="userDao"></property>
</bean>
<!-bean class="com.xs.dao.UserDao" name="userDao"></bean>-->

<!--Spring2的特性，要告诉spring，包在哪，是SSM最常见的方式，利用注解来注入bean-->
<comtext:component-scan base-package="com.xs"></comtext:component-scan>
<!--Spring3开始，可以用Java config配置注入-->
    
  用xml和注解的方式配置Bean，调用容器的方式和Configuration配置类的不一样：
@Test
    public void test(){
        System.out.println("this is a test");

        //如果要依赖spring的注入，需要从spring容器当中获取UserService
        //所以需要先获得一个spring容器ApplicationContext
        ApplicationContext ac = new ClassPathXmlApplicationContext("spring.xml");
        IUserService service = ac.getBean(IUserService.class);
        service.getUser();
    }
```

如果是Configuration配置类的方式注入Bean，要用AnnotationConfigApplicationContext

```java
public class Test01 {
    @Test
    public void test(){
        System.out.println("this is a test");
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SpringConfig.class);
        IUserService service = ac.getBean(IUserService.class);
        service.getUser();
    }
}
```

#### 基于SpringBoot快速启动Spring

```java
//注解的方式注入bean
@Component
public class UserService implements IUserService {
    //由spring注入
    @Autowired    //让spring注入bean
    IUserDao userDao;

    @Override
    public void getUser(){
        userDao.getUserDao();
    }
}
```

配置启动类run方法获取一个容器

```java
@SpringBootApplication
public class SpringbootDemoApplication {

    public static void main(String[] args) {
        ConfigurableApplicationContext ioc = SpringApplication.run(SpringbootDemoApplication.class, args);
        IUserService userService = ioc.getBean(IUserService.class);
        userService.getUser();
    }
}
```

### Bean：被Spring管理的对象

​	往容器注入bean的方式：

​	1、xml，<Bean class="xxxx.class" name="xxxxx"></Bean>

​	2、component注解，放在类上

​	**3、@Bean注解，1、通常放在配置类上，2、写在方法上，能做到比component更多的功能，从而干预bean的实例化的过程，相当于是一个实例方法，最后返回一个对象。在方法中注入的参数spring也会直接帮我们从spring容器中拿，包括在一个Bean里面获取另外一个bean，也是从容器中获取**

​	**注意，这里通常用于把外部的jar包里面的类，装配成bean来使用，因为你无法在jar包里面用@component注解**

```java
// 配置类
@Configuration
class AppConfig {
    @Bean
    public MyService myService() {
        return new MyServiceImpl();
    }
}

// 服务接口
interface MyService {
    void doSomething();
}

// 服务实现
class MyServiceImpl implements MyService {
    @Override
    public void doSomething() {
        System.out.println("执行服务方法");
    }
}		
		//main方法中
		// 通过类型获取Bean
        MyService service = context.getBean(MyService.class);
        
        // 通过名称和类型获取Bean
        MyService namedService = context.getBean("myService", MyService.class);
        
        // 使用Bean
        service.doSomething();
```

4、@Import注解

​	通常写在类上，必须类上，并且这个类必须是一个Bean，一般是哪里需要用到，就会把里面类型的bean注入到spring容器当中，然后spring容器会完成初始化

​	导入ImportSelector.class，需要实现这个接口，并且重写方法。可以返回一个数组，将多个bean注入spring容器

​	还有ImportBeanDefinitionsRegistrar.class这个接口，实现这个接口，可以手动创建一个类

​		定义一个类实现ImportBeanDefinitionRegistrar

​		根据beanDefinition注册bean

MyImportBeanDefinitionsRegister类里面：

```java
RootBeanDefinition definition=new RootBeanDefinition();
definition.setBeanClassName("com.xs.service.UserService");
registry.registerBeanDefinition("userService",definition);
```

​	然后注入bean就用@Import 这个MyImportBeanDefinitionsRegister.class，这是个利用底层源码实现更多业务功能的。



#### 实例化Bean

​	默认实例化方式，Bean是调用无参构造器，如果有多个构造器，依然会调用无参构造器

##### 	使用实例工厂方法实例化  @Bean

​	**用@Bean注解可以自由选择构造函数**

##### 	使用工厂Bean实例化-FactoryBean

​	FactoryBean是一个接口，需要有一个Bean，一旦这个Bean实现FactoryBean就成为工厂Bean，可以自由选择构造函数进行实例化，创建一类Bean

实现getObject-->通过Bean名字获取到的是getObject的对象，也可以通过类获得

实现getObjecType-->想通过容器去获取getObject返回的对象的类型，就需返回getObject返回对象的类型

```java
@Service
public class OrderService implements FactoryBean {
    @Autowired
    private IUserDao userDao;
    @Override
    public Object getObject() throws Exception {
        return new UserService(userDao);
    }

    @Override
    public Class<?> getObjectType() {
        return null;
    }
}
```

```java
public static void main(String[] args) {
        ConfigurableApplicationContext ioc = SpringApplication.run(SpringbootDemoApplication.class, args);
        System.out.println(ioc.getBean(OrderService.class));
    }
```

### 依赖注入DI，一定要配置成Bean才能相互注入

1、@Autowired，自动注入bean

​	**@Autowired可以用在参数、方法、字段、构造器**

​	**a.构造函数**

​		如果bean只有一个有参构造函数，是省略@Autowired，自动注入构造函数的参数

​		如果有多个有参构造函数，并且没有无参构造函数，会报错，解决就是用Autowired指定一个构造函数，required会失效，

​	**b.参数**

​		在参数上指定Autowired，并且设置required为false，这样就指定有效，**变成不是必须注入的bean**

​	        还可以在测试用例里面也可以用Autowired

​	**c.方法**

​		**@Autowired用在方法上，就是自动调用该方法，并且自动注入**

​	Autowired是先bytype再byname去找同名bean，

​	requir=false，说明不是必须的，true就是必须的bean，如果没找到就会报错

​	如果通过名字还匹配不到，用@primary注解设置主要的bean去找，@Qualifier("name")手动告诉spring找什么名字的bean

2、@Bean注解，注入，通过参数

3、构造函数上的参数会自动注入

​	@inject，也可以注入bean，但是inject要导入依赖

​	**优先用：@Autowired和@Resource  会先根据名字找，再根据类型找，不依赖与框架，是JDK官方提供的注入一个bean的方式**

​	

#### 	**--POJO用于接受前端请求参数，或者是把数据库查询到的数据封装到pojo--**

##### 但是有些属性、参数需要配置

##### @Value注解：

​	1、直接值（基本类型。String等等）

​	@Value("具体的值")

​	private string name;

​	2、用外部属性赋值，@PropertySource("文件夹.properties")

​	@Value($"AAA.age")

​	里面都是AAA.age=xxx，直接链入

​	3、springboot有一个配置文件，application.propertis，约定大于配置

​	这里面可以写配置，直接获取这里面的字段

​	AAA.age直接写在properties文件里面

​	非springboot的属性文件，需要用@PropertySource导入classpath：

​	

​	spel复杂类型

​	给map等类型

​	@Value("#{'key':'value'}")

​	

**@Order注解**

改变自动注入的顺序，如果Autowired注入的是List这种容器，所以List内的对象注入会有个先后顺序，





