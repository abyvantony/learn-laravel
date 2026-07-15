# Level 10: API development: RESTful APIs with Sanctum and resources

**Track:** Laravel · **Level:** 10/12 · **Cycle:** 3 · **Difficulty:** `advanced`

📚 **Today's lesson** — published 2026-07-15

## TL;DR

Modern Laravel apps are often a backend API + frontend (React, Vue, mobile). Sanctum handles authentication, API Resources control the JSON output, and rate limiting protects against abuse.

## Real-world analogies

- An API is a waiter in a restaurant. You place an order (request), the waiter goes to the kitchen (server), and brings back a plate (response). JSON is the language of the order.
- Sanctum is the security guard at the door. He checks your token before letting you place an order.

## Key concepts

### `API routes`

Define in `routes/api.php`. Prefixed with `/api` automatically. Stateless — no sessions.

### `Sanctum`

Laravel's lightweight auth package. Tokens for SPAs, mobile apps, simple API consumers. `composer require laravel/sanctum`.

### `API Resources`

Transform your Eloquent models into clean JSON. Control exactly which fields are exposed.

### `Rate limiting`

Throttle middleware: `->middleware('throttle:60,1')` — 60 requests per minute per IP.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Install Sanctum ===
// composer require laravel/sanctum
// php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
// php artisan migrate
// Add HasApiTokens to User model

namespace App\Models;

use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens;
    // ...
}

// === routes/api.php ===
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\PostController;

Route::post('/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::post('/logout', [AuthController::class, 'logout']);
    Route::apiResource('posts', PostController::class);
    Route::get('/user', fn(\Illuminate\Http\Request $r) => $r->user());
});

// === Auth controller ===
namespace App\Http\Controllers\Api;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function login(Request $request)
    {
        $data = $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $data['email'])->first();

        if (!$user || !Hash::check($data['password'], $user->password)) {
            return response()->json(['message' => 'Invalid credentials'], 401);
        }

        $token = $user->createToken('mobile-app')->plainTextToken;

        return response()->json([
            'user' => $user,
            'token' => $token,
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();
        return response()->json(['message' => 'Logged out']);
    }
}

// === API Resources (control JSON output) ===
// php artisan make:resource PostResource

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
            'published_at' => $this->published_at?->toIso8601String(),
            'author' => new UserResource($this->whenLoaded('user')),
            'tags' => TagResource::collection($this->whenLoaded('tags')),
            'created_at' => $this->created_at->toIso8601String(),
        ];
    }
}

// === Controller using Resources ===
public function show(Post $post)
{
    $post->load(['user', 'tags']);
    return new PostResource($post);
}

public function index()
{
    $posts = Post::with(['user', 'tags'])->paginate(20);
    return PostResource::collection($posts);
}

// === Validation errors return JSON automatically ===
// POST /api/posts with missing title:
// {
//   "message": "The title field is required.",
//   "errors": {
//     "title": ["The title field is required."]
//   }
// }

// === Rate limiting ===
// In RouteServiceProvider or routes/api.php:
Route::middleware('throttle:60,1')->group(function () {
    Route::apiResource('posts', PostController::class);
});

// Custom rate limit in AppServiceProvider::boot():
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('api', function (Request $request) {
    return $request->user()
        ? Limit::perMinute(100)->by($request->user()->id)
        : Limit::perMinute(20)->by($request->ip());
});

// === Test the API ===
public function test_can_create_post(): void
{
    $user = User::factory()->create();
    $token = $user->createToken('test')->plainTextToken;

    $response = $this->withHeaders([
        'Authorization' => "Bearer {$token}",
        'Accept' => 'application/json',
    ])->postJson('/api/posts', [
        'title' => 'Test Post',
        'body' => 'Body content',
    ]);

    $response->assertStatus(201)
             ->assertJsonPath('data.title', 'Test Post');
}

// === Frontend usage (JavaScript fetch) ===
// const response = await fetch('/api/posts', {
//   method: 'POST',
//   headers: {
//     'Authorization': `Bearer ${token}`,
//     'Content-Type': 'application/json',
//     'Accept': 'application/json',
//   },
//   body: JSON.stringify({ title: 'New', body: 'Body' }),
// });
// const data = await response.json();
```

## Try it yourself

Build a `/api/posts` endpoint with Sanctum auth. Add `PostResource` that returns id, title, body, author name, and tags. Test it with `php artisan test`.

## Common pitfalls

- ⚠️ Not adding `Accept: application/json` header in tests/frontend. Laravel returns HTML 404 instead of JSON 404 without it.
- ⚠️ Returning models directly. Always use API Resources — they prevent accidental exposure of sensitive fields (passwords, tokens).
- ⚠️ No rate limiting. Without it, a malicious user can DOS your API in seconds. Use `throttle` middleware.

## What's next?

**Level 11: Laravel ecosystem: Forge, Vapor, Nova, and the wider world** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 10 · Cycle 3_