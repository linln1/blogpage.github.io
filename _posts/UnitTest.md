---
layout: post
---

摘要
<!--more-->
<!-- CreateTime:2020/6/25 23:54:02 -->


<div id="toc"></div>

#Java笔记


###JUnit

###Fixture
    public class CalculatorTest {

        Calculator calculator;

        @BeforeEach
        public void setUp() {
            this.calculator = new Calculator();
        }

        @AfterEach
        public void tearDown() {
            this.calculator = null;
        }

        @Test
        void testAdd() {
            assertEquals(100, this.calculator.add(100));
            assertEquals(150, this.calculator.add(50));
            assertEquals(130, this.calculator.add(-20));
        }

        @Test
        void testSub() {
            assertEquals(-100, this.calculator.sub(100));
            assertEquals(-150, this.calculator.sub(50));
            assertEquals(-130, this.calculator.sub(-20));
        }
    }

        //invokeBeforeAll(CalculatorTest.class);
        for(Method testMethod : findTestMethods(CalculatorTest.class)){
            var test = new CalculatorTest();
            invokeBeforeEach(test):
                invokeTestMethod(test, testMethod);
            invokeAfterEach(test);
        }
        //invokeAfterAll(Calculator.class);

        public class DatabaseTest{
            static Database db;

            @BeforeAll
            public static void initDatabase(){
                db = createDb();
            }

            @AfterAll
            public static void dropDatabase(){
                ...
            }
        }

####1)对于实例变量，在@BeforeEach中初始化，在@AfterEach中清理，它们在各个@Test方法中互不影响，因为是不同的实例；

####2)对于静态变量，在@BeforeAll中初始化，在@AfterAll中清理，它们在各个@Test方法中均是唯一实例，会影响各个@Test方法。

###异常测试
####JUnit测试中，编写一个@Test的方法专门测试异常
    @Test
    void testNegative(){
        assertThrows(IllegalArgumentException class, new Executable(){
            @Override
            public void execute() throws Throwable{
                Factorial.fact(-1);
            }
        })
    }

####编写匿名类有点麻烦，所以使用函数式编程，单接口方式可以简写如下
    @Test
    void testNegative(){
        assertThrows(IllegalArgumentException.class, ()->{
            Factorial.fact(-1);
        })
    }

###条件测试
    @Disable
    @Test
    void testBug01{}//这个测试不会运行

    @Test
    @EnableOnOs(OS.WINDOWS)
    void testwindows(){
        assertEquals("C:\\test.ini", config.getConfigFile("test.ini"));
    }

    public class Config{
        public String getConfigFile(String filename){
            String os = System.getProperty("os.name").toLowerCase();
            if(os.contains("win")){
                return "C:\\" + filename;
            }
            if(os.contains("mac") || ...){
                return "/usr/local/" + filename;
            }
            throw new UnsupportedOperationException();
        }
    }

    @DisableOnJre(JRE.JAVA_8)

###参数化测试
    参数化测试和普通测试稍微不同的地方在于，测试方法至少要接受一个参数，然后传入一组数反复运行
    @ParameterizedTest
    @ValueSource(ints = {0, 1, 5, 100})
    void testAbs(int x){
        assertsEquals(x, Math.abs(x));
    }

    不仅要给输入，还要给出预期输出
    那么参数该如何输入？使用一个@MethodSource注解，编写一个同名的静态方法来提供测试参数
    @MethodSource
    void testCapitalize(String input, String results){
        assertEquals(results, StringUtils.capitalize(input));
    }

    static List<Arguments> testCapitalize(){
        return List.of(
            Arguments.arguments("abc", "Abc"),
            Arguments.arguments("APPLE", "Apple"),
            Arguments.arguments("gooD", "Good")
        );
    }

    @ParameterizedTest
    @CsvFileSource(resources = "/test-capitalize.csv")
    
