# Gift-Shop-Website
Web based website where you can choose gift and also suggest the best gifts. for those who is no so good at choosing gifts. 
# Giftoria — premium gift e-commerce for Nepal

Giftoria is a full-featured gift shop written in PHP (8.0+; 8.2+ recommended) with MySQL/MariaDB:

- an editorial storefront, with a rule-based **Gift Finder** recommendation engine and natural-language search;
- cart and checkout with NPR pricing and Nepali addresses;
- **eSewa** and **Khalti** payments, plus cash on delivery;
- an admin panel with role-based access control covering sales, marketing, inventory, content and reporting.

It runs on a stock **XAMPP** install (tested on XAMPP 8.0.30: PHP 8.0 + MariaDB 10.4). There is no framework, and Composer is not needed at runtime. It deploys to any PHP host.

> The brand name is configurable (`APP_NAME` in `.env` and **Admin → Settings → Store**).

### Images & copyright
- **Photography:** 131 demo photos in `public/assets/images/photos/` (tiles, banners, makers and about 60 product shots) are **CC0 1.0 / Public Domain Mark** works from Openverse, mostly from StockSnap and Rawpixel's public-domain collection. No attribution is required and commercial use is allowed. Provenance for every image is recorded in [CREDITS.md](CREDITS.md).
- **What was rejected:** every photo was reviewed by eye. Photos were rejected if they showed:
  - identifiable people, because CC0 photos rarely come with model releases;
  - trademarked characters or products, such as superhero figures or branded watches;
  - transparent clip-art;
  - the wrong subject.
- **Placeholders:** products without an accurate photo keep their generated SVG illustration. A stock photo of a *different* item on a product page would mislead buyers.
- **Before launch:** replace the demo product images with photos of your real products (Admin → Products → Media).
- **Tools:** `php bin/fetch-photos.php` re-runs the search (it needs the GD and cURL extensions). Rejected ids go in `database/seeders/photo-blocklist.txt`, and keys to leave as placeholders go in `photo-skip.txt`. `php bin/console.php photos` applies the photos to an existing database without touching images uploaded by staff.

---

