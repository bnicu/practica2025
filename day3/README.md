# Day 3: Seeders, Frontend Controllers & Basic Styling

## Duration: 1-2 hours

### Objectives
- Create database seeders for blog posts and comments
- Implement frontend controllers for displaying posts and comments
- Add basic CSS styling using Tailwind CSS
- Create responsive layouts for the blog

---

## Part 1: Database Seeders (30 minutes)

### 1. Create Factories
```bash
php artisan make:factory PostFactory
php artisan make:factory CommentFactory
```

**database/factories/PostFactory.php**
```php
<?php

namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Str;

class PostFactory extends Factory
{
    public function definition(): array
    {
        $title = fake()->sentence(6, true);

        return [
            'title' => $title,
            'slug' => Str::slug($title),
            'excerpt' => fake()->paragraph(2),
            'content' => fake()->paragraphs(8, true),
            'featured_image' => 'https://picsum.photos/800/400?random=' . fake()->numberBetween(1, 1000),
            'is_published' => fake()->boolean(80),
            'published_at' => fake()->optional(0.8)->dateTimeBetween('-1 year', 'now'),
            'user_id' => User::factory(),
            'created_at' => fake()->dateTimeBetween('-1 year', 'now'),
        ];
    }

    public function published(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_published' => true,
            'published_at' => fake()->dateTimeBetween('-6 months', 'now'),
        ]);
    }

    public function draft(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_published' => false,
            'published_at' => null,
        ]);
    }
}
```

**database/factories/CommentFactory.php**
```php
<?php

namespace Database\Factories;

use App\Models\Post;
use App\Models\User;
use App\Models\Comment;
use Illuminate\Database\Eloquent\Factories\Factory;

class CommentFactory extends Factory
{
    public function definition(): array
    {
        return [
            'content' => fake()->paragraph(2),
            'is_approved' => fake()->boolean(90),
            'post_id' => Post::factory(),
            'user_id' => User::factory(),
            'parent_id' => null,
            'created_at' => fake()->dateTimeBetween('-6 months', 'now'),
        ];
    }

    public function approved(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_approved' => true,
        ]);
    }

    public function pending(): static
    {
        return $this->state(fn (array $attributes) => [
            'is_approved' => false,
        ]);
    }

    public function reply(): static
    {
        return $this->state(fn (array $attributes) => [
            'parent_id' => Comment::factory(),
        ]);
    }
}
```

### 2. Create Seeders
```bash
php artisan make:seeder UserSeeder
php artisan make:seeder PostSeeder
php artisan make:seeder CommentSeeder
```

**database/seeders/UserSeeder.php**
```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class UserSeeder extends Seeder
{
    public function run(): void
    {
        // Create admin user
        User::create([
            'name' => 'Admin User',
            'email' => 'admin@example.com',
            'password' => Hash::make('password'),
            'is_admin' => true,
            'email_verified_at' => now(),
        ]);

        // Create regular users
        User::factory(10)->create();

        // Create specific test user
        User::factory()->create([
            'name' => 'John Doe',
            'email' => 'john@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

**database/seeders/PostSeeder.php**
```php
<?php

namespace Database\Seeders;

use App\Models\Post;
use App\Models\User;
use Illuminate\Database\Seeder;

class PostSeeder extends Seeder
{
    public function run(): void
    {
        $users = User::all();

        // Create published posts
        Post::factory(15)
            ->published()
            ->recycle($users)
            ->create();

        // Create draft posts
        Post::factory(5)
            ->draft()
            ->recycle($users)
            ->create();

        // Create specific featured post
        Post::create([
            'title' => 'Welcome to Our Laravel Blog Tutorial',
            'slug' => 'welcome-to-our-laravel-blog-tutorial',
            'excerpt' => 'This is the first post in our Laravel blog tutorial series. Learn how to build a complete blog application with Laravel and Livewire.',
            'content' => '
                <h2>Welcome to the Laravel Blog Tutorial</h2>
                <p>This tutorial series will guide you through building a complete blog application using Laravel 11 and Livewire 3.</p>

                <h3>What You\'ll Learn</h3>
                <ul>
                    <li>Setting up Laravel with Docker</li>
                    <li>Implementing authentication with Sanctum</li>
                    <li>Creating database relationships</li>
                    <li>Building interactive components with Livewire</li>
                    <li>Styling with Tailwind CSS</li>
                </ul>

                <h3>Prerequisites</h3>
                <p>Basic knowledge of PHP and Laravel is recommended but not required. We\'ll guide you through each step.</p>

                <h3>Getting Started</h3>
                <p>Follow along with the daily tutorials and build your own blog application. Each day builds upon the previous lessons.</p>

                <p>Happy coding! ðŸš€</p>
            ',
            'featured_image' => 'https://picsum.photos/800/400?random=1',
            'is_published' => true,
            'published_at' => now(),
            'user_id' => User::where('email', 'admin@example.com')->first()->id,
        ]);
    }
}
```

**database/seeders/CommentSeeder.php**
```php
<?php

