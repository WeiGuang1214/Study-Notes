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

**@Order注解**

​	改变自动注入的顺序，如果Autowired注入的是List这种容器，所以List内的对象注入会有个先后顺序，按数字小的排在前面 0 1这样

##### 通过@DependsOn改变Bean的创建顺序

​	创建bean：1、首先得把bean注入spring容器中----2、容器加载new applicationcontext----3、new A()；DI

​	默认的bean创建的顺序可能不一样，所以需要认为改变

​	按照bean的扫描顺序，按顺序创建

​	但是如果是按component注入的，就需要@Bean下面，用@DependsOn("d")

##### lazy加载Bean

​	@lazy，在容器加载的时候不会去初始化这个bean，只会实例化，只有等到使用这个bean的时候才进行初始化，可以优化启动速度

##### Bean的作用域@Scope

​	默认单例，只会new一次对象@Scope("singleton")，节省内存空间，减少GC次数；全局范围@Scope("prototype")，可能会导致线程不安全；

##### 	Bean默认是单例的，会不会有线程安全问题？

​	单例和原型都有线程安全问题，如果同时对一个共享资源Bean进行读写，才会有线程安全问题，但是只要别在单例bean中声明一些共享的成员变量进行读写，也不会出现线程安全问题。

##### @Conditional条件注解

​	用于动态判断bean是不是生效的，指定一个实现了Condition接口的类，由matches方法返回值决定当前bean是否生效，@Conditional("MyConditional.class")，什么地方用，比如根据依赖来决定连接哪个数据库，pom.xml；

##### 8、Bean的生命周期回调方法

​	1、要让一个类变成一个bean，通过component或者@bean，都可以注入

​	2、加载spring容器 ，new application

​	3、实例化(new 一个对象，放在容器中)

​	4、DI，如果有依赖注入要进行解析，@Autowired @Value

##### 	5、初始化，调用初始化回调方法afterpropertiesset，由开发者来配置

​		自动执行顺序：1接口，2注解，3基于initmethod

​	6、最终放入spring容器中，其实是Map<beanName，bean对象>

​	7、spring通过getBean("beanName")------>Map<beanName,bean对象>

##### 	8、spring容器关闭，bean就会销毁(调用销毁回调方法，由程序来配置)

​		自动执行顺序：1接口，2注解，3基于destorymethod

##### 9、循环依赖，需要解开

​	多个bean之间的初始化过程相互依赖，可能会导致死循环，循环依赖是设计缺陷，不建议使用，spring通过三级缓存解决循环依赖，只能解决单例情况下的bean循环依赖，还可以通过@lazy注解解决循环依赖问题

1、代码设计层面解决：

 	1、把依赖的方法直接写在本类中	2、把依赖的方法AB都抽象出来第三方bean C	3、把依赖类的构造函数引入，如果在构造函数上使用@lazy注解

## AOP，Spring面向切面编程

​	理解：把切面中的代码，插入到切点，不改变原有的代码基础上，对现有业务进行增强。，也就是说额外执行切面里面的代码。

​	<dependencies><dependencies>依赖管理，由父pom管理子模块的pom版本号，子模块只需要注入相应的依赖即可，不需要声明版本，这样就可以减少版本冲突，由父pom统一管理

```java
package com.example.aop;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

// AOP定义切面
@Aspect
@Component // 必须要声明为spring的bean
public class LogAspect {
    public void before() {
        System.out.println("this is a before method !");
    }
    // 切点表达式，方法具体实现，用环绕
    @Around("execution(* com.example.aop.UserService.*(..))") // * 表示所有的访问权限,*(..)表示所有方法，并且不需要参数
    public void log(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        try {
            System.out.println("this is a log method !");
            // 执行具体方法
            joinPoint.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
            System.out.println("方法异常执行");
        }
        long endTime = System.currentTimeMillis();
        System.out.println("this is a log method time : " + (endTime - startTime));
    }
}

```

@Aspect+@Component+通知+切点。4个要素，把额外的业务切入到现有的业务中



##### @EnableAspectJAutoProxy注解

