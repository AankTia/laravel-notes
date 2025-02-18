# Model & Migration Example Implementation

This example of a Blog system with Posts and Categories

## Step 1: Create the Migrations

First, let's create the necessary migrations:

```
php artisan make:migration create_categories_table
php artisan make:migration create_posts_table
```

### Categories Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateCategoriesTable extends Migration
{
    public function up()
    {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->boolean('is_active')->default(true);
            $table->timestamps();
            $table->softDeletes();
        });
    }

    public function down()
    {
        Schema::dropIfExists('categories');
    }
}
```

### Post Migration

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePostsTable extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('content');
            $table->string('featured_image')->nullable();
            $table->enum('status', ['draft', 'published', 'archived'])->default('draft');
            $table->timestamp('published_at')->nullable();
            $table->unsignedBigInteger('category_id');
            $table->unsignedBigInteger('user_id');
            $table->timestamps();
            $table->softDeletes();

            // Foreign key constraints
            $table->foreign('category_id')
                  ->references('id')
                  ->on('categories')
                  ->onDelete('restrict');

            $table->foreign('user_id')
                  ->references('id')
                  ->on('users')
                  ->onDelete('cascade');

            // Indexes
            $table->index(['status', 'published_at']);
            $table->fullText('content');
        });
    }

    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

## Step 2: Create the Models

```
php artisan make:model Category
php artisan make:model Post
```

### Category Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Support\Str;

class Category extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'name',
        'slug',
        'description',
        'is_active',
    ];

    protected $casts = [
        'is_active' => 'boolean',
    ];

    /**
     * Boot the model.
     */
    protected static function boot()
    {
        parent::boot();

        static::creating(function ($category) {
            if (empty($category->slug)) {
                $category->slug = Str::slug($category->name);
            }
        });

        static::updating(function ($category) {
            if ($category->isDirty('name') && !$category->isDirty('slug')) {
                $category->slug = Str::slug($category->name);
            }
        });
    }

    /**
     * Get the posts for the category.
     */
    public function posts()
    {
        return $this->hasMany(Post::class);
    }

    /**
     * Scope a query to only include active categories.
     */
    public function scopeActive($query)
    {
        return $query->where('is_active', true);
    }
}
```

### Post Model

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;
use Illuminate\Support\Str;

class Post extends Model
{
    use HasFactory, SoftDeletes;

    protected $fillable = [
        'title',
        'slug',
        'content',
        'featured_image',
        'status',
        'published_at',
        'category_id',
        'user_id',
    ];

    protected $casts = [
        'published_at' => 'datetime',
    ];

    /**
     * Boot the model.
     */
    protected static function boot()
    {
        parent::boot();

        static::creating(function ($post) {
            // Generate slug if not provided
            if (empty($post->slug)) {
                $post->slug = Str::slug($post->title);
            }

            // Set user_id if not provided
            if (empty($post->user_id) && auth()->check()) {
                $post->user_id = auth()->id();
            }
        });

        static::updating(function ($post) {
            // Update slug if title changed and slug wasn't manually changed
            if ($post->isDirty('title') && !$post->isDirty('slug')) {
                $post->slug = Str::slug($post->title);
            }

            // Set published_at when status changes to published
            if ($post->isDirty('status') && $post->status === 'published' && !$post->published_at) {
                $post->published_at = now();
            }
        });

        static::deleting(function ($post) {
            // Custom logic before deletion if needed
            \Log::info('Post being deleted: ' . $post->id);
        });
    }

    /**
     * Get the category that the post belongs to.
     */
    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    /**
     * Get the user who wrote the post.
     */
    public function author()
    {
        return $this->belongsTo(User::class, 'user_id');
    }

    /**
     * Scope a query to only include published posts.
     */
    public function scopePublished($query)
    {
        return $query->where('status', 'published')
                    ->where('published_at', '<=', now());
    }

    /**
     * Scope a query to include posts by status.
     */
    public function scopeByStatus($query, $status)
    {
        return $query->where('status', $status);
    }

    /**
     * Get the post's excerpt.
     */
    public function getExcerptAttribute()
    {
        return Str::limit(strip_tags($this->content), 150);
    }

