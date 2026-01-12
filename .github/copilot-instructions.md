# Copilot Instructions for Subscription SaaS Codebase 🤖

Guidance for AI coding agents working on this .NET Framework SaaS subscription management platform.

## Project Overview
- **Purpose:** A highly-integrated SaaS application for managing subscriptions across any system with flexible third-party integration capabilities.
- **Type:** ASP.NET Web API (OWIN) + layered architecture (API → BLL → DAL)
- **Tech Stack:** .NET Framework, Entity Framework 6, AutoMapper, Unity DI, OAuth, Swagger
- **Main entry:** `Subscription.API/Startup.cs`; controllers in `Subscription.API/Controllers/*`
- **BLL & Facades:** `Subscription.BLL/Services/*` (UserFacade, ProductFacade, BackgroundFacade)
- **Data:** `Subscription.DAL/Entities/*` with generic `Repository<TEntity>` pattern
- **Key domains:** Users, Products, Subscriptions, Backgrounds, Translations (multi-language support)

## Critical Workflows

### Adding a New API Endpoint
1. Create or modify controller in `Subscription.API/Controllers/` — inherit from `BaseApiController`.
2. Inject facade (e.g., `IProductFacade`) via constructor.
3. Use `Mapper.Map<DTO>(Model)` to convert API models to DTOs.
4. Call facade method and return via `Ok(result)` or `PagedResponse(...)` for paged data.
5. Add `[AuthorizeRoles(Enums.RoleType.GlobalAdmin)]` if admin-only.

### Adding a New Service/Facade
1. Create interface in `Subscription.BLL/Services/Interfaces/I*Service.cs`.
2. Implement in `Subscription.BLL/Services/*Service.cs` — inherit from `Service<TEntity>`.
3. Register in `Subscription.BLL/SubscriptionBllConfig.RegisterTypes()` with `IUnityContainer`.
4. Register AutoMapper mappings in `SubscriptionBllConfig.RegisterMappings()` if entity ↔ DTO needed.
5. Inject facade/service into controllers or other facades via constructor.

### Adding Data Access (Repository Pattern)
- Use generic `Repository<TEntity>` via `IRepositoryAsync<TEntity>` — already configured.
- Call `.Query()`, `.Query(expression)`, or `.Query(queryObject)` to fetch with includes/filtering.
- For async: `.FindAsync()`, `.DeleteAsync()`.
- Use `UnitOfWork.SaveChanges()` to commit — typically handled by facade or service.

## Key Files & Their Purpose
- `Subscription.API/Startup.cs` — OWIN config, OAuth, Web API pipeline, message handlers.
- `Subscription.API/App_Start/UnityConfig.cs` — DI bindings for controllers, facades.
- `Subscription.API/App_Start/AutoMapperConfig.cs` — API model ↔ DTO mappings.
- `Subscription.BLL/SubscriptionBLLConfig.cs` — BLL DI, entity ↔ DTO mappings, service registrations.
- `Subscription.API/Infrastructure/BaseApiController.cs` — shared response helpers (`PagedResponse`), paging constants, token claim extraction.
- `Subscription.API/Providers/SimpleAuthorizationServerProvider.cs` — OAuth grant validation, user claims.
- `Frameworks/Repository.Pattern/Repository.cs` — core generic repository; used by all services.
- `Frameworks/Service.Pattern/Service.cs` — base service class; all BLL services inherit.

## Codebase Conventions

1. **Controllers stay thin** — route to facade/service, map DTO, return response.
2. **Facades orchestrate** — call multiple services, handle cross-domain logic (e.g., UserFacade calls ProductFacade).
3. **Paging:** Services return `PagedResultsDto` with `.Data` and `.TotalCount`; controllers call `PagedResponse(...)`.
4. **DTOs everywhere** — API models ↔ DTOs ↔ Entities; use AutoMapper for all conversions.
5. **Auth:** Read `UserId`, `UserName`, `UserRole` from `BaseApiController` base class (claims-based).
6. **Localization:** `Language` property set from thread culture in `BaseApiController`; some endpoints respect it (e.g., product translations).
7. **File uploads:** Parsed via `JavaScriptSerializer` from form data (see `BackgroundsController`) — validate size, extension, store under `~/Images`.

## Integration Points
- **OAuth:** Token endpoint at `/api/token`; refresh at `/api/token` with refresh token grant.
- **CORS:** Enabled in `WebApiConfig` (currently `AllowAll`).
- **Swagger:** Registered in `Startup.ConfigureWebApi()`; visit `/swagger` for docs.
- **Message Handlers:** `LanguageMessageHandler` extracts language from request headers.
- **Exception Filter:** `CustomExceptionFilter` catches and standardizes exceptions globally.

## When Modifying...

- **Controllers:** Ensure DI is injected, routes use `[Route(...)]`, `[AuthorizeRoles(...)]` applied if needed.
- **DTOs/Models:** Update both AutoMapper registrations (`AutoMapperConfig` + `SubscriptionBllConfig`).
- **Services:** Register in DI container and implement `IService<TEntity>` or custom interface.
- **Entities:** Update migrations (EF6 Code-First) in `Subscription.DAL/Migrations/`.
- **Response format:** Always use `Ok(...)` or return `PagedResponse(...)` for consistency.

## Common Pitfalls
- **Forgetting DI registration** — new service won't be injected; register in `UnityConfig` or `SubscriptionBllConfig`.
- **Missing AutoMapper config** — DTO ↔ entity mapping will fail at runtime; add to `RegisterMappings`.
- **Skipping `BaseApiController`** — lose access to `UserId`, `Language`, `PagedResponse` helpers.
- **Mixing concerns** — don't put data access in controllers; use facade/service layer.

## Quick Tips for AI Agents
- Check `Subscription.API/App_Start/` for DI & mapping setup before adding endpoints.
- Look at existing facades (e.g., `UserFacade.cs`) as templates for new ones.
- Use `Mapper.Map<T>(source)` freely; it's pre-configured.
- For paged endpoints, return `PagedResponse(routeName, page, pageSize, totalCount, data, isParentTranslated)`.
- Respect `[AuthorizeRoles(...)]` attribute; don't bypass authentication.

---

**See also:** `README.md` for architecture overview and build/run instructions.
