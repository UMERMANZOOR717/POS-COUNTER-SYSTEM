# Counter POS

**A multi-outlet, AI-assisted Point-of-Sale system** built with SQL Server, Node.js/Express, and vanilla JavaScript — designed to be usable by any small-to-medium shopkeeper, from a general grocery store to a pharmacy.

![Node.js](https://img.shields.io/badge/Node.js-339933?style=flat&logo=node.js&logoColor=white)
![Express](https://img.shields.io/badge/Express.js-000000?style=flat&logo=express&logoColor=white)
![SQL Server](https://img.shields.io/badge/SQL%20Server-CC2927?style=flat&logo=microsoft-sql-server&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Tests](https://img.shields.io/badge/tests-34%20passing-brightgreen)
![License](https://img.shields.io/badge/license-MIT-blue)

---

## Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Database Design](#database-design)
- [System Workflow](#system-workflow)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Running Tests](#running-tests)
- [Project Structure](#project-structure)
- [Team](#team)
- [License](#license)

---

## Overview

Counter POS is a complete three-tier point-of-sale system built as a Database Management Systems project, then extended well beyond the original assignment scope. Every checkout runs as a single atomic SQL transaction with explicit row-level locking — guaranteeing two cashiers can never sell the same last unit of stock — and an integrated AI assistant answers natural-language business questions using only the store's real, live data.

## Key Features

- 🏢 **Multi-outlet support** — each branch's products, sales, and staff are fully isolated
- 🔐 **Role-based access control** — Owner / Manager / Cashier, enforced at the API level (not just hidden UI)
- 🔒 **Concurrency-safe checkout** — `UPDLOCK` + `HOLDLOCK` row locking inside a single stored procedure transaction
- 💰 **Automatic discounts** — loyalty (5+ past orders) and bulk-purchase (Rs. 50,000+) discounts computed inside the database
- 📦 **Barcode scanning** — works with any standard USB "keyboard wedge" scanner, zero drivers required
- 📅 **Expiry & manufacturing date tracking** — built for perishables and pharmacy use cases
- 📊 **Full reporting** — daily/weekly/monthly/yearly summaries, best/worst sellers via `RANK()` window functions
- 🤖 **AI Store Assistant** — natural-language Q&A powered by Google Gemini, grounded strictly in real data
- 📧 **Secure password recovery** — hashed, time-limited reset codes delivered by email
- 🧪 **34 automated tests** — Jest + Supertest, covering every critical authorization path

## Architecture

Three-tier architecture with an additional external AI service layer:

![Architecture Diagram](docs/diagrams/architecture.png)

| Layer | Technology |
|---|---|
| Front-end | HTML5, CSS3, vanilla JavaScript |
| Backend | Node.js + Express REST API |
| Database | Microsoft SQL Server |
| AI | Google Gemini API |

## Database Design

Normalized to 3NF across 13 entities, with full referential integrity and multi-outlet scoping throughout:

![Entity-Relationship Diagram](docs/diagrams/erd.png)

**Advanced database features:**
- Stored procedure (`sp_checkout`) wrapping the entire sale in one transaction with row-level locking
- Triggers enforcing stock rules independently of the application layer (`trg_prevent_overselling`, `trg_deduct_stock_on_sale`)
- 8 views powering reports without duplicating query logic
- Window functions (`RANK() OVER (PARTITION BY ...)`) for best/worst-seller rankings per outlet

## System Workflow

![System Workflow](docs/diagrams/workflow.png)

1. Owner registers the outlet + their own account (first-time setup only)
2. Manager/Cashier accounts register against that outlet
3. Cashier scans/searches a product → adds to cart → checks out
4. `sp_checkout` locks the relevant rows, validates stock, applies any automatic discount, and commits or rolls back atomically
5. A trigger deducts stock and logs the movement
6. The sale becomes instantly available to Reports and the AI Assistant

## Tech Stack

| Category | Technology |
|---|---|
| Database | Microsoft SQL Server 2019+ |
| Backend runtime | Node.js |
| Backend framework | Express.js |
| SQL driver | [`mssql`](https://www.npmjs.com/package/mssql) |
| Password hashing | [`bcryptjs`](https://www.npmjs.com/package/bcryptjs) |
| Email delivery | [`nodemailer`](https://www.npmjs.com/package/nodemailer) (Gmail SMTP) |
| AI provider | [Google Gemini API](https://ai.google.dev/) |
| Testing | Jest + Supertest |

## Getting Started

Full setup instructions are in [`START_HERE.md`](START_HERE.md), but in short:

```bash
# 1. Database — run these in SSMS, in order, each file on its own
database/00_create_database.sql
database/01_tables.sql
database/02_cart_type.sql
database/03-triggers/*.sql
database/04-procedures/*.sql
database/05-views/*.sql

# 2. Backend
cd backend
cp .env.example .env   # then fill in your SQL Server + Gemini credentials
npm install
npm start

# 3. Frontend
# Just open frontend/index.html in your browser
```

On first run, since the database is empty, the app shows an outlet registration screen instead of a login screen — register your outlet and owner account, then optionally run `database/06_seed_catalog.sql` for demo data.

## Running Tests

```bash
cd backend
npm test
```

34 tests run against a mocked database connection, so no live SQL Server instance is required. Coverage includes login/registration, role-based authorization (the critical Cashier-vs-Manager-vs-Owner logic for adding/deleting products), checkout success/failure paths, password reset, and the AI assistant endpoint.

## Project Structure

```
COUNTER-POS-SYSTEM/
├── database/
│   ├── 00_create_database.sql
│   ├── 01_tables.sql
│   ├── 02_cart_type.sql
│   ├── 03-triggers/
│   ├── 04-procedures/
│   ├── 05-views/
│   └── 06_seed_catalog.sql
├── backend/
│   ├── routes/          # setup, auth, products, sales, customers, reports, outlets, assistant
│   ├── db/pool.js        # SQL Server connection
│   ├── tests/            # Jest + Supertest suite
│   └── app.js / server.js
├── frontend/
│   ├── index.html
│   ├── js/ (app.js, api.js)
│   └── css/style.css
└── START_HERE.md
```

## Team

Built for our Database Management Systems coursework by:

- **Umer Manzoor** — database schema, triggers & stored procedures, concurrency testing, backend API
- **Mustafa Rehman** — front-end implementation, reporting views, AI Assistant integration, testing

## License

This project is available under the [MIT License](LICENSE).
