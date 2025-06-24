# Day 2: Authentication, Migrations & Environment Setup

## Duration: 1-2 hours

### Objectives
- Install and configure Laravel Sanctum
- Create database migrations for blog posts and comments
- Set up authentication middleware
- Configure environment for different stages

---

## Part 1: Laravel Sanctum Setup (30 minutes)

### 1. Install Sanctum
```bash
composer require laravel/sanctum
```

### 2. Publish Sanctum Configuration
```bash
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

### 3. Run Sanctum Migrations
```bash
php artisan migrate
```

### 4. Update User Model
**app/Models/User.php**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];

    public function posts()
    {
        return $this->hasMany(Post::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }
}
```

### 5. Configure Sanctum Middleware
**app/Http/Kernel.php** (or **bootstrap/app.php** for Laravel 12)

For Laravel 12, update **bootstrap/app.php**:
```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->api(prepend: [
            \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        ]);

        $middleware->alias([
            'verified' => \App\Http\Middleware\EnsureEmailIsVerified::class,
        ]);

        $middleware->validateCsrfTokens(except: [
            'stripe/*',
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

---

## Part 2: Database Migrations (30 minutes)

### 1. Create Post Migration
```bash
php artisan make:migration create_posts_table
```

**database/migrations/YYYY_MM_DD_HHMMSS_create_posts_table.php**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->string('slug')->unique();
            $table->text('excerpt')->nullable();
            $table->longText('content');
            $table->string('featured_image')->nullable();
            $table->boolean('is_published')->default(false);
            $table->timestamp('published_at')->nullable();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->timestamps();
            
            $table->index(['is_published', 'published_at']);
            $table->index('slug');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('posts');
    }
};
```

### 2. Create Comment Migration
```bash
php artisan make:migration create_comments_table
```

