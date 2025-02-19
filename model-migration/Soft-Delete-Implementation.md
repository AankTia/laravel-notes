# Soft Delete Implementation

## 1. Migration

In migration adds a `deleted_at` column to your table using `$table->softDeletes()`

### Example

```php
// database/migrations/xxxx_xx_xx_create_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateUsersTable extends Migration
{
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamps();
            $table->softDeletes(); // Adds deleted_at column
        });
    }

    public function down()
    {
        Schema::dropIfExists('users');
    }
}
```

## 2. Model

In your model:

- Use the `SoftDeletes` trait
- Laravel will automatically handle the `deleted_at` timestamp

### Exampl

```php

// app/Models/User.php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class User extends Model
{
    use SoftDeletes;

    protected $fillable = ['name', 'email'];
}
```

## 3. Key Operations

- `$user->delete()` - Soft deletes by setting deleted_at
- `$user->forceDelete()` - Permanently deletes the record
- `$user->restore()` - Removes the deleted_at timestamp

### Example usage in a controller

```php

// app/Http/Controllers/UserController.php

namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    public function destroy(User $user)
    {
        $user->delete(); // Soft deletes the user
        return response()->json(['message' => 'User deleted successfully']);
    }

    public function forceDelete(User $user)
    {
        $user->forceDelete(); // Permanently deletes the user
        return response()->json(['message' => 'User permanently deleted']);
    }

    public function restore($id)
    {
        User::withTrashed()->find($id)->restore();
        return response()->json(['message' => 'User restored successfully']);
    }

    public function index()
    {
        // Different ways to query soft deleted records
        $activeUsers = User::all(); // Only active users
        $deletedUsers = User::onlyTrashed()->get(); // Only deleted users
        $allUsers = User::withTrashed()->get(); // Both active and deleted users

        return response()->json([
            'active_users' => $activeUsers,
            'deleted_users' => $deletedUsers,
            'all_users' => $allUsers
        ]);
    }
}
```

## 4. Querying options:

- `User::all()` - Gets only non-deleted records
- `User::withTrashed()` - Includes soft deleted records
- `User::onlyTrashed()` - Gets only soft deleted records
