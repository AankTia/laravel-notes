# Model Validator Example

Here's an example of how to implement validation in Laravel using the built-in validation system with models:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Facades\Validator;

class Post extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'title',
        'content',
        'user_id',
        'published_at',
    ];

    /**
     * Get validation rules for the model.
     *
     * @return array
     */
    public static function getValidationRules()
    {
        return [
            'title' => 'required|string|max:255',
            'content' => 'required|string',
            'user_id' => 'required|exists:users,id',
            'published_at' => 'nullable|date',
        ];
    }

    /**
     * Get validation messages for the model.
     *
     * @return array
     */
    public static function getValidationMessages()
    {
        return [
            'title.required' => 'The post title is required',
            'title.max' => 'The post title cannot exceed 255 characters',
            'content.required' => 'The post content is required',
            'user_id.required' => 'The post must be assigned to a user',
            'user_id.exists' => 'The specified user does not exist',
            'published_at.date' => 'The published date must be a valid date',
        ];
    }

    /**
     * Validate the model data.
     *
     * @param array $data
     * @return \Illuminate\Contracts\Validation\Validator
     */
    public static function validate(array $data)
    {
        return Validator::make(
            $data,
            self::getValidationRules(),
            self::getValidationMessages()
        );
    }
}
```

Now, let's see how to use this validation in a controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Store a newly created post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate the request data using model validation
        $validator = Post::validate($request->all());

        if ($validator->fails()) {
            return redirect()->back()
                ->withErrors($validator)
                ->withInput();
        }

        // Create the post
        $post = Post::create($validator->validated());

        return redirect()->route('posts.show', $post)
            ->with('success', 'Post created successfully!');
    }

    /**
     * Update the specified post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \App\Models\Post  $post
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, Post $post)
    {
        // Validate the request data using model validation
        $validator = Post::validate($request->all());

        if ($validator->fails()) {
            return redirect()->back()
                ->withErrors($validator)
                ->withInput();
        }

        // Update the post
        $post->update($validator->validated());

        return redirect()->route('posts.show', $post)
            ->with('success', 'Post updated successfully!');
    }
}
```

Alternatively, you can use Laravel's FormRequest for validation:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use App\Models\Post;

class StorePostRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true; // You can add authorization logic here
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return Post::getValidationRules();
    }

    /**
     * Get the error messages for the defined validation rules.
     *
     * @return array
     */
    public function messages()
    {
        return Post::getValidationMessages();
    }
}
```

Then in your controller:

```php
public function store(StorePostRequest $request)
{
    // The request is already validated
    $post = Post::create($request->validated());

    return redirect()->route('posts.show', $post)
        ->with('success', 'Post created successfully!');
}
```
