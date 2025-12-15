# 前端安全規範

## 憑證與存放
- JWT/固定 Token 一律放於 `Authorization: Bearer <token>` Header，不存於可被 JS 讀取之 Cookie（避免 CSRF）。
- 若需長存，優先使用安全的記憶體狀態或受保護的儲存抽象；LocalStorage 僅在風險可接受時使用，並搭配短效 Token + 旋轉策略。
- 記錄來源 IP、trace_id 由後端負責；前端於請求附帶必要追蹤 header。

## CSRF 與 XSS
- 使用 Header 方式攜帶 Token，避免表單自動夾帶 Cookie 所引發的 CSRF。
- 所有使用者輸入均需過濾/轉義；UI 禁止使用危險的 `innerHTML`，必要時使用可信白名單與 CSP。

## CORS 與來源控制
- 僅對受信任網域發送請求；避免在前端硬編任意可配置的 API 網域。
- 後端應設定嚴格 CORS；前端不嘗試繞過瀏覽器安全限制。

## API 使用與重試
- 設定全球 timeout、冪等 GET 可有限次重試；失敗需有使用者可理解的錯誤提示。
- 避免在未驗證狀態傳送敏感資料；重要操作需二次確認。

## 多語與錯誤訊息
- 錯誤訊息支援 i18n，不回顯後端內部細節；僅顯示 `message`，不顯示堆疊。

## PR 檢查清單
- [ ] Token 僅透過 Authorization Header 傳遞，無存於 Cookie
- [ ] 無危險 `innerHTML`；必要時有 CSP 與白名單
- [ ] 有全域 timeout 與錯誤處理；重試僅用於冪等 GET
- [ ] 不在前端硬編不受信任的 API 網域