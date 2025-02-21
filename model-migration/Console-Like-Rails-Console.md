# Console Like Rails Console

Laravel offers an interactive console similar to Rails console called `Tinker`. It provides a REPL (Read-Eval-Print Loop) environment where you can interact with your Laravel application.

To use Laravel's Tinker console:

```bash
php artisan tinker
```

Once in Tinker, you can:

- Create, read, update, and delete database records
- Test model relationships
- Execute any PHP code within your application's context
- Access all your application's classes and methods

Example usage:

```php
// Create a new user
$user = new App\Models\User;
$user->name = 'John';
$user->email = 'john@example.com';
$user->password = bcrypt('password');
$user->save();

// Find a user
$user = App\Models\User::find(1);

// Query with conditions
$activeUsers = App\Models\User::where('active', true)->get();
```

If you're coming from Rails and want better tab completion and a more powerful console experience, you might want to check out Laravel Tinkerwell, which is a desktop application that provides an enhanced Tinker experience.
