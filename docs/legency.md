# FTC_MES_MVC 技術架構盤點

本文件整理 /Users/pablo/Downloads/FTC_MES_MVC 專案的程式碼架構與主要技術內容，作為後續維護、交接或改版前的快速導覽。

## 1. 專案總覽

此資料夾是一個以 ASP.NET MVC 5 / Web API 2 為主的 MES 系統專案，目標框架為 .NET Framework 4.6。整體包含主 Web 系統、資料庫 SQL 腳本、排程服務程式與舊式 NuGet packages。

主要目錄如下：

| 路徑 | 說明 |
|---|---|
|FTC_MES_MVC.sln|Visual Studio Solution|

FTC_MES_MVC/

主要 ASP.NET MVC / Web API 專案

SQLScript/

建表、初始化、權限與維護用 SQL 腳本

Svc/

額外排程/服務程式

packages/

舊式 NuGet packages 還原目錄

專案規模概略如下：

類型

數量

C# 原始碼 .cs

約 624

Razor View .cshtml

約 925

MVC Areas

約 35

2. 技術棧

後端主要技術：

.NET Framework 4.6

ASP.NET MVC 5

ASP.NET Web API 2

Dapper

SQL Server

Oracle Managed DataAccess

NLog

Swashbuckle / Swagger

SignalR

Application Insights

前端主要技術：

Razor View

jQuery

Bootstrap

Kendo UI

Highcharts

AdminLTE

SignalR JavaScript client

NuGet 套件採用舊式 packages.config 與 packages/ 目錄管理，不是 SDK-style project，也不是 .NET Core / .NET 5+ 專案。

3. 主 Web 專案結構

主 Web 專案位於：

FTC_MES_MVC/

重要目錄如下：

目錄

說明

App_Start/

MVC、Web API、Bundle、Swagger 等啟動設定

Controllers/

主系統 MVC Controller 與 API Controller

Views/

主系統 Razor View

Models/

Model、ViewModel、DAL、Service

Base/

DAL 連線與資料存取基底類別

Filters/

登入驗證、權限、Log Filter

Hubs/

SignalR Hub

Areas/

依模組或廠區拆分的 MVC Area

Content/

CSS、圖片、UI 套件資源

Scripts/

JavaScript 套件與自訂腳本

App_Data/

上傳檔案、範本或系統資料

Web References/

舊式 Web Service 參考

4. 啟動流程

系統入口位於：

FTC_MES_MVC/Global.asax.cs

Application_Start() 會依序註冊：

AreaRegistration.RegisterAllAreas()

GlobalConfiguration.Configure(WebApiConfig.Register)

FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters)

RouteConfig.RegisterRoutes(RouteTable.Routes)

BundleConfig.RegisterBundles(BundleTable.Bundles)

也就是說，啟動時會載入所有 Areas、Web API 路由、MVC 路由、全域 Filter 與前端 bundle。

5. 路由設定

MVC Route

設定檔：

FTC_MES_MVC/App_Start/RouteConfig.cs

預設路由：

{controller}/{action}/{id}

預設頁面：

Home/Index

Web API Route

設定檔：

FTC_MES_MVC/App_Start/WebApiConfig.cs

預設 API 路由：

api/{controller}/{action}/{id}

系統也有啟用 Attribute Routing：

config.MapHttpAttributeRoutes();

並且啟用 CORS：

config.EnableCors();

6. Controller 架構

主 Controller 放在：

FTC_MES_MVC/Controllers/

常見類型：

類型

範例

說明

MVC Controller

HomeController, QCController, ATMSController

回傳 Razor View

API Controller

MESApiWipController, MESApiQcController, AtmsApiController

提供 Web API

管理功能

UserManageController, MESManageToolController

權限、使用者、MES 基礎資料維護

系統共用基底

BaseController, BaseApiController

共用登入、權限、連線、Log、工具方法

BaseController

檔案：

FTC_MES_MVC/Controllers/BaseController.cs

主要用途：

MVC Controller 共用基底

掛載 [Authentication]

掛載 [SysAuthorize]

