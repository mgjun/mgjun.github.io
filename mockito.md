## Mockito

#### Mock

​		Mock生成的mock类，其中所有的方法都不会真实的执行，而且返回值都是NULL。只能调用打桩(stubs)方法，调用不了真实的方法

```java
public class MockTestService {
    public String test() {
        System.out.println("mock method called");
        return "method mock real return";
    }
}
```

```java
// MockTestService对应的测试类
@ExtendWith(MockitoExtension.class)
class MockTestServiceTest {
    @Mock
    private MockTestService mockTestService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    //直接调用mock会获取到null值
    @Test
    void test() {
        assertNull(mockTestService.test());
    }

    // 不执行真实的方法，并返回"mock"
    @Test
    void test_do_return() {
        doReturn("mock").when(mockTestService).test();
        assertEquals("mock",mockTestService.test());
    }

    // 不执行真实的方法，并返回"mock"
    @Test
    void test_when() {
        when(mockTestService.test()).thenReturn("mock");
        assertEquals("mock", mockTestService.test());
    }

}
```

####Spy

​		spy可以监视真实的对象，可以对真实的对象打桩，也可以执行对象的真实方法

```java
public class SpyTestService {
    public String test() {
        System.out.println("spy method called");
        return "method spy real return";
    }
}
```

```java
// SpyTestService的测试类
@ExtendWith(MockitoExtension.class)
class SpyTestServiceTest {

    @Spy
    private SpyTestService spyTestService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    // 调用真实方法，在控制台输出"spy method called"，并返回真实结果
    @Test
    void test() {
        assertEquals("method spy real return", spyTestService.test());
    }
    
    // 不调用真实方法，返回mock值
    @Test
    void test_do_return() {
        doReturn("spy return").when(spyTestService).test();
        assertEquals(spyTestService.test(), "spy return");
    }

    // 调用真实方法，在控制台输出"spy method called"，并返回mock值
    @Test
    void test_when() {
        when(spyTestService.test()).thenReturn("spy return");
        assertEquals(spyTestService.test(), "spy return");
    }
}
```

#### InjectMocks

​		在mockito中进行依赖注入,@InjectMocks创建对象，它会根据类型来注入对象里面的成员方法和变量。

```java
public class InjectMocksTestService {
    
    private InjectMockDao injectMockDao;

    public String test() {
        System.out.println("InjectMocks method called");
        return injectMockDao.callDb();
    }

    public void setInjectMockDao(InjectMockDao injectMockDao) {
        this.injectMockDao = injectMockDao;
    }
}
```

```java
public class InjectMockDao {
    public String callDb() {
        return "db return";
    }
}
```

```java
@ExtendWith(MockitoExtension.class)
class InjectMocksTestServiceTest {
    
    @Mock
    private InjectMockDao injectMockDao;
    @InjectMocks
    public InjectMocksTestService injectMocksTestService;
    
    @Test
    void test() {
        when(injectMockDao.callDb()).thenReturn("mock db return");
        
        String result = injectMocksTestService.test();
        
        assertEquals("mock db return",result);
    }
}
```

#### 参数捕捉

​	在verification期间获取方法的参数，获取到参数之后可以对其进行测试

```java
@ExtendWith(MockitoExtension.class)
class ArgumentCaptorTest {

    @Captor
    private ArgumentCaptor<List<String>> captor;

    @Test
    void test() {
        List<String> lists = Arrays.asList("ele1", "ele2");
        List<String> mockList = mock(List.class);
        mockList.addAll(lists);

        verify(mockList).addAll(captor.capture());
        List<String> captorValue = captor.getValue();
        assertEquals(Arrays.asList("ele1", "ele2"), captorValue);
    }
}
```

#### Mockito的限制

​	以下三种类型不能被测试:

- final class

- anonymous classes
- primitive types

#### 参考资料

- [Mockito 官网](http://site.mockito.org)
- [Mockito github](https://github.com/mockito/mockito)
- [martinfowler 关于mock vs stub的文章](http://martinfowler.com/articles/mocksArentStubs.html)