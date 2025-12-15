# 後端風格與安全規範

## 命名與風格
- Controller/Action 避免模糊名稱（如 `index/getInfo` 作為萬用介面），採資源化命名。
- 例外類型具體化（如 `AdvertisementNotFoundException`），錯誤碼集中於 `ExceptionCode`。
- 常數集中於 `Constant/enum`，禁止魔術字串。

## 認證與授權
- 後台（Filament 或 web）：使用 `web` guard + CSRF + `AuthenticateSession`。
- API（前台/後台）：依場景使用 JWT/固定 Token；路由層宣告 middleware，不混入 Controller。
- 權限：後台 Controller 與前台 Controller 分流；後續以 JWT/Policy 控制資源權限。

## 限流與防護
- 登入/敏感操作需限流：如 `loginRateLimiter(maxAttempts:5,decayMinutes:15)`。
- API 路由至少套用 `throttle`；對爬蟲入口加更嚴格限流。
- 防重放：短效 Token、時間窗校驗、重要操作要求二次驗證。

## Session 與 Cookie（如使用 Session）
- `.env` 建議：
  - `SESSION_DRIVER=database`
  - `SESSION_LIFETIME=120`
  - `SESSION_SECURE_COOKIE=true`
  - `SESSION_HTTP_ONLY=true`
  - `SESSION_SAME_SITE=lax` 或 `strict`

## 密碼與帳號安全
- Hash：`'password' => 'hashed'`、使用 bcrypt；密碼不出現在回應（`$hidden`）。
- 密碼重設 Token：60 分鐘過期、60 秒限流。
- 密碼強度規則：至少 8 碼，含大小寫/數字/特殊字元（可用自訂 `StrongPassword` 規則）。

## 網路與 IP 控制
- 後台入口可加 IP 白名單 middleware（公司/辦公室 IP）。
- 實作 `AdminIpWhitelist` 並於後台路由/Panel 啟用。

## HTTP 安全標頭（建議透過中介層設定）
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`（或必要頁面 `SAMEORIGIN`）
- `Referrer-Policy: no-referrer`（或 `strict-origin-when-cross-origin`）
- `Content-Security-Policy` 最少限制腳本來源與內嵌執行

## 日誌與可觀測性
- 記錄認證嘗試（成功/失敗）、來源 IP、User-Agent、trace_id。
- 對敏感操作（登入、重設密碼、權限變更）建立審計日誌。

## PR 檢查清單
- [ ] 路由已套用適當 middleware（認證/限流/語系）
- [ ] `.env` session 安全設定到位（如適用）
- [ ] 密碼規則、重設流程與限流可追溯
- [ ] 例外與錯誤碼集中管理，無魔術字串