
mock 对象的验证行为的基本操作，也可以对行为的次数进行验证。

mock的内核不是对一个复杂对象做一个stub模拟，而是结合自己的业务逻辑，验证mock对象是不是做出了自己预期的行为

```java
   			List<String> mockedList = mock(List.class);

        // 对mock开始预测操作
        mockedList.add("one");
        mockedList.clear();

        // 对之前的预测进行验证
        verify(mockedList).add("one");
        verify(mockedList).clear();

        // 预测操作 ，同时指定行为，对mock对象指定行为
        when(mockedList.get(0)).thenReturn("0");
        assertEquals("0", mockedList.get(0));

        // 如果预测行为的时候，不知道具体的参数，可以使用参数匹配器
        when(mockedList.get(anyInt())).thenReturn("element");
        assertEquals("element", mockedList.get(999));

        // 如果一个行为存在多个添加次数，可以对指定的次数判断行为 , 判断重复执行的次数
        mockedList.add("once");
        verify(mockedList, times(1)).add("once");
        verify(mockedList, never()).add("never happened");

        // 可以对次数模糊确定
        verify(mockedList, atLeastOnce()).add("once");
```

可以对 mock 对象的执行顺序一一验证

```java
        // 验证执行的顺序
        List<String> singleMock = mock(List.class);
        singleMock.add("was added first");
        singleMock.add("was added second");
        InOrder inOrder = inOrder(mockedList);
        inOrder.verify(singleMock).add("was added first");
        inOrder.verify(singleMock).add("was added second");
        // 验证多个对象的顺序
        List<String> firstMock = mock(List.class);
        List<String> secondMock = mock(List.class);
        firstMock.add("first add");
        secondMock.add("second add");
        inOrder = inOrder(firstMock, secondMock);
        inOrder.verify(firstMock).add("first add");
        inOrder.verify(secondMock).add("second add");
```

验证mock对象是不是被使用过了，或者是不是还存在没有验证过的操作

```java
      	List<String> neverUsedMock = mock(List.class);
        // 验证一次都没有使用
        verifyZeroInteractions(neverUsedMock);
        neverUsedMock.add("one");
        neverUsedMock.add("two");
        verify(neverUsedMock).add("one");
        // 验证还存在没有验证的操作 - fail 
        // verifyNoMoreInteractions(neverUsedMock);
```

一个方法按照顺序返回结果

```java
        // 按照顺序返回，第一次 true  第二次 false
        when(mockedList.add("next"))
                .thenReturn(true, false);
        assertTrue(mockedList.add("next"));
        assertFalse(mockedList.add("next"));
```

[https://github.com/hehonghui/mockito-doc-zh](https://github.com/hehonghui/mockito-doc-zh)