namespace Database\Seeders;

use App\Models\Comment;
use App\Models\Post;
use App\Models\User;
use Illuminate\Database\Seeder;

class CommentSeeder extends Seeder
{
    public function run(): void
    {
        $posts = Post::where('is_published', true)->get();
        $users = User::all();

        foreach ($posts as $post) {
            // Create parent comments
            $parentComments = Comment::factory(rand(2, 5))
                ->approved()
                ->create([
                    'post_id' => $post->id,
                    'user_id' => $users->random()->id,
                ]);

            // Create replies to some parent comments
            foreach ($parentComments as $parentComment) {
                if (rand(0, 100) < 40) { // 40% chance of having replies
                    Comment::factory(rand(1, 3))
                        ->approved()
                        ->create([
                            'post_id' => $post->id,
                            'user_id' => $users->random()->id,
                            'parent_id' => $parentComment->id,
                        ]);
                }
            }

            // Create some pending comments
            Comment::factory(rand(0, 2))
                ->pending()
                ->create([
                    'post_id' => $post->id,
                    'user_id' => $users->random()->id,
                ]);
        }
    }
}
```

**database/seeders/DatabaseSeeder.php**
```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }
}
```

### 3. Run Seeders
```bash
php artisan migrate:fresh --seed
```

---

## Part 2: Frontend Controllers (30 minutes)

### 1. Update PostController
**app/Http/Controllers/PostController.php**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with('user')
            ->published()
            ->latest('published_at')
            ->paginate(6);

        return view('posts.index', compact('posts'));
    }

    public function show(Post $post)
    {
        if (!$post->is_published && (!auth()->check() || !auth()->user()->is_admin)) {
            abort(404);
        }

        $post->load([
            'user',
            'approvedComments' => function ($query) {
                $query->with('user', 'approvedReplies.user')
                      ->whereNull('parent_id')
                      ->latest();
            }
        ]);

        return view('posts.show', compact('post'));
    }
}
```

### 2. Update CommentController
**app/Http/Controllers/CommentController.php**
```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Http\Request;

class CommentController extends Controller
{
    public function store(Request $request, Post $post)
    {
        $request->validate([
            'content' => 'required|string|max:1000',
            'parent_id' => 'nullable|exists:comments,id',
        ]);

        $comment = $post->comments()->create([
            'content' => $request->content,
            'user_id' => auth()->id(),
            'parent_id' => $request->parent_id,
            'is_approved' => false, // Require approval
        ]);

        return back()->with('success', 'Comment submitted successfully! It will be visible after approval.');
    }

    public function destroy(Comment $comment)
    {
        if ($comment->user_id !== auth()->id() && !auth()->user()->is_admin) {
            abort(403);
        }

        $comment->delete();

        return back()->with('success', 'Comment deleted successfully.');
    }
}
```

