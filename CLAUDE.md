# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single ASP.NET Core Razor Pages app (`catalog/catalog.csproj`, .NET 8) showing a book catalog with a shopping cart, plus a weather page. Front-end libraries (Bootstrap, jQuery, jQuery Validation) are committed under `catalog/wwwroot/lib`; there is no Node/npm tooling.

## Commands

```bash
dotnet restore                 # from repo root (uses catalog-baseline-01.sln)
dotnet build                   # expect one NETSDK1206 warning (Alpine-only SQLite lib) — safe to ignore
cd catalog && dotnet run       # http://localhost:5000, https://localhost:5001 (Properties/launchSettings.json)
dotnet publish -c Release      # what CI runs before deploying
```

There is no test project and no linter configured.

## Architecture

- **Hosting**: old-style `Program.cs` + `Startup.cs` (generic host with `UseStartup<Startup>`), not minimal hosting. Services and the middleware pipeline live in `Startup.cs`.
- **Data**: EF Core `BookContext` (`Data/BookContext.cs`) with a single `DbSet<Book>`. `Startup.useInMemory` (hard-coded `true`) picks the provider: in-memory DB `"test"` vs. SQL Server via the `BooksDB` connection string. There are no migrations in the repo; switching to Azure SQL means setting `useInMemory = false` and following `catalog/connect_azure_sql.txt` (`dotnet ef migrations add -c BookContext InitialCreate`, `dotnet ef database update`).
- **Seeding**: the DB starts empty. `BookLoader.LoadBooks` wipes and re-seeds four hard-coded books; it's triggered by the "load" form on `/` (`OnPostLoad` in `Pages/Index.cshtml.cs`).
- **Page handlers**: pages pass data to views via `ViewData` (e.g. `ViewData["books"]`, `ViewData["Error"]`) rather than bound model properties. Forms use `asp-page-handler="x"`, mapping to `OnPostX` methods.
- **Redis cart**: `ServiceStack.Redis` is referenced, and `IndexModel.GetRedisClient()` reads `Redis:ConnectionString`, but the cart logic in `OnGet` and `OnPostAddToShoppingCart` is commented out (marked `UNCOMMENT AFTER ADDING REDIS`). With it disabled, "add to cart" is a no-op redirect.
- **Weather page**: `OnPostWeather` makes a synchronous `HttpClient` GET to `http://{ip}/api/weather`, where `ip` comes from user input. It depends on a separate weather service that is not in this repo.
- `appsettings.json` holds placeholder connection strings only; nothing needs to be filled in for local in-memory runs.

## Gotchas

- EF Core packages are pinned to 6.0.3 while the target framework is net8.0. It works; upgrading to 8.x would align them.
- CI (`.github/workflows/`): both workflows deploy to Azure Web Apps on every push to `main`. `main_readit.yml` builds/publishes the .NET app (app `Readit`). `main_group3.yml` is a Node.js template (`npm install`, app `Group3`) that doesn't match this project.
- Static image paths are case-sensitive on Linux: `Index.cshtml` references `~/images/cart.jpg` but the file is `cart.JPG`, and the seeded book `ImageUrl`s (`rama.jpg`, etc.) have no matching files in `wwwroot/images`.