管理 SQL connection

讀取 Menu 權限資料

寫入 NLog

在 OnActionExecuting 與 OnActionExecuted 記錄執行資訊

BaseApiController

檔案：

FTC_MES_MVC/Controllers/BaseApiController.cs

主要用途：

Web API Controller 共用基底

啟用 CORS

掛載 API Log

管理 Base_DB_ConnectString 與 MES_DB_ConnectString

依 CompanyID / FactoryID 動態取得 MES AP DB 連線

提供查詢條件組裝、日期處理、回傳訊息等共用方法

7. Models 與資料存取

Model 主要放在：

FTC_MES_MVC/Models/

主要子目錄：

目錄

說明

AtmsModels/

ATMS 相關資料模型

CommModels/

共用權限、角色、節點、清單資料模型

LimsModels/

LIMS 相關資料模型

MesModels/

MES 相關資料模型

Dals/

DAL 類別

Services/

Service 類別

UserManage/

使用者管理相關模型

ViewModels/

各功能 ViewModel / DTO

資料存取基底位於：

FTC_MES_MVC/Base/

包含：

檔案

說明

MesDalBase.cs

MES DB 連線與 DAL 共用邏輯

CommDalBase.cs

共用 DB / 權限資料相關 DAL 基底

LimsDalBase.cs

LIMS 相關 DAL 基底

AtmsDalBase.cs

ATMS 相關 DAL 基底

資料存取主要使用 Dapper，透過 SqlConnection 與 DynamicParameters 組合 SQL 查詢。部分功能也引用 Oracle Managed DataAccess。

8. 權限與登入

權限 Filter 位於：

FTC_MES_MVC/Filters/

主要檔案：

檔案

說明

AuthenticationAttribute.cs

登入驗證

SysAuthorizeAttribute.cs

頁面權限檢查

AvoidSysAuthorizeAttribute.cs

略過系統權限檢查

ApiLogAttribute.cs

API Log

LogAttribute.cs

一般 Log

權限開關由設定值控制：

FtcMesAuthentication

若設定不是 true，登入與權限檢查會直接略過。

權限邏輯大致如下：

判斷是否啟用 FtcMesAuthentication

判斷 Action / Controller 是否允許匿名或略過權限

透過 QueryString 的 NodeId 判斷頁面節點

讀取使用者角色

讀取角色可存取節點

無權限時導向 Home/NoAccess

Menu 也透過權限資料產生，主要查詢 V_TreeNode。

9. Areas 模組

系統大量使用 MVC Areas 切分模組或廠區功能。Areas 位於：

FTC_MES_MVC/Areas/

常見 Area 結構：

Areas/{AreaName}/
  Controllers/
  Models/
  Views/
  {AreaName}AreaRegistration.cs

部分規模較大的 Area：

Area

C# 檔數

View 數

說明

NYPVC

約 10

約 178

規模最大的廠區/業務模組之一

JI4_AP

約 16

約 53

含 React / JS bundle

NJJG

約 15

約 47

廠區模組

SHIEYU

約 10

約 46

含 WIP、EQP、COMMON、QC 等功能

SK_DW1

約 15

約 39

廠區模組

SL2_CCL

約 31

約 31

廠區模組

SPS

約 12

約 26

SPS 相關模組

LKP_PVC

約 15

約 27

廠區模組

這種架構表示系統是長期累積的企業內部平台，各廠區或業務線以 Area 方式獨立擴充。

10. 前端資源與 Bundle

Bundle 設定位於：

FTC_MES_MVC/App_Start/BundleConfig.cs

主要 bundle：

Bundle

包含內容

~/bundles/jquery

jQuery、blockUI、AdminLTE、mask、toast、SignalR、md5/base64

~/bundles/bootstrap

Bootstrap、Respond、i18next

~/Content/css

Bootstrap、font-awesome、AdminLTE、toast、iCheck、Iframe、UX UI CSS

~/Content/kendo/Css

Kendo UI CSS

~/bundles/kendo/Script

Kendo UI JavaScript

~/bundles/Highcharts/Script

