# 前端風格規範

## 命名與資料分離
- API 模組以資源命名：`ticketCategoriesApi`、`ordersApi`。
- 元件關注點分離：容器元件（資料抓取/狀態） vs. 呈現元件（僅 props 渲染）。
- i18n 使用 key，不硬編字串；與後端多語 header 協同（`Accept-Language`）。

## API 使用
- 統一請求器封裝（baseURL、auth header、錯誤攔截）。
- 分頁參數：`page/per_page/sort/filter`；設上限保護，避免過大 page size。
- 依回應協定解析 `meta.pagination`，統一表格/清單元件。

## 狀態管理
- 查詢資料使用快取策略（如 SWR/RTK Query），避免重複請求。
- 請求結果標準化：`{ data, meta, error }`，便於 UI 呈現。

## 組件與樣式
- 組件無副作用；表單驗證集中處理，錯誤訊息可國際化。
- 樣式採用原子/utility 優先，主題與色票集中管理。

## PR 檢查清單
- [ ] 不存在硬編文案，皆使用 i18n key
- [ ] 清單使用統一分頁/排序/篩選介面
- [ ] 資料抓取與渲染分離，易測試