# Configuration & Externalized Settings Inventory

This project has a minimal configuration landscape consisting of a single `Web.config` XML file and two transform overlays (`Web.Debug.config`, `Web.Release.config`), with no external configuration sources, secrets stores, or environment-specific property files.

## Configuration Sources

| Source | Type | Path/Location | Notes |
|---|---|---|---|
| Web.config | XML Application Config | `/Web.config` | Primary ASP.NET configuration; `appSettings`, Razor host, compilation, and assembly binding redirects |
| Web.Debug.config | XML Transform | `/Web.Debug.config` | Applied over Web.config in Debug builds |
| Web.Release.config | XML Transform | `/Web.Release.config` | Applied over Web.config in Release builds |
| packages.config | NuGet Package Manifest | `/packages.config` | Declares NuGet package versions; not a runtime config source |
| .nuget/NuGet.Config | NuGet Config | `/.nuget/NuGet.Config` | NuGet package source configuration |

No external configuration server (Spring Cloud Config, Azure App Configuration, etc.), secret stores (KeyVault, HashiCorp Vault), or Kubernetes ConfigMaps are present.

## Build Profiles

| Profile | Activation | Purpose | Key Changes |
|---|---|---|---|
| Debug | Default / manual (`/p:Configuration=Debug`) | Local development build with full debug symbols | `debug="true"` compilation; full PDB output; `DEBUG` and `TRACE` constants defined |
| Release | Manual (`/p:Configuration=Release`) | Production-ready build with optimizations | Optimizations enabled; PDB-only debug info; `TRACE` constant only |

Build transforms (`Web.Debug.config` / `Web.Release.config`) are applied via MSBuild Web.config transformation at publish time.

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
|---|---|---|---|
| Default (all environments) | Implicit — no ASPNETCORE_ENVIRONMENT equivalent in classic ASP.NET | `Web.config` | None — single config file serves all environments |

Classic ASP.NET MVC 5 on .NET Framework does not have the `appsettings.{Environment}.json` runtime profile system. Environment differentiation is achieved only via Web.config transforms at build/publish time.

## Properties Inventory

**CleanArchitectureAspNetMvc5 — Web.config `appSettings`**

| Property Key | Default Value | Profiles | Source |
|---|---|---|---|
| `webpages:Version` | `3.0.0.0` | All | Web.config |
| `webpages:Enabled` | `false` | All | Web.config |
| `ClientValidationEnabled` | `true` | All | Web.config |
| `UnobtrusiveJavaScriptEnabled` | `true` | All | Web.config |

**System Settings (`system.web`)**

| Property Key | Default Value | Profiles | Source |
|---|---|---|---|
| `compilation/@debug` | `true` | All (overridden by Release transform) | Web.config |
| `compilation/@targetFramework` | `4.5` | All | Web.config |
| `httpRuntime/@targetFramework` | `4.5` | All | Web.config |

**Assembly Binding Redirects (runtime section)**

| Assembly | Redirected From | Redirected To | Source |
|---|---|---|---|
| `Microsoft.Owin` | 1.0.0.0–3.0.0.0 | 3.0.0.0 | Web.config |
| `Microsoft.Owin.Security.OAuth` | 1.0.0.0–3.0.0.0 | 3.0.0.0 | Web.config |
| `Microsoft.Owin.Security.Cookies` | 1.0.0.0–3.0.0.0 | 3.0.0.0 | Web.config |
| `Microsoft.Owin.Security` | 1.0.0.0–3.0.0.0 | 3.0.0.0 | Web.config |
| `Newtonsoft.Json` | 0.0.0.0–6.0.0.0 | 6.0.0.0 | Web.config |
| `System.Web.Optimization` | 1.0.0.0–1.1.0.0 | 1.1.0.0 | Web.config |
| `WebGrease` | 1.0.0.0–1.5.2.14234 | 1.5.2.14234 | Web.config |
| `System.Web.Helpers` | 1.0.0.0–3.0.0.0 | 3.0.0.0 | Web.config |
| `System.Web.WebPages` | 1.0.0.0–3.0.0.0 | 3.0.0.0 | Web.config |
| `System.Web.Mvc` | 1.0.0.0–5.2.2.0 | 5.2.2.0 | Web.config |

> Note: Several assembly binding redirects reference assemblies (Microsoft.Owin, Newtonsoft.Json, System.Web.Optimization, WebGrease) that are **not declared** in `packages.config`. These redirects appear to be remnants of a Visual Studio project template and are not currently used by the application.

## Startup Parameters & Resource Requirements

| Service | Runtime Options | Memory | CPU | Instance Count |
|---|---|---|---|---|
| CleanArchitectureAspNetMvc5 | Hosted by IIS / IIS Express; no custom JVM/CLR startup parameters | Not specified | Not specified | 1 (single IIS application) |

No Docker, Kubernetes, or container definitions are present. The application is deployed as a classic IIS web application.

## Startup Dependency Chain

The application has no external service dependencies and no startup ordering requirements. Startup sequence:

1. IIS / IIS Express hosts the application pool.
2. `MvcApplication.Application_Start()` fires (`Global.asax.cs`):
   - Clears default view engines.
   - Registers `CustomRazorViewEngine`.
   - Calls `AreaRegistration.RegisterAllAreas()`.
   - Calls `RouteConfig.RegisterRoutes(RouteTable.Routes)`.
3. Application is ready to serve requests.

No health checks, readiness probes, `dockerize` wait mechanisms, or Spring Cloud Config retry logic are present.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage |
|---|---|---|
| None detected | — | — |

No database connection strings, API keys, passwords, or other secrets are present in any configuration file. The application requires no runtime credentials.

### Secrets Provisioning Workflow

No secrets provisioning workflow exists. The application has no external service dependencies that would require credentials. If the application were extended with a database or external API, a secrets management strategy (e.g., Azure Key Vault with managed identity, or environment variable injection at deploy time) would need to be introduced.

## Feature Flags

No feature flag framework or conditional configuration is used. There are no `@ConditionalOnProperty` equivalents, LaunchDarkly integrations, or custom toggle mechanisms.

| Flag Name | Default | Controlled By |
|---|---|---|
| None detected | — | — |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| .NET Framework | 4.5 | Web.config `targetFramework`, CleanArchitectureAspNetMvc5.csproj |
| ASP.NET MVC | 5.2.2 | packages.config, Web.config assembly binding redirect |
| ASP.NET Razor | 3.2.2 | packages.config |
| ASP.NET WebPages | 3.2.2 | packages.config |
| Microsoft.Web.Infrastructure | 1.0.0.0 | packages.config |
| MSBuild Tools | 12.0 | CleanArchitectureAspNetMvc5.csproj `ToolsVersion` |
| Visual Studio (project format) | 10.0 | CleanArchitectureAspNetMvc5.csproj `VisualStudioVersion` |
| IIS Express | Unspecified | CleanArchitectureAspNetMvc5.csproj `UseIISExpress=true` |
