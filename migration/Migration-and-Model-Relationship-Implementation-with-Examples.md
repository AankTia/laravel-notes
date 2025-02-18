# Laravel Migration and Model Relationship Implementation with Examples

## 1. One-to-One Relationship

Implement a one-to-one relationship between a User and a Profile.

### Migration Files

users table migration (probably already exists):

```php
 function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

profiles table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProfilesTable extends Migration
{
    public function up()
    {
        Schema::create('profiles', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id');
            $table->string('bio')->nullable();
            $table->string('phone_number')->nullable();
            $table->string('address')->nullable();
            $table->string('profile_picture')->nullable();
            $table->timestamps();
            
            // Foreign key constraint
            $table->foreign('user_id')
                  ->references('id')
                  ->on('users')
                  ->onDelete('cascade');
                  
            // One-to-one relationship constraint
            $table->unique('user_id');
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('profiles');
    }
}
```

### Model Implementation
User model:
```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    protected $fillable = [
        'name', 'email', 'password',
    ];

    protected $hidden = [
        'password', 'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
    ];

    // One-to-one relationship with Profile
    public function profile()
    {
        return $this->hasOne(Profile::class);
    }
}
```

Profile model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Profile extends Model
{
    protected $fillable = [
        'user_id', 'bio', 'phone_number', 'address', 'profile_picture',
    ];

    // Inverse of one-to-one relationship
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

## 2. One-to-Many Relationship

Let's implement a one-to-many relationship between a User and multiple Posts.

### Migration Files

posts table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePostsTable extends Migration
{
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id'); // Foreign key column
            $table->string('title');
            $table->text('content');
            $table->string('image')->nullable();
            $table->boolean('is_published')->default(false);
            $table->timestamps();
            
            // Foreign key constraint
            $table->foreign('user_id')
                  ->references('id')
                  ->on('users')
                  ->onDelete('cascade');
                  
            // Index for faster queries
            $table->index('user_id');
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('posts');
    }
}
```

### Model Implementation

#### User model (add the relationship):

```php
// One-to-many relationship with Post
public function posts()
{
    return $this->hasMany(Post::class);
}
```

#### Post model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    protected $fillable = [
        'user_id', 'title', 'content', 'image', 'is_published',
    ];

    protected $casts = [
        'is_published' => 'boolean',
    ];

    // Inverse of one-to-many relationship
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

## 3. Many-to-Many Relationship
Let's implement a many-to-many relationship between Posts and Tags.

### Migration Files

tags table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTagsTable extends Migration
{
    public function up()
    {
        Schema::create('tags', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('slug')->unique();
            $table->timestamps();
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('tags');
    }
}
```

