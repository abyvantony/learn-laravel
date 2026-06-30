# Level 9: Testing: PHPUnit, Feature tests, and TDD

**Track:** Laravel · **Level:** 9/12 · **Difficulty:** `advanced`

📚 **Today's lesson** — published 2026-06-30

## TL;DR

Tests prove your code works. Laravel ships with PHPUnit set up out of the box. Feature tests check end-to-end behavior; Unit tests check small pieces. The `RefreshDatabase` trait resets your DB between tests.

## Real-world analogies

- Tests are a quality control checklist. Each one says 'this thing should work this way'. After every change, run the checklist. If anything fails, you broke something.
- Feature tests are like a customer trying your app end-to-end. Unit tests are like a mechanic checking individual parts.

## Key concepts

### `PHPUnit`

PHP's standard testing framework. Laravel ships with it configured. `php artisan test` runs all tests.

### `Feature vs Unit`

Feature tests hit routes end-to-end (slower, catch integration bugs). Unit tests check a single function (fast, catch logic bugs).

### `RefreshDatabase`

Trait that migrates the DB before each test, rolls back after. Tests don't pollute each other.

### `Factories`

Generate fake models for tests. `User::factory()->create()` makes a user with random data.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Test directory structure ===
// tests/
//   Feature/        — End-to-end tests
//     UserTest.php
//   Unit/           — Small, isolated tests
//     HelpersTest.php
//   TestCase.php    — Base class

// === A feature test ===
// php artisan make:test UserTest

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class UserTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_register(): void
    {
        $response = $this->post('/register', [
            'name' => 'Aby',
            'email' => 'aby@example.com',
            'password' => 'password',
            'password_confirmation' => 'password',
        ]);

        $response->assertRedirect('/dashboard');
        $this->assertDatabaseHas('users', ['email' => 'aby@example.com']);
        $this->assertAuthenticated();
    }

    public function test_email_is_required(): void
    {
        $response = $this->post('/register', [
            'name' => 'Aby',
            'email' => '',
            'password' => 'password',
            'password_confirmation' => 'password',
        ]);

        $response->assertSessionHasErrors('email');
    }

    public function test_guest_cannot_view_users(): void
    {
        $response = $this->get('/users');
        $response->assertRedirect('/login');
    }

    public function test_user_can_only_edit_own_posts(): void
    {
        $user = User::factory()->create();
        $otherUser = User::factory()->create();
        $post = Post::factory()->for($otherUser)->create();

        $response = $this->actingAs($user)->put("/posts/{$post->id}", [
            'title' => 'Hacked',
        ]);

        $response->assertStatus(403);
    }
}

// === A unit test ===
namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class HelpersTest extends TestCase
{
    public function test_format_currency(): void
    {
        $this->assertSame('$10.00', formatCurrency(10));
        $this->assertSame('$1,234.56', formatCurrency(1234.56));
    }
}

// === Factories ===
// database/factories/UserFactory.php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected $model = User::class;

    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => bcrypt('password'),
            'remember_token' => Str::random(10),
        ];
    }

    public function unverified(): static
    {
        return $this->state(fn() => ['email_verified_at' => null]);
    }

    public function admin(): static
    {
        return $this->state(fn() => ['is_admin' => true]);
    }
}

// === Using factories in tests ===
$user = User::factory()->create();           // Save to DB
$user = User::factory()->make();            // Don't save
$users = User::factory()->count(10)->create();  // 10 users
$admin = User::factory()->admin()->create();  // Apply admin state

// === Common assertions ===
$response->assertOk();                    // 200
$response->assertStatus(403);
$response->assertJson(['name' => 'Aby']);
$response->assertJsonPath('user.email', 'aby@x.com');
$response->assertRedirect('/home');
$response->assertSee('Welcome');
$response->assertDontSee('Error');

$this->assertDatabaseHas('users', ['email' => 'aby@x.com']);
$this->assertDatabaseMissing('users', ['email' => 'spam@x.com']);
$this->assertDatabaseCount('users', 5);

$this->assertAuthenticated();
$this->assertGuest();

$this->assertSame($expected, $actual);
$this->assertTrue($condition);
$this->assertCount(3, $collection);

// === Running tests ===
// php artisan test                  — Run all
// php artisan test --filter=UserTest  — Run one file
// php artisan test --testsuite=Unit  — Run unit tests only
// php artisan test --coverage        — Generate coverage report
```

## Try it yourself

Write a feature test for the `PostController` that verifies: (1) authenticated users can create posts, (2) validation errors are shown for missing title, (3) only the post author can edit it. Use factories for test data.

## Common pitfalls

- ⚠️ Tests that depend on each other. Each test should set up its own data. Use `RefreshDatabase` + factories.
- ⚠️ Not testing the unhappy path. Always test: missing data, wrong data, unauthorized access, edge cases.
- ⚠️ Slow tests because they hit external APIs. Mock external services with `Http::fake()`.

## What's next?

**Level 10: API development: RESTful APIs with Sanctum and resources** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 9_