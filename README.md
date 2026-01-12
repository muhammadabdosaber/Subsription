# Subscription SaaS Platform 🏢

**A highly-integrated SaaS (Software-as-a-Service) application for managing subscriptions across any system.** This solution enables organizations to centralize subscription management with seamless integration capabilities for third-party systems.

- **Solution file:** Subscription.sln
- **Startup / run:** Open the solution in Visual Studio, restore NuGet packages, set `Subscription.API` as the startup project and run.

## High level architecture 🏗️

- **`Subscription.API` (Web API)** — OWIN-based Web API entry point. Key files:
  - `Subscription.API/Startup.cs` — OWIN startup, OAuth config and Web API pipeline.
  - `Subscription.API/App_Start/WebApiConfig.cs` — routing, CORS and JSON settings.
  - `Subscription.API/App_Start/UnityConfig.cs` — Unity DI registrations for controllers and facades.
  - `Subscription.API/App_Start/AutoMapperConfig.cs` — AutoMapper registration for API <-> DTO mappings.

- **`Subscription.BLL` (Business Layer)** — Facades and services implementing business rules.
  - Facade examples: `Subscription.BLL/Services/UserFacade.cs`, `ProductFacade.cs`, `BackgroundFacade.cs`.
  - `Subscription.BLL/SubscriptionBllConfig.cs` wires BLL DI and AutoMapper mappings.

- **`Subscription.DAL` (Data Access Layer)** — EF entities, context and migrations. DTO <-> Entity mapping lives here via AutoMapper config.

- **`Frameworks/Repository.Pattern` & `Frameworks/Service.Pattern`** — Generic repository and service abstractions used across the codebase (Repository<TEntity>, Service<TEntity>). Follow these patterns when adding data access.

- **`Subscription.Common` & `Subscription.Resources`** — shared enums, helpers (e.g., `PasswordHelper`, `Strings.cs`), and localization resources.

## Important runtime & integration details 🔌

- **Authentication:** OAuth bearer tokens via `SimpleAuthorizationServerProvider` — token endpoint exposed at `/api/token` (see `Subscription.API/Providers`). Refresh tokens are handled by `RefreshTokenProvider`.
- **Dependency Injection:** Unity is used. Register new services in `UnityConfig.RegisterTypes` and `SubscriptionBllConfig.RegisterTypes`.
- **Mapping:** AutoMapper initialized in `AutoMapperConfig` and `SubscriptionBllConfig`.
- **Swagger:** Enabled in `Startup.ConfigureWebApi` for API docs.
- **Global base controller:** `Subscription.API/Infrastructure/BaseApiController.cs` contains shared helpers (paging constants, token claim helpers, `PagedResponse`).
- **Message handlers & Filters:** `LanguageMessageHandler` and `CustomExceptionFilter` are wired globally — respect their expectations when changing headers, responses, or localization.

## Common patterns & conventions 🧭

- Use **Facades** (e.g., `IProductFacade`) in controllers to coordinate BLL operations — controllers keep thin.
- Services return DTOs and `PagedResultsDto` for paging. Controllers call `PagedResponse(...)` to return paged results.
- Use AutoMapper for DTO ⇄ model mapping; register mappings in `AutoMapperConfig` and `SubscriptionBllConfig`.
- File uploads for backgrounds are handled in `BackgroundsController` and saved under `~/Images` (max ~2MB, only `.jpg/.png/.jpeg`).

## Files & locations to inspect for changes 🗂️

- API startup and middleware: `Subscription.API/Startup.cs` and `App_Start/*`
- DI: `Subscription.API/App_Start/UnityConfig.cs` and `Subscription.BLL/SubscriptionBllConfig.cs`
- Controllers: `Subscription.API/Controllers/*` (notably `BackgroundsController`, `ProductsController`, `UsersController`)
- Facades & services: `Subscription.BLL/Services/*`
- Repository pattern: `Frameworks/Repository.Pattern/*`
- DTOs & Entities: `Subscription.BLL/DTOs` and `Subscription.DAL/Entities`

## Build & run (quick) 🚀

1. Open `Subscription.sln` in Visual Studio (Windows / .NET Framework).
2. Restore NuGet packages (Visual Studio or `nuget restore Subscription.sln`).
3. Set `Subscription.API` as startup project and run (IIS Express or local IIS).

> Note: This is a .NET Framework (not .NET Core) solution — prefer Visual Studio on Windows for local development and debugging.

## Notes for AI agents 🤖

- Prefer using existing DI registrations and facades when modifying business logic.
- Follow `Repository.Pattern` + `Service.Pattern` abstractions for data access to be consistent with the codebase.
- When adding routes, use attribute routing (existing controllers rely on it) and update Swagger registration if needed.
- Look at `BaseApiController` and `CustomExceptionFilter` before changing global response shapes or error codes.


