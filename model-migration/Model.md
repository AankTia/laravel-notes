# Model

## Laravel Model Events and Hooks

### 1. Using Model Events

Laravel's models dispatch events before and after various actions. You can listen for these events in the model's boot() method:

```php
protected static function boot()
{
    parent::boot();

    // Before creating
    static::creating(function ($model) {
        // This runs before a model is saved for the first time
    });

    // Before updating
    static::updating(function ($model) {
        // This runs before updates to an existing model
    });

    // Before saving (runs for both creating and updating)
    static::saving(function ($model) {
        // This runs before any save operation
    });

    // Before deleting
    static::deleting(function ($model) {
        // This runs before a model is deleted
    });
}
```

Other available model events include:

- `created`/`updated`/`saved`/`deleted` (after the action completes)
- `retrieved` (after fetching from database)
- `forceDeleted` (for soft deletes)
- `restored` (when restoring soft-deleted models)

### 2. Using Observers

For cleaner organization, Laravel also offers Model Observers which can group all event handlers for a model:

```
// Generate an observer
php artisan make:observer UserObserver --model=User
```

Then in the observer:

```php
class UserObserver
{
    public function creating(User $user)
    {
        // Before creating
    }

    public function updating(User $user)
    {
        // Before updating
    }

    public function saving(User $user)
    {
        // Before any save
    }

    public function deleting(User $user)
    {
        // Before deleting
    }

    // You can also define after-action hooks
    public function created(User $user)
    {
        // After creation
    }
}
```

Register the observer in a service provider, typically `AppServiceProvider`:

```php
use App\Models\User;
use App\Observers\UserObserver;

public function boot()
{
    User::observe(UserObserver::class);
}
```

### 3. Using Mutators (for Attribute Changes)

For transforming attributes before saving:

```php
// In your model
public function setNameAttribute($value)
{
    $this->attributes['name'] = ucfirst($value);
}
```

### 4. Using Global Scopes

For query-based modifications on all operations:

```php
protected static function boot()
{
    parent::boot();

    static::addGlobalScope('active', function (Builder $builder) {
        $builder->where('active', true);
    });
}
```

### Common Use Cases

#### 1. Generating UUIDs or slugs:

```php
protected static function boot()
{
    parent::boot();
    static::creating(function ($model) {
        $model->uuid = (string) Str::uuid();
    });
}
```

#### 2. Setting default values:

```php
protected static function boot()
{
    parent::boot();
    static::creating(function ($model) {
        $model->status = $model->status ?? 'pending';
    });
}
```

#### 3. Validation:

```php
protected static function boot()
{
    parent::boot();
    static::saving(function ($model) {
        if (empty($model->email)) {
            throw new \Exception('Email cannot be empty');
        }
    });
}
```

## Laravel Model Before Action - Example Implementation

Here's a practical example of implementing "before action" behavior in Laravel models, showing different approaches for common use cases:

