# 後端路由規範

來源對應：`docs/frontend-api-multilang.md`（路由分組示例）、`docs/SECURITY.md`（中介層/限流）。

## 路由檔案與分組
- `routes/api.php` 作為統一入口，載入子群組：
  - `Route::prefix('backend')->group(base_path('routes/backend.php'));`
  - `Route::prefix('frontend')->group(base_path('routes/frontend.php'));`
- 版本以子前綴：`/v1`、`/v2`。

## 中介層（Middleware）
- 後台（backend）：`auth:web` 或後台專屬認證 + CSRF（若使用 Session）
- 前台（frontend）：固定 API Token（`api.token`）或 JWT（依專案設定）
- 共用：`throttle`（限流）、`localization`（多語）、日誌/追蹤。

## 命名與慣例
- 路由以資源命名，遵循 REST：index/show/store/update/destroy。
- 僅暴露必要動作；避免使用含糊之 `index/getInfo` 作為通用介面。

## 範例
```php
// routes/api.php
Route::prefix('backend')->group(base_path('routes/backend.php'));
Route::prefix('frontend')->group(base_path('routes/frontend.php'));

// routes/backend.php
Route::prefix('v1')->middleware(['auth:web','throttle:60,1'])->group(function () {
    Route::apiResource('advertisements', AdvertisementController::class);
});
```

## PR 檢查清單
- [ ] 分組清晰（backend/frontend/版本）
- [ ] 必要中介層到位（認證/限流/語系）
- [ ] 動作語意正確，無雜項彙總接口