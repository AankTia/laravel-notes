# Implementing Laravel Auth::can() Permissions Step by Step

Here's a comprehensive guide to implementing permission checks with Laravel's `Auth::can()` method:

## 1. Install Laravel and Set Up Authentication

First, ensure you have Laravel installed with basic authentication:

```bash
composer require laravel/ui
php artisan ui bootstrap --auth
php artisan migrate
```

## 2. Create Permission and Role Models

Create the necessary models:

```bash
php artisan make:model Permission -m
php artisan make:model Role -m
```

## 3. Set Up Database Migrations

Modify the created migration files:

### For roles table:

```php
Schema::create('roles', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('slug')->unique();
    $table->timestamps();
});
```

### For permissions table:

```php
Schema::create('permissions', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('slug')->unique();
    $table->timestamps();
});
```

### Create pivot tables for many-to-many relationships:

```bash
php artisan make:migration create_role_user_table
php artisan make:migration create_permission_role_table
```

### Setup the pivot table migrations:

```php
// role_user pivot
Schema::create('role_user', function (Blueprint $table) {
    $table->foreignId('role_id')->constrained()->onDelete('cascade');
    $table->foreignId('user_id')->constrained()->onDelete('cascade');
    $table->primary(['role_id', 'user_id']);
});

// permission_role pivot
Schema::create('permission_role', function (Blueprint $table) {
    $table->foreignId('permission_id')->constrained()->onDelete('cascade');
    $table->foreignId('role_id')->constrained()->onDelete('cascade');
    $table->primary(['permission_id', 'role_id']);
});
```

## 4. Define Model Relationships

### Update your User model:

```php
class User extends Authenticatable
{
// ...existing code...

    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }

    public function hasRole($role)
    {
        return $this->roles->contains('slug', $role);
    }

}
```

### Update the Role model:

```php
class Role extends Model
{
protected $fillable = ['name', 'slug'];

    public function permissions()
    {
        return $this->belongsToMany(Permission::class);
    }

    public function users()
    {
        return $this->belongsToMany(User::class);
    }

}
```

### Update the Permission model:

```php
class Permission extends Model
{
protected $fillable = ['name', 'slug'];

    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }

}
```

## 5. Create an Authorization Gate Provider

### Modify the `AuthServiceProvider.php`:

```php
namespace App\Providers;

use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;
use App\Models\Permission;

class AuthServiceProvider extends ServiceProvider
{
// ...existing code...

    public function boot()
    {
        $this->registerPolicies();

        // Register permissions as Gates
        if (!$this->app->runningInConsole()) {
            foreach (Permission::all() as $permission) {
                Gate::define($permission->slug, function ($user) use ($permission) {
                    foreach ($user->roles as $role) {
                        if ($role->permissions->contains('slug', $permission->slug)) {
                            return true;
                        }
                    }
                    return false;
                });
            }
        }
    }

}
```

## 6. Seed Initial Roles and Permissions

### Create a seeder:

```bash
php artisan make:seeder RolesAndPermissionsSeeder
```

### Implement the seeder:

```php
use App\Models\Permission;
use App\Models\Role;
use App\Models\User;
use Illuminate\Database\Seeder;

class RolesAndPermissionsSeeder extends Seeder
{
    public function run()
    {
        // Create permissions
        $permissions = [
            ['name' => 'Create Post', 'slug' => 'create-post'],
            ['name' => 'Edit Post', 'slug' => 'edit-post'],
            ['name' => 'Delete Post', 'slug' => 'delete-post'],
        ];

        foreach ($permissions as $permissionData) {
            Permission::create($permissionData);
        }

        // Create roles
        $adminRole = Role::create([
            'name' => 'Administrator',
            'slug' => 'admin',
        ]);

        $editorRole = Role::create([
            'name' => 'Editor',
            'slug' => 'editor',
        ]);

        // Assign permissions to roles
        $adminRole->permissions()->attach(Permission::all());
        $editorRole->permissions()->attach(Permission::where('slug', 'create-post')->orWhere('slug', 'edit-post')->get());

        // Create admin user and assign admin role
        $admin = User::where('email', 'admin@example.com')->first();
        if (!$admin) {
            $admin = User::factory()->create([
                'name' => 'Admin User',
                'email' => 'admin@example.com',
            ]);
        }
        $admin->roles()->attach($adminRole);
    }
}
```

### Run the seeder:

```bash
php artisan db:seed --class=RolesAndPermissionsSeeder
```

## 7. Using Auth::can() in Controllers and Views

### In a controller:

```php
public function edit(Post $post)
{
    if (Auth::can('edit-post')) {
        return view('posts.edit', compact('post'));
    }

    return redirect()->route('posts.index')
        ->with('error', 'You do not have permission to edit posts.');

}
```

### In a Blade view:

```php
@if(Auth::can('create-post'))
    <a href="{{ route('posts.create') }}" class="btn btn-primary">Create New Post</a>
@endif
```

## 8. Protecting Routes with Middleware

You can use the built-in can middleware to protect routes:

```php
// In routes/web.php
Route::get('/posts/create', [PostController::class, 'create'])
    ->middleware('can:create-post');

Route::resource('posts', PostController::class)
    ->except(['index', 'show'])
    ->middleware('can:edit-post');
```

## 9. Creating a Custom Middleware (Optional)

For more complex permission checks:

```bash
php artisan make:middleware CheckPermission
```

Implement the middleware:

```php
public function handle($request, Closure $next, $permission)
{
    if (!Auth::can($permission)) {
return redirect()->route('home')
->with('error', 'You do not have permission to access this page.');
}

    return $next($request);

}
```

Register it in `app/Http/Kernel.php`:

```php
protected $routeMiddleware = [
    // ...existing middleware
    'permission' => \App\Http\Middleware\CheckPermission::class,
];
```

Use it in routes:

```php
Route::get('/admin/settings', [AdminController::class, 'settings'])
    ->middleware('permission:manage-settings');
```

Testing Your Implementation
Create a simple test to verify permissions are working:

```php
$user = User::factory()->create();
$adminRole = Role::where('slug', 'admin')->first();
$user->roles()->attach($adminRole);

$this->actingAs($user);
$this->assertTrue(Auth::can('create-post'));
```
