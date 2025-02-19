# Password Update Controller with Validation

Controller example for updating passwords with proper validation:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rules\Password;

class PasswordController extends Controller
{
    /**
     * Update the user's password.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request)
    {
        $user = Auth::user();

        // Validate the request
        $validated = $request->validate([
            'current_password' => ['required', 'current_password'],
            'password' => [
                'required',
                'string',
                'confirmed', // This automatically checks against password_confirmation field
                Password::min(8)
                    ->mixedCase()
                    ->numbers()
                    ->symbols()
                    ->uncompromised(),
            ],
        ]);

        // Update the user's password
        $user->password = Hash::make($validated['password']);
        $user->save();

        return response()->json([
            'message' => 'Password updated successfully'
        ]);
    }
}
```

## Key validation features:
- `'confirmed'` validation rule automatically checks that the `password` field matches the `password_confirmation` field
- `current_password` rule verifies the user's existing password
- Password strength requirements using Laravel's Password facade:
    - Minimum 8 characters
    - Mix of uppercase and lowercase letters
    - Contains numbers and symbols
    - Checks against known compromised passwords

## Form input example:

```php
<input type="password" name="current_password">
<input type="password" name="password">
<input type="password" name="password_confirmation">
```

## Route definition:
```php
Route::put('/password/update', [PasswordController::class, 'update'])->middleware('auth');
```