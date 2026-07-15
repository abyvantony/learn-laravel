# Level 11: Laravel ecosystem: Forge, Vapor, Nova, and the wider world

**Track:** Laravel · **Level:** 11/12 · **Cycle:** 3 · **Difficulty:** `ultra-advanced`

📚 **Today's lesson** — published 2026-07-15

## TL;DR

Laravel is more than a framework — it's an ecosystem. Forge deploys your apps. Vapor runs them serverlessly on AWS Lambda. Nova is an admin panel. Livewire lets you build reactive UIs without writing JS. The full stack has solutions for every problem.

## Real-world analogies

- Laravel is the kitchen. Forge is the restaurant chain that opens new locations for you. Vapor is the food truck that appears only when needed. Nova is the manager who sees everything happening in the kitchen.
- Livewire is the chef who can cook and serve at the same time — write PHP, get reactive UI. No JavaScript needed.

## Key concepts

### `Laravel Forge`

Server management panel. Provision servers on DigitalOcean/AWS/Linode, deploy from GitHub, manage SSL, queue workers, databases. Pay per server.

### `Laravel Vapor`

Serverless deployment on AWS Lambda. Auto-scales to zero, no server management. Pay per request.

### `Laravel Nova`

Admin panel. Generate CRUD interfaces for your Eloquent models. Beautiful, customizable, paid.

### `Laravel Livewire`

Build reactive UIs in pure PHP. No JavaScript framework needed. Server-rendered, AJAX-driven.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Laravel Livewire (reactive components in PHP) ===
// composer require livewire/livewire

// resources/views/livewire/counter.blade.php
// <div>
//     <h1>Count: {{ $count }}</h1>
//     <button wire:click="increment">+</button>
//     <button wire:click="decrement">-</button>
// </div>

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public int $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function decrement()
    {
        $this->count--;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}

// Use in any Blade view
// @livewire('counter')

// === Livewire with Eloquent ===
namespace App\Http\Livewire;

use Livewire\Component;
use Livewire\WithPagination;
use App\Models\Post;

class PostList extends Component
{
    use WithPagination;
    public string $search = '';

    public function render()
    {
        $posts = Post::where('title', 'like', "%{$this->search}%")
            ->paginate(10);

        return view('livewire.post-list', ['posts' => $posts]);
    }
}

// resources/views/livewire/post-list.blade.php
// <div>
//     <input wire:model.live="search" placeholder="Search...">
//     @foreach ($posts as $post)
//         <div>{{ $post->title }}</div>
//     @endforeach
//     {{ $posts->links() }}
// </div>

// === Laravel Forge (server management) ===
// - Push to GitHub → Forge deploys automatically
// - Free SSL via Let's Encrypt
// - Queue worker management
// - Database backups
// - Server monitoring

// Deployment script (Forge auto-generates):
// cd /home/forge/myapp.com
// git pull origin main
// composer install --no-dev --no-interaction --prefer-dist --optimize-autoloader
// php artisan migrate --force
// php artisan config:cache
// php artisan route:cache
// php artisan view:cache
// php artisan queue:restart

// === Laravel Vapor (serverless on AWS) ===
// vapor.yml
// id: 12345
// name: my-app
// environments:
//   production:
//     memory: 1024
//     cli-memory: 512
//     runtime: php-8.2
//     build:
//       - composer install --no-dev
//     deploy:
//       - php artisan migrate --force

// === Laravel Nova (admin panel) ===
// composer require laravel/nova
// php artisan nova:install
// php artisan migrate

// app/Nova/Post.php
namespace App\Nova;

use Laravel\Nova\Fields\ID;
use Laravel\Nova\Fields\Text;
use Laravel\Nova\Fields\Textarea;
use Laravel\Nova\Fields\BelongsTo;

class Post extends Resource
{
    public static $model = \App\Models\Post::class;

    public function fields(Request $request)
    {
        return [
            ID::make()->sortable(),
            Text::make('Title')->required(),
            Textarea::make('Body'),
            BelongsTo::make('Author', 'user', User::class),
        ];
    }
}

// Visit /nova in your browser — instant admin panel

// === Laravel Pint (code style fixer) ===
// vendor/bin/pint         — Fix all files
// vendor/bin/pint --test  — Check without fixing

// === Laravel Sail (Docker dev environment) ===
// ./vendor/bin/sail up
// ./vendor/bin/sail artisan migrate
// ./vendor/bin/sail npm install
```

## Try it yourself

Install Livewire. Build a `SearchUsers` component that has a search input and a live-updating list of users matching the query. No JavaScript required — just `wire:model.live`.

## Common pitfalls

- ⚠️ Not using the deployment script. Without `php artisan config:cache` and friends, your changes don't take effect.
- ⚠️ Forgetting to set `APP_ENV=production` and `APP_DEBUG=false` in production. Exposes sensitive error info to users.
- ⚠️ Using Nova without authorization. Lock down access with `Nova::auth(fn($request) => $request->user()->isAdmin)`.

## What's next?

**Level 12: Ultra-advanced: design patterns, package development, and the cutting edge** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 11 · Cycle 3_