​	启用AOP，没有这个注解AOP功能无法使用，SpringBoot会通过启动类自动加入这个注解配置，所以可以省略，但是还是加上。

​	名称释义：

​		目标对象：要增强的对象，通常会有多个，类下的所有方法

​		切面：要增强的代码放入的那个类，@Aspect

​		通知：Advice，放增强的代码的方法，在切面的连接点上前后所执行的方法，@Around是通用的到处可以用的通知，前，方法前；后，方法后；异常，出现了异常执行；返回通知，方法返回之前去执行。

​		切点：增强的代码切入到哪些方法当中，切点表达式。

​		连接点：通知和目标方法的桥梁，JoinPoint，可以获取目标方法的信息，比如参数，名字等等

​		顾问：源码当中的体现

​		织入：把通知切入到连接点的过程叫织入。

##### 前置通知：@Before，因为明确方法之前，所以不需要手动执行目标方法

##### 后置通知：@After，对应finally中的代码块，不管有没有出现异常都会执行。

##### 返回通知：@AfterReturning，获取返回值，对应方法结束之前的，和return一起。方法参数需要接收返回值。

##### 异常通知：@AfterThrowing，对应catch当中的代码，出现异常就不会执行返回通知，执行后置通知。获取异常信息，需要用Exception接收异常。



##### 切点：

1、表达式抽取成方法，切点表达式，pointcut("execution(* com.example.c4_aop.pointcut.Service.*(..))")

public void pointcut(){}

在后面只需要用@Before("pointcut()")即可，execution可以匹配到方法级别，within只能匹配到类这一级别。

2、@execution，访问修饰符可以省略，但是不能为**，返回值一定要写，可以为***，完整的限定名：包，类

​	包可以是com.ex，全部就是com.ex.*，层级任意就是com..=com.xs.ex.imple

​	类：说

​	指定参数就是类型就可以，所有类型参数是..，不写表示没有参数的方法

3、@within，只能匹配到类级别，within(包名.类名)

4、@annotation，匹配到注解粒度。比如可以记录每个方法的详细作用，比如通过反射获取一个方法的信息。

需要在注入的方法上用到@RetentionPolicy注解，声明为，RUNTIME是运行时，SOURCE是java文件中，CLASS是编译过程中保留方法信息。

@Target表示当前注解可以标记的地方@Target({ElementType.METHOD,ElementType.TYPE})  可以标记的地方

```java
@Pointcut("@annotation(Log)")  // 表示所有用了Log注解的方法都会匹配
public void pointcutAnnotation(){}
```
如果想获取注解的值，需要在方法中加一个参数，如果要在通知的参数中绑定注解，就需要声明参数名，而不是注解类型。

```java
@Pointcut("@annotation(Log)")  // 表示所有用了Log注解的方法都会匹配，不和注解绑定用类型即可
public void pointcutAnnotation(){}

@Before("pointcut()&& pointcutWithIn()&& @annotation(log)")) // 和注解绑定需要用参数名
public void before(joinPoint joinPoint,Log log){}
```



#### SpringAOP的底层原理-动态代理--分为JDK和CGLIB

1、创建spring容器new applicationContext()

2、spring把所有的bean进行创建，进行依赖注入

3、spring(new $$SpringCGLIB$$0())  ,用动态代理继承bean然后创建一个对象代替原本的类

4、当实现了AOP，spring会根据当前的bean创建动态代理

5、把bean进行替换-->所以自动装配的类就成为new $$SpringCGLIB$$0()

##### 代理模式，动态代理是运行时生成代理对象，静态代理是编译时：

​	使用代理对象来增强目标对象，不修改原本对象的功能，扩展目标对象的功能，将核心业务代码和非核心公共代码分离解耦，提高代码可维护性，让被代理类专注业务降低代码复杂度。

​	a、被代理类专注业务  b、代理类非核心的公共代码

**JDK的动态代理，被代理的类一定要实现接口**，所以JDK只能代理实现了接口的类

​	需要传入MyHandler，在myhandler里面执行目标方法，通过反射来接收，里面也可以写前置和后置增强。