### 1. Using Model Events in the Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Post extends Model
{
    protected $fillable = ['title', 'body', 'slug', 'user_id', 'status'];

    protected static function boot()
    {
        parent::boot();

        // Before creating a post
        static::creating(function ($post) {
            // Generate a slug from the title
            $post->slug = Str::slug($post->title);

            // Set the user_id if not already set
            if (!$post->user_id && auth()->check()) {
                $post->user_id = auth()->id();
            }

            // Default status is draft
            $post->status = $post->status ?? 'draft';

            // Log the creation attempt
            \Log::info('Attempting to create post: ' . $post->title);
        });

        // Before updating a post
        static::updating(function ($post) {
            // If title changed, update the slug
            if ($post->isDirty('title')) {
                $post->slug = Str::slug($post->title);
            }

            // Log the update attempt
            \Log::info('Updating post: ' . $post->id);
        });

        // Before deleting a post
        static::deleting(function ($post) {
            // Delete associated comments
            $post->comments()->delete();

            // Log the delete attempt
            \Log::info('Deleting post: ' . $post->id);
        });
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

### 2. Using an Observer

If you prefer to separate the event handling logic from the model, you can use an observer:

First, generate the observer:

`php artisan make:observer PostObserver --model=Post`

Then implement your logic in the observer:

```php
<?php

namespace App\Observers;

use App\Models\Post;
use Illuminate\Support\Str;

class PostObserver
{
    public function creating(Post $post)
    {
        // Generate a slug from the title
        $post->slug = Str::slug($post->title);

        // Set the user_id if not already set
        if (!$post->user_id && auth()->check()) {
            $post->user_id = auth()->id();
        }

        // Default status is draft
        $post->status = $post->status ?? 'draft';

        // Log the creation attempt
        \Log::info('Attempting to create post: ' . $post->title);
    }

    public function updating(Post $post)
    {
        // If title changed, update the slug
        if ($post->isDirty('title')) {
            $post->slug = Str::slug($post->title);
        }

        // Log the update attempt
        \Log::info('Updating post: ' . $post->id);
    }

    public function deleting(Post $post)
    {
        // Delete associated comments
        $post->comments()->delete();

        // Log the delete attempt
        \Log::info('Deleting post: ' . $post->id);
    }
}
```

Register the observer in your `AppServiceProvider.php`:

```php
<?php

namespace App\Providers;

use App\Models\Post;
use App\Observers\PostObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Post::observe(PostObserver::class);
    }
}
```

### 3. Using Mutators for Attribute-Specific Actions

If you need to transform specific attributes before saving, you can use mutators:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Post extends Model
{
    protected $fillable = ['title', 'body', 'slug', 'user_id', 'status'];

    // Automatically slug the title when it's set
    public function setTitleAttribute($value)
    {
        $this->attributes['title'] = $value;
        $this->attributes['slug'] = Str::slug($value);
    }

    // Ensure the status is valid
    public function setStatusAttribute($value)
    {
        $validStatuses = ['draft', 'published', 'archived'];

        $this->attributes['status'] = in_array($value, $validStatuses)
            ? $value
            : 'draft';
    }
}
```

### 4. Real-World Usage Example

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Support\Str;

class Product extends Model
{
    use SoftDeletes;

    protected $fillable = [
        'name', 'slug', 'description', 'price', 'stock', 'category_id', 'is_featured', 'status'
    ];

    protected static function boot()
    {
        parent::boot();

        static::creating(function ($product) {
            // Generate slug
            $product->slug = Str::slug($product->name);

            // Ensure uniqueness of slug
            $count = static::whereRaw("slug RLIKE '^{$product->slug}(-[0-9]+)?$'")->count();
            if ($count > 0) {
                $product->slug = "{$product->slug}-{$count}";
            }

            // Default status
            $product->status = $product->status ?? 'inactive';

            // Validate price
            if ($product->price < 0) {
                throw new \Exception('Product price cannot be negative.');
            }
        });

        static::saving(function ($product) {
            // Truncate description if too long
            if (strlen($product->description) > 1000) {
                $product->description = substr($product->description, 0, 997) . '...';
            }

            // Update slug if name changes
            if ($product->isDirty('name')) {
                $product->slug = Str::slug($product->name);

                // Ensure uniqueness of slug
                $count = static::whereRaw("slug RLIKE '^{$product->slug}(-[0-9]+)?$'")
                    ->where('id', '!=', $product->id)
                    ->count();

                if ($count > 0) {
                    $product->slug = "{$product->slug}-{$count}";
                }
            }
        });

        static::updating(function ($product) {
            // Log price changes
            if ($product->isDirty('price')) {
                \Log::info('Product price changed for #' . $product->id .
                    ' from ' . $product->getOriginal('price') .
                    ' to ' . $product->price);
            }
        });

        static::deleting(function ($product) {
            // Remove from featured products list if featured
            if ($product->is_featured) {
                \Cache::forget('featured_products');
            }
        });
    }

    // Relationships and other methods...
}
```

### Usage in Controllers

Here's how you might use this model in a controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;

class ProductController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'description' => 'required|string',
            'price' => 'required|numeric|min:0',
            'stock' => 'required|integer|min:0',
            'category_id' => 'required|exists:categories,id',
            'is_featured' => 'boolean',
        ]);

        // The model events will handle slug generation, validation, etc.
        $product = Product::create($validated);

        return redirect()->route('products.show', $product)
            ->with('success', 'Product created successfully!');
    }

    public function update(Request $request, Product $product)
    {
        $validated = $request->validate([
            'name' => 'string|max:255',
            'description' => 'string',
            'price' => 'numeric|min:0',
            'stock' => 'integer|min:0',
            'category_id' => 'exists:categories,id',
            'is_featured' => 'boolean',
            'status' => 'in:active,inactive,archived',
        ]);

        // Model events will handle slug updates, logging, etc.
        $product->update($validated);

        return back()->with('success', 'Product updated successfully!');
    }
}
```
