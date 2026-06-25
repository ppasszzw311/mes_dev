# mes_dev NuGet 套件清單

本文件列出各專案建議引入的 NuGet 套件，依 Clean Architecture 分層整理。
對應 legacy 系統（FTC_MES_MVC）的技術棧，升級至 .NET 8 版本。

---

## 1. mes_dev.domain

> 原則上不依賴任何外部套件，保持純 C# 邏輯。

| 套件 | 用途 |
|---|---|
| *(無需外部套件)* | Domain 層只包含 Entity、ValueObject、DomainException、Enum |

---

## 2. mes_dev.application

| 套件 | 版本建議 | 用途 |
|---|---|---|
| `MediatR` | 12.x | CQRS 模式，對應 UseCases/ 的 Command / Query Handler |
| `FluentValidation` | 11.x | 對應 Validators/ 資料驗證邏輯 |
| `AutoMapper` | 13.x | DTO 與 Entity 之間的物件映射 |
| `Microsoft.Extensions.DependencyInjection.Abstractions` | 8.x | DI 介面定義，不直接依賴容器實作 |
| `Microsoft.Extensions.Logging.Abstractions` | 8.x | 日誌介面，不直接綁定 NLog/Serilog |

---

## 3. mes_dev.infrastructure

> 對應 legacy 的 `Base/`（MesDalBase、CommDalBase 等）+ 外部服務整合。

| 套件 | 版本建議 | 用途 |
|---|---|---|
| `Dapper` | 2.x | 輕量 ORM，legacy 主要資料存取方式 |
| `Microsoft.Data.SqlClient` | 5.x | SQL Server 連線（對應 Base_DB_ConnectString / MES_DB_ConnectString） |
| `Oracle.ManagedDataAccess.Core` | 23.x | Oracle 連線（legacy 有引用 Oracle Managed DataAccess） |
| `NLog` | 5.x | 日誌實作（對應 legacy NLog 使用） |
| `NLog.Extensions.Logging` | 5.x | NLog 接入 Microsoft.Extensions.Logging |
| `Microsoft.Extensions.Options` | 8.x | 對應 Options/ 強型別設定讀取 |
| `Microsoft.Extensions.Http` | 8.x | HttpClient Factory，供 ExternalServices/ 使用 |
| `Polly` | 8.x | 外部 API 呼叫的 Retry / Circuit Breaker（對應 legacy Web Service 串接） |
| `Microsoft.AspNetCore.DataProtection` | 8.x | 檔案加密存取（對應 FileStorage/） |

---

## 4. mes_dev.web

> 對應 legacy 的 `FTC_MES_MVC/`，升級為 ASP.NET Core 8 MVC。

| 套件 | 版本建議 | 用途 |
|---|---|---|
| `Swashbuckle.AspNetCore` | 6.x | Swagger / OpenAPI 文件（對應 legacy SwaggerConfig.cs） |
| `Microsoft.AspNetCore.Authentication.JwtBearer` | 8.x | JWT 驗證（取代 legacy Session-based 登入） |
| `NLog.Web.AspNetCore` | 5.x | NLog 整合 ASP.NET Core（對應 legacy LogAttribute） |
| `AutoMapper.Extensions.Microsoft.DependencyInjection` | 13.x | AutoMapper DI 註冊 |
| `FluentValidation.AspNetCore` | 11.x | FluentValidation 整合 Model Binding |
| `Microsoft.AspNetCore.SignalR` | *(內建)* | SignalR Hub（對應 legacy Hubs/CounterHub.cs） |

---

## 5. mes_dev.worker

> 對應 legacy 的 `Svc/` 排程服務（FtcAtmsCreateCycleCheckPlanSvc 等）。

| 套件 | 版本建議 | 用途 |
|---|---|---|
| `Microsoft.Extensions.Hosting` | 8.x | Worker Service / BackgroundService 基底 |
| `Quartz.NET` | 3.x | Job 排程（取代 legacy 自製輪詢排程邏輯） |
| `Quartz.Extensions.Hosting` | 3.x | Quartz 整合 .NET Generic Host |
| `NLog.Extensions.Logging` | 5.x | 日誌整合 |

---

## 6. tests/mes_dev.application.tests & mes_dev.infrastructure.tests

> 已有 xunit、coverlet、Microsoft.NET.Test.Sdk，建議補充：

| 套件 | 版本建議 | 用途 |
|---|---|---|
| `NSubstitute` | 5.x | Mock 框架（建議優先，比 Moq 語法更簡潔） |
| `FluentAssertions` | 6.x | 語意化斷言，提升測試可讀性 |
| `Respawn` | 6.x | 整合測試時重置資料庫狀態 |

---

## 套件依賴關係圖

```
domain          ← 無外部依賴
application     ← 依賴 domain
infrastructure  ← 依賴 application + domain
web             ← 依賴 application + infrastructure
worker          ← 依賴 application + infrastructure
```

> **注意**：`web` 不應直接依賴 `infrastructure` 的具體實作，應透過 DI 注入介面。
> 建議在 `web/Program.cs` 或獨立的 `DependencyInjection.cs` 集中註冊所有服務。