post_tag pivot table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreatePostTagTable extends Migration
{
    public function up()
    {
        Schema::create('post_tag', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('post_id');
            $table->unsignedBigInteger('tag_id');
            $table->timestamps();
            
            // Foreign key constraints
            $table->foreign('post_id')
                  ->references('id')
                  ->on('posts')
                  ->onDelete('cascade');
                  
            $table->foreign('tag_id')
                  ->references('id')
                  ->on('tags')
                  ->onDelete('cascade');
                  
            // Prevent duplicate relationships
            $table->unique(['post_id', 'tag_id']);
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('post_tag');
    }
}
```

### Model Implementation

Post model (add relationship):

```php
// Many-to-many relationship with Tag
public function tags()
{
    return $this->belongsToMany(Tag::class);
}
```

Tag model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Tag extends Model
{
    protected $fillable = [
        'name', 'slug',
    ];

    // Many-to-many relationship with Post
    public function posts()
    {
        return $this->belongsToMany(Post::class);
    }
}
```

## 4. Has Many Through Relationship

Let's implement a 'has-many-through' relationship between a Country, its Users, and their Posts.

### Migration Files

countries table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateCountriesTable extends Migration
{
    public function up()
    {
        Schema::create('countries', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('code', 2)->unique();
            $table->timestamps();
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('countries');
    }
}
```

Modify users table migration to add country_id:

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->unsignedBigInteger('country_id')->nullable()->after('id');
        $table->foreign('country_id')
              ->references('id')
              ->on('countries')
              ->onDelete('set null');
    });
}
```

### Model Implementation

Country model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Country extends Model
{
    protected $fillable = [
        'name', 'code',
    ];

    // One-to-many relationship with User
    public function users()
    {
        return $this->hasMany(User::class);
    }
    
    // Has-many-through relationship with Post
    public function posts()
    {
        return $this->hasManyThrough(Post::class, User::class);
    }
}
```

User model (update for country relationship):

```php
// Belongs-to relationship with Country
public function country()
{
    return $this->belongsTo(Country::class);
}
```

## 5. Polymorphic Relationship

Let's implement a polymorphic relationship for Comments. Both Posts and Images can have comments.

### Migration Files

images table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateImagesTable extends Migration
{
    public function up()
    {
        Schema::create('images', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('user_id');
            $table->string('title');
            $table->string('url');
            $table->text('description')->nullable();
            $table->timestamps();
            
            $table->foreign('user_id')
                  ->references('id')
                  ->on('users')
                  ->onDelete('cascade');
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('images');
    }
}
```

comments table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateCommentsTable extends Migration
{
    public function up()
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->text('content');
            $table->unsignedBigInteger('user_id');
            // Polymorphic relationship fields
            $table->morphs('commentable'); // Creates commentable_id and commentable_type
            $table->timestamps();
            
            $table->foreign('user_id')
                  ->references('id')
                  ->on('users')
                  ->onDelete('cascade');
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('comments');
    }
}
```

### Model Implementation

Comment model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    protected $fillable = [
        'content', 'user_id', 'commentable_id', 'commentable_type'
    ];

    // Polymorphic relationship
    public function commentable()
    {
        return $this->morphTo();
    }
    
    // Relationship with User
    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
```

Post model (add polymorphic relationship):

```php
// Polymorphic relationship for comments
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}
```

Image model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Image extends Model
{
    protected $fillable = [
        'user_id', 'title', 'url', 'description'
    ];

    // Relationship with User
    public function user()
    {
        return $this->belongsTo(User::class);
    }
    
    // Polymorphic relationship for comments
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

## 6. Many-to-Many Polymorphic Relationship

Let's implement a polymorphic many-to-many relationship for tagging Posts and Images.

### Migration Files

taggables pivot table migration:

```php
Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTaggablesTable extends Migration
{
    public function up()
    {
        Schema::create('taggables', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('tag_id');
            // Polymorphic relationship fields
            $table->morphs('taggable'); // Creates taggable_id and taggable_type
            $table->timestamps();
            
            $table->foreign('tag_id')
                  ->references('id')
                  ->on('tags')
                  ->onDelete('cascade');
                  
            // Prevent duplicate relationships
            $table->unique(['tag_id', 'taggable_id', 'taggable_type'], 'taggable_unique');
        });
    }
    
    public function down()
    {
        Schema::dropIfExists('taggables');
    }
}
```

### Model Implementation

Tag model (update for polymorphic relationship):

```php
// Many-to-many polymorphic relationship
public function posts()
{
    return $this->morphedByMany(Post::class, 'taggable');
}

public function images()
{
    return $this->morphedByMany(Image::class, 'taggable');
}
```

Post model (update for polymorphic many-to-many):

```php
// Replace the previous tags() method with this polymorphic version
public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}
```

Image model (add tags relationship):

```php
// Polymorphic many-to-many relationship
public function tags()
{
    return $this->morphToMany(Tag::class, 'taggable');
}
```

## Example Usage

Here are examples of how to use these relationships in your controllers or elsewhere:

### One-to-One
```php
// Create a user with profile
$user = User::create([
    'name' => 'John Doe',
    'email' => 'john@example.com',
    'password' => bcrypt('password'),
]);

$user->profile()->create([
    'bio' => 'Laravel developer',
    'phone_number' => '123-456-7890',
]);

// Access profile from user
$profile = $user->profile;

// Access user from profile
$user = Profile::find(1)->user;
```

### One-to-Many

```php
// Create posts for a user
$user = User::find(1);
$user->posts()->create([
    'title' => 'My first post',
    'content' => 'This is the content',
]);

// Get all posts by a user
$posts = $user->posts;

// Get the author of a post
$author = Post::find(1)->user;
```

### Many-to-Many

```php
// Create tags
$tag1 = Tag::create(['name' => 'Laravel', 'slug' => 'laravel']);
$tag2 = Tag::create(['name' => 'PHP', 'slug' => 'php']);

// Attach tags to a post
$post = Post::find(1);
$post->tags()->attach([$tag1->id, $tag2->id]);

// Or with sync (removes existing relationships)
$post->tags()->sync([$tag1->id, $tag2->id]);

// Get all tags for a post
$tags = $post->tags;

// Get all posts with a specific tag
$posts = Tag::find(1)->posts;
```

### Has-Many-Through

```php
// Get all posts from a country
$country = Country::find(1);
$posts = $country->posts;
```

### Polymorphic Relationships
```php
// Create a comment on a post
$post = Post::find(1);
$post->comments()->create([
    'content' => 'Great post!',
    'user_id' => 1,
]);

// Create a comment on an image
$image = Image::find(1);
$image->comments()->create([
    'content' => 'Beautiful image!',
    'user_id' => 1,
]);

// Get comments for a post
$comments = Post::find(1)->comments;

// Get the commentable item (post or image)
$commentable = Comment::find(1)->commentable;
```

### Many-to-Many Polymorphic
```php
// Tag a post
$post = Post::find(1);
$post->tags()->attach([1, 2]); // Tag IDs

// Tag an image
$image = Image::find(1);
$image->tags()->attach([1, 3]); // Tag IDs

// Get all posts with a specific tag
$posts = Tag::find(1)->posts;

// Get all images with a specific tag
$images = Tag::find(1)->images;

// Get all tags for a post
$tags = Post::find(1)->tags;
```


This comprehensive guide covers the most common relationship types in Laravel with their migration and model implementations. The relationships are designed to maintain referential integrity using foreign key constraints and provide intuitive methods to access related models.