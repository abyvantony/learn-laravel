# Level 6: Authentication and authorization: who can do what

**Track:** Laravel · **Level:** 6/12 · **Cycle:** 3 · **Difficulty:** `intermediate`

📚 **Today's lesson** — published 2026-07-13

## TL;DR

Authentication is 'who are you?' (login). Authorization is 'what can you do?' (permissions). Laravel has both built-in: Breeze/Jetstream for auth scaffolding, Gates and Policies for permissions.

## Real-world analogies

- Authentication is showing your ID at the airport. Authorization is whether you're allowed in the cockpit — not everyone with an ID can go there.
- A Gate is a security guard at a specific door. 'Can this user enter?' A Policy is the rulebook the guard consults: 'Admins yes, others no, between 9-5 only'.

## Key concepts

### `Laravel Breeze`

Minimal auth scaffolding: login, register, password reset, email verification. `composer require laravel/breeze && php artisan breeze:install`.

### ``Auth` facade`

`Auth::user()`, `Auth::check()`, `Auth::id()`, `Auth::login($user)`, `Auth::logout()`.

### `Gates`

Closure-based authorization. `Gate::define('edit-post', fn($user, $post) => $user->id === $post->user_id)`.

### `Policies`

Class-based authorization. One Policy per Model. `PostPolicy@update`. Use `authorize('update', $post)` in controllers.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Installing Breeze ===
// composer require laravel/breeze --dev
// php artisan breeze:install
// npm install && npm run dev
// php artisan migrate
// This gives you: /login, /register, /logout, password reset, email verification

// === Auth facade ===
use Illuminate\Support\Facades\Auth;

Auth::check();         // Is anyone logged in?
Auth::user();          // The current user (or null)
Auth::id();            // Current user ID (or null)
Auth::guest();         // Is no one logged in?

// In a controller
public function dashboard()
{
    if (Auth::guest()) {
        return redirect('/login');
    }
    $user = Auth::user();
    return view('dashboard', compact('user'));
}

// Better: use the 'auth' middleware
Route::get('/dashboard', [DashboardController::class, 'index'])
    ->middleware('auth');

// === Manual login (for API or custom flows) ===
public function login(Request $request)
{
    $credentials = $request->validate([
        'email' => 'required|email',
        'password' => 'required',
    ]);

    if (Auth::attempt($credentials, $request->boolean('remember'))) {
        $request->session()->regenerate();  // Prevent session fixation
        return redirect()->intended('/dashboard');
    }

    return back()->withErrors(['email' => 'Invalid credentials']);
}

// === Authorization with Gates ===
// In a service provider (e.g., AuthServiceProvider)
use Illuminate\Support\Facades\Gate;

Gate::define('update-post', function (User $user, Post $post) {
    return $user->id === $post->user_id || $user->isAdmin();
});

// Use in a controller
public function update(Request $request, Post $post)
{
    if (Gate::denies('update-post', $post)) {
        abort(403);
    }
    $post->update($request->validated());
    return redirect()->route('posts.show', $post);
}

// Or in a Blade view
@can('update-post', $post)
    <a href="{{ route('posts.edit', $post) }}">Edit</a>
@endcan

// === Policies (preferred for model authorization) ===
// php artisan make:policy PostPolicy --model=Post

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }

    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id || $user->isAdmin();
    }

    public function viewAny(User $user): bool
    {
        return true;  // Anyone can view posts
    }
}

// Use in a controller
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);  // Throws 403 if denied
    $post->update($request->validated());
    return redirect()->route('posts.show', $post);
}

// === Roles (with Spatie/laravel-permission package) ===
// composer require spatie/laravel-permission
// Gives you: $user->assignRole('admin'), $user->hasRole('admin'),
//            $user->can('edit-posts'), @role('admin')

// === Middleware for route protection ===
Route::middleware(['auth', 'verified'])->group(function () {
    // Only authenticated, verified users
});

Route::middleware(['auth', 'can:admin'])->group(function () {
    // Only admins
});

// Custom middleware
// php artisan make:middleware EnsureUserIsAdmin
// In handle(): if (!auth()->user()->isAdmin()) abort(403);
```

## Try it yourself

Install Breeze. Add an `is_admin` boolean to users. Create a `PostPolicy` that lets only the post's author (or an admin) edit/delete. Use `@can` in the post views to show/hide the edit button.

## Common pitfalls

- ⚠️ Forgetting to hash passwords. Always use `Hash::make($password)` or `bcrypt($password)`. Never store plain text.
- ⚠️ Not regenerating the session on login. Vulnerable to session fixation attacks.
- ⚠️ Authorization only in the UI. Hiding the edit button is UX, not security. ALWAYS authorize in the controller too.

## What's next?

**Level 7: Database relationships: one-to-many, many-to-many, and beyond** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 6 · Cycle 3_