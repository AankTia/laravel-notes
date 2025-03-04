# Route Prefixing, Naming, and Namespacing

In Laravel, you can group routes using aliases by leveraging **Route Prefixing, Naming, and Namespacing**. Below are different ways to achieve alias-based route grouping.

---

### **1. Using Route Name Prefix (Alias for Route Names)**
You can prefix route names using the `name` method when grouping routes.

#### **Example: Grouping Routes with a Name Prefix**
```php
use App\Http\Controllers\AdminController;
use App\Http\Controllers\UserController;
use Illuminate\Support\Facades\Route;

Route::name('admin.')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard'])->name('dashboard');
    Route::get('/settings', [AdminController::class, 'settings'])->name('settings');
});
```
#### **Usage in Blade or Controller**
```php
route('admin.dashboard'); // Generates: /dashboard
route('admin.settings'); // Generates: /settings
```

---

### **2. Using Route Prefix and Naming Together**
You can also combine route prefixes and name prefixes for better aliasing.

#### **Example: Admin Panel Routes with Prefix & Name**
```php
Route::prefix('admin')->name('admin.')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard'])->name('dashboard');
    Route::get('/settings', [AdminController::class, 'settings'])->name('settings');
});
```
#### **Generated URLs & Named Routes**
| Route Name | URL |
|------------|----------------|
| `admin.dashboard` | `/admin/dashboard` |
| `admin.settings` | `/admin/settings` |

#### **Usage in Blade**
```html
<a href="{{ route('admin.dashboard') }}">Dashboard</a>
<a href="{{ route('admin.settings') }}">Settings</a>
```

---

### **3. Using Route Namespace for Controller Grouping (Laravel 8+)**
Since Laravel 8, the `namespace` property was removed, so you need to explicitly define namespaces in route groups.

```php
use App\Http\Controllers\Admin\AdminController;
use Illuminate\Support\Facades\Route;

Route::prefix('admin')->name('admin.')->namespace('App\Http\Controllers\Admin')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard'])->name('dashboard');
});
```

Alternatively, without the `namespace` key (recommended for Laravel 8+):

```php
use App\Http\Controllers\Admin\AdminController;
use Illuminate\Support\Facades\Route;

Route::prefix('admin')->name('admin.')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard'])->name('dashboard');
});
```

---

### **4. Middleware with Route Grouping**
You can also apply middleware along with route aliasing:

```php
Route::middleware(['auth', 'admin'])->prefix('admin')->name('admin.')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard'])->name('dashboard');
});
```

---

### **5. Route Aliases with Resource Controllers**
When defining resource routes, you can add name prefixes as well:

```php
use App\Http\Controllers\UserController;

Route::prefix('user')->name('user.')->group(function () {
    Route::resource('profiles', UserController::class);
});
```

This results in route names like:
- `user.profiles.index`
- `user.profiles.create`
- `user.profiles.store`
- `user.profiles.show`
- `user.profiles.edit`
- `user.profiles.update`
- `user.profiles.destroy`

```php
<?php
use App\Http\Controllers\PostController;
use Illuminate\Support\Facades\Route;

// Basic resource routing
Route::resource('posts', PostController::class);

// This single line creates the following routes:
// GET /posts (index)
// GET /posts/create (create)
// POST /posts (store)
// GET /posts/{post} (show)
// GET /posts/{post}/edit (edit)
// PUT/PATCH /posts/{post} (update)
// DELETE /posts/{post} (destroy)

// Customizing resource routes
Route::resource('posts', PostController::class)->only([
    'index', 'show'  // Only generate index and show routes
]);

Route::resource('posts', PostController::class)->except([
    'destroy'  // Generate all routes except destroy
]);

// Nested resource routes
Route::resource('posts.comments', CommentController::class);
// This creates routes like /posts/{post}/comments
```

Key points about `Route::resource()`:

- Automatically generates standard RESTful routes
- Maps directly to controller methods
- Can be customized with only() and except() methods
- Supports nested resource routing
- Reduces boilerplate routing code

Corresponding Controller Methods:

- index(): Display a list of resources
- create(): Show form to create a new resource
- store(): Save a new resource
- show(): Display a specific resource
- edit(): Show form to edit a resource
- update(): Update a specific resource
- destroy(): Delete a specific resource

Pro tip: Use php artisan route:list to see all generated routes for your resources.

---

### **Conclusion**
Using route grouping with aliasing (`name`), prefixes, and middleware improves code organization.