### 3. Create Admin Controllers
**app/Http/Controllers/Admin/PostController.php**
```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Post;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

class PostController extends Controller
{
    public function index()
    {
        $posts = Post::with('user')
            ->latest()
            ->paginate(10);

        return view('admin.posts.index', compact('posts'));
    }

    public function create()
    {
        return view('admin.posts.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'slug' => 'nullable|string|max:255|unique:posts,slug',
            'excerpt' => 'nullable|string',
            'content' => 'required|string',
            'featured_image' => 'nullable|url',
            'is_published' => 'boolean',
        ]);

        if (empty($validated['slug'])) {
            $validated['slug'] = Str::slug($validated['title']);
        }

        if ($validated['is_published'] ?? false) {
            $validated['published_at'] = now();
        }

        $validated['user_id'] = auth()->id();

        Post::create($validated);

        return redirect()->route('admin.posts.index')->with('success', 'Post created successfully.');
    }

    public function show(Post $post)
    {
        $post->load('user', 'comments.user');
        return view('admin.posts.show', compact('post'));
    }

    public function edit(Post $post)
    {
        return view('admin.posts.edit', compact('post'));
    }

    public function update(Request $request, Post $post)
    {
        $validated = $request->validate([
            'title' => 'required|string|max:255',
            'slug' => 'nullable|string|max:255|unique:posts,slug,' . $post->id,
            'excerpt' => 'nullable|string',
            'content' => 'required|string',
            'featured_image' => 'nullable|url',
            'is_published' => 'boolean',
        ]);

        if (empty($validated['slug'])) {
            $validated['slug'] = Str::slug($validated['title']);
        }

        if (($validated['is_published'] ?? false) && !$post->published_at) {
            $validated['published_at'] = now();
        } elseif (!($validated['is_published'] ?? false)) {
            $validated['published_at'] = null;
        }

        $post->update($validated);

        return redirect()->route('admin.posts.index')->with('success', 'Post updated successfully.');
    }

    public function destroy(Post $post)
    {
        $post->delete();
        return redirect()->route('admin.posts.index')->with('success', 'Post deleted successfully.');
    }
}
```

**app/Http/Controllers/Admin/CommentController.php**
```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Comment;
use Illuminate\Http\Request;

class CommentController extends Controller
{
    public function index()
    {
        $comments = Comment::with(['user', 'post'])
            ->latest()
            ->paginate(10);

        return view('admin.comments.index', compact('comments'));
    }

    public function show(Comment $comment)
    {
        $comment->load(['user', 'post', 'parent.user']);
        return view('admin.comments.show', compact('comment'));
    }

    public function approve(Comment $comment)
    {
        $comment->update(['is_approved' => !$comment->is_approved]);

        $status = $comment->is_approved ? 'approved' : 'unapproved';
        return back()->with('success', "Comment {$status} successfully.");
    }

    public function destroy(Comment $comment)
    {
        $comment->delete();
        return redirect()->route('admin.comments.index')->with('success', 'Comment deleted successfully.');
    }
}
```

---

## Part 3: Basic Views & Styling (60 minutes)

### 1. Update App Layout
**resources/views/layouts/app.blade.php**
```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Fonts -->
    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap" rel="stylesheet" />

    <!-- Scripts -->
    @vite(['resources/css/app.css', 'resources/js/app.js'])
    @livewireStyles
</head>
<body class="font-sans antialiased">
    <div class="min-h-screen bg-gray-100">
        @include('layouts.navigation')

        <!-- Page Heading -->
        @if (isset($header))
            <header class="bg-white shadow">
                <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
                    {{ $header }}
                </div>
            </header>
        @endif

        <!-- Page Content -->
        <main>
            @if (session('success'))
                <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-4">
                    <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded mb-4">
                        {{ session('success') }}
                    </div>
                </div>
            @endif

            @if (session('error'))
                <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-4">
                    <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
                        {{ session('error') }}
                    </div>
                </div>
            @endif

            {{ $slot }}
        </main>
    </div>
    @livewireScripts
</body>
</html>
```

