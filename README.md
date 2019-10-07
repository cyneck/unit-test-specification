# 单元测试规范和mock进阶使用实例
---------------------------

开发、测试流程示例：

![img](https://github.com/cyneck/unit-test-specification/blob/master/test%20process.jpg)

------------------------

## 一、单元测试目的

​		单元测试是编写测试代码，用来检测特定的、明确的、细颗粒的功能。单元测试并不一定保证程序功能是正确的，更不保证整体业务是准备的。单元测试不仅仅用来保证当前代码的正确性，更重要的是用来保证代码**修复**、**改进**或**重构**之后的正确性。

1. 接口功能测试：用来保证接口功能的正确性。
2. 局部数据结构测试（不常用）：用来保证接口中的数据结构是正确的
   1. 比如变量有无初始值
   2. 变量是否溢出
3. 边界条件测试
   1. **变量没有赋值**（即为NULL）
   2. 变量是数值（或字符)
      1. **主要边界**：最小值，最大值，无穷大（对于DOUBLE等）
      2. **溢出边界**（期望异常或拒绝服务）：最小值-1，最大值+1
      3. **临近边界**：最小值+1，最大值-1
   3. 变量是字符串
      1. 引用“字符变量”的边界
      2. **空字符串**
      3. 对字符串长度应用“数值变量”的边界
   4. 变量是集合
      1. **空集合**
      2. **对集合的大小应用“数值变量”的边界**
      3. **调整次序**：升序、降序
   5. 变量有规律
      1. 比如对于Math.sqrt，给出n^2-1，和n^2+1的边界
4. 所有独立执行通路测试：保证每一条代码，每个分支都经过测试
   1. 代码覆盖率
      1. **语句覆盖**：保证每一个语句都执行到了
      2. 判定覆盖（分支覆盖）：保证每一个分支都执行到
      3. 条件覆盖：保证每一个条件都覆盖到true和false（即if、while中的条件语句）
      4. 路径覆盖：保证每一个路径都覆盖到
   2. 相关软件
      1. Cobertura：语句覆盖
      2. Emma: Eclipse插件Eclemma
5. 各条错误处理通路测试：保证每一个异常都经过测试

-------------

## **二、基本原则**
1. <font color=red>【强制】</font>好的单元测试必须遵守 AIR 原则。
   说明：单元测试在线上运行时，感觉像空气（AIR）一样并不存在，但在测试质量的保障上，却是非常关键的。好的单元测试宏观上来说，具有自动化、独立性、可重复执行的特点。
   -  A：Automatic（自动化）
   -  I：Independent（独立性）
   -  R：Repeatable（可重复）
2. <font color=red>【强制】</font>单元测试应该是全自动执行的，并且非交互式的。测试用例通常是被定期执行的，
   执行过程必须完全自动化才有意义。输出结果需要人工检查的测试不是一个好的单元测试。
   单元测试中不准使用 System.out 来进行人肉验证，必须使用 assert 来验证。
3. <font color=red>【强制】</font>保持单元测试的独立性。为了保证单元测试稳定可靠且便于维护，单元测试用例之
   间决不能互相调用，也不能依赖执行的先后次序。
   反例：method2 需要依赖 method1 的执行，将执行结果作为 method2 的输入。
4. <font color=red>【强制】</font>单元测试是可以重复执行的，不能受到外界环境的影响。
   说明：单元测试通常会被放到持续集成中，每次有代码 check in 时单元测试都会被执行。如果单测对外部
   环境（网络、服务、中间件等）有依赖，容易导致持续集成机制的不可用。
   正例：为了不受外界环境影响，要求设计代码时就把 SUT 的依赖改成注入，在测试时用 spring  这样的 DI
   框架注入一个本地（内存）实现或者 Mock 实现。
5. <font color=red>【强制】</font>对于单元测试，要保证测试粒度足够小，有助于精确定位问题。单测粒度至多是类
   级别，一般是方法级别。
   说明：只有测试粒度小才能在出错时尽快定位到出错位置。单测不负责检查跨类或者跨系统的交互逻辑，
   那是集成测试的领域。
6. <font color=red>【强制】</font>核心业务、核心应用、核心模块的增量代码确保单元测试通过。
   说明：新增代码及时补充单元测试，如果新增代码影响了原有单元测试，请及时修正。
