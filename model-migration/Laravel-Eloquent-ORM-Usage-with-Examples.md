# Laravel Eloquent ORM Usage with Examples

Eloquent is Laravel's implementation of the Active Record pattern for working with databases. It's an elegant and intuitive ORM (Object-Relational Mapper) that makes database interactions simpler and more expressive. Here's a comprehensive guide to using Eloquent:

## Basic Model Definition

First, let's create a basic model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    // By default, Eloquent will use the plural, snake_case version of the class name as the table name
    // You can override this by setting the $table property
    protected $table = 'users';

    // By default, Eloquent expects 'id' as the primary key
    // You can override this by setting the $primaryKey property
    protected $primaryKey = 'id';

    // Attributes that are mass assignable
    protected $fillable = ['name', 'email', 'password'];

    // Attributes that should be hidden when converting to array/JSON
    protected $hidden = ['password', 'remember_token'];
}
```

## Basic CRUD Operations

### Create (Insert)

```php
// Method 1: Create and save an instance
$user = new User;
$user->name = 'John Doe';
$user->email = 'john@example.com';
$user->password = bcrypt('password');
$user->save();

// Method 2: Mass assignment using create() method
$user = User::create([
    'name' => 'Jane Doe',
    'email' => 'jane@example.com',
    'password' => bcrypt('password')
]);
```

### Read (Select)

```php
// Retrieve all users
$users = User::all();

// Find a user by primary key
$user = User::find(1);

// Find with exception if not found
$user = User::findOrFail(1);

// Basic where clauses
$users = User::where('active', 1)->get();
$users = User::where('age', '>', 18)->get();

// Multiple where clauses
$users = User::where('active', 1)
             ->where('age', '>', 18)
             ->get();

// Order by
$users = User::orderBy('name', 'asc')->get();

// Limit results
$users = User::limit(10)->get();

// Pagination
$users = User::paginate(15);
```

### Update

```php
// Method 1: Find and update
$user = User::find(1);
$user->name = 'New Name';
$user->save();

// Method 2: Mass update
User::where('active', 0)
    ->update(['status' => 'inactive']);
```

### Delete

```php
// Method 1: Find and delete
$user = User::find(1);
$user->delete();

// Method 2: Delete by ID
User::destroy(1);
User::destroy([1, 2, 3]); // Delete multiple IDs

// Method 3: Delete with where clause
User::where('active', 0)->delete();
```

## Relationships

### One to One

```php
// User.php
public function profile()
{
    return $this->hasOne(Profile::class);
}

// Profile.php
public function user()
{
    return $this->belongsTo(User::class);
}

// Usage
$profile = User::find(1)->profile;
$user = Profile::find(1)->user;
```

### One to Many

```php
// User.php
public function posts()
{
    return $this->hasMany(Post::class);
}

// Post.php
public function user()
{
    return $this->belongsTo(User::class);
}

// Usage
$posts = User::find(1)->posts;
$user = Post::find(1)->user;
```

### Many to Many

```php
// User.php
public function roles()
{
    return $this->belongsToMany(Role::class);
}

// Role.php
public function users()
{
    return $this->belongsToMany(User::class);
}

// Usage
$roles = User::find(1)->roles;
$users = Role::find(1)->users;

// Attach, detach, sync
$user->roles()->attach(1);        // Attach role with ID 1
$user->roles()->detach(1);        // Detach role with ID 1
$user->roles()->sync([1, 2, 3]);  // Sync with role IDs 1, 2, 3
```

## Query Scopes

```php
// User.php
public function scopeActive($query)
{
    return $query->where('active', 1);
}

public function scopePopular($query)
{
    return $query->where('followers', '>', 1000);
}

// Usage
$activeUsers = User::active()->get();
$popularActiveUsers = User::active()->popular()->get();
```

## Accessors and Mutators

```php
// User.php
// Accessor (get)
public function getFullNameAttribute()
{
    return "{$this->first_name} {$this->last_name}";
}

// Mutator (set)
public function setPasswordAttribute($value)
{
    $this->attributes['password'] = bcrypt($value);
}

// Usage
$fullName = $user->full_name;        // Uses the accessor
$user->password = 'plain-password';  // Uses the mutator
```

## Eager Loading

```php
// Without eager loading (N+1 problem)
$users = User::all();
foreach ($users as $user) {
    echo $user->profile->bio;  // Each iteration makes a separate query
}

// With eager loading
$users = User::with('profile')->get();
foreach ($users as $user) {
    echo $user->profile->bio;  // No additional queries
}

// Multiple relationships
$users = User::with(['profile', 'posts'])->get();

// Nested relationships
$users = User::with('posts.comments')->get();
```

## Advanced Querying

```php
// Aggregates
$count = User::count();
$max = User::max('age');
$avg = User::avg('age');

// Chunks (process large datasets)
User::chunk(100, function ($users) {
    foreach ($users as $user) {
        // Process each user
    }
});

// Subqueries
$users = User::where('votes', '>', function ($query) {
    $query->selectRaw('AVG(votes) from users');
})->get();

// Joins
$users = User::join('contacts', 'users.id', '=', 'contacts.user_id')
             ->select('users.*', 'contacts.phone')
             ->get();
```

## Events

Eloquent models fire several events, which you can hook into:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    protected static function boot()
    {
        parent::boot();

        static::creating(function ($user) {
            // Before a new user is created
        });

        static::created(function ($user) {
            // After a new user is created
        });

        static::updating(function ($user) {
            // Before an existing user is updated
        });

        static::updated(function ($user) {
            // After an existing user is updated
        });

        static::deleting(function ($user) {
            // Before an existing user is deleted
        });

        static::deleted(function ($user) {
            // After an existing user is deleted
        });
    }
}
```

These examples cover most of the common operations you'll perform with Laravel's Eloquent ORM. The beauty of Eloquent is its fluent, expressive syntax that makes database interactions feel natural and intuitive. As you become more familiar with it, you'll appreciate its power and flexibility in handling complex database operations.
