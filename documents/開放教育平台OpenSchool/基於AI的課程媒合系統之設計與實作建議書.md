# 基於 AI 的課程媒合系統之設計與實作建議書 v1.0

## 1. 背景與需求

開放教育平台（Open School）旨在促進教師與學生之間的高效媒合，建立一個以知識傳承和教學為核心的生態系統。通過 AI 技術，提升教師課程與學生需求的精確匹配，實現自動化課程推薦和學習追蹤的功能，從而提高教育效能並確保資源的合理分配。

## 2. 平台功能與挑戰

### 平台功能包括：
1. **教師註冊與課程創建**：教師可以註冊並創建課程，設置內容、時間、堂數等信息。
2. **學生報名與參與**：學生可以瀏覽和報名課程，系統支持課程的追蹤與學習進度管理。
3. **課程管理與學習追蹤**：提供教師對課程的管理工具，並追蹤學生的報名和學習歷史。

### 挑戰：
1. **課程與學生需求匹配的困難**：教師經常無法找到足夠的學生來支持課程開設，學生也難以迅速找到符合其學習興趣的課程。
2. **系統自動化不足**：目前的匹配和推薦功能主要依賴手動操作，尚未整合 AI 技術來實現自動化的課程推薦，效率仍需提高。

## 3. 解決方案與技術規劃

### 3.1 AI 媒合系統架構設計

透過 AI 分析學生的學習歷史、興趣與教師的課程需求，實現學生和課程的自動化匹配。系統將動態追蹤學生的學習偏好，並生成個性化的課程推薦列表。

### 3.2 資料庫設計與管理

基於 NopCommerce 的開源架構，設計和管理系統的資料庫，確保課程、學生、學習歷史數據能夠進行精確的分析與推薦。

#### 資料庫關鍵設計
- **使用者資料表**（Customer）：存儲所有教師和學生的基本信息，追蹤他們的活動和偏好。
- **課程資料表**（Product）：存儲所有課程的詳細資料，包括課程名稱、描述、價格等。
- **訂單歷史資料表**（Order）：記錄學生的報名與學習歷史，便於系統進行進一步分析。

### 3.3 多層 AI 結構設計

系統引入了多角色 AI 架構，以提升效率並確保推薦的準確性。每個 AI 角色負責特定任務，分工合作，實現精確匹配。

#### 角色 1：需求判斷與分揀
**任務**：初步判斷需求方向，並將其分揀至合適的查詢路徑。  
- **情境 1**：教師尋找學生，角色 1 進行初步篩選，呼叫後續 AI 進行學生推薦。
- **情境 2**：學生尋找課程，角色 1 進行課程篩選，並將需求傳遞至角色 2。

**System Prompt**：
"請判斷需求是教師尋找學生還是學生尋找課程，並根據結果進行分揀，進入下一步查詢。"

角色 2：數據查詢與篩選
任務：根據角色 1 的判斷進行具體的數據查詢，例如篩選符合學生需求的課程，或查詢符合教師需求的學生。
System Prompt：
"根據需求，從資料庫中查詢相應的學生或課程，並返回符合條件的結果。"

角色 3：結果檢查與優化
任務：檢查角色 2 查詢的結果，確保數據的正確性，並根據匹配度進行優化，進一步縮小查詢範圍或進行推薦優化。
System Prompt：
"檢查查詢結果並根據需求進行優化，生成精確的匹配列表。"

角色 4：最終推薦與審核
任務：將角色 3 優化過的結果呈現給用戶，提供最終的推薦並解釋選擇依據。
System Prompt：
"基於優化結果，向用戶生成推薦列表，並解釋選擇背後的依據。"

### 3.4 AI 自動化與用戶體驗優化
AI 聊天機器人：即時解答教師與學生的問題，並提供相關課程或學生的推薦。
LINE 推送通知整合：通知教師和學生最新的課程報名情況與學習進度，提升用戶體驗。

## 4. SQL 查詢範例
這裡提供幾個常用的 SQL 查詢範例，幫助教師找到潛在的學生，並優化課程推薦。

### 4.1 查詢曾經報名過某教師課程的學生
SELECT C.FirstName, C.LastName, C.Email, O.CreatedOnUtc, P.Name AS CourseName
FROM [Order] O
JOIN [Customer] C ON O.CustomerId = C.Id
JOIN [Product] P ON O.Id = P.Id
WHERE P.VendorId = @TeacherId AND O.OrderStatusId = 2;

### 4.2 查詢對特定課程主題有興趣的學生
SELECT DISTINCT C.FirstName, C.LastName, C.Email
FROM [Order] O
JOIN [Customer] C ON O.CustomerId = C.Id
JOIN [Product] P ON O.Id = P.Id
WHERE P.Name LIKE '%主題%' AND O.OrderStatusId = 2;

### 4.3 查詢活躍學生
SELECT C.FirstName, C.LastName, C.Email, C.LastActivityDateUtc
FROM [Customer] C
WHERE C.LastActivityDateUtc >= DATEADD(DAY, -30, GETDATE()) AND C.Active = 1;

### 4.4 查詢報名課程最多的學生
SELECT C.FirstName, C.LastName, C.Email, COUNT(O.Id) AS TotalCourses
FROM [Order] O
JOIN [Customer] C ON O.CustomerId = C.Id
WHERE O.OrderStatusId = 2
GROUP BY C.FirstName, C.LastName, C.Email
ORDER BY TotalCourses DESC;

### 4.5 查詢最近完成課程的學生
SELECT C.FirstName, C.LastName, C.Email, O.CreatedOnUtc
FROM [Order] O
JOIN [Customer] C ON O.CustomerId = C.Id
WHERE O.OrderStatusId = 2
AND O.CreatedOnUtc >= DATEADD(DAY, -30, GETDATE());

## 5. 系統開發與測試
開發階段：
設計並建立教師與學生需求的數據庫框架，實現 AI 自動化課程推薦功能。

測試階段：
進行小規模課程匹配測試，根據測試結果優化系統功能。

優化與擴展：
根據反饋進行算法調整，最終擴展至整個平台，並增加更多自動化功能。

## 6. 預期成果
課程媒合機制優化：提升教師課程與學生需求的匹配度，減少教師開課困難，促進學生參與課程。
自動化推薦系統：AI 技術將持續自動更新和優化推薦結果，減少人工干預，提升用戶體驗。
教師專業發展：幫助教師發展個人品牌，吸引更多學生參與，提高課程的可持續性。

## 7. 附錄：其他專題建議與技術提醒
模型選擇的建議
現有 LLM 模型處理複雜任務時可能效率不足，建議考慮使用如 Claude 3.5 等更高效的模型。

System Prompt 的撰寫
System Prompt 在與 LLM 互動時至關重要，建議具體描述欄位名稱、定義與用途，以減少模型的誤判。

多角色 AI 的設計模式
引入多角色 AI 結構，每個角色負責特定的查詢、優化和推薦任務，確保分工合作，提升系統效率。

檢視表（View Table）技術
使用 MS-SQL 檢視表技術，整合多張表進行高效查詢，增強資料管理能力。

## 8. 專案資訊與參考連結
開放教育平台 (Open School) 專案網站:
https://openact.my.canva.site/about

NopCommerce 開源架構官網:
https://www.nopcommerce.com/en

NopCommerce 開源架構 Github 原始碼:
https://github.com/nopSolutions/nopCommerce

本專案 NopCommerce 實習及團隊互動平台:
https://github.com/LangraceTechUniversityCollab/NopCommerceUniversityCollab
