# My Easy Vault 2026

Josh 的 side project，一個以 Go 開發的簡易 Vault 後端服務。
主要目標是提供可擴充的資金帳務核心能力，支援使用者錢包、幣別兌換、轉帳與交易紀錄查詢。

## 專案定位

這個專案聚焦在「簡單但工程化」的資金系統後端，強調：
- 清楚的三層架構：`handler -> service -> dao`
- 可維護的資料存取設計
- 可直接落地到實務場景的權限與流量控管

## 核心功能

- 使用者登入/註冊與登出（token-based）
- 錢包建立與列表查詢
- 匯率查詢
- 幣別兌換（Preview / Confirm）
- 使用者間轉帳（Preview / Confirm）
- 交易紀錄分頁查詢
- 測試路由提供資金入帳模擬（`/test/account/addAssets`）

## 亮點設計

### 1) Middleware：權限控管 + 流量控制

路由層統一掛載：
- `ApiAuthorityMiddleWare`：驗證 API 權限、解析 token、注入使用者上下文
- `RateLimitMiddleWare`：以漏桶演算法，依權限規則做限流，並回傳 `X-RateLimit-*` header

這種做法讓權限與限流都在入口層集中處理，業務程式碼更乾淨。

### 2) DAO：更佳 ORM 設計

DAO 層採用一致化查詢物件與 Scope 鏈式組合，常見特色：
- `Query` 結構 + `queryChain`，統一 where/in/order/page 條件拼裝
- `WithTx` 支援交易內切換 DAO，降低交易流程耦合
- 動態 `Update` 映射，減少重複 SQL 與樣板程式

這讓資料層在可讀性、擴充性與一致性上更好維護。

### 3) L2 Cache：非侵入式快取查詢

`infra/L2CQuery` 提供通用查詢快取包裝：
- 以查詢 SQL 自動產生 key
- 先讀 Redis，miss 再回源 DB
- 寫回快取與維護 table/key 關聯集合

業務端只需包一層查詢函式，不必把快取邏輯散落在各 service，達到「非侵入式」導入。

## 技術棧

- Language: Go (`go1.23`)
- API: Gin
- DI: Uber Fx
- DB: MySQL + GORM（含讀寫分離 plugin）
- Cache / MQ: Redis（快取、鎖、佇列/pubsub）
- Tooling: Cobra、Viper、Swagger

## 專案結構（節錄）

```text
api-server/
  api/            # handlers, routers, middlewares
  services/       # 業務邏輯
  dao/            # 資料存取與 ORM 組裝
  infra/          # DB/Redis/MQ/L2 cache/lock/router
  docs/           # swagger 文件
```

## 快速啟動（本地）

```bash
cd api-server
cp .env.example .env
go run main.go serve
```

或使用 Docker：

```bash
cd api-server
docker compose -f docker/docker-compose.yml up --build
```

## 備註

這是 Josh 持續迭代中的 side project，目標是把 Vault 後端核心做成可擴充、可實戰的工程模板。
