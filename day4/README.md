# Day 4: Livewire Components for Blog CRUD

## Duration: 1-2 hours

### Objectives
- Create Livewire components for blog post management
- Implement real-time CRUD operations
- Add form validation and error handling
- Create interactive admin dashboard
- Learn Livewire best practices

---

## Part 1: Basic Livewire Components (30 minutes)

### 1. Create Post Management Components
```bash
php artisan make:livewire Admin/PostManager
php artisan make:livewire Admin/PostForm
php artisan make:livewire Admin/PostList
php artisan make:livewire BlogPostCard
```

### 2. Create PostManager Component
**app/Livewire/Admin/PostManager.php**
```php
<?php

namespace App\Livewire\Admin;

use App\Models\Post;
use Livewire\Component;
use Livewire\WithPagination;

class PostManager extends Component
{
    use WithPagination;

    public $search = '';
    public $filter = 'all';
    public $showForm = false;
    public $editingPost = null;

    protected $queryString = [
        'search' => ['except' => ''],
        'filter' => ['except' => 'all'],
    ];

    public function mount()
    {
        if (!auth()->user()->is_admin) {
            abort(403);
        }
    }

    public function updatingSearch()
    {
        $this->resetPage();
    }

    public function updatingFilter()
    {
        $this->resetPage();
    }

    public function createPost()
    {
        $this->editingPost = null;
        $this->showForm = true;
    }

    public function editPost(Post $post)
    {
        $this->editingPost = $post;
        $this->showForm = true;
    }

    public function closeForm()
    {
        $this->showForm = false;
        $this->editingPost = null;
    }

    public function deletePost(Post $post)
    {
        $post->delete();
        session()->flash('success', 'Post deleted successfully.');
    }

    public function togglePublished(Post $post)
    {
        $post->update([
            'is_published' => !$post->is_published,
            'published_at' => !$post->is_published ? now() : null,
        ]);

        $status = $post->is_published ? 'published' : 'unpublished';
        session()->flash('success', "Post {$status} successfully.");
    }

    public function render()
    {
        $query = Post::with('user');

        if ($this->search) {
            $query->where('title', 'like', '%' . $this->search . '%')
                  ->orWhere('content', 'like', '%' . $this->search . '%');
        }

        switch ($this->filter) {
            case 'published':
                $query->where('is_published', true);
                break;
            case 'draft':
                $query->where('is_published', false);
                break;
            case 'mine':
                $query->where('user_id', auth()->id());
                break;
        }

        $posts = $query->latest()->paginate(10);

        return view('livewire.admin.post-manager', compact('posts'));
    }
}
```

### 3. Create PostForm Component
**app/Livewire/Admin/PostForm.php**
```php
<?php

namespace App\Livewire\Admin;

use App\Models\Post;
use Livewire\Component;
use Illuminate\Support\Str;

class PostForm extends Component
{
    public $post;
    public $title = '';
    public $slug = '';
    public $excerpt = '';
    public $content = '';
    public $featured_image = '';
    public $is_published = false;

    protected $rules = [
        'title' => 'required|string|max:255',
        'slug' => 'nullable|string|max:255',
        'excerpt' => 'nullable|string|max:500',
        'content' => 'required|string',
        'featured_image' => 'nullable|url',
        'is_published' => 'boolean',
    ];

    protected $messages = [
        'title.required' => 'Please enter a title for your post.',
        'content.required' => 'Please enter some content for your post.',
        'featured_image.url' => 'Please enter a valid URL for the featured image.',
    ];

    public function mount($post = null)
    {
        if ($post) {
            $this->post = $post;
            $this->title = $post->title;
            $this->slug = $post->slug;
            $this->excerpt = $post->excerpt;
            $this->content = $post->content;
            $this->featured_image = $post->featured_image;
            $this->is_published = $post->is_published;
        }
    }

    public function updatedTitle()
    {
        if (empty($this->slug)) {
            $this->slug = Str::slug($this->title);
        }
    }

    public function generateSlug()
    {
        $this->slug = Str::slug($this->title);
    }

    public function save()
    {
        $this->validate();

        if (empty($this->slug)) {
            $this->slug = Str::slug($this->title);
        }

        // Check for unique slug
        $slugExists = Post::where('slug', $this->slug)
            ->when($this->post, fn($query) => $query->where('id', '!=', $this->post->id))
            ->exists();

        if ($slugExists) {
            $this->addError('slug', 'This slug is already taken.');
            return;
        }

        $data = [
            'title' => $this->title,
            'slug' => $this->slug,
            'excerpt' => $this->excerpt,
            'content' => $this->content,
            'featured_image' => $this->featured_image ?: null,
            'is_published' => $this->is_published,
        ];

        if ($this->is_published && (!$this->post || !$this->post->published_at)) {
            $data['published_at'] = now();
        } elseif (!$this->is_published) {
            $data['published_at'] = null;
        }

        if ($this->post) {
            $this->post->update($data);
            $message = 'Post updated successfully.';
        } else {
            $data['user_id'] = auth()->id();
            Post::create($data);
            $message = 'Post created successfully.';
        }

        session()->flash('success', $message);
        $this->dispatch('post-saved');
        $this->dispatch('close-form');
    }

    public function render()
    {
        return view('livewire.admin.post-form');
    }
}
```

