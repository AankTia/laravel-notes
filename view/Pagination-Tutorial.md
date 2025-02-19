# Laravel Pagination Tutorial

This tutorial will walk you through implementing pagination in Laravel applications, from basic setups to advanced customization.

## Basic Pagination

### Step 1: Set Up a Controller

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index()
    {
        // Paginate users with 10 items per page
        $users = User::paginate(10);

        return view('users.index', compact('users'));
    }
}
```

### Step 2: Create the Route

```php
// routes/web.php
Route::get('/users', [UserController::class, 'index'])->name('users.index');
```

### Step 3: Create the View

```php
<!-- resources/views/users/index.blade.php -->
<div class="container">
    <h1>Users List</h1>

    <table class="table">
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>Created At</th>
            </tr>
        </thead>
        <tbody>
            @foreach($users as $user)
                <tr>
                    <td>{{ $user->id }}</td>
                    <td>{{ $user->name }}</td>
                    <td>{{ $user->email }}</td>
                    <td>{{ $user->created_at->format('Y-m-d') }}</td>
                </tr>
            @endforeach
        </tbody>
    </table>

    <!-- Pagination Links -->
    {{ $users->links() }}
</div>
```

## Pagination with Query Parameters

If you need to maintain query parameters in your pagination links:

```php
// Controller
public function search(Request $request)
{
    $query = $request->input('query');

    $users = User::where('name', 'LIKE', "%{$query}%")
                 ->orWhere('email', 'LIKE', "%{$query}%")
                 ->paginate(10);

    // Appends the query parameters to pagination links
    $users->appends(['query' => $query]);

    return view('users.search', compact('users', 'query'));
}
```

```php
<!-- Template with search form -->
<form action="{{ route('users.search') }}" method="GET">
    <input type="text" name="query" value="{{ $query ?? '' }}">
    <button type="submit">Search</button>
</form>

<!-- Display results and pagination -->
<table><!-- Table structure same as before --></table>

{{ $users->links() }}
```

## API Pagination

For JSON responses in an API:

```php
public function index()
{
    $users = User::paginate(15);

    return response()->json([
        'data' => $users->items(),
        'pagination' => [
            'total' => $users->total(),
            'per_page' => $users->perPage(),
            'current_page' => $users->currentPage(),
            'last_page' => $users->lastPage(),
            'from' => $users->firstItem(),
            'to' => $users->lastItem(),
            'next_page_url' => $users->nextPageUrl(),
            'prev_page_url' => $users->previousPageUrl(),
        ]
    ]);
}
```

## Customizing the Pagination View

### Step 1: Publish the Pagination Views

```bash
php artisan vendor:publish --tag=laravel-pagination
```

This will publish the default pagination views to `resources/views/vendor/pagination`.

### Step 2: Modify the Templates

Edit the files in `resources/views/vendor/pagination` to match your design requirements.

### Step 3: Specify Which View to Use (Optional)

```php
{{ $users->links('pagination::bootstrap-4') }}
```

## Simple Pagination (Lighter Query)

If you only need "Previous" and "Next" links:

```php
$users = User::simplePaginate(15);
```

## Using with Eloquent Collections

```php
// For a custom collection of items
$usersArray = [/* ... array of user objects ... */];
$collection = collect($usersArray);

$perPage = 10;
$page = request()->input('page', 1);
$paginator = new \Illuminate\Pagination\LengthAwarePaginator(
    $collection->forPage($page, $perPage),
    $collection->count(),
    $perPage,
    $page,
    ['path' => request()->url(), 'query' => request()->query()]
);

return view('users.index', ['users' => $paginator]);
```

## Complete Working Example

Here's a complete example with filtering, sorting, and pagination:

### Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'description', 'price', 'category_id'];

    public function category()
    {
        return $this->belongsTo(Category::class);
    }
}
```

### Controller

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use App\Models\Category;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function index(Request $request)
    {
        $query = Product::query();

        // Apply filters
        if ($request->has('category')) {
            $query->where('category_id', $request->category);
        }

        if ($request->has('min_price')) {
            $query->where('price', '>=', $request->min_price);
        }

        if ($request->has('max_price')) {
            $query->where('price', '<=', $request->max_price);
        }

        if ($request->has('search')) {
            $search = $request->search;
            $query->where(function($q) use ($search) {
                $q->where('name', 'LIKE', "%{$search}%")
                  ->orWhere('description', 'LIKE', "%{$search}%");
            });
        }

        // Apply sorting
        $sortField = $request->input('sort', 'name');
        $sortDirection = $request->input('direction', 'asc');
        $allowedSortFields = ['name', 'price', 'created_at'];

        if (in_array($sortField, $allowedSortFields)) {
            $query->orderBy($sortField, $sortDirection === 'desc' ? 'desc' : 'asc');
        }

        // Paginate results
        $products = $query->with('category')->paginate(12);

        // Append all query parameters to pagination links
        $products->appends($request->all());

        // Get categories for filter dropdown
        $categories = Category::all();

        return view('products.index', compact('products', 'categories'));
    }
}
```

### Route

```php
// routes/web.php
Route::get('/products', [ProductController::class, 'index'])->name('products.index');
```

### View

```php
<!-- resources/views/products/index.blade.php -->
@extends('layouts.app')

