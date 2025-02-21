# Caching CSS and JS assets in Laravel.

## 1. Using Laravel Mix with Versioning

Laravel Mix provides an easy way to version your assets:

```javascript
// webpack.mix.js
mix
  .js("resources/js/app.js", "public/js")
  .sass("resources/sass/app.scss", "public/css")
  .version();
```

Then in your Blade template:

```php
<link rel="stylesheet" href="{{ mix('css/app.css') }}">
<script src="{{ mix('js/app.js') }}"></script>
```

This adds a unique hash to filenames, forcing browsers to download new versions when the content changes.

## 2. Using Cache Busting with Query Strings

```php
<link rel="stylesheet" href="{{ asset('css/app.css') }}?v={{ filemtime(public_path('css/app.css')) }}">
<script src="{{ asset('js/app.js') }}?v={{ filemtime(public_path('js/app.js')) }}"></script>
```

## 3. Setting HTTP Cache Headers

In your` AppServiceProvider.php`:

```php
public function boot()
{
    // Set cache headers for assets
    Route::middleware('web')->group(function () {
        Route::get('/css/{file}', function ($file) {
            $path = public_path('css/' . $file);
            if (!file_exists($path)) {
                abort(404);
            }
            $response = response()->file($path);
            $response->header('Cache-Control', 'public, max-age=31536000');
            return $response;
        });

        Route::get('/js/{file}', function ($file) {
            $path = public_path('js/' . $file);
            if (!file_exists($path)) {
                abort(404);
            }
            $response = response()->file($path);
            $response->header('Cache-Control', 'public, max-age=31536000');
            return $response;
        });
    });
}
```

## 4. Using Laravel's Built-in Cache System

Wrap asset loading in a cache function:

```php
$cssTimestamp = Cache::remember('css-timestamp', 60*24, function () {
    return filemtime(public_path('css/app.css'));
});

<link rel="stylesheet" href="{{ asset('css/app.css') }}?v={{ $cssTimestamp }}">
```

## 5. Server Configuration (production)

For production, configure your web server (Nginx/Apache) with proper caching rules:
Nginx example:
```
location ~* \.(css|js)$ {
    expires 1y;
    add_header Cache-Control "public, max-age=31536000";
}
```
