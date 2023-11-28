
```java
import static org.easymock.EasyMock.*;   
import static org.powermock.api.easymock.PowerMock.*;  

@RunWith(PowerMockRunner.class)   
@PrepareForTest({AuthorityContext.class, AdaptUtils.class})
```


对于一般的mock需求 ， 使用 easyMock 就可以完成
先 mock ,然后 expect 设置预期行为和次数 ，然后 replay  , 开始到真实的测试逻辑里面看看 ， 最后验证一下是不是符合需求 verify

涉及到静态变量 ，使用 powerMock

```java
DecisionCacheController cacheController = mock(DecisionCacheController.class);  
mockStatic(DecisionCacheMonitor.class);  
expect(DecisionCacheController.getInstance()).andReturn(cacheController).anyTimes();
```






