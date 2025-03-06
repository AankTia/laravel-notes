Here's a simple example of a Laravel CRUD (Create, Read, Update, Delete) controller and view.  

---

### **Step 1: Create a Model and Migration**  
Run the following command to generate a model with a migration file:  

```bash
php artisan make:model Post -m
```

Inside `database/migrations/YYYY_MM_DD_create_posts_table.php`, modify the schema:  

```php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->timestamps();
    });
}
```

Run the migration:  

```bash
php artisan migrate
```

---

### **Step 2: Create a Controller**  
Generate a controller:  

```bash
php artisan make:controller PostController --resource
```

Modify `app/Http/Controllers/PostController.php`:  

```php
namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::all();
        return view('posts.index', compact('posts'));
    }

    public function create()
    {
        return view('posts.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required',
            'content' => 'required',
        ]);

        Post::create($request->all());

        return redirect()->route('posts.index')->with('success', 'Post created successfully.');
    }

    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }

    public function edit(Post $post)
    {
        return view('posts.edit', compact('post'));
    }

    public function update(Request $request, Post $post)
    {
        $request->validate([
            'title' => 'required',
            'content' => 'required',
        ]);

        $post->update($request->all());

        return redirect()->route('posts.index')->with('success', 'Post updated successfully.');
    }

    public function destroy(Post $post)
    {
        $post->delete();
        return redirect()->route('posts.index')->with('success', 'Post deleted successfully.');
    }
}
```

---

### **Step 3: Define Routes**  
Modify `routes/web.php`:  

```php
use App\Http\Controllers\PostController;

Route::resource('posts', PostController::class);
```

---

### **Step 4: Create Blade Views**  

#### **Index View (`resources/views/posts/index.blade.php`)**  
```blade
@extends('layout')

@section('content')
    <h1>Posts</h1>
    <a href="{{ route('posts.create') }}">Create Post</a>

    @if (session('success'))
        <p>{{ session('success') }}</p>
    @endif

    <ul>
        @foreach ($posts as $post)
            <li>
                <a href="{{ route('posts.show', $post->id) }}">{{ $post->title }}</a>
                <a href="{{ route('posts.edit', $post->id) }}">Edit</a>
                <form action="{{ route('posts.destroy', $post->id) }}" method="POST" style="display:inline;">
                    @csrf
                    @method('DELETE')
                    <button type="submit">Delete</button>
                </form>
            </li>
        @endforeach
    </ul>
@endsection
```

---

#### **Create View (`resources/views/posts/create.blade.php`)**  
```blade
@extends('layout')

@section('content')
    <h1>Create Post</h1>
    <form action="{{ route('posts.store') }}" method="POST">
        @csrf
        <label>Title:</label>
        <input type="text" name="title" required>
        
        <label>Content:</label>
        <textarea name="content" required></textarea>

        <button type="submit">Submit</button>
    </form>
@endsection
```

---

#### **Edit View (`resources/views/posts/edit.blade.php`)**  
```blade
@extends('layout')

@section('content')
    <h1>Edit Post</h1>
    <form action="{{ route('posts.update', $post->id) }}" method="POST">
        @csrf
        @method('PUT')

        <label>Title:</label>
        <input type="text" name="title" value="{{ $post->title }}" required>

        <label>Content:</label>
        <textarea name="content" required>{{ $post->content }}</textarea>

        <button type="submit">Update</button>
    </form>
@endsection
```

---

#### **Show View (`resources/views/posts/show.blade.php`)**  
```blade
@extends('layout')

@section('content')
    <h1>{{ $post->title }}</h1>
    <p>{{ $post->content }}</p>
    <a href="{{ route('posts.index') }}">Back to List</a>
@endsection
```

---

### **Step 5: Layout File (`resources/views/layout.blade.php`)**  
```blade
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Laravel CRUD</title>
</head>
<body>
    @yield('content')
</body>
</html>
```

---

### **Final Step: Run the Application**  
```bash
php artisan serve
```

Visit:  
- `http://127.0.0.1:8000/posts` to see the CRUD in action!  

This provides a basic Laravel CRUD example. Let me know if you need enhancements! 🚀