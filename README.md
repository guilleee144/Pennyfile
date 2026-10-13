# Pennyfile

**Simple expense and invoice management for freelancers and micro-businesses.**

Pennyfile is a web application that centralises a small business's income and expenses in one place. Receipts and invoices are photographed with a phone, an OCR service extracts the data automatically, and the app handles expense approvals, client invoicing, VAT calculation and cash-flow reporting.

> **Status:** early development. The foundation (Blazor Server project with ASP.NET Core Identity) is in place; features below are being built incrementally and are marked accordingly.

This project is the final-year project for the BSc in Computing at Dorset College, but it is designed with the aim of becoming a real, sellable product rather than only an academic exercise.

---

## Table of contents

- [The problem](#the-problem)
- [Target users](#target-users)
- [Features](#features)
- [Tech stack](#tech-stack)
- [Architecture](#architecture)
- [Getting started](#getting-started)
- [Project structure](#project-structure)
- [Testing](#testing)
- [Deployment](#deployment)
- [Roadmap](#roadmap)
- [Academic context](#academic-context)

---

## The problem

Freelancers and micro-businesses (1–5 employees) typically keep their accounts on paper, in spreadsheets, or as loose photos on a phone. Receipts get lost, deductible expenses are forgotten, and preparing tax returns is slow and error-prone. Existing tools (Xero, QuickBooks, Sage) are expensive and far more complex than a business of this size needs.

Pennyfile aims to be the lightweight alternative, focused on the Irish and UK market.

## Target users

| Role | Permissions |
|------|-------------|
| **Admin** (business owner) | Manages users, approves expenses, creates and sends invoices, sees the full financial dashboard. |
| **Employee** | Uploads their own receipts and expenses (pending approval). Cannot see invoicing or the dashboard. |
| **Accountant** | Read-only access to reports and invoicing. |

Access control is always enforced **on the server**, never only by hiding UI elements.

## Features

### MVP

- [ ] Authentication with roles (ASP.NET Core Identity)
- [ ] User management
- [ ] Receipt scanning with OCR (Azure `prebuilt-receipt`)
- [ ] Income invoice scanning with OCR (Azure `prebuilt-invoice`) and automatic client matching
- [ ] Expense approval workflow
- [ ] Client management (CRUD)
- [ ] Invoicing with line items and VAT calculation
- [ ] Dashboard: expenses vs income
- [ ] Tax report export (CSV)

### Planned additions

- [ ] Duplicate expense detection
- [ ] Recurring expenses
- [ ] Category budgets with alerts
- [ ] Payments with Stripe (payment link + webhook that marks invoices as paid)
- [ ] Automatic reminders for overdue invoices
- [ ] Accountant role

See the [Roadmap](#roadmap) for longer-term ideas.

## Tech stack

| Area | Technology |
|------|------------|
| Frontend + backend | Blazor Server on ASP.NET Core (C#) |
| Authentication | ASP.NET Core Identity (roles) |
| Data access | Entity Framework Core (migrations) |
| Database | PostgreSQL hosted on Supabase (database and image storage only; Supabase Auth is **not** used) |
| Background jobs | `IHostedService` (recurring expenses, reminders, budget checks) |
| OCR | Azure AI Document Intelligence |
| Payments | Stripe |
| Email | SendGrid or Azure Communication Services |
| Testing | xUnit (unit and integration with `WebApplicationFactory`), Playwright (end-to-end) |
| Deployment | Docker, Azure App Service, GitHub Actions (CI/CD) |

**Why Blazor Server?** A single language (C#) for the whole application, and no need to manage tokens between client and server since the UI runs on the server.

## Architecture

Pennyfile follows a layered architecture with dependency injection:

```
Presentation (Blazor components)
        │
        ▼
Application services (business logic)
        │
        ▼
Data access (EF Core / repositories)
        │
        ▼
PostgreSQL (Supabase)
```

Key design points:

- **Multi-tenant by design:** every business-owned entity carries a `BusinessId`, and all queries are filtered by the current user's business.
- **Server-side authorization:** roles and policies are enforced in services and endpoints, not just in the UI.
- **Separation of concerns:** components do not access the database directly; they call application services.

### Main entities

`Business` (tenant), `ApplicationUser`, `Client`, `ExpenseCategory`, `Expense`, `Invoice`, `InvoiceLineItem`, `Budget`, `RecurringExpense`, `Payment`, `AuditLog`.

## Getting started

### Prerequisites

- [.NET SDK](https://dotnet.microsoft.com/download) (the version targeted by `Pennyfile.csproj`)
- Visual Studio 2022 or later (or any editor with the .NET CLI)
- Git
- A PostgreSQL database (a free Supabase project works)
- Optional, for later features: Azure AI Document Intelligence, Stripe and email provider accounts

### Setup

1. **Clone the repository**

   ```bash
   git clone https://github.com/<your-username>/Pennyfile.git
   cd Pennyfile/Pennyfile
   ```

2. **Configure secrets.** Never commit credentials. Use [.NET User Secrets](https://learn.microsoft.com/aspnet/core/security/app-secrets) for local development:

   ```bash
   dotnet user-secrets init
   dotnet user-secrets set "ConnectionStrings:DefaultConnection" "<your-connection-string>"
   ```

   Further settings (added as the features are built) will also go in User Secrets, for example Azure Document Intelligence, Stripe and email keys.

3. **Apply database migrations**

   ```bash
   dotnet ef database update
   ```

4. **Run the application**

   ```bash
   dotnet run
   ```

   Or press **F5** in Visual Studio.

## Project structure

Current layout:

```
Pennyfile/
├── .gitignore
├── README.md
└── Pennyfile/                  # Blazor Server project
    ├── Components/             # Blazor components, pages and Identity UI
    ├── Data/                   # DbContext, entities, EF Core migrations
    ├── Properties/
    ├── wwwroot/
    ├── appsettings.json
    └── Program.cs
```

The folders for application services and domain models will be added as the layered architecture is built out.

## Testing

Automated testing is a core requirement of the project:

- **Unit tests** (xUnit) for business logic such as VAT calculation and approval rules
- **Integration tests** (xUnit + `WebApplicationFactory`) for services, endpoints and authorization
- **End-to-end tests** (Playwright) for the main user flows

```bash
dotnet test
```

## Deployment

The target deployment is a Docker container on **Azure App Service**, with **GitHub Actions** running build, test and deploy on every push to the main branch. Configuration and secrets are supplied through environment variables in production.

## Roadmap

Not implemented, but planned for the future:

- Multi-currency support
- Bank integration (Open Banking)
- Cash-flow forecasting
- Standalone REST API for a future mobile app

## Academic context

This project is developed to meet Dorset College's minimum technical standards for the final project:

- No low-code or no-code tools
- Authentication through an established framework (no custom auth)
- Real, server-enforced authorization
- Layered architecture
- Automated testing
- Version control with meaningful commits
- Deployment outside the development machine
- Full documentation: README, architecture, data model and design decisions

---

*Author: <Guillermo Pando & Guillem Fernández> — BSc in Computing, Dorset College.*