---

## Part 2: Livewire Views (30 minutes)

### 1. Create PostManager View
**resources/views/livewire/admin/post-manager.blade.php**
```php
<div class="space-y-6">
    <!-- Header -->
    <div class="flex justify-between items-center">
        <h1 class="text-2xl font-bold text-gray-900">Manage Posts</h1>
        <button wire:click="createPost" 
                class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700">
            Create New Post
        </button>
    </div>

    <!-- Filters -->
    <div class="flex flex-col sm:flex-row gap-4">
        <div class="flex-1">
            <input wire:model.live.debounce.300ms="search" 
                   type="text" 
                   placeholder="Search posts..." 
                   class="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
        </div>
        
        <select wire:model.live="filter" 
                class="px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500">
            <option value="all">All Posts</option>
            <option value="published">Published</option>
            <option value="draft">Drafts</option>
            <option value="mine">My Posts</option>
        </select>
    </div>

    <!-- Posts Table -->
    <div class="bg-white rounded-lg shadow overflow-hidden">
        <table class="min-w-full divide-y divide-gray-200">
            <thead class="bg-gray-50">
                <tr>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                        Title
                    </th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                        Author
                    </th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                        Status
                    </th>
                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                        Date
                    </th>
                    <th class="px-6 py-3 text-right text-xs font-medium text-gray-500 uppercase tracking-wider">
                        Actions
                    </th>
                </tr>
            </thead>
            <tbody class="bg-white divide-y divide-gray-200">
                @forelse($posts as $post)
                    <tr>
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div>
                                <div class="text-sm font-medium text-gray-900">
                                    {{ $post->title }}
                                </div>
                                @if($post->excerpt)
                                    <div class="text-sm text-gray-500">
                                        {{ Str::limit($post->excerpt, 60) }}
                                    </div>
                                @endif
                            </div>
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                            {{ $post->user->name }}
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap">
                            @if($post->is_published)
                                <span class="inline-flex px-2 py-1 text-xs font-semibold rounded-full bg-green-100 text-green-800">
                                    Published
                                </span>
                            @else
                                <span class="inline-flex px-2 py-1 text-xs font-semibold rounded-full bg-gray-100 text-gray-800">
                                    Draft
                                </span>
                            @endif
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                            {{ $post->created_at->format('M j, Y') }}
                        </td>
                        <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium space-x-2">
                            <a href="{{ route('posts.show', $post) }}" 
                               target="_blank"
                               class="text-blue-600 hover:text-blue-900">View</a>
                            
                            <button wire:click="editPost({{ $post->id }})" 
                                    class="text-indigo-600 hover:text-indigo-900">Edit</button>
                            
                            <button wire:click="togglePublished({{ $post->id }})" 
                                    class="text-{{ $post->is_published ? 'yellow' : 'green' }}-600 hover:text-{{ $post->is_published ? 'yellow' : 'green' }}-900">
                                {{ $post->is_published ? 'Unpublish' : 'Publish' }}
                            </button>
                            
                            <button wire:click="deletePost({{ $post->id }})" 
                                    wire:confirm="Are you sure you want to delete this post?"
                                    class="text-red-600 hover:text-red-900">Delete</button>
                        </td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="5" class="px-6 py-4 text-center text-gray-500">
                            No posts found.
                        </td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>

    <!-- Pagination -->
    @if($posts->hasPages())
        <div class="mt-6">
            {{ $posts->links() }}
        </div>
    @endif

    <!-- Modal Form -->
    @if($showForm)
        <div class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full z-50" 
             wire:click="closeForm">
            <div class="relative top-20 mx-auto p-5 border w-11/12 md:w-3/4 lg:w-1/2 shadow-lg rounded-md bg-white"
                 wire:click.stop>
                <div class="flex justify-between items-center mb-4">
                    <h3 class="text-lg font-medium text-gray-900">
                        {{ $editingPost ? 'Edit Post' : 'Create New Post' }}
                    </h3>
                    <button wire:click="closeForm" class="text-gray-400 hover:text-gray-600">
                        <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                        </svg>
                    </button>
                </div>
                
                @livewire('admin.post-form', ['post' => $editingPost], key($editingPost?->id ?? 'create'))
            </div>
        </div>
    @endif
</div>

@script
<script>
    $wire.on('post-saved', () => {
        $wire.showForm = false;
        $wire.$refresh();
    });
    
    $wire.on('close-form', () => {
        $wire.showForm = false;
    });
</script>
@endscript
```