@section('content')
<div class="container">
    <h1>Products</h1>

    <!-- Filters -->
    <form action="{{ route('products.index') }}" method="GET" class="mb-4">
        <div class="row">
            <div class="col-md-3">
                <select name="category" class="form-control">
                    <option value="">All Categories</option>
                    @foreach($categories as $category)
                        <option value="{{ $category->id }}" {{ request('category') == $category->id ? 'selected' : '' }}>
                            {{ $category->name }}
                        </option>
                    @endforeach
                </select>
            </div>

            <div class="col-md-2">
                <input type="number" name="min_price" placeholder="Min Price" class="form-control"
                       value="{{ request('min_price') }}">
            </div>

            <div class="col-md-2">
                <input type="number" name="max_price" placeholder="Max Price" class="form-control"
                       value="{{ request('max_price') }}">
            </div>

            <div class="col-md-3">
                <input type="text" name="search" placeholder="Search products..." class="form-control"
                       value="{{ request('search') }}">
            </div>

            <div class="col-md-2">
                <button type="submit" class="btn btn-primary">Filter</button>
                <a href="{{ route('products.index') }}" class="btn btn-secondary">Reset</a>
            </div>
        </div>

        <!-- Sort options -->
        <div class="row mt-3">
            <div class="col-md-6">
                <select name="sort" class="form-control d-inline-block w-auto mr-2">
                    <option value="name" {{ request('sort') == 'name' ? 'selected' : '' }}>Sort by Name</option>
                    <option value="price" {{ request('sort') == 'price' ? 'selected' : '' }}>Sort by Price</option>
                    <option value="created_at" {{ request('sort') == 'created_at' ? 'selected' : '' }}>Sort by Newest</option>
                </select>

                <select name="direction" class="form-control d-inline-block w-auto">
                    <option value="asc" {{ request('direction') != 'desc' ? 'selected' : '' }}>Ascending</option>
                    <option value="desc" {{ request('direction') == 'desc' ? 'selected' : '' }}>Descending</option>
                </select>

                <button type="submit" class="btn btn-sm btn-primary">Apply Sort</button>
            </div>
        </div>
    </form>

    <!-- Products Grid -->
    <div class="row">
        @forelse($products as $product)
            <div class="col-md-4 mb-4">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">{{ $product->name }}</h5>
                        <h6 class="card-subtitle mb-2 text-muted">{{ $product->category->name }}</h6>
                        <p class="card-text">{{ Str::limit($product->description, 100) }}</p>
                        <div class="d-flex justify-content-between align-items-center">
                            <span class="font-weight-bold">${{ number_format($product->price, 2) }}</span>
                            <a href="{{ route('products.show', $product) }}" class="btn btn-sm btn-info">View Details</a>
                        </div>
                    </div>
                </div>
            </div>
        @empty
            <div class="col-12">
                <div class="alert alert-info">
                    No products found matching your criteria.
                </div>
            </div>
        @endforelse
    </div>

    <!-- Pagination Links with Bootstrap 5 Styling -->
    <div class="d-flex justify-content-center">
        {{ $products->links() }}
    </div>

    <!-- Pagination Information -->
    <div class="text-center text-muted mt-2">
        Showing {{ $products->firstItem() ?? 0 }} to {{ $products->lastItem() ?? 0 }} of {{ $products->total() }} products
    </div>
</div>
@endsection
```

## Customize the "Previous" and "Next" link titles

### Method 1: Using Translation Files

Edit your resources/lang/en/pagination.php file (or create it if it doesn't exist):

```php
// resources/lang/en/pagination.php
return [
    'previous' => '« Back',
    'next' => 'Forward »',
];
```

### Method 2: Creating Custom Pagination View

1.  First, publish the pagination views:
    `php artisan vendor:publish --tag=laravel-pagination`
2.  Edit the simple pagination view in `resources/views/vendor/pagination/simple-bootstrap-4.blade.php` or similar:

    ```php
    <ul class="pagination">
    {{-- Previous Page Link --}}
    @if ($paginator->onFirstPage())
    <li class="page-item disabled" aria-disabled="true">
    <span class="page-link">← Previous Page</span>
    </li>
    @else
    <li class="page-item">
    <a class="page-link" href="{{ $paginator->previousPageUrl() }}" rel="prev">← Previous Page</a>
    </li>
    @endif

        {{-- Next Page Link --}}
        @if ($paginator->hasMorePages())
            <li class="page-item">
                <a class="page-link" href="{{ $paginator->nextPageUrl() }}" rel="next">Next Page →</a>
            </li>
        @else
            <li class="page-item disabled" aria-disabled="true">
                <span class="page-link">Next Page →</span>
            </li>
        @endif

    </ul>
    ```

3.  Use your custom view:
    ```php
    {{ $posts->links('vendor.pagination.simple-bootstrap-4') }}
    ```

### Method 3: Inline Custom Pagination Template

```php
{!! $posts->links('pagination::simple-bootstrap-4', [
    'paginator' => $posts,
    'elements' => [
        'previous' => '← Go Back',
        'next' => 'Continue →',
    ]
]) !!}
```

> Note that the third method may need a custom view that accepts the elements parameter, which isn't standard in Laravel's default views.

## Tips and Best Practices

1. **Performance Considerations**:

   - Use `simplePaginate()` instead of `paginate()` for large datasets when you don't need to show the total count
   - Add proper indexes to columns used in sorting and filtering

2. **Security**:

   - Validate and whitelist sortable fields to prevent SQL injection
   - Use query scopes in your models to keep controllers clean

3. **UX Improvements**:
   - Show loading indicators during AJAX pagination
   - Preserve filters and search parameters across pagination
   - Display summary info (e.g., "Showing 1-10 of 100 results")
