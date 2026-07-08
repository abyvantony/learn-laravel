# Level 8: Queues, jobs, and async processing

**Track:** Laravel · **Level:** 8/12 · **Cycle:** 2 · **Difficulty:** `advanced`

📚 **Today's lesson** — published 2026-07-08

## TL;DR

Some tasks take too long to do in a request: send email, generate PDF, process video, call slow APIs. Laravel queues let you defer these to a background worker. The user gets an instant response; the heavy lifting happens later.

## Real-world analogies

- A queue is a coat check. You drop off your coat (job) and get a ticket. The clerk (worker) takes care of it in the background. You don't wait.
- A synchronous request is cooking a meal for the customer while they wait. A queued job is giving them the menu and cooking the meal after they've gone home.

## Key concepts

### `Jobs`

PHP classes that do a unit of work. Implement `ShouldQueue` to make them async. Dispatch with `Job::dispatch($data)`.

### `Queue drivers`

Database (default, simple), Redis (fast, recommended for production), SQS (AWS), Beanstalkd. Configured in `config/queue.php`.

### `Workers`

`php artisan queue:work` runs in the background, picks up jobs, processes them. Supervisord keeps it running.

### `Failed jobs`

When a job fails repeatedly, it goes to the `failed_jobs` table. Inspect, retry, or delete with `php artisan queue:failed`.

## Code with comments

Every line has a comment. Read it slowly.

```
<?php
// === Create a job ===
// php artisan make:job SendWelcomeEmail

namespace App\Jobs;

use App\Models\User;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Mail;

class SendWelcomeEmail implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public int $tries = 3;             // Retry up to 3 times
    public int $backoff = 60;          // Wait 60s between retries
    public int $timeout = 120;         // Kill job after 2 min

    public function __construct(public User $user) {}

    public function handle(): void
    {
        Mail::to($this->user)->send(new WelcomeEmail($this->user));
    }

    public function failed(\Throwable $exception): void
    {
        // Called when all retries exhausted
        \Log::error("Failed to send welcome email to {$this->user->id}");
    }
}

// === Dispatch the job (don't run it now, queue it) ===
use App\Jobs\SendWelcomeEmail;

public function store(StoreUserRequest $request)
{
    $user = User::create($request->validated());
    SendWelcomeEmail::dispatch($user);  // Returns immediately
    return redirect()->route('users.show', $user);
}

// === Sync dispatch (for testing) ===
SendWelcomeEmail::dispatchSync($user);

// === Delay a job ===
SendWelcomeEmail::dispatch($user)->delay(now()->addMinutes(10));

// === Queue on a specific queue ===
SendWelcomeEmail::dispatch($user)->onQueue('emails');

// === Job chaining (run jobs in sequence) ===
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();

// === Queue configuration (config/queue.php) ===
// Default: 'sync' (no queue, runs immediately — good for testing)
// In production: set QUEUE_CONNECTION=redis in .env

// .env
// QUEUE_CONNECTION=redis
// REDIS_HOST=127.0.0.1

// === Running workers ===
// php artisan queue:work              — Start a worker (one process)
// php artisan queue:work --queue=high,default
// php artisan queue:listen            — Auto-reload code changes (dev)
// php artisan queue:restart           — Graceful restart after deploy
// php artisan queue:failed            — List failed jobs
// php artisan queue:retry all         — Retry all failed jobs
// php artisan queue:flush             — Delete all failed jobs

// === In production: use Supervisor to keep workers running ===
// /etc/supervisor/conf.d/laravel-worker.conf
// [program:laravel-worker]
// process_name=%(program_name)s_%(process_num)02d
// command=php /var/www/artisan queue:work --sleep=3 --tries=3
// autostart=true
// autorestart=true
// user=www-data
// numprocs=4
// redirect_stderr=true
// stdout_logfile=/var/www/storage/logs/worker.log

// === Common patterns ===
// 1. Send email: Mail::to($user)->send() in a job
// 2. Generate PDF: Snappy / DomPDF in a job
// 3. Process upload: Image intervention in a job
// 4. Call slow API: Http::get() in a job
// 5. Database cleanup: delete old records in a scheduled job
```

## Try it yourself

Create a `SendWelcomeEmail` job. Configure Redis as the queue driver. In your `UserController::store`, dispatch the job. Run `php artisan queue:work` in another terminal and create a user. Watch the worker process the job.

## Common pitfalls

- ⚠️ Passing Eloquent models directly: `$user` works, but if the model changes between dispatch and processing, you get stale data. Use IDs + re-fetch in the job.
- ⚠️ Forgetting to run `queue:work` in production. Jobs are queued but never processed. Use Supervisor.
- ⚠️ Long-running jobs without timeouts. A stuck job blocks the worker. Set `$timeout` and `$tries`.

## What's next?

**Level 9: Testing: PHPUnit, Feature tests, and TDD** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 8 · Cycle 2_