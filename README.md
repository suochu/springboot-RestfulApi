拆分功能
實踐
練習表達
論壇問問題,chatgpt,開發者交流
## 使用 springboot 開發 RESTful API 
1. 什麼是 API？
     - 使用API過程如同點菜(Input)、實作過程(Call API )、出餐點(Output)
     - 設計好API，前端可以透過Call API，獲取來自server資料庫的資料，並將資料render到頁面上
    
2. 什麼是 REST？
     - REST的無狀態特性是指後端server不會保留前端Http Request的狀態
     - 有狀態例子，例如：servlet A保留前端Http Request的狀態並forward到servlet B去
     - 思考有狀態有什麼的壞處？
      
5. 什麼是 RESTful API特性？
     - 無狀態、前後端分離
     - 前後端分離會碰什麼問題？
     
      
7. RESTful API 請求資源的方法
    - 是指基於Http協定創造方法與Server溝通要向資料庫進行哪些資料操作
    - Get request：獲取資料庫資料，常用於資料查詢
    - Post request：新增資料庫資料
    - Put request：資料庫資料沒資料直接新增的修改
    - Patch request：修改資料庫部分欄位資料
    - Delete request:刪除資料庫資料


9. 回應狀態碼


11. 開發上差異
12. 如何實作 
13. 開發思考路徑：
    - 建立資料表 -> 寫entityspring data jpa @entity