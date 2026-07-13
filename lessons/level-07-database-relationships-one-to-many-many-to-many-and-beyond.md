# Level 7: Database relationships: one-to-many, many-to-many, and beyond

**Track:** Laravel · **Level:** 7/12 · **Cycle:** 3 · **Difficulty:** `intermediate-plus`

📚 **Today's lesson** — published 2026-07-13

## TL;DR

Real apps have data that's connected: users have posts, posts have tags, students enroll in courses. Eloquent's relationships make these connections effortless — and fast, if you use eager loading.

## Real-world analogies

- A `hasMany` is a parent-children relationship. A user has many posts. Each post belongs to one user.
- A `belongsToMany` is a friendship — two people connected by a 'friendship' table. Neither owns the other; both are equal.
- A `hasOne` is marriage — one partner belongs to another. Each user has one profile.

## Key concepts

### `Eager loading`

`User::with('posts')->get()` loads users AND their posts in 2 queries. Without it, you get N+1 queries (1 for users, N for posts).

### `Polymorphic`

Comments can belong to posts OR videos OR photos. One table, multiple parent types. `morphTo()` / `morphMany()`.

### `Pivot tables`

Many-to-many needs a join table. `post_tag` connects `posts` and `tags`. Stores the relationship.

### `Querying relationships`

`User::whereHas('posts', fn($q) => $q->where('views', '>', 100))` — users who have popular posts.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === One to Many ===
class User extends Model
{
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}

class Post extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
        // Foreign key defaults to 'user_id' on posts table
    }
}

// Usage
$user = User::find(1);
$user->posts;                              // Collection of posts
$user->posts()->where('published', true)->get();  // Query
$user->posts()->create(['title' => 'New']);  // Create for this user

$post = Post::find(1);
$post->user;                               // The author
$post->user->name;

// === Many to Many ===
class Post extends Model
{
    public function tags(): BelongsToMany
    {
        return $this->belongsToMany(Tag::class)
            ->withTimestamps()       // adds created_at, updated_at
            ->withPivot('weight');   // extra column on pivot
    }
}

// Migration for pivot table:
// Schema::create('post_tag', function (Blueprint $table) {
//     $table->id();
//     $table->foreignId('post_id')->constrained();
//     $table->foreignId('tag_id')->constrained();
//     $table->integer('weight')->default(0);
//     $table->timestamps();
// });

// Usage
$post = Post::find(1);
$post->tags;                              // All tags for this post
$post->tags()->attach($tagId);            // Add a tag
$post->tags()->detach($tagId);            // Remove a tag
$post->tags()->sync([1, 2, 3]);           // Replace all tags
$post->tags()->syncWithoutDetaching([4]); // Add without removing

$tag = Tag::find(1);
$tag->posts;                              // All posts with this tag

// === One to One ===
class User extends Model
{
    public function profile(): HasOne
    {
        return $this->hasOne(Profile::class);
    }
}

class Profile extends Model
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}

// Usage
$user = User::find(1);
$user->profile;            // Single profile object
$user->profile()->create([...]);  // Create for this user

// === Has Many Through ===
class Country extends Model
{
    public function posts(): HasManyThrough
    {
        return $this->hasManyThrough(Post::class, User::class);
    }
}
// $country->posts — all posts by users in this country

// === Polymorphic ===
class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

class Video extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

// Migration: comments table has commentable_id and commentable_type
// Schema::create('comments', function (Blueprint $table) {
//     $table->id();
//     $table->text('body');
//     $table->morphs('commentable');  // commentable_id, commentable_type
//     $table->timestamps();
// });

// Usage
$post->comments;          // All comments on this post
$video->comments;         // All comments on this video
$comment->commentable;    // The post OR video the comment belongs to

// === Querying relationships ===
$users = User::has('posts')->get();  // Users with at least one post
$users = User::has('posts', '>=', 5)->get();  // 5+ posts
$users = User::whereHas('posts', fn($q) => $q->where('views', '>', 100))->get();
$users = User::withCount('posts')->orderBy('posts_count', 'desc')->get();

// Eager loading with constraints
$users = User::with(['posts' => fn($q) => $q->where('published', true)])->get();

// Nested eager loading
$users = User::with('posts.comments')->get();
// Loads users → their posts → each post's comments

// Lazy eager loading (when you already have the models)
$users = User::all();
$users->load('posts');
```

## Try it yourself

Build a blog with `User`, `Post`, `Tag`, and `Comment` models. Set up `User hasMany Post`, `Post belongsToMany Tag`, `Post morphMany Comment`. Use eager loading when displaying a post with its tags and comments.

## Common pitfalls

- ⚠️ N+1 queries: `User::all()` then `$user->posts` in a loop = 1 + N queries. Use `User::with('posts')`.
- ⚠️ Forgetting `withTimestamps()` on pivot tables. Without it, you lose pivot table timestamps.
- ⚠️ Mixing up `belongsTo` and `hasMany` direction. The model that has the foreign key uses `belongsTo`.

## What's next?

**Level 8: Queues, jobs, and async processing** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 7 · Cycle 3_