    /**
     * Check if the post is published.
     */
    public function isPublished()
    {
        return $this->status === 'published' && $this->published_at <= now();
    }
}
```

## Step 3: Run the Migrations

```
php artisan migrate
```

## Step 4: Create Factories and Seeders (Optional but Recommended)

For testing and development, it's helpful to have seed data:

```
php artisan make:factory CategoryFactory --model=Category
php artisan make:factory PostFactory --model=Post
```

### CategoryFactory

```php
<?php

namespace Database\Factories;

use App\Models\Category;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class CategoryFactory extends Factory
{
    protected $model = Category::class;

    public function definition()
    {
        $name = $this->faker->unique()->word;
        return [
            'name' => ucfirst($name),
            'slug' => Str::slug($name),
            'description' => $this->faker->paragraph,
            'is_active' => $this->faker->boolean(80), // 80% chance of being active
        ];
    }
}
```

### PostFactory

```php
<?php

namespace Database\Factories;

use App\Models\Category;
use App\Models\Post;
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class PostFactory extends Factory
{
    protected $model = Post::class;

    public function definition()
    {
        $title = $this->faker->sentence;
        $statuses = ['draft', 'published', 'archived'];
        $status = $this->faker->randomElement($statuses);

        return [
            'title' => $title,
            'slug' => Str::slug($title),
            'content' => $this->faker->paragraphs(5, true),
            'featured_image' => 'images/posts/' . $this->faker->numberBetween(1, 10) . '.jpg',
            'status' => $status,
            'published_at' => $status === 'published' ? $this->faker->dateTimeBetween('-1 year', 'now') : null,
            'category_id' => Category::factory(),
            'user_id' => User::factory(),
        ];
    }

    /**
     * Indicate that the post is published.
     */
    public function published()
    {
        return $this->state(function (array $attributes) {
            return [
                'status' => 'published',
                'published_at' => $this->faker->dateTimeBetween('-1 year', 'now'),
            ];
        });
    }

    /**
     * Indicate that the post is a draft.
     */
    public function draft()
    {
        return $this->state(function (array $attributes) {
            return [
                'status' => 'draft',
                'published_at' => null,
            ];
        });
    }
}
```

### Database Seeder

```php
<?php

namespace Database\Seeders;

use App\Models\Category;
use App\Models\Post;
use App\Models\User;
use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run()
    {
        // Create users with posts
        User::factory(10)->create()->each(function ($user) {
            // Create 3-7 posts for each user
            Post::factory(rand(3, 7))
                ->create([
                    'user_id' => $user->id,
                    'category_id' => Category::factory()->create()->id,
                ]);
        });

        // Create additional categories
        Category::factory(5)->create();
    }
}
```

Run the seeder:

`php artisan db:seed`

## Step 5: Using the Models in a Controller

Here's a basic PostController to demonstrate how to use these models:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Category;
use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with(['category', 'author'])
                     ->published()
                     ->latest('published_at')
                     ->paginate(10);
                     
        return view('posts.index', compact('posts'));
    }
    
    public function show($slug)
    {
        $post = Post::with(['category', 'author'])
                    ->whereSlug($slug)
                    ->firstOrFail();
                    
        return view('posts.show', compact('post'));
    }
    
    public function create()
    {
        $categories = Category::active()->get();
        return view('posts.create', compact('categories'));
    }
    
    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'content' => 'required|string',
            'featured_image' => 'nullable|image|max:2048',
            'category_id' => 'required|exists:categories,id',
            'status' => 'required|in:draft,published',
        ]);
        
        // Handle featured image upload
        if ($request->hasFile('featured_image')) {
            $path = $request->file('featured_image')->store('images/posts', 'public');
            $validated['featured_image'] = $path;
        }
        
        // Create the post (user_id and slug are handled by model events)
        $post = Post::create($validated);
        
        return redirect()->route('posts.show', $post->slug)
                         ->with('success', 'Post created successfully!');
    }
    
    // Additional methods for update, delete, etc.
}
```

## Benefits of This Implementation

- **Automatic Slug Generation**: The model handles slug creation and uniqueness
- **User Association**: Automatically sets the current user as the author
- **Status Management**: Handles published date when status changes
- **Soft Deletes**: Allows recovery of accidentally deleted items
- **Factory & Seeder**: Makes testing and development easier with realistic data
- **Model Scopes**: Makes filtering and querying cleaner