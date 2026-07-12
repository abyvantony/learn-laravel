# Level 5: Form validation and request handling

**Track:** Laravel · **Level:** 5/12 · **Cycle:** 3 · **Difficulty:** `intermediate`

📚 **Today's lesson** — published 2026-07-12

## TL;DR

Never trust user input. Laravel's validation is one of the framework's best features — declarative, automatic, with helpful error messages. Use Form Requests for complex validation, or `$request->validate()` for quick cases.

## Real-world analogies

- Validation is a bouncer at a club. It checks every input (ID) against a list of rules. Bad ID? You're not getting in. The request stops right there.
- Form Requests are reusable ID templates. Define the rules once, apply to any controller method.

## Key concepts

### `Inline validation`

`$request->validate(['email' => 'required|email'])` — quick and simple, perfect for one-off cases.

### `Form Request classes`

Separate classes that encapsulate validation + authorization. Reusable, testable, clean controllers.

### `Custom rules`

When the built-in rules aren't enough, write your own: `Rule::unique('users')->ignore($user->id)`.

### `Error display`

`@error('field'){{ $message }}@enderror` shows validation errors next to form fields. `old('field')` repopulates inputs.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Inline validation ===
public function store(Request $request)
{
    $data = $request->validate([
        'name'     => 'required|string|max:255',
        'email'    => 'required|email|unique:users,email',
        'age'      => 'required|integer|min:18|max:120',
        'website'  => 'nullable|url',
        'password' => 'required|string|min:8|confirmed',
        'avatar'   => 'nullable|image|max:2048',  // Max 2MB
        'role'     => 'required|in:admin,user,guest',
    ]);

    $user = User::create($data);
    return redirect()->route('users.show', $user);
}

// If validation fails, Laravel auto-redirects back with errors in session.

// === Form Request classes ===
// php artisan make:request StoreUserRequest

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;  // Or check if user can do this
    }

    public function rules(): array
    {
        return [
            'name'  => 'required|string|max:255',
            'email' => 'required|email|unique:users',
            'password' => ['required', 'confirmed', Password::min(8)->mixedCase()->numbers()],
        ];
    }

    public function messages(): array
    {
        return [
            'email.required' => 'We need your email address.',
            'email.unique'   => 'This email is already registered.',
        ];
    }

    public function attributes(): array
    {
        return [
            'email' => 'email address',
        ];
    }
}

// Use it in the controller:
public function store(StoreUserRequest $request)
{
    // Validation already ran. Data is clean.
    $user = User::create($request->validated());
    return redirect()->route('users.show', $user);
}

// === Update request (often differs from create) ===
class UpdateUserRequest extends FormRequest
{
    public function rules(): array
    {
        $userId = $this->route('user')->id;  // From route param
        return [
            'name'  => 'sometimes|string|max:255',
            'email' => "sometimes|email|unique:users,email,{$userId}",
        ];
    }
}

// === Common validation rules ===
// 'required'              — must be present
// 'nullable'              — may be null
// 'string' / 'integer' / 'numeric' / 'boolean' / 'array'
// 'email' / 'url' / 'uuid' / 'ip' / 'json'
// 'min:5' / 'max:255' — for strings, numbers, arrays, files
// 'in:a,b,c' / 'not_in:x,y'
// 'between:1,10' — inclusive
// 'regex:/^[A-Z]+$/'
// 'confirmed' — field_confirmation must match
// 'unique:table,column' / 'exists:table,column'
// 'date' / 'date_format:Y-m-d' / 'before:2026-01-01' / 'after:today'
// 'same:other_field' / 'different:other_field'
// 'file' / 'image' / 'mimes:jpg,png' / 'max:2048' (KB)

// === Conditional validation ===
public function rules(): array
{
    return [
        'email' => 'required_if:contact_method,email',
        'phone' => 'required_if:contact_method,phone',
        'company_name' => 'required_if:account_type,business',
    ];
}

// === Custom validation messages ===
public function messages(): array
{
    return [
        'name.required' => 'Please tell us your name.',
        'email.email'   => 'That doesn\'t look like a valid email.',
    ];
}

// === Displaying errors in Blade ===
{{-- Show all errors at top of form --}}
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{-- Show error next to a specific field --}}
<input name="email" value="{{ old('email') }}" class="@error('email') is-invalid @enderror">
@error('email')
    <div class="invalid-feedback">{{ $message }}</div>
@enderror

{{-- Custom rule (advanced) ===
// php artisan make:rule Uppercase
namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    public function passes($attribute, $value): bool
    {
        return strtoupper($value) === $value;
    }

    public function message(): string
    {
        return 'The :attribute must be uppercase.';
    }
}

// Use: 'name' => ['required', new Uppercase],
```

## Try it yourself

Build a registration form: name, email, password, password_confirmation, age. Use a Form Request class for validation. Display all errors next to their fields. On success, redirect to a welcome page.

## Common pitfalls

- ⚠️ Forgetting to handle validation failure. Laravel auto-redirects back, but make sure your form repopulates with `old()`.
- ⚠️ Validating after processing. Validate FIRST, then use the cleaned data. Never use `$request->all()` directly without validation.
- ⚠️ Putting all rules in one giant array. Use Form Request classes for anything non-trivial — keeps controllers clean.

## What's next?

**Level 6: Authentication and authorization: who can do what** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 5 · Cycle 3_