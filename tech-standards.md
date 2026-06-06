# 團隊技術標準建議（Tech Standards）

> 版本：1.0.0｜性質：**建議參考**，非強制憲章。
>
> 本文件提供團隊預設的技術選型建議，協助新專案快速取得一致、可維護的技術基礎。
> 與 [`constitution.md`](constitution.md) 等憲章不同，本文件條目為 **SHOULD（建議）**——
> 採用其他成熟、社群活躍的替代方案是允許的，但 **SHOULD** 於 spec / PR 說明選型理由，
> 以利知識傳承與後續維護。

## 建議選型總覽

| 應用類型 | 建議框架 / 技術 | 說明 |
|---------|----------------|------|
| Node.js API 服務 | **NestJS** | 內建分層架構（module / controller / provider）、DI、guard，契合憲章責任分離要求 |
| Python API 服務 | **FastAPI** | 原生 OpenAPI 3.1、型別驗證（Pydantic）、非同步支援 |
| 工具服務（worker / 批次 / CLI）| 依語言採該語言成熟框架 | Python（FastAPI / Typer）、Shell（見 §SH）；容器化與部署見 §DEP |
| 前端網頁應用 | **Vue 3 以上** | Composition API、`<script setup>`、Vite、Pinia 生態成熟 |
| 容器化 / 部署 | **Docker + Docker Compose** | 所有自建服務以容器為優先、Compose 為主要部署實現；K8s 為低優先客戶選項（見 §DEP）|

## 建議的責任分離實作（API 服務）

憲章 §API-4 只要求**責任分離**（transport / application / domain / infrastructure），不綁定形狀。團隊建議的一種具體實作為四層：

| 層 | 對應責任 |
|----|---------|
| Middleware / Guard | transport 橫切：authn / authz / rate limit / request logging |
| Controller | transport 接入：路由、請求解析、回應組裝、輸入驗證 |
| Service | application + domain：用例編排與商務邏輯 |
| Repository | infrastructure：資料存取封裝 |

採用 DDD、Hexagonal、CQRS 等其他達成同等分離的架構亦合規。

## 選型原則

- 框架 **SHOULD** 為成熟、社群活躍、有長期維護承諾者；**SHOULD NOT** 自製不成熟的核心框架。
- 同一專案 / 服務內 **SHOULD** 維持選型一致，避免混用多種同質框架增加維護成本。
- 偏離本建議時，**SHOULD** 在 spec / PR 說明替代方案及其理由（如既有系統相容性、團隊熟悉度、特定需求）。

## 與憲章的關係

- 憲章規範的是**架構與行為要求**（如分層、OpenAPI、安全標頭、測試），這些為強制項。
- 本文件規範的是**達成上述要求的建議工具**。只要替代方案同樣能滿足憲章的強制要求，即為合規。
- 對應憲章條款：API 責任分離見 [`constitution-api.md`](constitution-api.md) §API-4；前端框架見 [`constitution-frontend.md`](constitution-frontend.md) §FE-1；容器化與部署見 [`constitution-deployment.md`](constitution-deployment.md) §DEP。
