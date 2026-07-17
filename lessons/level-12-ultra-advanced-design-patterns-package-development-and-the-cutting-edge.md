# Level 12: Ultra-advanced: design patterns, package development, and the cutting edge

**Track:** Laravel · **Level:** 12/12 · **Cycle:** 3 · **Difficulty:** `ultra-advanced`

📚 **Today's lesson** — published 2026-07-17

## TL;DR

You've got the foundations. Now: design patterns in Laravel (Repository, Service, Strategy), writing your own Composer packages, Octane for high-performance apps, and the modern Laravel stack (Laravel 11+ simplified structure, Reverb for WebSockets).

## Real-world analogies

- A Repository is a librarian. You ask for a book, they get it for you. The librarian knows where the books are, but you don't have to.
- A Service is a specialist contractor. The controller is the project manager, but for complex jobs (payments, reports), they bring in a specialist.
- Octane is a sports car engine. Swap it in and your app goes 10x faster — but you have to know how to drive it.

## Key concepts

### `Repository pattern`

Hide database access behind a clean interface. Swap implementations (MySQL → MongoDB) without changing controllers.

### `Service pattern`

Complex business logic lives in a service class. Controllers stay thin (HTTP concerns only).

### `Laravel Octane`

Runs Laravel on Swoole/RoadRunner. Keeps the app in memory between requests. 10x faster, but stateless-aware code required.

### `Package development`

Build reusable Composer packages. Useful for company-internal code or contributing to the community.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Repository pattern ===

namespace App\Contracts;

interface PostRepository
{
    public function find(int $id): ?Post;
    public function create(array $data): Post;
    public function update(Post $post, array $data): Post;
    public function delete(Post $post): bool;
    public function published(): Collection;
}

namespace App\Repositories;

use App\Models\Post;
use App\Contracts\PostRepository;
use Illuminate\Support\Collection;

class EloquentPostRepository implements PostRepository
{
    public function find(int $id): ?Post
    {
        return Post::find($id);
    }

    public function create(array $data): Post
    {
        return Post::create($data);
    }

    public function update(Post $post, array $data): Post
    {
        $post->update($data);
        return $post;
    }

    public function delete(Post $post): bool
    {
        return $post->delete();
    }

    public function published(): Collection
    {
        return Post::whereNotNull('published_at')
            ->with('user', 'tags')
            ->latest()
            ->get();
    }
}

// Bind in a Service Provider
$this->app->bind(PostRepository::class, EloquentPostRepository::class);

// Use in a controller
class PostController
{
    public function __construct(private PostRepository $posts) {}

    public function show(int $id)
    {
        $post = $this->posts->find($id);
        return new PostResource($post);
    }
}

// === Service pattern ===

namespace App\Services;

use App\Models\Order;
use App\Models\Payment;

class CheckoutService
{
    public function __construct(
        private PaymentGateway $payments,
        private InventoryService $inventory,
        private NotificationService $notifications
    ) {}

    public function checkout(User $user, array $items): Order
    {
        $order = Order::create(['user_id' => $user->id]);

        $total = $this->inventory->reserve($items);

        $this->payments->charge($user, $total);

        $this->notifications->send($user, "Order #{$order->id} confirmed!");

        return $order;
    }
}

// Controller stays thin
class CheckoutController
{
    public function __construct(private CheckoutService $checkout) {}

    public function store(CheckoutRequest $request)
    {
        $order = $this->checkout->checkout(
            $request->user(),
            $request->input('items')
        );
        return new OrderResource($order);
    }
}

// === Laravel Octane (high-performance) ===
// composer require laravel/octane
// php artisan octane:install
// php artisan octane:start

// Run with Swoole
// php artisan octane:start --server=swoole --workers=4

// Gotcha: Octane keeps app in memory. Don't use:
// - Static state that mutates
// - Service container singletons with request-specific data
// - Global variables

// Do use:
// - Database connections (Octane manages pooling)
// - Caching
// - Stateless services

// === Package development ===

// composer.json
{
    "name": "vendor/my-package",
    "description": "A reusable Laravel package",
    "type": "library",
    "require": {
        "php": "^8.1",
        "illuminate/support": "^10.0|^11.0"
    },
    "autoload": {
        "psr-4": {
            "Vendor\\MyPackage\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": [
                "Vendor\\MyPackage\\MyPackageServiceProvider"
            ]
        }
    }
}

// src/MyPackageServiceProvider.php
namespace Vendor\MyPackage;

use Illuminate\Support\ServiceProvider;

class MyPackageServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->mergeConfigFrom(__DIR__ . '/../config/my-package.php', 'my-package');
    }

    public function boot(): void
    {
        $this->loadRoutesFrom(__DIR__ . '/../routes/web.php');
        $this->loadViewsFrom(__DIR__ . '/../resources/views', 'my-package');
        $this->loadMigrationsFrom(__DIR__ . '/../database/migrations');

        if ($this->app->runningInConsole()) {
            $this->publishes([
                __DIR__ . '/../config/my-package.php' => config_path('my-package.php'),
            ], 'config');
        }
    }
}

// === Reading list ===
// - Laravel Docs (laravel.com/docs) — best in the business
// - "Laravel Up & Running" by Matt Stauffer — book
// - Laravel News (laravel-news.com) — daily updates
// - Laracasts (laracasts.com) — video tutorials
// - Spatie (spatie.be) — high-quality Laravel packages
```

## Try it yourself

Refactor a small Laravel app to use the Service pattern. Move business logic from your controllers into a service class. Inject it via the constructor. Notice how much cleaner the controller becomes.

## Common pitfalls

- ⚠️ Over-applying patterns. Don't add a Repository for a 2-method controller. Use patterns when they solve a real problem.
- ⚠️ Octane without understanding its constraints. Static state and singletons break in Octane. Read the docs carefully.
- ⚠️ Making packages too small. The overhead of a Composer package isn't worth it for a single class. Use it for genuinely reusable code.

## What's next?

🎉 You've completed the track! Time to build something real.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 12 · Cycle 3_