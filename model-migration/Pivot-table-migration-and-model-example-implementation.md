# Pivot table migration and model example implementation between users, role and permission table

In a Laravel application, when you're working with many-to-many relationships, a pivot table is used to represent the connection between two models. In this case, we have a `User`, `Role`, and `Permission` model, where `users` can have multiple `roles`, `roles` can have multiple `permissions`, and `users` can have multiple `permissions` through the `roles`.

Let's implement this relationship step by step.

### 1. Migrations

We will create 3 tables:
1. `users` (already exists, if using Laravel's default authentication system).
2. `roles`
3. `permissions`
4. Pivot table `role_user` for the relationship between `users` and `roles`.
5. Pivot table `permission_role` for the relationship between `roles` and `permissions`.

Here is how you would set up the migrations:

#### `roles` Table Migration:

```php
// database/migrations/xxxx_xx_xx_create_roles_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateRolesTable extends Migration
{
    public function up()
    {
        Schema::create('roles', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('roles');
    }
}
```

#### `permissions` Table Migration:

```php
// database/migrations/xxxx_xx_xx_create_permissions_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePermissionsTable extends Migration
{
    public function up()
    {
        Schema::create('permissions', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('permissions');
    }
}
```

#### `role_user` Pivot Table Migration:

```php
// database/migrations/xxxx_xx_xx_create_role_user_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateRoleUserTable extends Migration
{
    public function up()
    {
        Schema::create('role_user', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->foreignId('role_id')->constrained()->onDelete('cascade');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('role_user');
    }
}
```

#### `permission_role` Pivot Table Migration:

```php
// database/migrations/xxxx_xx_xx_create_permission_role_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePermissionRoleTable extends Migration
{
    public function up()
    {
        Schema::create('permission_role', function (Blueprint $table) {
            $table->id();
            $table->foreignId('role_id')->constrained()->onDelete('cascade');
            $table->foreignId('permission_id')->constrained()->onDelete('cascade');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('permission_role');
    }
}
```

### 2. Models

Now, let’s define the relationships in the corresponding models.

#### User Model:

```php
// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    use HasFactory;

    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }

    public function permissions()
    {
        return $this->belongsToMany(Permission::class, 'permission_role', 'role_id', 'permission_id')
                    ->through(Role::class);
    }
}
```

#### Role Model:

```php
// app/Models/Role.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    use HasFactory;

    public function users()
    {
        return $this->belongsToMany(User::class);
    }

    public function permissions()
    {
        return $this->belongsToMany(Permission::class);
    }
}
```

#### Permission Model:

```php
// app/Models/Permission.php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Permission extends Model
{
    use HasFactory;

    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
}
```

### 3. Seeding Data

You may want to seed some roles and permissions for testing purposes. Here’s an example of how to do that:

#### Role and Permission Seeder:

```php
// database/seeders/RolePermissionSeeder.php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Role;
use App\Models\Permission;

class RolePermissionSeeder extends Seeder
{
    public function run()
    {
        $adminRole = Role::create(['name' => 'admin']);
        $editorRole = Role::create(['name' => 'editor']);
        
        $viewPermission = Permission::create(['name' => 'view']);
        $editPermission = Permission::create(['name' => 'edit']);
        
        // Assign permissions to roles
        $adminRole->permissions()->attach([$viewPermission->id, $editPermission->id]);
        $editorRole->permissions()->attach([$viewPermission->id]);
    }
}
```

Run the seeder:

```bash
php artisan db:seed --class=RolePermissionSeeder
```

### 4. Using the Relationships

Here’s an example of how to use the relationships in your code:

#### Assigning roles and permissions to a user:

```php
$user = User::find(1);
$adminRole = Role::where('name', 'admin')->first();
$user->roles()->attach($adminRole);

$editPermission = Permission::where('name', 'edit')->first();
$adminRole->permissions()->attach($editPermission);
```

#### Checking if a user has a specific permission:

```php
$user = User::find(1);

if ($user->hasPermissionTo('edit')) {
    // Do something
}
```

This example should set up the basic relationships between the `User`, `Role`, and `Permission` models, as well as define the necessary pivot tables. You can then expand on this structure by adding more methods for more specific use cases, such as checking permissions on a per-user basis.