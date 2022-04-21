# **Unity 傳送系統**

---

![](D:\User_Andongni\Desktop\MD筆記\Markdown-Notes\PNG\transition PNG\transition-show.gif)

[toc]



## 新增 *Tansition Manager*

![Add the "Tansition Manager"](https://drive.google.com/file/d/1KjSeUL47gw6KQBQyB2gHFDzeBmgVCsl7/view?usp=sharing)



## 新增 *TansitionManager.cs*

![Add the "TansitionManager.cs"](D:\User_Andongni\Desktop\MD筆記\Markdown-Notes\PNG\transition PNG\transition2.png)

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace AnFarm.Transition
{
    public class TransitionManager : MonoBehaviour
    {
        public string startSceneName = string.Empty;

        private void Start()
        {
            StartCoroutine(LoadSceneSetActive(startSceneName));
        }

        /// <summary>
        /// Load scene and set active
        /// </summary>
        /// <param name="sceneName">Scene Name</param>
        /// <returns></returns>
        private IEnumerator LoadSceneSetActive(string sceneName)
        {
            yield return SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);

            Scene newSceme = SceneManager.GetSceneAt(SceneManager.sceneCount - 1);

            SceneManager.SetActiveScene(newSceme);
        }
    }
}

```



![](D:\User_Andongni\Desktop\MD筆記\Markdown-Notes\PNG\transition PNG\transition3.png)  在 StartSceneName 填入開始遊戲的場景



### 在TransitionManager.cs 添加更多程式碼

```c#
// 添加在 Start() 下面

	    /// <summary>
        /// Change Scene
        /// </summary>
        /// <param name="sceneName">Scene Name</param>
        /// <param name="targetPosition">Target Position</param>
        /// <returns></returns>
        private IEnumerator Transition(string sceneName, Vector3 targetPosition)
        {
            EventHandler.CallBeforeSceneUnloadEvent();

            yield return SceneManager.UnloadSceneAsync(SceneManager.GetActiveScene());

            yield return LoadSceneSetActive(sceneName);

            EventHandler.CallAfterSceneLoadedEvent();
        }
```



## 添加地圖*傳送點*

![](D:\User_Andongni\Desktop\MD筆記\Markdown-Notes\PNG\transition PNG\transition4.png)



## 新增*Teleport.cs* 並放在 *傳送點* 上

![](D:\User_Andongni\Desktop\MD筆記\Markdown-Notes\PNG\transition PNG\transition6.png)

## 編寫*Teleport.cs*

```c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace AnFarm.Transition
{
    public class Teleport : MonoBehaviour
    {
        public string sceneToGo;
        public Vector3 positionToGo;

        private void OnTriggerEnter2D(Collider2D other)
        {
            if(other.CompareTag("Player"))
            {
                
            }
        }
    }
}
```



### 新增事件

用[Unity 事件](UnityEvent-Logic)語法新增**一個事件**

**EventHandler.cs**

```c#
public static event Action<string, Vector3> TransitionEvent;
public static void CallTransitionEvent(string sceneToGo, Vector3 positionToGo)
{
    TransitionEvent?.Invoke(sceneToGo, positionToGo);
}
```



### 回到 *Teleport.cs*並添加更多程式碼

```c#
// 添加在'If(other.CompareTag("Player"))'下面
EventHandler.CallTransitionEvent(sceneToGo, positionToGo);
```



### 在 'PositionToGo'  中填上傳送後的位置

![](D:\User_Andongni\Desktop\MD筆記\Markdown-Notes\PNG\transition PNG\transition7.png)



## 在*TransitionManager.cs* 添加更多程式碼

``` c#
// 添加在 'public string startSceneName = string.Empty;' 下面

private void OnEnable()
{
    EventHandler.TransitionEvent += OnTransitionEvent;
}

private void OnDisable() {
    EventHandler.TransitionEvent -= OnTransitionEvent;
}

private void OnTransitionEvent(string sceneToGo, Vector3 positionToGo)
{
    StartCoroutine(Transition(sceneToGo, positionToGo));
}
```



### 新增事件

用[Unity 事件](UnityEvent-Logic)語法新增**二個事件**

**EventHandler.cs**

```c#
public static event Action BeforeSceneUnloadEvent;
public static void CallBeforeSceneUnloadEvent()
{
    BeforeSceneUnloadEvent?.Invoke();
}

public static event Action AfterSceneLoadedEvent;
public static void CallAfterSceneLoadedEvent()
{
    AfterSceneLoadedEvent?.Invoke();
}
```



### 回到 *Teleport.cs*並添加更多程式碼

新增兩行程式

```c#
private IEnumerator Transition(string sceneName, Vector3 targetPosition)
        {
            EventHandler.CallBeforeSceneUnloadEvent(); // 新增

            yield return SceneManager.UnloadSceneAsync(SceneManager.GetActiveScene());

            yield return LoadSceneSetActive(sceneName);

            EventHandler.CallAfterSceneLoadedEvent();  // 新增
        }
```

> 可以在需要傳送事件的腳本中添加事件



### 新增事件

用[Unity 事件](UnityEvent-Logic)語法新增**一個事件**

**EventHandler.cs**

```c#
public static event Action<Vector3> MoveToPosition;
public static void CallMoveToPosition(Vector3 targetPosition)
{
	MoveToPosition?.Invoke(targetPosition);
}
```



### 回到 *Teleport.cs*並添加更多程式碼

新增一行程式

```c#
private IEnumerator Transition(string sceneName, Vector3 targetPosition)
{
    EventHandler.CallBeforeSceneUnloadEvent();

    yield return SceneManager.UnloadSceneAsync(SceneManager.GetActiveScene());

    // Move player position
    EventHandler.CallMoveToPosition(targetPosition); // 新增
    
    yield return LoadSceneSetActive(sceneName);

    EventHandler.CallAfterSceneLoadedEvent();
}
```



## 在 *Player.cs* 添加更程式碼

``` c#
// <新增>
private bool inputDisable;

