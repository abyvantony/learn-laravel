# Level 2: Routing, controllers, and request handling

**Track:** Laravel · **Level:** 2/12 · **Cycle:** 3 · **Difficulty:** `beginner`

📚 **Today's lesson** — published 2026-07-11

## TL;DR

Routes define what URLs your app responds to. Controllers handle the logic. Requests come in, responses go out. Master these and you can build any web app.

## Real-world analogies

- A route is a doorman at a hotel. He checks your reservation (URL pattern) and directs you to your room (controller method).
- A controller method is a hotel concierge. You tell them what you want (request data), they get it done and hand you back a result (response).

## Key concepts

### `HTTP verbs`

GET (read), POST (create), PUT/PATCH (update), DELETE (delete). RESTful routes map one verb per action.

### `Route parameters`

`/users/{id}` captures the `id` from the URL. `{id?}` makes it optional. `{id?}` with a default in the controller.

### `Named routes`

`Route::get(...)->name('users.show')` lets you reference routes by name: `route('users.show', $id)`. Refactor-friendly.

### `Middleware`

Code that runs before/after a request. Used for auth, logging, CORS, rate limiting. Apply per-route, per-group, or globally.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === routes/web.php ===
use App\Http\Controllers\UserController;

// Basic routes
Route::get('/about', function () {
    return 'About page';
});

// Routes with parameters
Route::get('/users/{id}', [UserController::class, 'show'])
    ->where('id', '[0-9]+');  // Only matches digits

// Optional parameter with default
Route::get('/greet/{name?}', function ($name = 'World') {
    return "Hello, {$name}!";
});

// HTTP verbs (RESTful)
Route::get('/users', [UserController::class, 'index']);     // List
Route::get('/users/{id}', [UserController::class, 'show']); // Read
Route::post('/users', [UserController::class, 'store']);    // Create
Route::put('/users/{id}', [UserController::class, 'update']); // Replace
Route::patch('/users/{id}', [UserController::class, 'update']); // Partial update
Route::delete('/users/{id}', [UserController::class, 'destroy']); // Delete

// Named routes
Route::get('/profile', [UserController::class, 'profile'])->name('profile');
// In Blade: <a href="{{ route('profile') }}">Profile</a>
// In controller: return redirect()->route('profile');

// Route groups
Route::prefix('admin')->middleware('auth')->group(function () {
    Route::get('/dashboard', [AdminController::class, 'dashboard']);
    Route::resource('users', AdminUserController::class);
});

// Resource routes (one line creates all 7 RESTful routes)
Route::resource('posts', PostController::class);
// Creates: GET /posts, GET /posts/create, POST /posts, GET /posts/{id},
//          GET /posts/{id}/edit, PUT/PATCH /posts/{id}, DELETE /posts/{id}

// API routes (no sessions, no CSRF — for JSON APIs)
Route::apiResource('api/posts', ApiPostController::class);

// === A controller ===
namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;

class UserController extends Controller
{
    public function index(Request $request)
    {
        $users = User::all();
        return view('users.index', compact('users'));
    }

    public function show(int $id)
    {
        $user = User::findOrFail($id);
        return view('users.show', compact('user'));
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:users',
        ]);

        $user = User::create($data);
        return redirect()->route('users.show', $user);
    }

    public function update(Request $request, int $id)
    {
        $user = User::findOrFail($id);
        $user->update($request->all());
        return redirect()->route('users.show', $user);
    }

    public function destroy(int $id)
    {
        User::destroy($id);
        return redirect()->route('users.index');
    }
}

// === Middleware ===
// php artisan make:middleware CheckAge
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckAge
{
    public function handle(Request $request, Closure $next, int $minAge = 18)
    {
        if ($request->user()->age < $minAge) {
            return redirect('home');
        }
        return $next($request);
    }
}

// Apply in routes/web.php:
Route::get('/admin', function () { /* ... */ })->middleware('check.age:21');

// === Request helper methods ===
$request->input('name');        // Get input field
$request->all();                 // All input
$request->only(['name', 'email']); // Whitelist
$request->except(['_token']);   // Blacklist
$request->has('name');           // Check presence
$request->boolean('is_admin');  // Boolean coercion
$request->file('avatar');       // Uploaded file
$request->cookie('name');       // Cookie
$request->header('X-Token');    // Header
$request->ip();                  // Client IP
$request->user();                // Authenticated user
```

## Try it yourself

Create a `PostController` with `index`, `show`, `create`, `store` methods. Use `Route::resource('posts', PostController::class)`. Create a Blade view listing posts and a form to create one.

## Common pitfalls

- ⚠️ Forgetting to add routes. New controllers don't appear in your app until you wire up routes for them.
- ⚠️ Not validating input. Always use `$request->validate(...)` — never trust user input.
- ⚠️ Using GET for state-changing actions (delete, update). Use POST/DELETE — they can have CSRF protection and don't get cached.

## What's next?

**Level 3: Eloquent ORM: working with databases the elegant way** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 2 · Cycle 3_