```java
@Test
public void testO{
	IUserService UserService = (IUserService) Proxy.newProxyInstance(
		UserService.class.getClassLoader(),
		UserService.class.getInterfaces(),
		new MyHandler(new UserService()) // 在myhandler中的invoke方法中写目标方法和增强方法
	);
	userService.add();
	}
}
```

##### 代理对象的生成途径：

​	借助 Proxy.newProxyInstance() 方法能够创建代理对象。在创建时，需要传入类加载器、接口数组，还有 MyHandler 实例。

##### 方法调用的流转过程：

​	**当代理对象的方法被调用时，会触发 MyHandler 中的 invoke 方法，进而实现增强逻辑和目标方法的调用。**

##### CGLIB是第三方注解，但是被Spirng整合，所以可以直接用

```java
public class TestCGlibProxy{
	@Test
	public void test(){
		Enhancer enhancer = new Enhancer();
        // 设置被代理的类
        enhancer.setSuperclass(UserService.class); //直接继承，不需要实现接口
        // 设置处理类
        enhancer.setCallback();
        // 生成代理类，继承自UserService
        Userservice userService = (Userservice)enhancer.create();
        
        userService.add
	}
}
// 需要写一个MyCallback继承自MethodInterceptor，在intercept方法中需要写前置方法，增强，目标对象，和后置方法

目标对象：
    返回值 = proxy.invoke(参数)，注意参数和返回值
```

#### 平常的业务开发用不到动态代理，但是不能不学，只有底层框架才会用

​	spring默认jdk动态代理，springboot默认cglib动态代理

静态代理弊端：代理类不需要自己声明，一个代理类只能代理一个类型

### Spring声明式事务

#### Spring操作数据库---MaBatis-plus

JDBC连接方法需要构建链接，写DataSource，数据库的名字、密码、URL等等，然后datasource.getconnection

自动配置成bean

datasource是一个接口，提供了getConnection方法，更为高效、灵活、可维护的方式，不同的提供商只需要实现DataSource，程序员不需要关心链接的提供方式，可以通过DriverManager也可以通过连接池。

连接池：管理数据库连接的技术，一次性创建很多连接，可以复用，避免重复创建和关闭的开销，提高效率。

##### Spring单独配置DataSource

```java
@SpringBootTest(classes = SpringDataSourceTest.class) // 当前配置类
@ComponentScan // 扫包
@SpringBootApplication // 如果是用springboot的datasource，一定要用这个注解
class SpringDataSourceTest {
    @Test
	void contextLoads(@Autowired DataSource dataSource) throws SQLException{
		System.out.println(dataSource.getConnection());
	}	
}
```

```java
// 标准做法应该是做一个配置类
@Configuration
public class DataSource {
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(driverClassName);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);
        return dataSource;
    }
}
```

##### 利用JDBCTemplate连接数据库

jdbc太捞了，spring提供了jdbcTemplate封装了

Spring中要单独配bean，SpringBoot直接注入使用

```java
@Bean
public JdbcTemplate jdbcTemplate(){
	return new JdbcTemplate(dataSource());
}

@Test
void testJDBCTemplate(@Autowired JdbcTemplate){
    String sql = "update t_user set username=? where id=?";
    int result = jdbcTemplate.update(sql,"xxx",1);
}
```

#### Spring 声明式事务

声明式事务，直接写注解。

JDBC的编程式事务：

```java
public void test(oAutowired DataSource dataSource) {
	try (Connection connection = dataSource.getConnection()){
		connection.setAutoCommit(false);
		//开启事务
	try(Statement statement = connection.createStatement()) {
		String sql="insert into t_user(username) values ("xxx")
		statement.execute(sql);
		connection.commit();  // 执行完提交
	}
	catch (SQLException e) {
		connection.rollback(); // 异常了回滚
} catch (SQLException e){
	e.printStackTrace();
	// 处理异常
	}
}
```

##### SpringBoot声明式事务：

