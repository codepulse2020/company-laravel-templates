# 後端 API 規範（請求/回應/錯誤）

來源對應：`docs/RULES.md`（E/F 節）、`docs/validator.md`、`docs/presenter.md`、`docs/api-md/README.md`。

## 請求驗證（FormRequest 必須）
- 每個端點都有對應專屬 FormRequest（如 `AdvertisementStoreRequest`）。
- Controller 方法參數一律注入專屬 FormRequest 型別；嚴禁注入 `Illuminate\Http\Request`。
- 僅使用 `$request->validated()`；嚴禁 `$request->all()` 與未驗證輸入。
- List 查詢使用專屬 Request，實作：
  - `listRules()` 定義查詢欄位與規則（page/per_page/sort/filter…）。
  - `listValidated()` 回傳標準化查詢參數物件/陣列。

## 回應格式（統一包裝）
- 統一格式：`{ code, message, data, meta }`。
- 資料結構由 Resource/Presenter 格式化：
  - 單筆：`Resource::make($entity)`
  - 清單：`Resource::collection($paginator)` 並在 `meta.pagination` 帶上分頁資訊。

### 範例（JSON）
```json
{
  "code": 0,
  "message": "OK",
  "data": [{"id":1,"name":"..."}],
  "meta": {"pagination":{"current_page":1,"per_page":15,"total":3,"last_page":1}}
}
```

## 錯誤與例外
- Controller 不直接 `abort()`；由 Service 拋出具體業務例外，對應 `ExceptionCode`。
- 全域 Handler/包裝器統一轉為 `code/message` 與適當 HTTP 狀態碼。
- 驗證錯誤輸出結構一致，支援國際化（`field`/`code`/`message`）。

## 命名與版本
- 路由遵循 REST 動詞與資源命名：`GET /advertisements`、`POST /advertisements`、`GET /advertisements/{id}`…
- API 版本以前綴區隔：`/api/backend/v1/...`。

## 分頁與查詢
- 統一支援：`page`、`per_page`（上限保護）、`sort`、`filter`。
- 回應 `meta.pagination`：`current_page/per_page/total/last_page`。

## PR 檢查清單
- [ ] Controller 僅接收專屬 FormRequest；無 `$request->all()`
- [ ] List 查詢以 `listRules()/listValidated()` 驗證與整形
- [ ] 回應格式一致，清單含 `meta.pagination`
- [ ] 例外使用具體類型並對應 `ExceptionCode`