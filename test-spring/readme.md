# Testing (without @SpringBootTest)

@RunWith(SpringRunner.class) is used to provide a bridge between Spring Boot test features and JUnit. Whenever we are using any Spring Boot testing features in out JUnit tests, this annotation will be required.

Difference: 
- @ContextConfiguration is an annotation from the Spring Test Framework, which is suitable for every Spring application, 
- (removed) @SpringApplicationConfiguration is from Spring Boot and is actually a composite annotation, which includes ContextConfiguration with the custom SpringApplicationContextLoader as loader.

Configuration db

1. create database see `src/main/resources/db.sql`
2. `cp src/main/resources/application.properties.dist src/main/resources/application.properties`
3. `cp src/test/resources/application.properties.dist src/test/resources/application.properties`
4. set db config in application.properties

## Unit testing POJO component - bare jUnit, Mockito (no Spring config, no @Autowired)

see Test:
```
/src/test/java/bare_junit/EmployeeTest.java
```

## Unit testing with Spring annotation (ex.: @Autowired) but with mocking

#### Use @RunWith(SpringRunner.class) with @Mock

!Important, when use SpringRunner.class - spring context will be loaded

see Test:  
```
/src/test/java/unit/mockito_spring_runner/MyServiceTest.java
```

```
@Mock
private Data mockData;

@InjectMocks
private MyService service;

@Before
public void setUp() {
    MockitoAnnotations.initMocks(this);
}

@Test
public void getMainData() {
    when(mockData.getInfo()).thenReturn("ahaha");
    String info = service.getMainData();
    assertThat(info, is("ahaha"));
}
```

but if in your service there are @Autowired classes that you do not want to get wet - 
then you need to remember to add @Autowired for @InjectMocks.

See example: `/src/test/java/unit/mockito_spring_runner/MyServiceWithTwoDepsTest.java`  
When use class ConcatStr for non-mocking

```
@Mock
private Data mockData;

@Autowired
@InjectMocks
private MyServiceWithTwoDeps service;

@Before
public void setUp() {
    MockitoAnnotations.initMocks(this);
}

@Test
public void getMainData() {
    when(mockData.getInfo()).thenReturn("ahaha");
    String info = service.getMainData();
    assertThat(info, is("ahaha"));
}
```

#### Use @RunWith(MockitoJUnitRunner.class) with @Mock (the best)

!Important, when use MockitoJUnitRunner.class - spring context will not be loaded

see Test:  
```
/src/test/java/unit.mockito_junit_runner/MyServiceTest.java
```

#### Set manual mock in config bean with scan

see Test:
```
/src/test/java/unit/mockito_bean_with_scan/MyServiceTest.java
```

#### Set manual mock in config bean without scan

see Test:
```
/src/test/java/unit/mockito_bean_without_scan/MyServiceTest.java
```

## Integration test without mocking - get dependencies from Spring

For create test with spring features:
1. `@RunWith(SpringRunner.class)` is used to provide a bridge between Spring Boot test features and JUnit. Whenever we are using any Spring Boot testing features in out JUnit tests, this annotation will be required.
2. Add `ConfigClass` - where configuration @Bean (or use main Application for scan all package)

Example `ConfigClass`:
```
@Configuration
@ComponentScan("hello.simplelogic")
public class MyServiceTestConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

#### Use main Application for scan (without ConfigClass)

`@ContextConfiguration(classes = Application.class)`  
see `/src/test/java/integration/with_config_main_package/EmployeeDaoImplTest.java`

#### Local package scan Config

see `/src/test/java/integration.with_config_local/MyServiceTest.java`

#### Global package scan Config

see `/src/test/java/integration.with_config_main_package/EmployeeDaoImplTest.java`

#### Import common config file 

So as not to create different configuration files for all tests - you can create common for many tests.   
Or create an abstract test class with a configuration class

see `/src/test/java/integration.spring_integration_with_common_config/MyServiceTest.java`