## Contents
1. [Requirements](#1-requirements)
2. [Install on XAMPP (Windows)](#2-install-on-xampp-windows)
3. [First login & demo data](#3-first-login--demo-data)
4. [Payments (sandbox)](#4-payments-sandbox)
5. [Scheduled tasks (cron)](#5-scheduled-tasks-cron)
6. [Running the tests](#6-running-the-tests)
7. [Acceptance checklist](#7-acceptance-checklist)
8. [Project structure](#8-project-structure)
9. [Further documentation](#9-further-documentation)

---

## 1. Requirements

| Component | Version |
|---|---|
| XAMPP | 8.0.30 or newer (PHP ≥ 8.0, MariaDB ≥ 10.4). **PHP 8.2+ recommended for production**: PHP 8.0 no longer receives security fixes. |
| PHP extensions | `pdo_mysql`, `mbstring`, `curl`, `openssl`, `fileinfo`, `json` (all enabled in XAMPP by default) |
| Apache | `mod_rewrite` enabled (on by default in XAMPP) |
| Internet | Needed for live eSewa/Khalti sandbox calls and Google Fonts. Everything else works offline. |

## 2. Install on XAMPP (Windows)

1. **Install XAMPP** 8.2+ from https://www.apachefriends.org and open the XAMPP Control Panel.
2. **Start Apache and MySQL.**
3. **Copy the project** so that this README is at `C:\xampp\htdocs\giftoria\README.md`.
4. **Create the configuration file:** copy `.env.example` to `.env` in the same folder. For a default XAMPP install the defaults already work:
   ```ini
   APP_URL=http://localhost/giftoria
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=giftoria
   DB_USERNAME=root
   DB_PASSWORD=
   ```
   If you use the web installer, set `INSTALL_TOKEN` to a long random value:
   ```bash
   C:\xampp\php\php.exe -r "echo bin2hex(random_bytes(24));"
   ```
5. **Composer is optional.** Nothing is needed to run the site. Composer is only used to get PHPUnit (`composer install`).
6. **Create the database, tables and base data, and the first administrator.** Choose **one** option:

   **Option A: command line (recommended).** Open a terminal in `C:\xampp\htdocs\giftoria`:
   ```bash
   C:\xampp\php\php.exe bin\console.php install --demo
   ```
   This creates the `giftoria` database if needed, runs `database/migrations/*.sql`, seeds the base data (roles, permissions, gift taxonomies, delivery zones, email templates, homepage sections) and the demo catalogue (`--demo`). It then asks for the **first Super Admin** name, email and password (minimum 12 characters). Add `--staff` to also create demo staff accounts for each role. Demo passwords are random and printed once.

   **Option B: browser.** Open http://localhost/giftoria/install, enter the `INSTALL_TOKEN`, and fill in the administrator details. Tick "Load demo catalogue" if you want sample data.

   Both options refuse to run again once a Super Admin exists. `storage/installed.lock` is written, the installer route returns 404, and `create-admin` exits with an error. There is no default or backdoor account.
7. **Open the site.**
   - Storefront: **http://localhost/giftoria/**
   - Admin: **http://localhost/giftoria/admin** (the path is configurable with `ADMIN_PATH`)

   **Apache rewrite rules.** `giftoria/.htaccess` forwards every request into `public/`, and `public/.htaccess` routes everything except real files to `public/index.php`. As a result, `app/`, `config/`, `storage/` and `.env` are never reachable from the web. If you see "Not Found" for every page, `mod_rewrite` or `AllowOverride All` is disabled (see `docs/TROUBLESHOOTING.md`).
8. **Seed more data later:** `php bin\console.php seed --demo` (skips safely if products already exist).
9. **Mail.** `MAIL_MODE=log` (the default) records every email in **Admin → Email templates → Outbox** and in `storage/mail/*.html`, clearly marked "Logged · not sent". Switch to `MAIL_MODE=smtp` and fill in `MAIL_*` to deliver real email. An email is marked *sent* only when the SMTP server accepts it.

## 3. First login & demo data

- Sign in at `/admin/login` with the Super Admin you created. Create staff from **Admin → Staff**. Each new staff member gets an invitation link to set their own password; in local development the link is also shown on screen.
- The demo catalogue includes:
  - 61 products tagged with recipients, occasions, interests, age ranges and safety data;
  - 6 makers, two coupons (`WELCOME10`, `FREESHIP`) and one live seasonal campaign;
  - 2 demo customers and 17 demo orders (cash on delivery, in various statuses) so the dashboard has data.
- No reviews or ratings are invented. Ratings appear only from real, moderated customer reviews.
- **Before going live,** delete the demo customers and staff (Admin → Customers / Staff), or reinstall without `--demo`.

## 4. Payments (sandbox)

Payments are covered in detail in **[docs/PAYMENTS.md](docs/PAYMENTS.md)**. In short:

- **eSewa:** works out of the box in test mode with eSewa's public sandbox merchant (`EPAYTEST`). Leave `ESEWA_ENV=test`, and the product code and secret empty.
- **Khalti:** create a free merchant account at https://test-admin.khalti.com and put its sandbox secret key in `KHALTI_SECRET_KEY`. The sandbox wallets are 9800000000–9800000005, MPIN `1111`, OTP `987654`.
- **Offline development:** set `PAYMENT_SIMULATOR=true` to replace the gateway pages with a local simulator. The real verification code still runs, through a simulated HTTP transport. It is disabled automatically when `APP_ENV=production`.
- **When an order counts as paid:** only after a **server-side** status check (eSewa Status API / Khalti lookup) confirms success for the exact amount and reference. Repeated callbacks are idempotent.

## 5. Scheduled tasks (cron)

Run this every 5–15 minutes. On Windows use Task Scheduler; on Linux use `crontab -e`.
```bash
php bin/console.php cron
```
It does the following:
- Cancels unpaid online orders after `PAYMENT_TIMEOUT_MINUTES`. It re-verifies with the gateway first, then releases the reserved stock once.
- Sends gift-occasion reminders and abandoned-cart reminders. Both are opt-in and sent once.
- Sends a daily low-stock email to the address set in Settings.
- Cleans up expired rate-limit rows.

## 6. Running the tests

Integration tests build a throwaway database named `<DB_DATABASE>_test`. The runner refuses to use any other name.
```bash
php tests/run.php
```
This needs no Composer. With Composer you can run real PHPUnit instead: `composer install`, then `vendor/bin/phpunit`.

The suite covers the scoring rules, query parsing (synonyms, prices, typos), eSewa signatures, the state machine, validation, and the acceptance scenarios below (29 tests).

## 7. Acceptance checklist

| Test | Covered by | How to check by hand |
|---|---|---|
| A: "gift for baby girl" | `testA_…` | Search box → *gift for baby girl* |
| B: filtered recommendations | `testB_…` | Gift Finder → Baby girl → First birthday → Princess toys + Educational toys → 1–2 years → custom 500–2,500 |
| C: misspelled/unknown interest | `testC_…` | Search *pricess toyz*; type an unknown word in the Gift Finder's "Something else" box |
| D: register/verify/login/wishlist/addresses/orders | `testD_…` | `/register`, then the verification link in Admin → Outbox, `/account` |
| E: cart totals recalculated on the server | `testE_…` | Change quantities on `/cart` |
| F/G/H: checkout, verification, duplicate callbacks | `testFGH_…`, `testKhalti…` | Checkout with eSewa (sandbox or simulator) |
| I: marketing staff can't touch inventory or refunds | `testIJ_…` | Sign in as `marketing-staff` (demo `--staff`), open `/admin/inventory` and get 403 |
| J: no public route grants admin | `testIJ_…` | Registration only creates customers; there are no public staff/role routes |
| K: product manager manages products | `testK_…` | Sign in as `product-manager`, then Products → New / Publish / Deactivate |
| L: coupon + scheduled campaign | `testL_…` | Admin → Coupons / Campaigns, set start dates in the future |
| M: mobile UI | manual | Every storefront and admin page fits a 375 px screen with no horizontal scrolling |
| N: last unit can't be sold twice | `testN_…` | The demo product "Rosewood Wedding Keepsake Box" has exactly 1 unit |
| O: installation | this README | Follow section 2 on a clean XAMPP |

## 8. Project structure

```
giftoria/
  app/
    Core/            framework: Kernel, Router, Request/Response, Database (PDO), Session, Csrf,
                     Validator, Auth (customer & staff guards), Gate (RBAC), RateLimiter, Cache,
                     Uploader, SmtpTransport, AuditLogger, Logger
    Controllers/     storefront controllers, Account/ (customer area), Admin/ (back office)
    Middleware/      auth, guest, staff, can:<permission>
    Repositories/    ProductRepository, TaxonomyRepository, Admin/* reporting queries
    Services/        RecommendationEngine (+ Scorer, QueryParser, Vocabulary), SearchService,
                     CartService, PricingService, CouponService, ShippingService, OrderService,
                     OrderStateMachine, InventoryService, PaymentService, Payments/ (eSewa, Khalti),
                     MailService, AuthService, NewsletterService, ReminderService, HomepageService…
    Helpers/         global helpers (e(), url(), money()…), inline Lucide icons
  bin/console.php    install, migrate, seed, create-admin, cron, cache:clear, routes
  config/            app, database, mail, payments, permissions (roles), nepal (provinces/districts)
  database/
    migrations/      ordered SQL files
    seeders/         BaseSeeder (configuration), DemoSeeder (catalogue), ArtSeeder (SVG artwork)
  public/            web root: index.php, .htaccess, assets/, uploads/ (PHP execution disabled)
  resources/views/   layouts, components, storefront/, account/, admin/, emails/, errors/
  routes/            web.php, account.php, admin.php, admin/*.php (one file per admin module)
  storage/           logs, cache, sessions, mail previews (blocked from the web)
  tests/             Unit/, Integration/, run.php (zero-dependency runner), PHPUnit config
  docs/              architecture, payments, deployment & security, troubleshooting, conventions
```

## 9. Further documentation

- **[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md):** request flow, database entities and relationships, the recommendation algorithm, and order/payment state machines.
- **[docs/PAYMENTS.md](docs/PAYMENTS.md):** eSewa and Khalti sandbox setup, verification, refunds, and going live.
- **[docs/DEPLOYMENT.md](docs/DEPLOYMENT.md):** production checklist, security review, backups and restore.
- **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md):** common problems and fixes.
- **[docs/CONVENTIONS.md](docs/CONVENTIONS.md):** coding conventions for contributors.

### What still needs your input
- **Credentials:**
  - Khalti sandbox/live secret key.
  - eSewa production merchant code and secret, after eSewa merchant onboarding.
  - SMTP account.
- **Content:**
  - Real product photography and copy.
  - Your store address and phone (Admin → Settings).
  - Delivery zone rates.
  - Legal review of the policy pages in `resources/views/storefront/pages/`.
- **Optional AI re-ranking:** `AI_PROVIDER` is reserved for a future provider integration. The Gift Finder is fully rule-based and does not call any AI service.