7. <font color=red>【强制】</font>单元测试代码必须写在如下工程目录：src/test/java，不允许写在业务代码目录下。
   说明：源码编译时会跳过此目录，而单元测试框架默认是扫描此目录。
8. <font color=#FF9900>【推荐】</font>单元测试的基本目标：语句覆盖率达到 70%；核心模块的语句覆盖率和分支覆盖率
   都要达到 100%
   说明：在工程规约的应用分层中提到的 DAO 层，Manager 层，可重用度高的 Service，都应该进行单元
   测试。
9. <font color=#FF9900>【推荐】</font>编写单元测试代码遵守 BCDE 原则，以保证被测试模块的交付质量。
   -  B：Border，边界值测试，包括循环边界、特殊取值、特殊时间点、数据顺序等。
   -  C：Correct，正确的输入，并得到预期的结果。
   -  D：Design，与设计文档相结合，来编写单元测试。
   -  E：Error，强制错误信息输入（如：非法数据、异常流程、业务允许外等），并得到预期的结果。
10. <font color=#FF9900>【推荐】</font>对于数据库相关的查询，更新，删除等操作，不能假设数据库里的数据是存在的，
    或者直接操作数据库把数据插入进去，请使用程序插入或者导入数据的方式来准备数据。
    反例：删除某一行数据的单元测试，在数据库中，先直接手动增加一行作为删除目标，但是这一行新增数
    据并不符合业务插入规则，导致测试结果异常。
11. <font color=#FF9900>【推荐】</font>和数据库相关的单元测试，可以设定自动回滚机制，不给数据库造成脏数据。或者
    对单元测试产生的数据有明确的前后缀标识。
    正例：在企业智能事业部的内部单元测试中，使用 ENTERPRISE_INTELLIGENCE _UNIT_TEST_的前缀来
    标识单元测试相关代码。
12. <font color=#FF9900>【推荐】</font>对于不可测的代码在适当的时机做必要的重构，使代码变得可测，避免为了达到测
    试要求而书写不规范测试代码。
13. <font color=#FF9900>【推荐】</font>在设计评审阶段，开发人员需要和测试人员一起确定单元测试范围，单元测试最好
    覆盖所有测试用例。
14. <font color=#FF9900>【推荐】</font>单元测试作为一种质量保障手段，在项目提测前完成单元测试，不建议项目发布后
    补充单元测试用例。
15. <font color=#55AA77>【参考】</font>为了更方便地进行单元测试，业务代码应避免以下情况：
    构造方法中做的事情过多。
    -  存在过多的全局变量和静态方法。
    -  存在过多的外部依赖。
    -  存在过多的条件语句。
    说明：多层条件语句建议使用卫语句、策略模式、状态模式等方式重构。
16. <font color=#55AA77>【参考】</font>不要对单元测试存在如下误解：
    -  那是测试同学干的事情。本文是开发手册，凡是本文内容都是与开发同学强相关的。
    -  单元测试代码是多余的。系统的整体功能与各单元部件的测试正常与否是强相关的。
    -  单元测试代码不需要维护。一年半载后，那么单元测试几乎处于废弃状态。
    -  单元测试与线上故障没有辩证关系。好的单元测试能够最大限度地规避线上故障。

-------------------------

## 三、Spring boot单元测试框架

