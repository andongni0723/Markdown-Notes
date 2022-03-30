# **Unity事件邏輯**

---

[toc]



這個邏輯是在當發生某一事件時，呼叫 自身腳本的**On函數**去調用發生後的事情。



## 新增*EventHandler.cs*

在*EventHandler.cs*內添加**新事件**和**呼叫事件的函數**。

> 在事件Component中可以填**超過1個Component**。

EventHandler.cs

```csharp
public static class EventHandler
{
    public static event Action<Component1> Event01;
    public static void CallEvent01(Component1 com1)
    {
        Event01?.Invoke(com1);
        // 這"?"是檢查此事件是否為null
        //
        // ▼▼▼ 下面的代碼是原始碼 ▼▼▼
        // if(Event01 == null)
        //     Event01.Invoke(com1);
    }
}
```



## 在自身腳本內添加*On函數*

在*A腳本*內添加**事件發生後要執行的程式**。

A.cs

```csharp
private void OnEvent01Happen(Component1 com1)
{
    //裡面添加要執行的程式
}
```



## 在自身腳本內把*On函數*添加進事件中

>注意是'Event01' 不是'CallEvent01'。

A.cs

```cs
private void OnEnable()
{
    // ▼ 注意是'Event01' 不是'CallEvent01'。
    EventHandler.Event01 += OnEvent01Happen;
}

private void OnDisable()
{
    EventHandler.Event01 -= OnEvent01Happen;
}
```

**這樣功能就完成了。**
