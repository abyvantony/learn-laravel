# Level 3: Eloquent ORM: working with databases the elegant way

**Track:** Laravel · **Level:** 3/12 · **Cycle:** 2 · **Difficulty:** `beginner-plus`

📚 **Today's lesson** — published 2026-07-05

## TL;DR

Eloquent is Laravel's ORM (Object-Relational Mapper). Each database table maps to a PHP class (Model). You write `User::find(1)` instead of `SELECT * FROM users WHERE id = 1`. Cleaner, safer, and powerful.

## Real-world analogies

- Eloquent is a translator. You speak PHP (objects), the database speaks SQL. Eloquent translates between them seamlessly.
- A model is a smart representative for a database row. It knows how to save itself, find others, and even relate to other tables.

## Key concepts

### `Models`

PHP classes that represent database tables. `User` model → `users` table. Conventions: model is singular, table is plural.

### `Query builder`

Chainable methods: `User::where('active', true)->orderBy('name')->get()`. Translates to SQL behind the scenes.

### `Relationships`

Models relate to each other: `User hasMany Post`, `Post belongsTo User`. Eloquent handles the JOINs for you.

### `Migrations`

PHP files that define your database schema. Version-controlled. `php artisan migrate` applies them. `rollback` to undo.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Define a model ===
// php artisan make:model User

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    use HasFactory;

    // Mass-assignable fields (for User::create([...]))
    protected $fillable = ['name', 'email', 'password'];

    // Hidden fields (excluded from JSON)
    protected $hidden = ['password', 'remember_token'];

    // Casts (auto-convert types)
    protected $casts = [
        'email_verified_at' => 'datetime',
        'is_admin' => 'boolean',
    ];

    // Relationship: a user has many posts
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}

// === Define related model ===
class Post extends Model
{
    protected $fillable = ['title', 'body', 'user_id'];

    public function user(): \Illuminate\Database\Eloquent\Relations\BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function tags(): \Illuminate\Database\Eloquent\Relations\BelongsToMany
    {
        return $this->belongsToMany(Tag::class);
    }
}

// === CRUD operations ===
// Create
$user = User::create([
    'name' => 'Aby',
    'email' => 'aby@example.com',
    'password' => bcrypt('secret'),
]);

// Read
$user = User::find(1);                      // By ID
$user = User::findOrFail(1);                // 404 if not found
$user = User::where('email', 'aby@x.com')->first();
$users = User::all();                        // All users
$active = User::where('active', true)->get(); // Query
$count = User::where('active', true)->count();

// Update
$user = User::find(1);
$user->name = 'Aby Antony';
$user->save();

// Or mass update
User::where('active', false)->update(['banned' => true]);

// Delete
$user->delete();
User::destroy(1);                  // By ID
User::destroy([1, 2, 3]);          // Multiple

// === Query builder chaining ===
$posts = Post::query()
    ->where('published', true)
    ->where('views', '>', 100)
    ->orderByDesc('created_at')
    ->limit(10)
    ->get();

// Eager loading (avoids N+1 query problem)
$users = User::with('posts')->get();        // Loads users + their posts in 2 queries
foreach ($users as $user) {
    echo $user->posts->count();  // No extra query per user
}

// === Relationships in action ===
$user = User::find(1);
$user->posts;                    // All posts by this user (lazy load)
$user->posts()->create([...]);   // Create a post for this user

$post = Post::find(1);
$post->user;                     // The user who wrote this post
$post->user->name;

// === Migrations ===
// php artisan make:migration create_users_table
// database/migrations/2026_06_27_000000_create_users_table.php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->boolean('is_admin')->default(false);
            $table->timestamps();  // created_at, updated_at
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};

// Common column types: string, text, integer, bigInteger, decimal, boolean,
// date, dateTime, timestamp, json, foreignId, morphs
// Modifiers: ->nullable(), ->default($value), ->unique(), ->index()

// Migration commands:
// php artisan migrate              — Run all pending migrations
// php artisan migrate:rollback     — Roll back the last batch
// php artisan migrate:fresh        — Drop everything and re-migrate (dev only!)
// php artisan migrate:status       — See what's been run
```

## Try it yourself

Create `User` and `Post` models. Define the `hasMany` / `belongsTo` relationship. Write a controller method that returns a user with all their posts, using eager loading to avoid N+1 queries.

## Common pitfalls

- ⚠️ N+1 query problem: looping over users and accessing `$user->posts` runs a new query each time. Use `User::with('posts')` to fix.
- ⚠️ Forgetting `$fillable` causes mass-assignment errors. List every field that's safe to mass-assign.
- ⚠️ Not running migrations. New tables don't exist in the DB until you `php artisan migrate`.

## What's next?

**Level 4: Blade templating: building reusable views** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 3 · Cycle 2_