**database/migrations/YYYY_MM_DD_HHMMSS_create_comments_table.php**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('comments', function (Blueprint $table) {
            $table->id();
            $table->text('content');
            $table->boolean('is_approved')->default(false);
            $table->foreignId('post_id')->constrained()->onDelete('cascade');
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->foreignId('parent_id')->nullable()->constrained('comments')->onDelete('cascade');
            $table->timestamps();
            
            $table->index(['post_id', 'is_approved']);
            $table->index('parent_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('comments');
    }
};
```

### 3. Create Models
```bash
php artisan make:model Post
php artisan make:model Comment
```

**app/Models/Post.php**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Support\Str;

class Post extends Model
{
    use HasFactory;

    protected $fillable = [
        'title',
        'slug',
        'excerpt',
        'content',
        'featured_image',
        'is_published',
        'published_at',
        'user_id',
    ];

    protected $casts = [
        'is_published' => 'boolean',
        'published_at' => 'datetime',
    ];

    protected static function boot()
    {
        parent::boot();

        static::creating(function ($post) {
            if (empty($post->slug)) {
                $post->slug = Str::slug($post->title);
            }
        });

        static::updating(function ($post) {
            if ($post->isDirty('title') && empty($post->slug)) {
                $post->slug = Str::slug($post->title);
            }
        });
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }

    public function approvedComments()
    {
        return $this->hasMany(Comment::class)->where('is_approved', true);
    }

    public function scopePublished($query)
    {
        return $query->where('is_published', true)
                    ->whereNotNull('published_at')
                    ->where('published_at', '<=', now());
    }

    public function getRouteKeyName()
    {
        return 'slug';
    }
}
```

**app/Models/Comment.php**
```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    use HasFactory;

    protected $fillable = [
        'content',
        'is_approved',
        'post_id',
        'user_id',
        'parent_id',
    ];

    protected $casts = [
        'is_approved' => 'boolean',
    ];

    public function post()
    {
        return $this->belongsTo(Post::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function parent()
    {
        return $this->belongsTo(Comment::class, 'parent_id');
    }

    public function replies()
    {
        return $this->hasMany(Comment::class, 'parent_id');
    }

    public function approvedReplies()
    {
        return $this->hasMany(Comment::class, 'parent_id')->where('is_approved', true);
    }

    public function scopeApproved($query)
    {
        return $query->where('is_approved', true);
    }

    public function scopeParent($query)
    {
        return $query->whereNull('parent_id');
    }
}
```

### 4. Run Migrations
```bash
php artisan migrate
```

---

## Part 3: Authentication Setup (30 minutes)

### 1. Install Laravel Breeze (Simple Authentication)
```bash
composer require laravel/breeze --dev
php artisan breeze:install

# Choose: Blade with Alpine
# When prompted, choose: No (for dark mode)
# When prompted, choose: No (for testing)

npm install && npm run build
```

### 2. Create Authentication Routes
**routes/web.php**
```php
<?php

use App\Http\Controllers\ProfileController;
use App\Http\Controllers\PostController;
use App\Http\Controllers\CommentController;
use Illuminate\Support\Facades\Route;

Route::get('/', [PostController::class, 'index'])->name('home');
Route::get('/posts/{post}', [PostController::class, 'show'])->name('posts.show');

Route::middleware('auth')->group(function () {
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
    
    // Admin routes
    Route::prefix('admin')->name('admin.')->group(function () {
        Route::resource('posts', \App\Http\Controllers\Admin\PostController::class);
        Route::resource('comments', \App\Http\Controllers\Admin\CommentController::class);
        Route::patch('comments/{comment}/approve', [\App\Http\Controllers\Admin\CommentController::class, 'approve'])->name('comments.approve');
    });
    
    // Comment routes
    Route::post('/posts/{post}/comments', [CommentController::class, 'store'])->name('comments.store');
    Route::delete('/comments/{comment}', [CommentController::class, 'destroy'])->name('comments.destroy');
});

require __DIR__.'/auth.php';
```

### 3. Create Controllers
```bash
php artisan make:controller PostController
php artisan make:controller CommentController
php artisan make:controller Admin/PostController --resource
php artisan make:controller Admin/CommentController --resource
```

### 4. Environment Configuration
**config/sanctum.php** - Update stateful domains:
```php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,localhost:8000,127.0.0.1,127.0.0.1:8000,::1',
    env('APP_URL') ? ','.parse_url(env('APP_URL'), PHP_URL_HOST) : ''
))),
```

**.env** - Add additional configuration:
```env
# Session Configuration
SESSION_DRIVER=database
SESSION_LIFETIME=120

# Sanctum Configuration
SANCTUM_STATEFUL_DOMAINS=localhost:8000,127.0.0.1:8000

# Mail Configuration (for development)
MAIL_MAILER=log
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS="noreply@laravel-blog.local"
MAIL_FROM_NAME="${APP_NAME}"

# Cache Configuration
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
```

### 5. Create Session Table
```bash
php artisan session:table
php artisan migrate
```

---

## Part 4: Middleware Setup (30 minutes)

### 1. Create Custom Middleware
```bash
php artisan make:middleware EnsureUserIsAdmin
```

**app/Http/Middleware/EnsureUserIsAdmin.php**
```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsAdmin
{
    public function handle(Request $request, Closure $next): Response
    {
        if (!auth()->check() || !auth()->user()->is_admin) {
            abort(403, 'Access denied. Admin privileges required.');
        }

        return $next($request);
    }
}
```

### 2. Add is_admin Column to Users
```bash
php artisan make:migration add_is_admin_to_users_table
```

**database/migrations/YYYY_MM_DD_HHMMSS_add_is_admin_to_users_table.php**
```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->boolean('is_admin')->default(false)->after('email_verified_at');
        });
    }

    public function down(): void
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('is_admin');
        });
    }
};
```

### 3. Register Middleware
**bootstrap/app.php** - Add to middleware aliases:
```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
        'verified' => \App\Http\Middleware\EnsureEmailIsVerified::class,
    ]);
})
```

### 4. Update Routes with Middleware
**routes/web.php** - Add admin middleware:
```php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::prefix('admin')->name('admin.')->group(function () {
        Route::resource('posts', \App\Http\Controllers\Admin\PostController::class);
        Route::resource('comments', \App\Http\Controllers\Admin\CommentController::class);
        Route::patch('comments/{comment}/approve', [\App\Http\Controllers\Admin\CommentController::class, 'approve'])->name('comments.approve');
    });
});
```

### 5. Run Final Migration
```bash
php artisan migrate
```

---

## Testing the Setup

### 1. Create a Test Admin User
```bash
php artisan tinker
```

```php
$user = \App\Models\User::create([
    'name' => 'Admin User',
    'email' => 'admin@example.com',
    'password' => \Hash::make('password'),
    'is_admin' => true,
]);

$user->markEmailAsVerified();
```

### 2. Test Authentication
1. Visit http://localhost:8000
2. Register a new account or login with admin credentials
3. Check that admin routes are protected

### 3. Verify Database Structure
```bash
php artisan migrate:status
```

---

## Verification Checklist

- [ ] Laravel Sanctum installed and configured
- [ ] Database migrations created and run
- [ ] User, Post, and Comment models created with relationships
- [ ] Laravel Breeze authentication installed
- [ ] Admin middleware created and configured
- [ ] Session table created
- [ ] Environment variables configured
- [ ] Test admin user created
- [ ] All routes protected by appropriate middleware

---

## Troubleshooting

### Common Issues:

1. **Migration errors**: Check database connection in .env
2. **Middleware not working**: Clear config cache: `php artisan config:clear`
3. **Session issues**: Clear all caches: `php artisan optimize:clear`
4. **Sanctum not working**: Ensure domains are correctly configured

### Useful Commands:
```bash
# Clear all caches
php artisan optimize:clear

# Check routes
php artisan route:list

# Check migrations
php artisan migrate:status

# Rollback last migration
php artisan migrate:rollback

# Fresh migration with seeding
php artisan migrate:fresh --seed
```

---

## Next Steps
Tomorrow (Day 3) we'll implement:
- Database seeders for blog posts and comments
- Frontend controllers for displaying posts and comments
- Basic CSS styling for the blog interface