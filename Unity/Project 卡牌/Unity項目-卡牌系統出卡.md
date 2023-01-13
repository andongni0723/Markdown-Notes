# Unity卡牌系統項目-出卡

[toc]
---
## 流程圖

### 大流程

 ![](https://i.imgur.com/wGFRFG6.png)
```mermaid
sequenceDiagram
    客戶端->>BasicCard.cs: 打出卡牌
    BasicCard.cs->>BasicCard.cs:EventAtCardEndDrag()
    BasicCard.cs->>EventHandler.cs: EventHanlder.CallCardEndDrag()
    
    EventHandler.cs->>GridMouseManager.cs: Action 關閉Mouse判定
    EventHandler.cs->>BasicCard.cs: Func<CardDetail_SO> EndDragCardUpdateData
    BasicCard.cs-->>EventHandler.cs: return 'CardDetail_SO'
    EventHandler.cs->>GridManager.cs: Func<List<ConfirmGrid>> EndDragGridUpdateData
    GridManager.cs-->>EventHandler.cs: return 'List<ConfirmGrid>'
    EventHandler.cs->>GameManager.cs: Data
    
    Note right of GameManager.cs: "Next..."
```


### 小流程