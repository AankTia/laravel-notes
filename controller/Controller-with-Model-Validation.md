# Controller with Model Validation

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class ProductController extends Controller
{
    /**
     * Store a newly created product
     */
    public function store(Request $request)
    {
        // Define validation rules for creating
        $rules = [
            'name' => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
            'description' => 'nullable|string',
            'stock' => 'required|integer|min:0',
        ];
        
        // Validate the request
        $validator = Validator::make($request->all(), $rules);
        
        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'errors' => $validator->errors()
            ], 422);
        }
        
        // Create and save the product
        $product = new Product();
        $product->name = $request->name;
        $product->price = $request->price;
        $product->description = $request->description;
        $product->stock = $request->stock;
        $product->save();
        
        return response()->json([
            'success' => true,
            'message' => 'Product created successfully',
            'data' => $product
        ], 201);
    }
    
    /**
     * Update the specified product
     */
    public function update(Request $request, $id)
    {
        // Find the product
        $product = Product::find($id);
        
        if (!$product) {
            return response()->json([
                'success' => false,
                'message' => 'Product not found'
            ], 404);
        }
        
        // Define validation rules for updating
        $rules = [
            'name' => 'sometimes|string|max:255',
            'price' => 'sometimes|numeric|min:0',
            'description' => 'sometimes|nullable|string',
            'stock' => 'sometimes|integer|min:0',
        ];
        
        // Validate the request
        $validator = Validator::make($request->all(), $rules);
        
        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'errors' => $validator->errors()
            ], 422);
        }
        
        // Update product fields if they exist in the request
        if ($request->has('name')) {
            $product->name = $request->name;
        }
        
        if ($request->has('price')) {
            $product->price = $request->price;
        }
        
        if ($request->has('description')) {
            $product->description = $request->description;
        }
        
        if ($request->has('stock')) {
            $product->stock = $request->stock;
        }
        
        $product->save();
        
        return response()->json([
            'success' => true,
            'message' => 'Product updated successfully',
            'data' => $product
        ]);
    }
    
    /**
     * Alternative approach using Form Request validation
     */
    public function storeWithFormRequest(StoreProductRequest $request)
    {
        // The request is automatically validated by StoreProductRequest
        
        $product = new Product();
        $product->name = $request->name;
        $product->price = $request->price;
        $product->description = $request->description;
        $product->stock = $request->stock;
        $product->save();
        
        return response()->json([
            'success' => true,
            'message' => 'Product created successfully',
            'data' => $product
        ], 201);
    }
}

/**
 * Form Request class example (would be in a separate file)
 */
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    public function authorize()
    {
        return true; // Or check permission logic here
    }
    
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
            'description' => 'nullable|string',
            'stock' => 'required|integer|min:0',
        ];
    }
}
```

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class ProductController extends Controller
{
    /**
     * Store a newly created product
     */
    public function store(Request $request)
    {
        // Define validation rules for creating
        $rules = [
            'name' => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
            'description' => 'nullable|string',
            'stock' => 'required|integer|min:0',
        ];
        
        // Validate the request
        $validator = Validator::make($request->all(), $rules);
        
        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'errors' => $validator->errors()
            ], 422);
        }
        
        // Create and save the product
        $product = new Product();
        $product->name = $request->name;
        $product->price = $request->price;
        $product->description = $request->description;
        $product->stock = $request->stock;
        $product->save();
        
        return response()->json([
            'success' => true,
            'message' => 'Product created successfully',
            'data' => $product
        ], 201);
    }
    
    /**
     * Update the specified product
     */
    public function update(Request $request, $id)
    {
        // Find the product
        $product = Product::find($id);
        
        if (!$product) {
            return response()->json([
                'success' => false,
                'message' => 'Product not found'
            ], 404);
        }
        
        // Define validation rules for updating
        $rules = [
            'name' => 'sometimes|string|max:255',
            'price' => 'sometimes|numeric|min:0',
            'description' => 'sometimes|nullable|string',
            'stock' => 'sometimes|integer|min:0',
        ];
        
        // Validate the request
        $validator = Validator::make($request->all(), $rules);
        
        if ($validator->fails()) {
            return response()->json([
                'success' => false,
                'errors' => $validator->errors()
            ], 422);
        }
        
        // Update product fields if they exist in the request
        if ($request->has('name')) {
            $product->name = $request->name;
        }
        
        if ($request->has('price')) {
            $product->price = $request->price;
        }
        
        if ($request->has('description')) {
            $product->description = $request->description;
        }
        
        if ($request->has('stock')) {
            $product->stock = $request->stock;
        }
        
        $product->save();
        
        return response()->json([
            'success' => true,
            'message' => 'Product updated successfully',
            'data' => $product
        ]);
    }
    
    /**
     * Alternative approach using Form Request validation
     */
    public function storeWithFormRequest(StoreProductRequest $request)
    {
        // The request is automatically validated by StoreProductRequest
        
        $product = new Product();
        $product->name = $request->name;
        $product->price = $request->price;
        $product->description = $request->description;
        $product->stock = $request->stock;
        $product->save();
        
        return response()->json([
            'success' => true,
            'message' => 'Product created successfully',
            'data' => $product
        ], 201);
    }
}

/**
 * Form Request class example (would be in a separate file)
 */
namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreProductRequest extends FormRequest
{
    public function authorize()
    {
        return true; // Or check permission logic here
    }
    
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'price' => 'required|numeric|min:0',
            'description' => 'nullable|string',
            'stock' => 'required|integer|min:0',
        ];
    }
}

```

This example shows how to implement validation in Laravel controllers before saving or updating models. There are two main approaches demonstrated:

1. **Direct Controller Validation:**
   - Uses the `Validator` facade in controller methods
   - Has separate validation rules for create (`store`) and update operations
   - Returns appropriate JSON responses with error details when validation fails
   - Only updates fields that are present in the request during update operations

2. **Form Request Validation:**
   - Uses Laravel's Form Request classes for more reusable validation
   - Keeps controllers cleaner by moving validation logic to dedicated request classes
   - Validation happens automatically before the controller method is executed

To use the Form Request approach, you would need to create separate request classes for store and update operations, and then typehint these in your controller methods.

The key benefit of controller validation is that you can handle validation errors in a consistent way across your application and return appropriate responses to your frontend.