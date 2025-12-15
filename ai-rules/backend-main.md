# 後端規範總覽（分層/職責/邊界）

## 架構分層與資料流程
- Controller（控制流程）
  - 調用 Service，組合回應；不處理交易；不直接呼叫 Repository/Model。
  - 方法參數使用對應的 FormRequest 型別（如 `AdvertisementStoreRequest`）。
  - 回應走統一包裝（code/message/data）。
- Service（商業邏輯/交易）
  - 負責交易控制（DB Transaction）、快取策略（Cache-Aside/失效）、外部依賴（timeout/重試/斷路器/降級）。
  - 僅接收已驗證資料；不得重複輸入驗證。
- Repository（資料來源存取）
  - 封裝 DB/API/File 存取；回傳 Entity/DTO；不處理快取。
- Model（資料庫模型）
  - 資料表設定，避免業務邏輯。
- Request/FormRequest（輸入驗證）
  - 每個端點具專屬 Request，使用 `$request->validated()`；禁用 `$request->all()`。
- Resource/Presenter（輸出整形）
  - 統一格式化 API 回應；處理狀態文字等轉換。
- Validator（驗證器）
  - 僅資料合法性檢查，不做業務決策；輸出標準化錯誤。

## 存取限制
- 不得跨越超過 2 階層：Controller 不可直接呼叫 Repository/Model；Service 不可直接呼叫 Model。
- 同層不得互相呼叫：Service 不呼叫 Service；Repository 不呼叫 Repository；Validator 不呼叫 Validator…
- 獨立結構（CacheManager/Constant/Support/ExceptionCode）可在任意層使用。

## 交易與例外
- 所有資料庫交易在 Service 層執行；Controller 禁用 DB Facade 交易。
- Service 拋出具體業務例外（對應 `ExceptionCode`）；由全域 Handler/回應包裝器統一格式化。

## FormRequest 規範
- 每個 API 端點搭配專屬 FormRequest；Controller 方法必須注入該型別。
- List 查詢 Request 需提供 `listRules()` 與 `listValidated()`，Controller 的 `index` 使用之。

## 快取與一致性（摘要）
- Key 命名：`{domain}:{context}:{entity}:{id}:{field}:{ver}`；一律設定 TTL 並加入 ±5% Jitter。
- 讀取：Cache-Aside；寫入：精準失效/Tag/版本號策略；熱點加鎖（`Cache::lock()`）。

## PR 檢查重點
- 邊界：單一職責、分層與存取限制未違反。
- 輸入/輸出：FormRequest 驗證齊備，回應格式一致。
- 交易/例外：交易在 Service，例外具體且可對應 HTTP 狀態。
- 快取：Key/TTL/Jitter/失效策略清楚，無業務邏輯落在 Repository。