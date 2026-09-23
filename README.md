# catalog-baseline-01

An ASP.NET Core Razor Pages app on .NET 8 (`catalog/catalog.csproj`).

It has no Node or Python dependencies. The front-end libraries (Bootstrap, jQuery, jQuery Validation) are already committed under `catalog/wwwroot/lib`.

## Prerequisites

- [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0). Tested with 8.0.416 and 8.0.420.

## Setup

```bash
dotnet restore
dotnet build
```

`dotnet restore` downloads the NuGet packages:

- Microsoft.EntityFrameworkCore.InMemory, .Sqlite, .SqlServer and .Design
- ServiceStack.Redis
- Microsoft.Extensions.Logging.Debug
- Microsoft.VisualStudio.Web.CodeGeneration.Design

The build shows one warning (`NETSDK1206`) about an Alpine-only SQLite library. You can ignore it when running on Ubuntu or WSL.

## Running

```bash
cd catalog && dotnet run
```

By default the app runs at http://localhost:5000 and https://localhost:5001 (see `catalog/Properties/launchSettings.json`). There are two pages: `/` and `/Weather`.

## Notes

- **You don't need a database or Redis to run it locally.** `Startup.cs` sets `useInMemory = true`, so the app uses an in-memory database. The Redis calls in `Pages/Index.cshtml.cs` are commented out. You only need to fill in the placeholder connection strings in `appsettings.json` if you switch to Azure SQL or turn Redis on.
- **Switching to Azure SQL:** follow `catalog/connect_azure_sql.txt`. You'll also need the EF Core CLI tool:
  ```bash
  dotnet tool install --global dotnet-ef
  ```
- **Version mismatch:** the EF Core packages are version 6.0.3 but the project targets .NET 8. The app works as is, but upgrading the packages to 8.x would be cleaner.
- **Build output:** `.gitignore` excludes the `bin/` and `obj/` folders, so building doesn't create git changes. If you cloned an older commit that still tracks `obj/`, run `git rm -r --cached catalog/obj` to stop tracking it.
