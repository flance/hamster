## Spring 常用注解

| 注解 | 路径 | 解释 |  
| ------ | ------ | ------ |  
|@Bean|org.springframework.context.annotation|注解在方法上，声明当前方法的返回值为一个Bean。返回的Bean对应的类中可以定义init()方法和destroy()方法，然后在@Bean(initMethod=“init”,destroyMethod=“destroy”)定义，在构造之后执行init，在销毁之前执行destroy。
|@Component|org.springframework.stereotype|表示一个带注释的类是一个“组件”，成为Spring管理的Bean。当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。同时@Component还是一个元注解。
|@Service|org.springframework.stereotype|组合注解（组合了@Component注解），应用在service层（业务逻辑层）
|@Resource|javax.annotation.Resource|JSR-250提供的注解
|@Reponsitory|org.springframework.stereotype|组合注解（组合了@Component注解），应用在dao层（数据访问层）
|@Autowired|org.springframework.beans.factory.annotation|Spring提供的工具（由Spring的依赖注入工具（BeanPostProcessor、BeanFactoryPostProcessor）自动注入。）
|@Inject||JSR-330提供的注解
|@Controller|org.springframework.stereotype|组合注解（组合了@Component注解），应用在MVC层（控制层）,DispatcherServlet会自动扫描注解了此注解的类，然后将web请求映射到注解了@RequestMapping的方法上。
|@Configuration|org.springframework.context.annotation|声明当前类是一个配置类（相当于一个Spring配置的xml文件）
|@ComponentScan|org.springframework.context.annotation|自动扫描指定包下所有使用@Service,@Component,@Controller,@Repository的类并注册
|@Aspect|org.aspectj.lang.annotation|声明一个切面（就是说这是一个额外功能）
|@After|org.aspectj.lang.annotation|后置建言（advice），在原方法前执行。
|@Before|org.aspectj.lang.annotation|前置建言（advice），在原方法后执行。
|@Around|org.aspectj.lang.annotation|环绕建言（advice），在原方法执行前执行，在原方法执行后再执行（@Around可以实现其他两种advice）
|@PointCut|org.aspectj.lang.annotation|声明切点，即定义拦截规则，确定有哪些方法会被切入
|@Transactional|org.springframework.transaction.annotation javax.transaction|声明事务（一般默认配置即可满足要求，当然也可以自定义）
|@Cacheable|org.springframework.cache.annotation javax.persistence"|声明数据缓存
|@EnableAspectJAutoProxy|org.springframework.context.annotation|开启Spring对AspectJ的支持
|@Value|org.springframework.beans.factory.annotation|值得注入。经常与Sping EL表达式语言一起使用，注入普通字符，系统属性，表达式运算结果，其他Bean的属性，文件内容，网址请求内容，配置文件属性值等等
|@PropertySource|org.springframework.context.annotation|指定文件地址。提供了一种方便的、声明性的机制，用于向Spring的环境添加PropertySource。与@configuration类一起使用。
|@PostConstruct|javax.annotation|标注在方法上，该方法在构造函数执行完成之后执行。
|@PreDestroy||标注在方法上，该方法在对象销毁之前执行。
|@Profile|org.springframework.context.annotation|表示当一个或多个指定的文件是活动的时，一个组件是有资格注册的。使用@Profile注解类或者方法，达到在不同情况下选择实例化不同的Bean。@Profile(“dev”)表示为dev时实例化。
|@EnableAsync|org.springframework.scheduling.annotation|开启异步任务支持。注解在配置类上。
|@Async|org.springframework.scheduling.annotation|注解在方法上标示这是一个异步方法，在类上标示这个类所有的方法都是异步方法。
|@EnableScheduling|org.springframework.scheduling.annotation|注解在配置类上，开启对计划任务的支持。
|@Scheduled|org.springframework.scheduling.annotation|注解在方法上，声明该方法是计划任务。支持多种类型的计划任务：cron,fixDelay,fixRate
|@Conditional||根据满足某一特定条件创建特定的Bean
|@Enable*||通过简单的@Enable来开启一项功能的支持。所有@Enable注解都有一个@Import注解，@Import是用来导入配置类的，这也就意味着这些自动开启的实现其实是导入了一些自动配置的Bean(1.直接导入配置类2.依据条件选择配置类3.动态注册配置类)
|@RunWith||这个是Junit的注解，springboot集成了junit。一般在测试类里使用:@RunWith(SpringJUnit4ClassRunner.class) — SpringJUnit4ClassRunner在JUnit环境下提供Sprng TestContext Framework的功能
|@ContextConfiguration|org.springframework.test.context|用来加载配置ApplicationContext，其中classes属性用来加载配置类:@ContextConfiguration(classes = {TestConfig.class(自定义的一个配置类)})
|@ActiveProfiles|org.springframework.test.context|用来声明活动的profile–@ActiveProfiles(“prod”(这个prod定义在配置类中))
|@EnableWebMvc|org.springframework.web.servlet.config.annotation|用在配置类上，开启SpringMvc的Mvc的一些默认配置：如ViewResolver，MessageConverter等。同时在自己定制SpringMvc的相关配置时需要做到两点：1.配置类继承WebMvcConfigurerAdapter类2.就是必须使用这个@EnableWebMvc注解。
|@RequestMapping||用来映射web请求（访问路径和参数），处理类和方法的。可以注解在类和方法上，注解在方法上的@RequestMapping路径会继承注解在类上的路径。同时支持Serlvet的request和response作为参数，也支持对request和response的媒体类型进行配置。其中有value(路径)，produces(定义返回的媒体类型和字符集)，method(指定请求方式)等属性。
|@GetMapping||GET方式的@RequestMapping
|@PostMapping||POST方式的@RequestMapping
|@ResponseBody||将返回值放在response体内。返回的是数据而不是页面
|@RequestBody||允许request的参数在request体中，而不是在直接链接在地址的后面。此注解放置在参数前。
|@PathVariable||放置在参数前，用来接受路径参数。
|@RestController||组合注解，组合了@Controller和@ResponseBody,当我们只开发一个和页面交互数据的控制层的时候可以使用此注解。
|@ControllerAdvice||用在类上，声明一个控制器建言，它也组合了@Component注解，会自动注册为Spring的Bean。
|@ExceptionHandler||用在方法上定义全局处理，通过他的value属性可以过滤拦截的条件：@ExceptionHandler(value=Exception.class)–表示拦截所有的Exception。
|@ModelAttribute||将键值对添加到全局，所有注解了@RequestMapping的方法可获得次键值对（就是在请求到达之前，往model里addAttribute一对name-value而已）。
|@InitBinder||通过@InitBinder注解定制WebDataBinder（用在方法上，方法有一个WebDataBinder作为参数，用WebDataBinder在方法内定制数据绑定，例如可以忽略request传过来的参数Id等）。
|@WebAppConfiguration||一般用在测试上，注解在类上，用来声明加载的ApplicationContext是一个WebApplicationContext。他的属性指定的是Web资源的位置，默认为src/main/webapp,我们可以修改为：@WebAppConfiguration(“src/main/resources”)。
|@EnableAutoConfiguration|org.springframework.boot.autoconfigure|此注释自动载入应用程序所需的所有Bean——这依赖于Spring Boot在类路径中的查找。该注解组合了@Import注解，@Import注解导入了EnableAutoCofigurationImportSelector类，它使用SpringFactoriesLoader.loaderFactoryNames方法来扫描具有META-INF/spring.factories文件的jar包。而spring.factories里声明了有哪些自动配置。
|@SpingBootApplication|org.springframework.boot.autoconfigure|SpringBoot的核心注解，主要目的是开启自动配置。它也是一个组合注解，主要组合了@Configurer，@EnableAutoConfiguration（核心）和@ComponentScan。可以通过@SpringBootApplication(exclude={想要关闭的自动配置的类名.class})来关闭特定的自动配置。
|@ImportResource|org.springframework.context.annotation|虽然Spring提倡零配置，但是还是提供了对xml文件的支持，这个注解就是用来加载xml配置的。例：@ImportResource({"classpath
|@ConfigurationProperties||将properties属性与一个Bean及其属性相关联，从而实现类型安全的配置。例：@ConfigurationProperties(prefix=“authot”，locations={"classpath
|@ConditionalOnBean||条件注解。当容器里有指定Bean的条件下。
|@ConditionalOnClass||条件注解。当类路径下有指定的类的条件下。
|@ConditionalOnExpression||条件注解。基于SpEL表达式作为判断条件。
|@ConditionalOnJava||条件注解。基于JVM版本作为判断条件。
|@ConditionalOnJndi||条件注解。在JNDI存在的条件下查找指定的位置。
|@ConditionalOnMissingBean||条件注解。当容器里没有指定Bean的情况下。
|@ConditionalOnMissingClass||条件注解。当类路径下没有指定的类的情况下。
|@ConditionalOnNotWebApplication||条件注解。当前项目不是web项目的条件下。
|@ConditionalOnResource||条件注解。类路径是否有指定的值。
|@ConditionalOnSingleCandidate||条件注解。当指定Bean在容器中只有一个，后者虽然有多个但是指定首选的Bean。
|@ConditionalOnWebApplication||条件注解。当前项目是web项目的情况下。
|@EnableConfigurationProperties||注解在类上，声明开启属性注入，使用@Autowired注入。例：@EnableConfigurationProperties(HttpEncodingProperties.class)。
|@AutoConfigureAfter||在指定的自动配置类之后再配置。例：@AutoConfigureAfter(WebMvcAutoConfiguration.class)
