# ৪. সেপারেশন অফ কনসার্নস(Separation of concerns (SoC)): 🎭

- প্রেজেন্টেশন লেয়ার
- বিজনেস লজিক
- ডাটা অ্যাক্সেস
- কনফিগারেশন

# Laravel-এ Separation of Concerns (SoC) উদাহরণ

Separation of Concerns (SoC) হল একটি নকশা নীতি যা কোডকে ভিন্ন ভিন্ন স্তরে সংগঠিত করে। প্রতিটি স্তর নির্দিষ্ট একটি কাজ বা দায়িত্ব নিয়ে কাজ করে। Laravel-এ SoC সহজেই বাস্তবায়ন করা যায় বিভিন্ন উপাদান (Controllers, Services, Repositories) ব্যবহার করে।

### উদাহরণ: ব্লগ ম্যানেজমেন্ট

আমরা একটি ব্লগ ম্যানেজমেন্ট সিস্টেম তৈরি করব যেখানে থাকবে:

1. সকল পোস্টের তালিকা।
2. একটি নির্দিষ্ট পোস্ট দেখানো।
3. নতুন পোস্ট তৈরি।

---

## ধাপ ১: প্রজেক্ট সেটআপ

Laravel প্রজেক্ট তৈরি করুন:

```bash
composer create-project laravel/laravel blog-app
```

---

## ধাপ ২: ডেটাবেস এবং মাইগ্রেশন তৈরি

`.env` ফাইল আপডেট করুন ডেটাবেসের তথ্য দিয়ে।

`posts` টেবিলের জন্য মাইগ্রেশন তৈরি করুন:

```bash
php artisan make:migration create_posts_table --create=posts
```

মাইগ্রেশন ফাইল সম্পাদনা করুন:

```php
// database/migrations/xxxx_xx_xx_create_posts_table.php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('content');
        $table->timestamps();
    });
}
```

মাইগ্রেশন চালান:

```bash
php artisan migrate
```

---

## ধাপ ৩: Post মডেল তৈরি

```bash
php artisan make:model Post
```

---

## ধাপ ৪: Controller এবং Service তৈরি

### PostController তৈরি করুন:

```bash
php artisan make:controller PostController
```

`PostController`:

```php
namespace App\Http\Controllers;

use App\Services\PostService;
use Illuminate\Http\Request;

class PostController extends Controller
{
    private $postService;

    public function __construct(PostService $postService)
    {
        $this->postService = $postService;
    }

    public function index()
    {
        $posts = $this->postService->getAllPosts();
        return view('posts.index', compact('posts'));
    }

    public function show($id)
    {
        $post = $this->postService->getPostById($id);
        return view('posts.show', compact('post'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'title' => 'required|string|max:255',
            'content' => 'required|string',
        ]);

        $this->postService->createPost($request->all());

        return redirect()->route('posts.index');
    }
}
```

### PostService তৈরি করুন:

```bash
php artisan make:service PostService
```

`PostService`:

```php
namespace App\Services;

use App\Models\Post;

class PostService
{
    public function getAllPosts()
    {
        return Post::all();
    }

    public function getPostById($id)
    {
        return Post::findOrFail($id);
    }

    public function createPost($data)
    {
        return Post::create($data);
    }
}
```

---

## ধাপ ৫: Routing এবং View

### Route সেটআপ:

```php
use App\Http\Controllers\PostController;

Route::get('/posts', [PostController::class, 'index'])->name('posts.index');
Route::get('/posts/{id}', [PostController::class, 'show'])->name('posts.show');
Route::post('/posts', [PostController::class, 'store'])->name('posts.store');
```

### View তৈরি করুন:

`resources/views/posts/index.blade.php`:

```html
<h1>All Posts</h1>
<ul>
  @foreach($posts as $post)
  <li>
    <a href="{{ route('posts.show', $post->id) }}">{{ $post->title }}</a>
  </li>
  @endforeach
</ul>
```

`resources/views/posts/show.blade.php`:

```html
<h1>{{ $post->title }}</h1>
<p>{{ $post->content }}</p>
<a href="{{ route('posts.index') }}">Back to Posts</a>
```

---

## উপসংহার

এই উদাহরণটি দেখায় কিভাবে Laravel-এ Separation of Concerns (SoC) ব্যবহার করা যায়।

- **Controller**: শুধুমাত্র HTTP request এবং response পরিচালনা করে।
- **Service**: ব্যবসায়িক লজিক পরিচালনা করে।
- **Model**: ডেটা ম্যানেজমেন্ট এবং ডেটাবেস সংযোগ পরিচালনা করে।