### 2. Create PostForm View
**resources/views/livewire/admin/post-form.blade.php**
```php
<form wire:submit="save" class="space-y-6">
    <!-- Title -->
    <div>
        <label for="title" class="block text-sm font-medium text-gray-700">Title</label>
        <input wire:model.live="title" 
               type="text" 
               id="title"
               class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500">
        @error('title') <span class="text-red-500 text-sm">{{ $message }}</span> @enderror
    </div>

    <!-- Slug -->
    <div>
        <label for="slug" class="block text-sm font-medium text-gray-700">Slug</label>
        <div class="mt-1 flex rounded-md shadow-sm">
            <input wire:model="slug" 
                   type="text" 
                   id="slug"
                   class="flex-1 block w-full px-3 py-2 border border-gray-300 rounded-l-md focus:outline-none focus:ring-blue-500 focus:border-blue-500">
            <button type="button" 
                    wire:click="generateSlug"
                    class="inline-flex items-center px-3 py-2 border border-l-0 border-gray-300 bg-gray-50 text-gray-500 rounded-r-md">
                Generate
            </button>
        </div>
        @error('slug') <span class="text-red-500 text-sm">{{ $message }}</span> @enderror
    </div>

    <!-- Excerpt -->
    <div>
        <label for="excerpt" class="block text-sm font-medium text-gray-700">Excerpt</label>
        <textarea wire:model="excerpt" 
                  id="excerpt" 
                  rows="3"
                  class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
                  placeholder="Brief description of the post..."></textarea>
        @error('excerpt') <span class="text-red-500 text-sm">{{ $message }}</span> @enderror
    </div>

    <!-- Content -->
    <div>
        <label for="content" class="block text-sm font-medium text-gray-700">Content</label>
        <textarea wire:model="content" 
                  id="content" 
                  rows="10"
                  class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
                  placeholder="Write your post content here..."></textarea>
        @error('content') <span class="text-red-500 text-sm">{{ $message }}</span> @enderror
    </div>

    <!-- Featured Image -->
    <div>
        <label for="featured_image" class="block text-sm font-medium text-gray-700">Featured Image URL</label>
        <input wire:model="featured_image" 
               type="url" 
               id="featured_image"
               class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500"
               placeholder="https://example.com/image.jpg">
        @error('featured_image') <span class="text-red-500 text-sm">{{ $message }}</span> @enderror
    </div>

    <!-- Published Status -->
    <div class="flex items-center">
        <input wire:model="is_published" 
               type="checkbox" 
               id="is_published"
               class="h-4 w-4 text-blue-600 focus:ring-blue-500 border-gray-300 rounded">
        <label for="is_published" class="ml-2 block text-sm text-gray-900">
            Publish immediately
        </label>
    </div>

    <!-- Form Actions -->
    <div class="flex justify-end space-x-3 pt-6 border-t">
        <button type="button" 
                wire:click="$dispatch('close-form')"
                class="px-4 py-2 border border-gray-300 rounded-md text-sm font-medium text-gray-700 hover:bg-gray-50">
            Cancel
        </button>
        <button type="submit" 
                class="px-4 py-2 bg-blue-600 border border-transparent rounded-md text-sm font-medium text-white hover:bg-blue-700">
            {{ $post ? 'Update Post' : 'Create Post' }}
        </button>
    </div>
</form>
```