### 2. Create Blog Layout
**resources/views/layouts/blog.blade.php**
```php
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>@yield('title', 'Laravel Blog Tutorial')</title>
    <meta name="description" content="@yield('description', 'A Laravel blog tutorial project with Livewire')">

    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=figtree:400,500,600&display=swap" rel="stylesheet" />

    @vite(['resources/css/app.css', 'resources/js/app.js'])
    @livewireStyles
</head>
<body class="font-sans antialiased bg-gray-50">
    <!-- Navigation -->
    <nav class="bg-white shadow-sm border-b">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between h-16">
                <div class="flex items-center">
                    <a href="{{ route('home') }}" class="text-xl font-bold text-gray-800">
                        Laravel Blog
                    </a>
                </div>

                <div class="flex items-center space-x-4">
                    @guest
                        <a href="{{ route('login') }}" class="text-gray-600 hover:text-gray-900">Login</a>
                        <a href="{{ route('register') }}" class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700">
                            Register
                        </a>
                    @else
                        <span class="text-gray-600">Hello, {{ auth()->user()->name }}!</span>

                        @if(auth()->user()->is_admin)
                            <a href="{{ route('admin.posts.index') }}" class="text-blue-600 hover:text-blue-800">
                                Admin Panel
                            </a>
                        @endif

                        <form method="POST" action="{{ route('logout') }}" class="inline">
                            @csrf
                            <button type="submit" class="text-gray-600 hover:text-gray-900">
                                Logout
                            </button>
                        </form>
                    @endguest
                </div>
            </div>
        </div>
    </nav>

    <!-- Main Content -->
    <main class="py-8">
        @if (session('success'))
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mb-6">
                <div class="bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded">
                    {{ session('success') }}
                </div>
            </div>
        @endif

        @if (session('error'))
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 mb-6">
                <div class="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded">
                    {{ session('error') }}
                </div>
            </div>
        @endif

        @yield('content')
    </main>

    <!-- Footer -->
    <footer class="bg-gray-800 text-white py-8 mt-12">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
            <p>&copy; {{ date('Y') }} Laravel Blog Tutorial. Built with Laravel & Livewire.</p>
        </div>
    </footer>

    @livewireScripts
</body>
</html>
```

### 3. Create Posts Index View
**resources/views/posts/index.blade.php**
```php
@extends('layouts.blog')

@section('title', 'Laravel Blog Tutorial')
@section('description', 'Welcome to our Laravel blog tutorial. Learn Laravel and Livewire step by step.')

@section('content')
<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <!-- Hero Section -->
    <div class="text-center mb-12">
        <h1 class="text-4xl font-bold text-gray-900 mb-4">
            Welcome to Laravel Blog Tutorial
        </h1>
        <p class="text-xl text-gray-600 max-w-2xl mx-auto">
            Learn how to build a complete blog application with Laravel 11 and Livewire 3.
            Follow our step-by-step tutorials and master modern web development.
        </p>
    </div>

    <!-- Posts Grid -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        @forelse($posts as $post)
            <article class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
                @if($post->featured_image)
                    <img src="{{ $post->featured_image }}" alt="{{ $post->title }}"
                         class="w-full h-48 object-cover">
                @endif

                <div class="p-6">
                    <div class="flex items-center text-sm text-gray-500 mb-2">
                        <span>{{ $post->user->name }}</span>
                        <span class="mx-2">â€¢</span>
                        <time datetime="{{ $post->published_at->toISOString() }}">
                            {{ $post->published_at->format('M j, Y') }}
                        </time>
                    </div>

                    <h2 class="text-xl font-semibold text-gray-900 mb-3">
                        <a href="{{ route('posts.show', $post) }}" class="hover:text-blue-600">
                            {{ $post->title }}
                        </a>
                    </h2>

                    @if($post->excerpt)
                        <p class="text-gray-600 mb-4">
                            {{ Str::limit($post->excerpt, 120) }}
                        </p>
                    @endif

                    <div class="flex items-center justify-between">
                        <a href="{{ route('posts.show', $post) }}"
                           class="text-blue-600 hover:text-blue-800 font-medium">
                            Read More â†’
                        </a>

                        <span class="text-sm text-gray-500">
                            {{ $post->comments_count ?? $post->approvedComments->count() }} comments
                        </span>
                    </div>
                </div>
            </article>
        @empty
            <div class="col-span-3 text-center py-12">
                <h3 class="text-lg font-medium text-gray-900 mb-2">No posts yet</h3>
                <p class="text-gray-600">Check back later for new content!</p>
            </div>
        @endforelse
    </div>

    <!-- Pagination -->
    @if($posts->hasPages())
        <div class="mt-12">
            {{ $posts->links() }}
        </div>
    @endif
</div>
@endsection
```

### 4. Run and Test
```bash
# Install Tailwind (if not already installed)
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# Build assets
npm run build

# Clear caches
php artisan optimize:clear
```

---

## Verification Checklist

- [ ] Database seeders created and run successfully
- [ ] Frontend controllers implemented
- [ ] Admin controllers created
- [ ] Blog layout and navigation created
- [ ] Posts index page displays correctly
- [ ] Tailwind CSS styling applied
- [ ] Responsive design implemented
- [ ] Sample data populated in database

---

## Next Steps
Tomorrow (Day 4) we'll implement:
- Livewire components for blog post CRUD operations
- Interactive post management interface
- Real-time updates and form validation