Highcharts 與匯出相關套件

整體前端屬於傳統企業後台型態，主要以 Razor View + jQuery plugin + Kendo widget 組成。

11. Swagger / API 文件

Swagger 設定位於：

FTC_MES_MVC/App_Start/SwaggerConfig.cs

使用 Swashbuckle，API 文件版本設定為：

FTC_MES_MVC v1

設定中有啟用 XML comments 與 full type name schema id：

c.IncludeXmlComments(GetXmlCommentsPath());
c.UseFullTypeNameInSchemaIds();

12. SignalR

SignalR Hub 位於：

FTC_MES_MVC/Hubs/CounterHub.cs

目前主要功能：

使用者連線時加入線上清單

回傳線上使用者列表給前端

使用者離線時移除清單

13. 設定檔注意事項

主專案根目錄未看到標準命名的 Web.config，但存在：

FTC_MES_MVC/正是部屬使用，請記得自改名稱_Web.config

從檔名判斷，這應該是正式部署或本機執行前需要手動改名成 Web.config 的設定檔。

該設定檔包含：

Base_DB_ConnectString

MES_DB_ConnectString

公司 / 廠別設定

AF / PI Server 設定

API path

驗證開關

上傳路徑

CORS header

Application Insights module

注意：設定檔中可見明文資料庫帳密。若要交付、上版控或部署到正式環境，建議改用環境變數、部署轉換檔、Secret Manager 或 CI/CD secret injection。

14. SQLScript

SQL 腳本位於：

SQLScript/

主要包含：

類型

範例

建表腳本

FtcMESDB_CreateAllTable_Sql_20190802.sql

建表加資料

FtcMESDB_CreateAllTable_Sql_And_Data_201909024.sql

共用 DB 腳本

FtcMESComDB_CreateAllTable_Sql_20190711.sql

權限資料

FtcMESDB_權限資料建立_Sql_20190711.sql

角色/功能/節點

COMM_ROLE.sql, COMM_FUNCTIONLIST.sql, COMM_NODEFUNCTION.sql

公司/工廠

Company.sql, Factory.sql

維護腳本

清除所有生產資料.sql, QMS_需要修正table修正sql.sql

這些腳本看起來是此系統初始化與權限資料建立的重要依據。

15. 排程服務

排程服務位於：

Svc/FtcAtmsCreateCycleCheckPlanSvc/

其中主要程式：

Svc/FtcAtmsCreateCycleCheckPlanSvc/FtcAtmsCreateCycleCheckPlanSvc/Program.cs

功能概略：

啟動時檢查是否已有同名 Process 正在執行

讀取 App.config 中的 Web API URL、JobName、ServiceName、CompanyID、FactoryID 等設定

查詢 Job 執行紀錄

判斷是否到達 EXECSTARTTIME 與 NEXTEXECTIME

到時間後呼叫指定 Web API

更新下一次執行時間

使用 NLog 記錄執行結果

目前可見用途包含：

ATMS 定期更新

模具剩餘壽命更新

定保開單計畫建立

16. 整體架構判讀

此專案屬於大型企業內部 MES 平台，具有以下特徵：

長期累積、多廠區、多模組

傳統 ASP.NET MVC 5 架構

後端商業邏輯多半集中在 Controller、DAL 與 ViewModel

前端以 Razor + jQuery/Kendo 為主

權限系統以使用者、角色、節點、功能控制資料表為核心

資料庫以 SQL Server 為主，部分功能可能串 Oracle、AF/PI、外部 Web Service

部署方式偏舊式 IIS / Web.config / packages.config 模式

17. 後續維護建議

建議優先處理：

補齊或確認正式 Web.config 管理方式

移除或保護明文資料庫帳密

建立本機建置與啟動說明

釐清各 Area 對應的廠區與業務邏輯

整理 API 清單與 Swagger 是否能正常產生

針對大型 Area，例如 NYPVC、JI4_AP、SHIEYU，建立獨立模組說明

若要現代化，先從設定管理、資料存取封裝與 API 文件化開始，不建議一次性大改架構

