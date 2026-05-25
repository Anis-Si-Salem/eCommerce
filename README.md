# eCommerce

A full-stack eCommerce web application built with **ASP.NET Core MVC** following clean N-tier architecture with Entity Framework Core, AutoMapper, and SQL Server.

---

## Features

- **User management** — registration, login, profile viewing
- **Product catalog** — paginated product listing with search by name and tags
- **Order processing** — create orders with line items, dynamic price calculation, payment tracking
- **Chat system** — domain model for real-time messaging between users (UI pending)
- **Many-to-many tagging** — products linked to search tags via join table
- **Role-based user model** — User and Admin roles defined at domain level

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│  eCommerce.Web          (ASP.NET Core MVC)       │
│  Razor Views · Controllers · Bootstrap 5         │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  eCommerce.Service      (Business Logic Layer)   │
│  Services · DTOs · AutoMapper · Validation       │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  eCommerce.Data         (Data Access Layer)      │
│  EF Core DbContext · Generic Repository          │
└──────────────────┬──────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────┐
│  eCommerce.Domain       (Domain Layer)           │
│  Entities · Enums · Base Classes                 │
└─────────────────────────────────────────────────┘
```

### Layer Responsibilities

| Layer | Project | Description |
|-------|---------|-------------|
| **Domain** | `eCommerce.Domain` | 14 entity classes, 3 enums, abstract `Auditable` base class. Zero dependencies — pure domain logic. |
| **Data** | `eCommerce.Data` | Entity Framework Core 6 `DbContext` with 14 `DbSet` tables, 3 code-first migrations, generic `IRepository<T>` / `Repository<T>` with full CRUD + `IQueryable` support. |
| **Service** | `eCommerce.Service` | 8 service interfaces and implementations, 21 DTOs, AutoMapper 12 profiles, pagination via extension methods, custom exception handling. |
| **Web** | `eCommerce.Web` | ASP.NET Core MVC with Razor views, 6 controllers, 3 layout templates, Bootstrap 5, jQuery, client-side validation. Dependency injection wiring via custom `ServiceExtensions`. |

---

## Domain Model

```
Users Aggregate     Products Aggregate      Orders Aggregate       Chats Aggregate
─────────────       ─────────────────       ────────────────       ───────────────
User                Product                 Order                  Chat
 ├─ Address          ├─ ProductCategory      ├─ OrderItem            └─ Message
 ├─ Cart (stub)      ├─ SearchTag            ├─ Payment                 (self-referencing
 └─ CreditCard        └─ ProductSearchTag     └─ CreditCard              reply support)
                         (M2M join)
```

All entities inherit from `Auditable`: `Id (long)`, `CreatedAt (DateTime)`, `UpdatedAt (DateTime?)`.

**Enums:** `UserRole` (User, Admin), `PaymentType` (Cash, CreditCard), `OrderStatus` (Picking, Pending, Processing, Shipping, Shipped, Rejected)

---

## Tech Stack

| Category | Technology |
|----------|-----------|
| **Framework** | .NET 6.0, ASP.NET Core MVC |
| **Language** | C# |
| **Database** | SQL Server (LocalDB) |
| **ORM** | Entity Framework Core 6.0.15 |
| **Object Mapping** | AutoMapper 12.0.1 |
| **Frontend** | Razor Views, Bootstrap 5, jQuery 3 |
| **Validation** | jQuery Validation + Unobtrusive |
| **IDE** | Visual Studio 2022 (VS 17) |
| **Version Control** | Git |

---

## Patterns Used

- **N-Tier Architecture** — clear separation of concerns across 4 layers
- **Generic Repository Pattern** — `IRepository<T>` with expression-based querying
- **DTO Pattern** — domain entities never exposed to the UI layer
- **Dependency Injection** — all services and repositories registered via DI container
- **Code-First Migrations** — database schema managed through EF Core migrations
- **Fluent API Configuration** — all relationships configured in `OnModelCreating`

---

## Getting Started

### Prerequisites

- [.NET 6.0 SDK](https://dotnet.microsoft.com/en-us/download/dotnet/6.0)
- [SQL Server LocalDB](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/sql-server-express-localdb) (included with Visual Studio)
- Visual Studio 2022 (recommended) or any IDE with .NET support

### Setup

```bash
# Clone the repository
git clone <repo-url>
cd eCommerce-main

# Restore dependencies
dotnet restore

# Apply database migrations (auto-applied on first run, or manually)
dotnet ef database update --project src\eCommerce.Data --startup-project src\eCommerce.Web

# Run the application
dotnet run --project src\eCommerce.Web
```

The app will be available at `https://localhost:7160` or `http://localhost:5171`.

### Configuration

Connection string is in `src/eCommerce.Web/appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=eCommerceDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

---

## Project Structure

```
eCommerce-main/
├── eCommerce.sln
├── README.md
└── src/
    ├── eCommerce.Domain/         # Domain entities & enums
    │   ├── Commons/              # Auditable base class
    │   ├── Configurations/       # PaginationParams
    │   ├── Entities/             # Users, Products, Orders, Chats
    │   └── Enums/                # UserRole, PaymentType, OrderStatus
    ├── eCommerce.Data/           # Data access layer
    │   ├── DbContexts/           # eCommerceDbContext
    │   ├── IRepositories/        # IRepository<T>
    │   ├── Migrations/           # EF Core migrations
    │   └── Repositories/         # Repository<T>
    ├── eCommerce.Service/        # Business logic layer
    │   ├── Dtos/                 # Data transfer objects
    │   ├── Exceptions/           # CustomException
    │   ├── Extensions/           # Collection pagination extensions
    │   ├── Interfaces/           # Service interfaces
    │   ├── Mappers/              # AutoMapper profiles
    │   └── Services/             # Service implementations
    └── eCommerce.Web/            # Presentation layer
        ├── Controllers/          # MVC controllers
        ├── Extensions/           # DI service registration
        ├── Models/               # View models
        ├── Views/                # Razor views
        └── wwwroot/              # Static assets (Bootstrap, jQuery, CSS)
```

---

## Key Technical Highlights

- **Expression-based repository queries** for flexible, type-safe data access (`Expression<Func<T, bool>>`)
- **Two-phase product search** — direct name match combined with search-tag lookup via many-to-many join
- **Manual Unit of Work** — services explicitly control `SaveAsync` for atomic persistence
- **Deduplication pattern** in `SearchTagService` — reuses existing tags rather than creating duplicates
- **Pagination extension** with overflow protection — snaps to last page if index exceeds bounds
- **Auto-apply migrations** — `Database.Migrate()` called on DbContext instantiation for zero-config deployments