```java
@Transactional
@Repository
public class UserDao{
    @Autowired 
    JdbcTemplate jdbcTemplate;
    
    public int insert(){
        jdbcTemplate.update("insert into t_user(username) values(?)","xushu");
    }
}
// 写在业务逻辑类上

@Service
public class UserService{
    
    @Autowired
    UserDao userDao;
    
    @Transactional  // SpringBoot的事务,类似AOP，SpringBoot自动开启了
    public int insert(){
        return userDao.insert();
    }
}

```

##### 如果是Spring，需要单独开启事务

```java

// 如果是Spring，需要单独开启事务
@EnableTransactionManagement
// 配置事务管理器
@Bean
public TranscationManager transactionManager(){
    return new DataSourceTransactionManager(dataSource);
}
```

除了写在方法上，也可以写在类上，表示所有的方法都开启事务

#### 事务配置的属性：

##### 1、isolation 事务的隔离级别，是数据库提供的功能：用来解决并发事务所产生的一些问题。

​	并发：同一时间、多个线程同时进行请求，对同一个数据进行读写操作，产生脏读、不可重复度、幻读

脏读解决  提交读(行锁，读不会加锁)

@Transaction(isolation = Isolation.READ-COMMOTTED)，不用设置，数据库自动帮我们设置了。

不可重复度解决：重复读(行锁，读和写都会上锁)

@Transaction(isolation = Isolation.REPEATABLE_READ)

幻读解决：串行化(表锁，读和写都锁)

@Transaction(isolation = Isolation.SERIALIZABLE)

##### 2、事务传播行为

@Transaction(propagation = REQUIRES_NEW)

事务的传播性是指当一个事务方法被另外一个事务方法调用的时候，这个事务方法应该如何进行？

希望外部如果存在就用外部，外部没有就自己开启

常用的：

​	默认REQUIRED，开启一个新的事物，有就融入，使用于增删查改

​	SUPPORTS，外部不存在就不开启，如果外部存在，就融入，适用于查询

​	REQUIRED_NEW ，开启一个新事务，如果外部有，仍然创建一个新事务，

​	NOT_SUPPORTED，不开启新的事务，不用外部事务

​	NEVER  ，不开启新事务，如果外层有，抛出异常

##### 设置事务只读 readOnly=true

只在查询业务中，当前数据库操作是只读，所以就会对当前的只读做优化。

##### 超时属性timeout，事务最长等待时间。否则线程会阻塞

##### 异常属性

默认对runtimeexception都回滚，只会回滚runtimeexception，不是runtimeexception就不会回滚

设置哪些异常不回滚noRollbackFor 

设置哪些异常回滚rollbackFor，如果是exception.class，所有异常都回滚

@Transaction(rollbackFor = exception.class)

#### 事务(AOP)失效原因

事务原理：动态代理

AOP底层原理：动态代理

userDao.add()  //  执行数据库操作

失效的原因：

1、保证事务的类，配置为了一个Bean，如果不是配置成Bean，就不会被Spring管理

2、如果是private，也没有办法代理

3、不能在事务方法中捕获异常，如果捕获了，一定要抛出，没有抛出外部事务就感知不到

4、如果要让动态代理生效，必须使用动态代理的对象调用目标方法，不能通过普通对象调用‘

动态代理层面的失效原因：

​	直接调用本类的方法，也会失效，因为没有走动态代理的对象

#### 事务失效了怎么解决？

​	1、直接将本类Bean自动装配进来，但是会导致循环依赖，需要打开支持

​	2、((本类)AopContext.currentProxy()).lose2()，获取当前的代理对象，并且调用方法，可以实现动态代理和事务、AOP等等

​	需要配置：

​	@EnableAspectJAutoProxy(exposeProxy = true)，意味着通过一个AOP动态代理对象执行方法的时候，当前动态代理对象会存在一个ThreadLocal里面，

​	3、把本类的方法移动到其他的Service中，然后再把对应的Bean自动装配进来，调用该方法，也可以生效。



#### Spring AOT提前优化

一种编译方式，提升启动速度

JIT，即Just-in-time，动态编译，运行时编译

AOT，Ahead of Time，指运行前编译，是两种程序的编译方式，编译期就直接生成机器码，程序运行时直接翻译机器码

实际开发用不上，除非自研底层框架





