# Level 4: Blade templating: building reusable views

**Track:** Laravel · **Level:** 4/12 · **Cycle:** 3 · **Difficulty:** `intermediate`

📚 **Today's lesson** — published 2026-07-12

## TL;DR

Blade is Laravel's templating engine. It looks like HTML with sprinkle of `{{ }}` and `@directive`. Compiles to plain PHP. Lets you build layouts, components, and partials without writing a single line of PHP.

## Real-world analogies

- Blade is HTML with superpowers. You can use variables, loops, conditionals — all without leaving your editor or context-switching to PHP.
- A Blade layout is a picture frame. The frame stays the same (header, footer, nav). The picture inside (the page content) changes.

## Key concepts

### `Layouts and sections`

Define a master layout with `@yield('content')`. Each page extends it and fills the section with `@section('content')`.

### `Components`

Reusable UI pieces. `<x-button>Click</x-button>` renders a button. Define once, use everywhere.

### `Loops and conditionals`

`@foreach`, `@if`, `@unless`. Familiar PHP syntax, but cleaner.

### `Includes and partials`

`@include('partials.header')` includes a sub-view. Use for repeated bits like navs, footers, error messages.

## Code with comments

Every line has a comment. Read it slowly.

```
{{-- resources/views/layouts/app.blade.php --}}
<!DOCTYPE html>
<html>
<head>
    <title>@yield('title', 'My App')</title>
    <link rel="stylesheet" href="/css/app.css">
    @stack('styles')
</head>
<body>
    @include('partials.nav')

    <main>
        @yield('content')
    </main>

    @include('partials.footer')

    <script src="/js/app.js"></script>
    @stack('scripts')
</body>
</html>

{{-- resources/views/users/index.blade.php --}}
@extends('layouts.app')

@section('title', 'All Users')

@section('content')
    <h1>All Users</h1>

    @if (session('success'))
        <div class="alert alert-success">{{ session('success') }}</div>
    @endif

    @if ($users->isEmpty())
        <p>No users yet.</p>
    @else
        <ul>
            @foreach ($users as $user)
                <li>
                    <a href="{{ route('users.show', $user) }}">
                        {{ $user->name }} — {{ $user->email }}
                    </a>
                </li>
            @endforeach
        </ul>

        {{ $users->links() }}  {{-- Pagination --}}
    @endif
@endsection

{{-- resources/views/components/button.blade.php --}}
@props(['type' => 'button', 'color' => 'blue'])

<button type="{{ $type }}" class="btn btn-{{ $color }}">
    {{ $slot }}
</button>

{{-- Use the component --}}
<x-button color="red" type="submit">Delete</x-button>
<x-button>Save</x-button>

{{-- === Blade directives === --}}

{{-- Conditionals --}}
@if ($user->isAdmin())
    <p>Admin tools</p>
@elseif ($user->isModerator())
    <p>Mod tools</p>
@else
    <p>Welcome, {{ $user->name }}</p>
@endif

@unless ($user->hasVerifiedEmail())
    Please verify your email.
@endunless

@auth
    <p>Logged in as {{ auth()->user()->name }}</p>
@endauth

@guest
    <a href="/login">Log in</a>
@endguest

{{-- Loops --}}
@foreach ($items as $item)
    <p>{{ $item->name }}</p>
@empty
    <p>No items.</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <li>No users.</li>
@endforelse

@for ($i = 0; $i < 10; $i++)
    <p>{{ $i }}</p>
@endfor

@while ($condition)
    <p>Looping...</p>
@endwhile

{{-- Loop variable --}}
@foreach ($users as $user)
    @if ($loop->first)
        <p>First user!</p>
    @endif
    {{ $loop->iteration }}. {{ $user->name }}
    @if ($loop->last)
        <p>Last user!</p>
    @endif
@endforeach

{{-- CSRF protection --}}
<form method="POST" action="/users">
    @csrf  {{-- Required for POST/PUT/DELETE --}}
    <input name="name">
    <button>Save</button>
</form>

{{-- Error display --}}
@error('email')
    <p class="error">{{ $message }}</p>
@enderror

{{-- Including sub-views --}}
@include('partials.user-card', ['user' => $user])

{{-- Stack for assets --}}
@push('scripts')
    <script src="/js/users.js"></script>
@endpush

{{-- === Blade components: class-based === --}}
// php artisan make:component Alert

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(
        public string $type = 'info',
        public string $message = ''
    ) {}

    public function render()
    {
        return view('components.alert');
    }
}

{{-- resources/views/components/alert.blade.php --}}
<div class="alert alert-{{ $type }}">
    {{ $message ?: $slot }}
</div>

{{-- Use --}}
<x-alert type="success" message="User created!" />
<x-alert type="error">Something went wrong.</x-alert>
```

## Try it yourself

Create a layout (`layouts.app`) with header, nav, footer, and content area. Create a page (`posts.show`) that extends it and shows a single post with author, date, and body. Add a 'Back to all posts' link.

## Common pitfalls

- ⚠️ Forgetting `@csrf` in forms. POST/PUT/DELETE requests need it or Laravel rejects them with 419.
- ⚠️ Using `{!! $user->bio !!}` (unescaped) for user input. Always use `{{ }}` to prevent XSS.
- ⚠️ Putting too much logic in views. If a `@foreach` is 30 lines, move it to a partial or component.

## What's next?

**Level 5: Form validation and request handling** — coming in the next drop.

---

_Generated by Hermes · Aby's learning cron · Track: Laravel · Level 4 · Cycle 3_