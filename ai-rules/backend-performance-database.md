# 後端效能與資料庫規範

## 快取策略（Cache）
- Key 命名：`{domain}:{context}:{entity}:{id}:{field}:{ver}`，多租戶/多語系需帶 `tenant/locale`。
- TTL 必填，加入 ±5% Jitter 分散過期；禁止永久快取。
- 讀：Cache-Aside；寫：精準失效/Tag/版本號；熱點加鎖（`Cache::lock()`）。
- 觀測：記錄 hit/miss、latency、error rate；具備 tracing。

## 服務層（Service）
- 維持交易邊界：交易內禁止外部呼叫（HTTP/RPC/Queue）。
- 外部依賴：設定 timeout、有限次重試（僅冪等讀）、必要時斷路器/降級。
- I/O 使用 DTO/VO，不曝露 ORM。

## 資料存取層（Repository）
- 僅封裝資料來源；方法語意清楚（`findById/search/save/delete`）。
- 回傳 Entity/DTO；不回傳 raw array 或 ORM 實體。
- 禁止處理快取邏輯。

## 分頁與查詢
- 一律標準化分頁：`page/per_page`（上限保護）、`sort`、`filter`（白名單）。
- 回應附帶 `meta.pagination`：`current_page/per_page/total/last_page`。

## 併發與反雪崩
- 熱點路徑使用 single-flight 或分布式鎖，避免 dogpile。
- 失效採事件驅動（Domain Event）或版本號 bump，避免不一致。

## 序列化與資料大小
- 預設 JSON；>1MB 分片；避免大型二進位直接快取。

## PR 檢查清單
- [ ] Key/TTL/Jitter 合規，失效策略清楚
- [ ] Service 交易邊界清楚，無跨界 IO
- [ ] Repository 不含快取與業務邏輯
- [ ] 分頁/排序/篩選標準化，且有上限保護