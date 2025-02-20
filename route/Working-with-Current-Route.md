# Working with Current Route

Here's a simple tutorial on working with current route information in Laravel:

## Laravel Current Route Examples

### Getting Current Route Information

```php
// Get current route name
$routeName = Route::currentRouteName();

// Get the current route action
$routeAction = Route::currentRouteAction();

// Get the current route object
$route = Route::current();

// Check if current route matches a pattern
if (Route::is('admin.*')) {
    // This matches any route name that begins with "admin."
}
```

### Accessing Route Parameters

```php
// Get all route parameters
$parameters = Route::current()->parameters();

// Get a specific route parameter
$userId = Route::current()->parameter('user');

// Alternative using the request
$userId = request()->route('user');
```

### Checking Current Route in Blade Templates

```php
{{-- Check if current route matches exactly --}}
@if(Route::currentRouteName() == 'users.show')
    <p>Currently viewing a user profile</p>
@endif

{{-- Check if route name starts with a pattern --}}
@if(Request::routeIs('admin.*'))
    <p>Currently in admin section</p>
@endif
```

### Practical Example

Let's build a simple navigation with active link highlighting:

```php
// routes/web.php
Route::get('/', function () {
    return view('home');
})->name('home');

Route::get('/about', function () {
    return view('about');
})->name('about');

Route::get('/services', function () {
    return view('services');
})->name('services');
```

```php
{{-- navigation.blade.php --}}
<nav>
    <ul>
        <li class="{{ Route::currentRouteName() == 'home' ? 'active' : '' }}">
            <a href="{{ route('home') }}">Home</a>
        </li>
        <li class="{{ Route::currentRouteName() == 'about' ? 'active' : '' }}">
            <a href="{{ route('about') }}">About</a>
        </li>
        <li class="{{ Route::currentRouteName() == 'services' ? 'active' : '' }}">
            <a href="{{ route('services') }}">Services</a>
        </li>
    </ul>
</nav>
```