---

## Part 3: Blog Post Card Component (30 minutes)

### 1. Create BlogPostCard Component
**app/Livewire/BlogPostCard.php**
```php
<?php

namespace App\Livewire;

use App\Models\Post;
use Livewire\Component;

class BlogPostCard extends Component
{
    public Post $post;
    public $showFullContent = false;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    public function toggleContent()
    {
        $this->showFullContent = !$this->showFullContent;
    }

    public function render()
    {
        return view('livewire.blog-post-card');
    }
}
```

### 2. Create BlogPostCard View
**resources/views/livewire/blog-post-card.blade.php**
```php
<article class="bg-white rounded-lg shadow-md overflow-hidden hover:shadow-lg transition-shadow">
    @if($post->featured_image)
        <img src="{{ $post->featured_image }}" 
             alt="{{ $post->title }}" 
             class="w-full h-48 object-cover">
    @endif
    
    <div class="p-6">
        <!-- Meta Information -->
        <div class="flex items-center text-sm text-gray-500 mb-2">
            <span>{{ $post->user->name }}</span>
            <span class="mx-2">•</span>
            <time datetime="{{ $post->published_at->toISOString() }}">
                {{ $post->published_at->format('M j, Y') }}
            </time>
            @if($post->published_at->diffInDays() < 7)
                <span class="ml-2 inline-flex px-2 py-1 text-xs font-semibold rounded-full bg-blue-100 text-blue-800">
                    New
                </span>
            @endif
        </div>
        
        <!-- Title -->
        <h2 class="text-xl font-semibold text-gray-900 mb-3">
            <a href="{{ route('posts.show', $post) }}" 
               class="hover:text-blue-600 transition-colors">
                {{ $post->title }}
            </a>
        </h2>
        
        <!-- Excerpt/Content -->
        <div class="text-gray-600 mb-4">
            @if($showFullContent && $post->content)
                <div class="prose max-w-none">
                    {!! Str::markdown($post->content) !!}
                </div>
            @elseif($post->excerpt)
                <p>{{ Str::limit($post->excerpt, 120) }}</p>
            @else
                <p>{{ Str::limit(strip_tags($post->content), 120) }}</p>
            @endif
        </div>
        
        <!-- Actions -->
        <div class="flex items-center justify-between">
            <div class="flex space-x-4">
                <a href="{{ route('posts.show', $post) }}" 
                   class="text-blue-600 hover:text-blue-800 font-medium transition-colors">
                    Read More →
                </a>
                
                @if($post->content && !$showFullContent)
                    <button wire:click="toggleContent" 
                            class="text-gray-600 hover:text-gray-800 font-medium transition-colors">
                        Quick Preview
                    </button>
                @elseif($showFullContent)
                    <button wire:click="toggleContent" 
                            class="text-gray-600 hover:text-gray-800 font-medium transition-colors">
                        Show Less
                    </button>
                @endif
            </div>
            
            <!-- Comments Count -->
            <div class="flex items-center text-sm text-gray-500">
                <svg class="w-4 h-4 mr-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
                          d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z">
                    </path>
                </svg>
                {{ $post->approvedComments->count() }} 
                {{ Str::plural('comment', $post->approvedComments->count()) }}
            </div>
        </div>
    </div>
</article>
```