​		统一使用spring boot对应版本的单元测试框架，避免junit3、junit4、junit5各种测试框架混合使用，当前spring boot主要的单元测试框架是junit4，通过`mvn test`命令运行所有单元测试模块。同时，spring boot官方集成了mockito框架用于mock测试。（spring boot测试框架参考[spring boot 官方文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-testing)，mockito测试框架参考[mockito官方文档](https://site.mockito.org)）

​		首先引入依赖如下依赖。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

### 1.测试注解使用

目前支持的主要注解有：

- `@BeforeClass` 全局只会执行一次，而且是第一个运行
- `@Before` 在测试方法运行之前运行
- `@Test` 测试方法
- `@After` 在测试方法运行之后允许
- `@AfterClass` 全局只会执行一次，而且是最后一个运行
- `@Ignore` 忽略此方法

​         执行次序是`@BeforeClass` -> `@Before` -> `@Test` -> `@After` -> `@Before` -> `@Test` -> `@After` -> `@AfterClass`。`@Ignore`会被忽略。

### 2.Assert类（断言的使用）

​		Junit4都提供了一个Assert类。Assert类中定义了很多静态方法来进行断言。

- assertTrue(String message, boolean condition) 要求condition == true
- assertFalse(String message, boolean condition) 要求condition == false
- fail(String message) 必然失败，同样要求代码不可达
- assertEquals(String message, XXX expected,XXX actual) 要求expected.equals(actual)
- assertArrayEquals(String message, XXX[] expecteds,XXX [] actuals) 要求expected.equalsArray(actual)
- assertNotNull(String message, Object object) 要求object!=null
- assertNull(String message, Object object) 要求object==null
- assertSame(String message, Object expected, Object actual) 要求expected == actual
- assertNotSame(String message, Object unexpected,Object actual) 要求expected != actual
- assertThat(String reason, T actual, Matcher matcher) 要求matcher.matches(actual) == true

### 3.Mock测试

​		Mock和Stub是两种测试代码功能的方法。Mock测重于对功能的模拟。Stub测重于对功能的测试重现。强烈建议优先选择Mock方式，因为Mock方式下，模拟代码与测试代码放在一起，易读性好，而且扩展性、灵活性都比Stub好。Spring boot提供了MockMvc有关的mock测试

#### (1)、MockMvc测试注解

（详细使用，参考：[官方指南](https://spring.io/guides/gs/testing-web/)、[文档指导](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html)）需要在测试类中增加如下注解。

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class MockXXXTest {
  
}
```

- **@RunWith(SpringRunner.class)**

  就是指用SpringRunner来运行，其中SpringJUnit4ClassRunner 和 SpringRunner 区别是什么？在官方文档中有如下这句话：“SpringRunner is an alias for the SpringJUnit4ClassRunner”。

- **@SpringBootTest**

  该注解是SpringBoot的一个用于测试的注解，通过SpringApplication在测试中创建ApplicationContext。

- **@AutoConfigureMockMvc**

  该注解是用于自动配置MockMvc。

附加：@Transactional：增加该注解实现数据回滚，可以避免数据污染；

​			@SpringBootTest( webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)，可以防止spring boot单元测试中的端口冲突。

​		依赖于application context的测试，还需要模拟WebApplicationContext，HttpServletRequest，HttpServletResponse，session等。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
public class MockXXXTest {
  	@Resource
    private WebApplicationContext webApplicationContext;

    private MockMvc mockMvc;
    private MockHttpServletRequest mockHttpServletRequest;
    private MockHttpServletResponse mockHttpServletResponse;
    protected MockHttpSession session;

    @Before
    public void setup() {
        mockHttpServletRequest = new MockHttpServletRequest(webApplicationContext.getServletContext());
        mockHttpServletResponse = new MockHttpServletResponse();
        MockHttpSession mockHttpSession = new MockHttpSession(webApplicationContext.getServletContext());
        mockHttpServletRequest.setSession(mockHttpSession);
        mockMvc = MockMvcBuilders
                .webAppContextSetup(webApplicationContext)
                .build();
    }
}
```

#### (2)、使用MockMvc发送请求

​		mockmvc测试，主要测试是Controller层单元测试，可以模拟数据从浏览器请求到后端整个mvc过程逻辑。

```java
//配置MockMvc
@Autowired
protected MockMvc mockMvc;

@Test
public void TestXXX() throws Exception {
       MvcResult result = mockMvc.perform(
                MockMvcRequestBuilders.get("/xxxController/xxx_query")
                        .contentType(MediaType.APPLICATION_JSON_UTF8)      
                        .param("xxx","xxx")
        ).andExpect(MockMvcResultMatchers.status().isOk())
         .andDo(MockMvcResultHandlers.print())
         .andReturn();
       
    }
｝
```

#### (3)、MockBean（@SpyBean）模拟相应对象

​		这里的主要作用是：使用mock对象代替原来spring的bean，然后模拟底层数据的返回，而不是调用原本真正的实现。

​		**注**：与@MockBean 对应的还有@SpyBean。 @SpyBean与 @Spy 的关系类似于 @MockBean 与 @Mock 的关系。和 @MockBean 不同的是，它不会生成一个 Bean 的替代品装配到类中，而是会监听一个真正的 Bean 中某些特定的方法，并在调用这些方法时给出指定的反馈。基于@SypBean的特性，可以构建依赖其他类、服务等的真实模拟，比如远程调用服务中的依赖。

​		如下示例，SpringBoot 中, @MockBean 会将mock的bean替换掉 SpringBoot 管理的原生bean，从而达到mock的效果。：

```java
public class MockXXXTest {

    @MockBean
    private XXXDao xxxtDao;
 
}
```

或者采用XXXDao mockBean = Mockito.mock(XXXDao.class)的代码方式模拟创建

​		下面是一个完整示例。

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class TenantServiceTest {
	@Test
    public void mockitoExampleGetTest() {
        //构造数据
        //设置期望数据
        TenantDetailRspDTO expectedResult = new TenantDetailRspDTO();
        expectedResult.setTenantDTO(new TenantDTO().setTenantId("0000"));
        expectedResult.setTenantParameterDTO(new TenantParameterDTO().setTenantId("0000"));
        expectedResult.setTenantConfigDTO(new TenantConfigDTO().setTenantId("0000"));

        //创建一个mock对象，或者通过属性注解的方式SpringBoot封装框架使用@MockBean，当对ApplicationContext有依赖时可以使用；原生mockito框架使用@Mock
        TenantService mockBean = Mockito.mock(TenantService.class);

        //创建一个测试桩stub，定义在service层该方法的返回值定义。
        Mockito.when(mockBean.getTenantDetail("0000")).thenReturn(expectedResult);

        //执行方法，使用模拟对象
        TenantDetailRspDTO actualResult = mockBean.getTenantDetail("0000");

        //验证调用方法是否执行过
        Mockito.verify(mockBean).getTenantDetail("0000");

        //断言
        Assert.isTrue(actualResult.getTenantDTO().getTenantId() == "0000", ApiErrorCode.FAILED);

    }
}
```

#### (4)、单元测试覆盖率

##### 1.代码覆盖率的意义。

代码覆盖率统计是用来发现没有被测试覆盖的代码；代码覆盖率统计不能完全用来衡量代码质量。

- 分析未覆盖部分的代码，从而反推在前期测试设计是否充分，没有覆盖到的代码是否是测试设计的盲点，为什么没有考虑到？需求/设计不够清晰，测试设计的理解有误，工程方法应用后的造成的策略性放弃等等，之后进行补充[**测试用例**](javascript:;)设计。
- 检测出程序中的废代码，可以逆向反推在代码设计中思维混乱点，提醒设计/开发人员理清代码逻辑关系，提升代码质量。
- 代码覆盖率高不能说明代码质量高，但是反过来看，代码覆盖率低，代码质量不会高到哪里去，可以作为测试自我审视的重要工具之一。

##### 2.使用工具

​		开发人员可以通过idea自身的测试代码覆盖率功能进行自测。在代码覆盖率测试中，构造一个高质量的测试数据，可以覆盖80%~90%以上的逻辑，开发人员可以导出测试覆盖率详细数据，根据核心功能的要求，构造合适的数据，进行逻辑功能的覆盖。

---------------------------------

## 四、Junit测试代码编写命名规范

使用测试idea提供的自动生成测试模板为样例。

**1.测试类的命名定义规范**

​		Junit自动生成测试类的命名如下：被测试的业务+Test、被测试的接口+Test、被测试的类+Test

**2.测试用例的命名定义规范**

​		测试用例的命名规则是：test+例操作名。避免使用test1、test2没有含义的名称。其次需要有必要的函数方法注释。

**3.测试程序的包名定义规范**

- 测试程序包的命名规则是：<公司域名>.<子级组织部门缩写>.<项目名>.<模块名>；
- 测试公共类包的命名规则是：<公司域名>.<子级组织部门缩写>.<项目名>.common；
- java包的名称都是由小写字母组成。
- 测试开发包，测试包保持和被测包一致。

**4.变量的命名规范**

​		保持跟开发规范一致

**5.常量的命名规范**

​		保持跟开发规范一致