private void OnEnable()
{
    EventHandler.BeforeSceneUnloadEvent += OnBeforeSceneUnloadEvent;
    EventHandler.AfterSceneLoadedEvent += OnAfterSceneLoadedEvent;
    EventHandler.MoveToPosition += OnMoveToPosition;
}

private void OnDisable()
{
    EventHandler.BeforeSceneUnloadEvent -= OnBeforeSceneUnloadEvent;
    EventHandler.AfterSceneLoadedEvent -= OnAfterSceneLoadedEvent;
    EventHandler.MoveToPosition -= OnMoveToPosition;
}

private void OnMoveToPosition(Vector3 targetPosition)
{
    transform.position = targetPosition;
}

private void OnAfterSceneLoadedEvent()
{
    inputDisable = false;
}

private void OnBeforeSceneUnloadEvent()
{
    inputDisable = true;
}
// <新增/>

private void Update()
{
    if (inputDisable == false) // 新增
        PlayerInput();		  // 新增
    
    SwitchAnimator();
}
```



# 總結

---

## TransitionManager.cs

``` c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace AnFarm.Transition
{
    public class TransitionManager : MonoBehaviour
    {
        public string startSceneName = string.Empty;

        private void OnEnable() {
            EventHandler.TransitionEvent += OnTransitionEvent;
        }

        private void OnDisable() {
            EventHandler.TransitionEvent -= OnTransitionEvent;
        }

        private void Start()
        {
            StartCoroutine(LoadSceneSetActive(startSceneName));
        }

        private void OnTransitionEvent(string sceneToGo, Vector3 positionToGo)
        {
            StartCoroutine(Transition(sceneToGo, positionToGo));
        }

        /// <summary>
        /// Change Scene
        /// </summary>
        /// <param name="sceneName">Scene Name</param>
        /// <param name="targetPosition">Target Position</param>
        /// <returns></returns>
        private IEnumerator Transition(string sceneName, Vector3 targetPosition)
        {
            EventHandler.CallBeforeSceneUnloadEvent();

            yield return SceneManager.UnloadSceneAsync(SceneManager.GetActiveScene());

            yield return LoadSceneSetActive(sceneName);

            // Move player position
            EventHandler.CallMoveToPosition(targetPosition);

            EventHandler.CallAfterSceneLoadedEvent();
        }

        /// <summary>
        /// Load scene and set active
        /// </summary>
        /// <param name="sceneName">Scene Name</param>
        /// <returns></returns>
        private IEnumerator LoadSceneSetActive(string sceneName)
        {
            yield return SceneManager.LoadSceneAsync(sceneName, LoadSceneMode.Additive);

            Scene newSceme = SceneManager.GetSceneAt(SceneManager.sceneCount - 1);

            SceneManager.SetActiveScene(newSceme);
        }
    }
}
```



## Teleport.cs

``` c#
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace AnFarm.Transition
{
    public class Teleport : MonoBehaviour
    {
        public string sceneToGo;
        public Vector3 positionToGo;

        private void OnTriggerEnter2D(Collider2D other)
        {
            if(other.CompareTag("Player"))
            {
                EventHandler.CallTransitionEvent(sceneToGo, positionToGo);
            }
        }
    }
}
```

## EventHandler.cs

``` c#
public static event Action<string, Vector3> TransitionEvent;
    public static void CallTransitionEvent(string sceneToGo, Vector3 positionToGo)
    {
        TransitionEvent?.Invoke(sceneToGo, positionToGo);
    }

    public static event Action BeforeSceneUnloadEvent;
    public static void CallBeforeSceneUnloadEvent()
    {
        BeforeSceneUnloadEvent?.Invoke();
    }

    public static event Action AfterSceneLoadedEvent;
    public static void CallAfterSceneLoadedEvent()
    {
        AfterSceneLoadedEvent?.Invoke();
    }

    public static event Action<Vector3> MoveToPosition;
    public static void CallMoveToPosition(Vector3 targetPosition)
    {
        MoveToPosition?.Invoke(targetPosition);
    }
```



## Player.cs

```c#
private void OnEnable()
    {
        EventHandler.BeforeSceneUnloadEvent += OnBeforeSceneUnloadEvent;
        EventHandler.AfterSceneLoadedEvent += OnAfterSceneLoadedEvent;
        EventHandler.MoveToPosition += OnMoveToPosition;
    }

    private void OnDisable()
    {
        EventHandler.BeforeSceneUnloadEvent -= OnBeforeSceneUnloadEvent;
        EventHandler.AfterSceneLoadedEvent -= OnAfterSceneLoadedEvent;
        EventHandler.MoveToPosition -= OnMoveToPosition;
    }

    private void OnMoveToPosition(Vector3 targetPosition)
    {
        transform.position = targetPosition;
    }

    private void OnAfterSceneLoadedEvent()
    {
        inputDisable = false;
    }

    private void OnBeforeSceneUnloadEvent()
    {
        inputDisable = true;
    }

    private void Update()
    {
        if (inputDisable == false)
            PlayerInput();
    }
```





