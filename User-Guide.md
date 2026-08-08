# Lime PHP Framework — User Guide

Version 1.0.0 · [shaik-obydullah/lime-php-framework](https://packagist.org/packages/shaik-obydullah/lime-php-framework)

---

## Table of Contents

1. [Welcome to Lime](#1-welcome-to-lime)
   - [Is Lime Right for You?](#11-is-lime-right-for-you)
   - [Feature Overview](#12-feature-overview)
2. [Server Requirements](#2-server-requirements)
3. [Installation](#3-installation)
   - [Composer Installation](#31-composer-installation-recommended)
   - [Manual Installation](#32-manual-installation)
   - [Configuration](#33-configuration)
   - [Running Migrations](#34-running-migrations)
   - [Serving the Application](#35-serving-the-application)
   - [Troubleshooting](#36-troubleshooting)
4. [Application Structure](#4-application-structure)
5. [Models, Views, and Controllers](#5-models-views-and-controllers)
6. [Autoloading](#6-autoloading)
7. [Request Lifecycle](#7-request-lifecycle)
8. [Routing](#8-routing)
   - [Defining Routes](#81-defining-routes)
   - [Route Registration Methods](#82-route-registration-methods)
   - [URL Parameters](#83-url-parameters)
   - [Unmatched Routes](#84-unmatched-routes)
9. [Controllers](#9-controllers)
   - [Base Controller](#91-the-base-controller)
   - [Rendering a View](#92-rendering-a-view)
   - [Redirecting](#93-redirecting)
   - [Returning JSON](#94-returning-json)
10. [Views](#10-views)
    - [View Paths](#101-view-paths)
    - [Passing Data to Views](#102-passing-data-to-views)
    - [Escaping Output](#103-escaping-output)
11. [Models](#11-models)
    - [Creating a Model](#111-creating-a-model)
    - [Base Model Methods](#112-base-model-methods)
12. [Working with Databases](#12-working-with-databases)
    - [Database Configuration](#121-database-configuration)
    - [Database Methods](#122-database-methods)
    - [Query Examples](#123-query-examples)
    - [Prepared Statements](#124-prepared-statements)
13. [Database Migrations](#13-database-migrations)
    - [Creating a Migration](#131-creating-a-migration)
    - [Running Migrations](#132-running-migrations)
    - [Migration Tracking](#133-migration-tracking)
14. [Environment Configuration](#14-environment-configuration)
15. [Command Line Reference](#15-command-line-reference)
16. [Security Guidelines](#16-security-guidelines)
17. [Serving in Production](#17-serving-in-production)
    - [Apache](#171-apache)
    - [Nginx](#172-nginx)
18. [Extending Lime](#18-extending-lime)
    - [Adding Framework Features](#181-adding-framework-features)
    - [Adding Application Features](#182-adding-application-features)
19. [Design and Architectural Goals](#19-design-and-architectural-goals)
20. [License Agreement](#20-license-agreement)

---

## 1. Welcome to Lime

Lime is a lightweight, dependency-free PHP MVC framework designed for small to medium web applications. It provides the essentials &mdash; routing, a PDO-based database abstraction, SQL-file migrations, plain-PHP templates, and a CLI tool &mdash; without the overhead of a full-stack framework.

It is a toolkit for people who build websites with PHP. Its goal is to let you develop projects faster than you could from scratch, while keeping every line of the framework small enough to read and understand. Lime minimizes the amount of code needed for a given task so you can focus on your project.

Lime stays out of your way. There is no service container, no ORM, and no templating language to learn &mdash; just clean, well-organized PHP that works the way you expect.

### 1.1 Is Lime Right for You?

Lime is right for you if:

- You want a framework with a small footprint (~300 lines of core).
- You want **zero runtime dependencies** &mdash; nothing to install beyond PHP itself.
- You need PHP 8.1+ features: strict types, match expressions, typed properties.
- You want a non-magical framework whose source you can read end to end in an afternoon.
- You want a framework that requires nearly zero configuration.
- You do not want to be forced to learn a templating language &mdash; views are plain PHP.
- You eschew complexity, favoring simple solutions.
- You want clear, thorough documentation (you're reading it).

Lime may **not** be right for you if you need built-in authentication, an ORM/query builder, form validation, session management, or a rich plugin ecosystem &mdash; those live in your application layer, not the core.

### 1.2 Feature Overview

- **MVC Architecture** &mdash; Clean separation of concerns with Controllers, Models, and Views.
- **Simple Router** &mdash; GET/POST/match registration with `{param}` URL capture.
- **Database Layer** &mdash; PDO-based, prepared-statement-only abstraction for MySQL/MariaDB.
- **Migration System** &mdash; Run plain SQL migration files from the command line.
- **View Engine** &mdash; Plain PHP templates with data extraction.
- **Autoloading** &mdash; Built-in PSR-4-style autoloader (Composer optional).
- **Environment Config** &mdash; `.env` file support.
- **CLI Tool** &mdash; `php lime` for running migrations.
- **JSON & Redirect Helpers** &mdash; Built into the base Controller.

---

## 2. Server Requirements

Before installing Lime, ensure your server meets these requirements:

| Requirement            | Minimum                        |
| ---------------------- | ------------------------------ |
| **PHP**                | 8.1 or later                   |
| **Database**           | MySQL 5.7+ or MariaDB 10.2+    |
| **Composer**           | Optional (only for Packagist / `vendor/autoload.php`) |
| **Web Server**         | Apache with `mod_rewrite` (or nginx equivalent) |
| **Extensions**         | `pdo_mysql`                    |

Lime has **no runtime PHP dependencies** (no Composer packages are required to run the framework). PHP 8.1 language features used include `declare(strict_types=1)`, typed properties, `match`, and `never` return types.

---

## 3. Installation

### 3.1 Composer Installation (Recommended)

```bash
composer create-project shaik-obydullah/lime-php-framework my-app
```

This installs the framework skeleton and creates Composer's `vendor/autoload.php`.

### 3.2 Manual Installation

Create a project directory and copy the repository contents into it:

```bash
mkdir my-app && cd my-app
```

Copy the repository contents into your project root:

```
my-app/
├── App/
│   ├── Controller/
│   ├── Database/
│   │   └── Migrations/
│   ├── Router/
│   │   └── web.php
│   └── View/
├── Public/
│   ├── index.php
│   └── .htaccess
├── System/
│   ├── Bootstrap.php
│   ├── Controller.php
│   ├── Database.php
│   ├── Migration.php
│   ├── Model.php
│   ├── Router.php
│   └── View.php
├── storage/
├── .env
├── .htaccess
├── composer.json
└── lime
```

(Optional) If you want Composer's autoloader:

```bash
composer install
```

The framework works fine without it &mdash; `System/Bootstrap.php` registers its own PSR-4-style autoloader when `vendor/autoload.php` is absent.

### 3.3 Configuration

Copy the environment template and set your database credentials:

```bash
cp .env.example .env
```

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=my_database
DB_USER=root
DB_PASS=secret
APP_KEY=your-32-char-secret-key
```

### 3.4 Running Migrations

```bash
php lime migrate
```

### 3.5 Serving the Application

Development server (web root is `Public/`):

```bash
php -S localhost:8000 -t Public
```

Point your browser to `http://localhost:8000`. You should see the welcome page.

### 3.6 Troubleshooting

| Symptom                                   | Likely cause                                   |
| ----------------------------------------- | ---------------------------------------------- |
| `404 - Route not found`                   | Route not defined in `App/Router/web.php`      |
| Blank page / connection refused           | `.env` missing or wrong DB credentials         |
| `View not found: ...`                     | View file missing or dot-path incorrect        |
| Database connection errors                | `pdo_mysql` extension not installed/enabled    |
| URL rewriting not working                 | `mod_rewrite` off or `AllowOverride All` not set |

---

## 4. Application Structure

Lime uses a clear, conventional layout that separates **your application code** (`App/`), the **web root** (`Public/`), and the **framework core** (`System/`).

```
├── App/                    # Application code (App\ namespace)
│   ├── Controller/         # Controllers (e.g., HomeController.php)
│   ├── Database/
│   │   └── Migrations/     # SQL migration files (e.g., 001_create_users.sql)
│   ├── Model/              # Model classes (optional)
│   ├── Router/
│   │   └── web.php         # Route definitions (the only route file)
│   └── View/               # PHP view templates
├── Public/                 # Web root (front controller)
│   ├── index.php           # Entry point for all web requests
│   ├── style.css           # Static assets
│   └── .htaccess           # URL rewriting rules
├── System/                 # Framework core (Lime\ namespace)
│   ├── Bootstrap.php       # Autoloader, env loader, constants, loads routes
│   ├── Controller.php      # Base controller class
│   ├── Database.php        # Singleton PDO wrapper
│   ├── Migration.php       # Migration runner
│   ├── Model.php           # Base model class
│   ├── Router.php          # Route matching & dispatching
│   └── View.php            # View renderer
├── storage/                # Writable file storage
├── vendor/                 # Composer dependencies (optional)
├── .env                    # Environment variables (git-ignore this)
├── .htaccess               # Root-level rewrite rules
├── composer.json           # Composer config
└── lime                    # CLI entry point
```

The only directory exposed to the web server is `Public/`. Everything else is blocked by `.htaccess` and the front controller.

---

## 5. Models, Views, and Controllers

Lime follows the classic Model-View-Controller pattern:

- **Model** &mdash; Encapsulates data access. Models extend `Lime\Model` and expose static methods that run prepared queries.
- **View** &mdash; Plain PHP files that present data. Controllers `extract()` data into the template scope.
- **Controller** &mdash; Receives the request via the router, coordinates the model and view, and sends the response.

A typical flow: the router calls `UserController@show` &rarr; the controller fetches a user through a model &rarr; the controller renders a view (or redirects, or returns JSON).

---

## 6. Autoloading

PSR-4-style autoloading is registered in `System/Bootstrap.php`. If `vendor/autoload.php` exists, it is used first; otherwise the built-in loader handles it.

| Namespace | Directory   |
| --------- | ----------- |
| `Lime\`   | `System/`   |
| `App\`    | `App/`      |

- New **framework core** classes go in `System/` under the `Lime\` namespace.
- New **application** classes go in `App/` under a matching sub-namespace (e.g. `App\Controller\`).
- Note: `App/Controller/` sub-directories are not auto-mapped by the built-in autoloader; a sub-directory requires the matching sub-namespace to resolve.

---

## 7. Request Lifecycle

A single request passes through seven steps:

1. Apache `.htaccess` (root + `Public/.htaccess`) rewrites all requests to `Public/index.php`.
2. `Public/index.php` defines `BASE_PATH`, blocks direct requests to `/App`, `/System`, `/vendor`, and `/.env`, then requires `System/Bootstrap.php`.
3. `Bootstrap.php` defines `APP_PATH` / `SYSTEM_PATH`, loads `.env` into `$_ENV`, registers the autoloader, and calls `Lime\Router::loadRoutes(APP_PATH . 'Router/web.php')`.
4. `index.php` instantiates `Lime\Router` and calls `dispatch()`.
5. `dispatch()` matches the HTTP method + URI against the route table, extracts `{param}` captures, instantiates `App\Controller\{Controller}`, and calls `{action}(...$params)`.
6. The controller renders a view, redirects, or returns JSON.
7. The response is emitted to the client.

Nothing runs that is not needed for the current request &mdash; the database connection, for example, is only created on first use.

---

## 8. Routing

### 8.1 Defining Routes

Define routes only in `App/Router/web.php`. Handlers are strings in `ControllerName@methodName` form; the namespace `App\Controller\` is prepended automatically.

```php
use Lime\Router;

Router::get('/', 'HomeController@index');
Router::post('/login', 'AuthController@login');
Router::match(['GET', 'POST'], '/register', 'AuthController@register');
Router::get('/user/{id}', 'UserController@show');
```

### 8.2 Route Registration Methods

| Method                                       | Description                         |
| -------------------------------------------- | ----------------------------------- |
| `Router::get($uri, $handler)`                | Register a GET route                |
| `Router::post($uri, $handler)`               | Register a POST route               |
| `Router::match($methods, $uri, $handler)`    | Register for multiple HTTP methods  |

### 8.3 URL Parameters

Parameters use `{name}` syntax and are passed positionally to the controller action:

```php
// Route: /user/{id}
Router::get('/user/{id}', 'UserController@show');

class UserController extends Controller
{
    public function show(string $id): void
    {
        // $id is extracted from the URL
    }
}
```

Captured values are always strings (they come from the URI); type-hint accordingly or cast inside the action.

### 8.4 Unmatched Routes

Unmatched routes, missing controllers, and missing methods return a plain-text `404` response (not JSON):

```
404 - Route not found
```

---

## 9. Controllers

Controllers extend `Lime\Controller` (abstract) and live in `App/Controller/` under the `App\Controller` namespace.

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use Lime\Controller;

class HomeController extends Controller
{
    public function index(): void
    {
        $this->view('home/index', [
            'title' => 'Welcome',
        ]);
    }
}
```

### 9.1 The Base Controller

The abstract base class provides three helpers for the three most common response types:

| Method                            | Description                |
| --------------------------------- | -------------------------- |
| `$this->view($path, $data)`       | Render a view with data    |
| `$this->redirect($url): never`    | Redirect to a URL and halt |
| `$this->json($data, $status = 200)` | Return a JSON response     |

### 9.2 Rendering a View

```php
$this->view('home/index', ['title' => 'Hello']);
```

### 9.3 Redirecting

```php
$this->redirect('/login');
```

### 9.4 Returning JSON

```php
$this->json(['ok' => true, 'user' => $user], 200);
```

---

## 10. Views

Views are plain PHP files in `App/View/`. No templating language is required &mdash; just PHP.

### 10.1 View Paths

View paths use dots for subdirectories: `home/index` maps to `App/View/home/index.php`.

### 10.2 Passing Data to Views

Controller data is `extract()`ed into the template scope:

```php
// In controller: $this->view('home/index', ['title' => 'Hello']);
```

```php
<!-- App/View/home/index.php -->
<h1><?= htmlspecialchars($title) ?></h1>
```

### 10.3 Escaping Output

Always escape output with `htmlspecialchars()` to prevent XSS:

```php
<?= htmlspecialchars($user['name']) ?>
```

---

## 11. Models

Models extend `Lime\Model` (abstract) and provide direct database access, sharing the same PDO configuration as the `Database` class.

### 11.1 Creating a Model

```php
<?php

declare(strict_types=1);

namespace App\Model;

use Lime\Model;

class User extends Model
{
    public static function findByEmail(string $email): ?array
    {
        return self::fetchOne('SELECT * FROM users WHERE email = ?', [$email]);
    }

    public static function create(array $data): int
    {
        return self::execute(
            'INSERT INTO users (name, email, password) VALUES (?, ?, ?)',
            [$data['name'], $data['email'], $data['password']]
        );
    }
}
```

Base model helpers are `protected static`; expose them through `public static` methods in your model.

### 11.2 Base Model Methods

| Method                           | Returns        | Description                     |
| -------------------------------- | -------------- | ------------------------------- |
| `Model::query($sql, $params)`    | `PDOStatement` | Execute a prepared query        |
| `Model::fetchAll($sql, $params)` | `array`        | Fetch all rows                  |
| `Model::fetchOne($sql, $params)` | `?array`       | Fetch one row or null           |
| `Model::execute($sql, $params)`  | `int`          | Execute and return lastInsertId |

> **Note:** `Model::execute()` returns the **last insert ID**, not a row count. Use `Database::execute()` (see below) when you need affected-row counts.

---

## 12. Working with Databases

### 12.1 Database Configuration

The `Lime\Database` class is a static, singleton PDO wrapper. It connects lazily (on first use) using these `.env` variables:

| Env var    | Default        |
| ---------- | -------------- |
| `DB_HOST`  | `chat_server`  |
| `DB_PORT`  | `3306`         |
| `DB_NAME`  | `chat_user`    |
| `DB_USER`  | `root`         |
| `DB_PASS`  | `root`         |

> The defaults are legacy placeholders &mdash; always set real values in your `.env` file.

Connection defaults: charset `utf8mb4`, `ERRMODE_EXCEPTION`, `FETCH_ASSOC`, native prepared statements (`EMULATE_PREPARES = false`).

### 12.2 Database Methods

| Method                              | Returns        | Description                    |
| ----------------------------------- | -------------- | ------------------------------ |
| `Database::connect()`               | `PDO`          | Get the singleton PDO instance |
| `Database::query($sql, $params)`    | `PDOStatement` | Execute a prepared query       |
| `Database::fetch($sql, $params)`    | `?array`       | Fetch one row or null          |
| `Database::fetchAll($sql, $params)` | `array`        | Fetch all rows                 |
| `Database::insert($sql, $params)`   | `int`          | Insert and return lastInsertId |
| `Database::execute($sql, $params)`  | `int`          | Execute and return rowCount    |

### 12.3 Query Examples

```php
use Lime\Database;

// Fetch a single row
$user = Database::fetch('SELECT * FROM users WHERE id = ?', [1]);

// Fetch all rows
$users = Database::fetchAll('SELECT * FROM users');

// Insert and get the last insert ID
$id = Database::insert(
    'INSERT INTO users (name, email) VALUES (?, ?)',
    ['John', 'john@example.com']
);

// UPDATE / DELETE (returns affected row count)
$affected = Database::execute(
    'UPDATE users SET status = ? WHERE id = ?',
    ['online', 1]
);
```

### 12.4 Prepared Statements

All queries **must** use `?` placeholders with parameter arrays &mdash; never interpolate values into SQL strings. This guarantees protection against SQL injection:

```php
// Correct
Database::fetch('SELECT * FROM users WHERE id = ?', [$id]);

// Wrong — never do this
Database::fetch("SELECT * FROM users WHERE id = $id");
```

---

## 13. Database Migrations

### 13.1 Creating a Migration

Migration files are plain SQL in `App/Database/Migrations/`, named like `001_create_users.sql`. They execute in filename order (glob + `sort`).

```sql
-- 001_create_users.sql
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

### 13.2 Running Migrations

```bash
php lime migrate        # run pending migrations
php lime migrate:reset  # drop the migrations table and reset
php lime                # show help
```

`migrate:reset` only drops the `migrations` table; it does **not** roll back schema. There is no down-migration support.

### 13.3 Migration Tracking

A `migrations` table tracks completed files:

| Column        | Type        |
| ------------- | ----------- |
| `id`          | INT (PK)    |
| `name`        | VARCHAR(255), UNIQUE |
| `executed_at` | TIMESTAMP   |

Only files that have not been run yet will execute.

---

## 14. Environment Configuration

The `.env` file lives at the project root (copy `.env.example`; do not commit it). The loader parses `KEY=VALUE` lines into `$_ENV`; `#` comments are skipped. There is no quoting, interpolation, or escaping support &mdash; keep values simple.

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=database
DB_USER=root
DB_PASS=secret
APP_KEY=your-secret-key
```

Values are available via `$_ENV` and `getenv()`:

```php
$host = $_ENV['DB_HOST'] ?? 'localhost';
```

> `APP_KEY` is defined in the example but is not currently consumed by the framework.

---

## 15. Command Line Reference

| Command                 | Description                          |
| ----------------------- | ------------------------------------ |
| `php lime`              | Show help                            |
| `php lime migrate`      | Run pending migrations               |
| `php lime migrate:reset`| Drop the migrations table and reset  |

Run all commands from the project root. The `lime` script boots the framework the same way `Public/index.php` does, so migrations use the same `.env` configuration.

---

## 16. Security Guidelines

- **SQL injection** &mdash; All queries use PDO prepared statements with `?` placeholders; never interpolate values.
- **XSS** &mdash; Escape all view output with `htmlspecialchars()`.
- **Source disclosure** &mdash; Direct access to `App/`, `System/`, `vendor/`, `storage/`, and `/.env` is blocked by the front controller and root `.htaccess`.
- **Credentials** &mdash; Never commit `.env`; keep `DB_PASS` and `APP_KEY` out of version control.
- **Input handling** &mdash; The core does not validate input; validate in your controllers/models. The suggested `vlucas/valitron` package is a lightweight option.

---

## 17. Serving in Production

### 17.1 Apache

The included `.htaccess` files handle URL rewriting. Ensure `mod_rewrite` is enabled and `AllowOverride All` is set for the document root. The web root is `Public/`.

### 17.2 Nginx

```nginx
server {
    listen 80;
    server_name example.com;
    root /path/to/project/Public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /(App|System|vendor|storage)/ {
        deny all;
        return 404;
    }
}
```

---

## 18. Extending Lime

### 18.1 Adding Framework Features

Create or edit a class under `System/` (namespace `Lime\`). Keep it dependency-free and document any new behavior in the `README.md`.

```php
<?php

declare(strict_types=1);

namespace Lime;

class Logger
{
    public static function info(string $message): void
    {
        // write to storage/logs/...
    }
}
```

### 18.2 Adding Application Features

- **Add a page:** add a route in `App/Router/web.php`, a controller in `App/Controller/`, and a view in `App/View/`.
- **Add a table:** create `App/Database/Migrations/00N_*.sql` and run `php lime migrate`.
- **Add a model:** extend `Lime\Model` and add `public static` methods using the protected helpers.

---

## 19. Design and Architectural Goals

- **Static APIs, not containers.** Framework services (Router/Database/View) use clean static APIs instead of a dependency-injection container &mdash; less abstraction, less to learn.
- **Plain SQL, not an ORM.** A thin, safe PDO wrapper lets you write the SQL you already know.
- **Plain PHP templates, not a DSL.** No compilation step, no new syntax.
- **SQL files as migrations.** Simple, reviewable, versioned schema changes.
- **Front-controller pattern.** Every request flows through `Public/index.php`.
- **Security by convention.** Prepared statements enforced in the API; escaping enforced by documentation.
- **Zero dependencies in the core.** Anything genuinely optional is added to `composer.json` under `suggest`.

---

## 20. License Agreement

Lime PHP Framework is open-source software licensed under the **MIT License**.

> Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
>
> The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

See the `LICENSE` file in the repository root for the full text.

---

© 2026 Shaik Obydullah · Lime PHP Framework v1.0.0
