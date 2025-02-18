# Laravel Model Find by Attribute

In Laravel Eloquent, finding models by specific attributes (other than the primary key) can be accomplished in several ways. Here are the most common approaches:

## Basic Query Methods

### Using where() with first()

The most common approach is to use where() followed by first():

```php
// Find the first user with email 'john@example.com'
$user = User::where('email', 'john@example.com')->first();

// If no matching record is found, $user will be null
if ($user) {
    // User was found
} else {
    // User was not found
}
```

### Using firstWhere()

Laravel provides a shorthand method firstWhere() for this common operation:

```php
// Same as where('email', 'john@example.com')->first()
$user = User::firstWhere('email', 'john@example.com');
```

## Methods With Exception Handling

### Using firstOrFail()

If you want to throw an exception when no model is found:

```php
try {
    // Throws Illuminate\Database\Eloquent\ModelNotFoundException if no user is found
    $user = User::where('email', 'john@example.com')->firstOrFail();

    // Process the user...
} catch (\Illuminate\Database\Eloquent\ModelNotFoundException $e) {
    // Handle not found scenario
    return redirect()->back()->with('error', 'User not found');
}
```

### Using firstWhereOrFail()

Combines firstWhere() and firstOrFail():

```php
$user = User::firstWhereOrFail('email', 'john@example.com');
```

## With Default Values

### Using firstOr()

Returns a default value when no model is found:

```php
// If no user is found, execute the closure
$user = User::where('email', 'john@example.com')->firstOr(function() {
    // Return a default user or perform other logic
    return new User(['name' => 'Guest User']);
});
```

### Using firstOrNew()

Returns a new model instance with the given attributes if no matching model exists in the database:

```php
// If no match is found, returns a new User instance with the attributes
// Note: this doesn't save to the database automatically
$user = User::firstOrNew([
    'email' => 'john@example.com'
]);

// Optional: you can provide default values for a new instance
$user = User::firstOrNew(
    ['email' => 'john@example.com'], // attributes to search by
    ['name' => 'John Doe', 'active' => true] // default values if new
);
```

### Using firstOrCreate()

Similar to firstOrNew() but automatically saves the new model to the database if no match is found:

```php
// If no matching user is found, creates and saves a new one
$user = User::firstOrCreate([
    'email' => 'john@example.com'
]);

// With default values for new records
$user = User::firstOrCreate(
    ['email' => 'john@example.com'], // attributes to search by
    ['name' => 'John Doe', 'password' => bcrypt('password')] // values for new record
);
```

### Using Multiple Attributes

All of these methods support searching by multiple attributes:

```php
// Find by multiple attributes
$user = User::where([
    'email' => 'john@example.com',
    'active' => true
])->first();

// Or more explicitly
$user = User::where('email', 'john@example.com')
            ->where('active', true)
            ->first();

// Using firstWhere with multiple attributes
$user = User::firstWhere([
    'email' => 'john@example.com',
    'active' => true
]);
```

### Custom Scopes

For frequently used queries, you can define custom scopes:

```php
// In your User model
public function scopeWhereEmail($query, $email)
{
    return $query->where('email', $email);
}

// Usage
$user = User::whereEmail('john@example.com')->first();
```

## Advanced Finding

### Using Advanced Where Clauses

```php
// Find users with emails that match a pattern
$users = User::where('email', 'like', '%@example.com')->get();

// Find users with null values
$inactiveUsers = User::whereNull('activated_at')->get();

// Finding using SQL expressions
$recentUsers = User::whereRaw('created_at > DATE_SUB(NOW(), INTERVAL 7 DAY)')->get();
```

### With Custom SQL

For complex scenarios, you can use direct SQL:

```php
$users = User::fromQuery("SELECT * FROM users WHERE email = ? AND active = ?", [
    'john@example.com',
    true
]);
```

These methods give you flexibility for finding models by various attributes in different scenarios, whether you need exception handling, default values, or advanced SQL conditions.
