# Lime PHP Framework

[![Latest Stable Version](https://poser.pugx.org/shaik-obydullah/lime-php-framework/v)](https://packagist.org/packages/shaik-obydullah/lime-php-framework)
[![License](https://poser.pugx.org/shaik-obydullah/lime-php-framework/license)](https://packagist.org/packages/shaik-obydullah/lime-php-framework)
[![PHP Version](https://poser.pugx.org/shaik-obydullah/lime-php-framework/require/php)](https://packagist.org/packages/shaik-obydullah/lime-php-framework)

A lightweight, minimal PHP MVC framework with routing, database abstraction, migrations, and a simple templating engine. Designed for small to medium web applications.

## Packagist

Available on [Packagist](https://packagist.org/packages/shaik-obydullah/lime-php-framework).

## Features

- **MVC Architecture** – Clean separation of concerns with Controllers, Models, and Views
- **Simple Router** – Define routes with support for URL parameters (`{id}`), GET/POST/PUT/DELETE
- **Database Layer** – PDO-based query builder with prepared statements (MySQL)
- **Migration System** – Run SQL migration files from the command line
- **View Engine** – Plain PHP templates with data extraction
- **Autoloading** – PSR-4-like autoloading for `Lime\` and `App\` namespaces
- **Environment Config** – `.env` file support
- **CLI Tool** – `php lime` for running migrations
- **JSON & Redirect Helpers** – Built into the base Controller

## Requirements

- PHP 8.1+
- MySQL 5.7+ / MariaDB 10.2+
- Composer (for third-party dependencies)
- Apache mod_rewrite (or nginx equivalent)

## Installation

### Via Composer (Recommended)

```bash
composer create-project shaik-obydullah/lime-php-framework my-app
```

### Manual Installation

```bash
mkdir my-app && cd my-app
```

Copy the contents of this repository into your project root:

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

### 2. Install dependencies

```bash
composer install
```

### 3. Configure environment

Copy `.env` and set your database credentials:

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=my_database
DB_USER=root
DB_PASS=secret
APP_KEY=your-32-char-secret-key
```

### 4. Run migrations

```bash
php lime migrate
```

### 5. Serve the application

```bash
php -S localhost:8000 -t Public
```

Point your browser to `http://localhost:8000`.

## Directory Structure

```
├── App/                    # Application code
│   ├── Controller/         # Controllers (e.g., HomeController.php)
│   ├── Database/
│   │   └── Migrations/     # SQL migration files (e.g., 001_create_users.sql)
│   ├── Model/              # Model classes (optional)
│   ├── Router/
│   │   └── web.php         # Route definitions
│   └── View/               # PHP view templates
├── Public/                 # Web root
│   ├── index.php           # Front controller (entry point)
│   ├── style.css           # Static assets
│   └── .htaccess           # URL rewriting rules
├── System/                 # Framework core (Lime namespace)
│   ├── Bootstrap.php       # Autoloader, env loader, constants
│   ├── Controller.php      # Base controller class
│   ├── Database.php        # Database connection & helpers
│   ├── Migration.php       # Migration runner
│   ├── Model.php           # Base model class
│   ├── Router.php          # Route matching & dispatching
│   └── View.php            # View renderer
├── storage/                # Writable file storage
├── vendor/                 # Composer dependencies
├── .env                    # Environment variables (git-ignore this)
├── .htaccess               # Root-level rewrite rules
├── composer.json           # Composer config
└── lime                    # CLI entry point
```

## Routing

Define routes in `App/Router/web.php`:

```php
use Lime\Router;

Router::get('/', 'HomeController@index');
Router::post('/login', 'AuthController@login');
Router::match(['GET', 'POST'], '/register', 'AuthController@register');
Router::get('/user/{id}', 'UserController@show');
```

### Available methods

| Method                                    | Description                        |
| ----------------------------------------- | ---------------------------------- |
| `Router::get($uri, $handler)`             | Register a GET route               |
| `Router::post($uri, $handler)`            | Register a POST route              |
| `Router::match($methods, $uri, $handler)` | Register for multiple HTTP methods |

Route handlers follow the format: `ControllerName@methodName`

### URL Parameters

Parameters are defined with curly braces and passed as method arguments:

```php
// Route: /user/{id}
Router::get('/user/{id}', 'UserController@show');

// Controller
class UserController extends Controller
{
    public function show(string $id): void
    {
        // $id is extracted from the URL
    }
}
```

## Controllers

Controllers extend `Lime\Controller` and are placed in `App/Controller/` under the `App\Controller` namespace.

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

### Base Controller Methods

| Method                        | Description             |
| ----------------------------- | ----------------------- |
| `$this->view($path, $data)`   | Render a view with data |
| `$this->redirect($url)`       | Redirect to a URL       |
| `$this->json($data, $status)` | Return JSON response    |

## Views

Views are plain PHP files in `App/View/`. Data passed from the controller is extracted as variables.

```php
// In controller: $this->view('home/index', ['title' => 'Hello']);
```

```php
<!-- App/View/home/index.php -->
<h1><?= htmlspecialchars($title) ?></h1>
```

View paths use dots for subdirectories: `home/index` maps to `App/View/home/index.php`.

## Database

The `Lime\Database` class provides a singleton PDO connection and helper methods.

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

// Execute a statement (UPDATE, DELETE)
$affected = Database::execute(
    'UPDATE users SET status = ? WHERE id = ?',
    ['online', 1]
);
```

### Methods

| Method                              | Returns        | Description                    |
| ----------------------------------- | -------------- | ------------------------------ |
| `Database::connect()`               | `PDO`          | Get the singleton PDO instance |
| `Database::query($sql, $params)`    | `PDOStatement` | Execute a prepared query       |
| `Database::fetch($sql, $params)`    | `?array`       | Fetch one row or null          |
| `Database::fetchAll($sql, $params)` | `array`        | Fetch all rows                 |
| `Database::insert($sql, $params)`   | `int`          | Insert and return lastInsertId |
| `Database::execute($sql, $params)`  | `int`          | Execute and return rowCount    |

All queries use prepared statements with automatic parameter binding for security.

## Models

Models extend `Lime\Model` and provide direct database access. They share the same PDO connection.

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

### Model Methods

| Method                           | Returns        | Description                     |
| -------------------------------- | -------------- | ------------------------------- |
| `Model::query($sql, $params)`    | `PDOStatement` | Execute a prepared query        |
| `Model::fetchAll($sql, $params)` | `array`        | Fetch all rows                  |
| `Model::fetchOne($sql, $params)` | `?array`       | Fetch one row or null           |
| `Model::execute($sql, $params)`  | `int`          | Execute and return lastInsertId |

## Migrations

Migration files are SQL files placed in `App/Database/Migrations/`. They are executed in filename order.

### Creating a migration

Create a file like `001_create_users.sql`:

```sql
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB;
```

### Running migrations

```bash
php lime migrate        # Run pending migrations
php lime migrate:reset  # Drop migrations table and reset
```

Migrations are tracked in a `migrations` table. Only files that haven't been run yet will execute.

## Environment Configuration

The framework loads a `.env` file from the project root. Variables are available via `$_ENV` and `getenv()`.

```env
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=database
DB_USER=root
DB_PASS=secret
APP_KEY=your-secret-key
```

Lines starting with `#` are treated as comments. The `.env` file should NOT be committed to version control.

## Serving in Production

### Apache

The included `.htaccess` files handle URL rewriting. Ensure `mod_rewrite` is enabled and `AllowOverride All` is set for the document root.

### Nginx

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

## Architecture Overview

### Request Lifecycle

1. `.htaccess` rewrites all requests to `Public/index.php`
2. `index.php` blocks direct access to internal directories
3. `System/Bootstrap.php` loads environment, registers autoloader, loads routes
4. `Router::dispatch()` matches the request URI and method
5. The matched controller method is called with extracted parameters
6. The controller renders a view or returns a JSON response

### Namespace Mapping

| Namespace | Directory |
| --------- | --------- |
| `Lime\`   | `System/` |
| `App\`    | `App/`    |

The autoloader in `Bootstrap.php` handles PSR-4-style class loading for both namespaces.

## Security

- All database queries use PDO prepared statements to prevent SQL injection
- View output should be escaped with `htmlspecialchars()`
- Direct access to `App/`, `System/`, `vendor/`, and `storage/` is blocked
- Root `.htaccess` restricts access to system directories

## License

MIT