---

## Part 4: Update Routes and Views (30 minutes)

### 1. Create Admin Routes
**routes/web.php** - Add these routes:
```php
Route::middleware(['auth', 'admin'])->group(function () {
    Route::get('/admin', function () {
        return view('admin.dashboard');
    })->name('admin.dashboard');
    
    Route::get('/admin/posts', function () {
        return view('admin.posts.index');
    })->name('admin.posts.index');
});
```

### 2. Create Admin Dashboard View
**resources/views/admin/dashboard.blade.php**
```php
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Admin Dashboard') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <!-- Stats Cards -->
            <div class="grid grid-cols-1 md:grid-cols-4 gap-6 mb-8">
                <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <div class="p-6 text-center">
                        <div class="text-3xl font-bold text-blue-600">
                            {{ \App\Models\Post::count() }}
                        </div>
                        <div class="text-gray-600">Total Posts</div>
                    </div>
                </div>
                
                <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <div class="p-6 text-center">
                        <div class="text-3xl font-bold text-green-600">
                            {{ \App\Models\Post::where('is_published', true)->count() }}
                        </div>
                        <div class="text-gray-600">Published</div>
                    </div>
                </div>
                
                <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <div class="p-6 text-center">
                        <div class="text-3xl font-bold text-yellow-600">
                            {{ \App\Models\Comment::where('is_approved', false)->count() }}
                        </div>
                        <div class="text-gray-600">Pending Comments</div>
                    </div>
                </div>
                
                <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                    <div class="p-6 text-center">
                        <div class="text-3xl font-bold text-purple-600">
                            {{ \App\Models\User::count() }}
                        </div>
                        <div class="text-gray-600">Total Users</div>
                    </div>
                </div>
            </div>

            <!-- Quick Actions -->
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg mb-8">
                <div class="p-6">
                    <h3 class="text-lg font-medium text-gray-900 mb-4">Quick Actions</h3>
                    <div class="flex flex-wrap gap-4">
                        <a href="{{ route('admin.posts.index') }}" 
                           class="bg-blue-600 text-white px-4 py-2 rounded-md hover:bg-blue-700">
                            Manage Posts
                        </a>
                        <a href="{{ route('admin.comments.index') }}" 
                           class="bg-green-600 text-white px-4 py-2 rounded-md hover:bg-green-700">
                            Manage Comments
                        </a>
                        <a href="{{ route('home') }}" 
                           class="bg-gray-600 text-white px-4 py-2 rounded-md hover:bg-gray-700">
                            View Blog
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

### 3. Create Admin Posts Index View
**resources/views/admin/posts/index.blade.php**
```php
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Manage Posts') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6">
                    @livewire('admin.post-manager')
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

### 4. Update Posts Index to Use Livewire Cards
**resources/views/posts/index.blade.php** - Update the posts grid section:
```php
<!-- Posts Grid -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
    @forelse($posts as $post)
        @livewire('blog-post-card', ['post' => $post], key($post->id))
    @empty
        <div class="col-span-3 text-center py-12">
            <h3 class="text-lg font-medium text-gray-900 mb-2">No posts yet</h3>
            <p class="text-gray-600">Check back later for new content!</p>
        </div>
    @endforelse
</div>
```

---

## Testing & Verification

### 1. Test Commands
```bash
# Clear caches
php artisan optimize:clear

# Test Livewire components
php artisan livewire:list
```

### 2. Manual Testing
1. Login as admin user
2. Navigate to `/admin/posts`
3. Test creating, editing, and deleting posts
4. Test publishing/unpublishing posts
5. Test search and filter functionality
6. Check real-time updates

---

## Verification Checklist

- [ ] Livewire components created and working
- [ ] Post CRUD operations functional
- [ ] Real-time form validation working
- [ ] Admin dashboard created
- [ ] Interactive post cards implemented
- [ ] Search and filter functionality working
- [ ] Modal forms working properly

---

## Next Steps
Tomorrow (Day 5) we'll implement:
- Livewire components for comment management
- Custom comment component with nested replies
- Real-time comment approval system
- Interactive comment features