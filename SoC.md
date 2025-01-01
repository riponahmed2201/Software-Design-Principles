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

# Laravel E-Commerce Project Example: Category, Brand, Product

এই উদাহরণটি একটি Laravel E-Commerce প্রজেক্টে Category, Brand, এবং Product মডিউল তৈরির জন্য একটি পূর্ণাঙ্গ গাইড। এখানে Service Layer ব্যবহারের মাধ্যমে কোড আরও সংগঠিত করা হবে।

---

## ধাপ ১: প্রকল্প সেটআপ

নতুন Laravel প্রকল্প তৈরি করুন:

```bash
composer create-project laravel/laravel ecommerce-app
```

`.env` ফাইলটি আপডেট করে ডেটাবেস সেটআপ করুন।

---

## ধাপ ২: মাইগ্রেশন তৈরি

### ২.১: Category মাইগ্রেশন

```bash
php artisan make:migration create_categories_table --create=categories
```

**categories টেবিলের জন্য মাইগ্রেশন ফাইলটি আপডেট করুন:**

```php
// database/migrations/xxxx_xx_xx_create_categories_table.php
public function up()
{
    Schema::create('categories', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}
```

### ২.২: Brand মাইগ্রেশন

```bash
php artisan make:migration create_brands_table --create=brands
```

**brands টেবিলের জন্য মাইগ্রেশন ফাইলটি আপডেট করুন:**

```php
// database/migrations/xxxx_xx_xx_create_brands_table.php
public function up()
{
    Schema::create('brands', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->timestamps();
    });
}
```

### ২.৩: Product মাইগ্রেশন

```bash
php artisan make:migration create_products_table --create=products
```

**products টেবিলের জন্য মাইগ্রেশন ফাইলটি আপডেট করুন:**

```php
// database/migrations/xxxx_xx_xx_create_products_table.php
public function up()
{
    Schema::create('products', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->foreignId('category_id')->constrained()->cascadeOnDelete();
        $table->foreignId('brand_id')->constrained()->cascadeOnDelete();
        $table->decimal('price', 8, 2);
        $table->integer('stock');
        $table->timestamps();
    });
}
```

মাইগ্রেশন রান করুন:

```bash
php artisan migrate
```

---

## ধাপ ৩: মডেল তৈরি

### ৩.১: Category মডেল

```bash
php artisan make:model Category
```

**Category.php:**

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Category extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function products()
    {
        return $this->hasMany(Product::class);
    }
}
```

### ৩.২: Brand মডেল

```bash
php artisan make:model Brand
```

**Brand.php:**

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Brand extends Model
{
    use HasFactory;

    protected $fillable = ['name'];

    public function products()
    {
        return $this->hasMany(Product::class);
    }
}
```

### ৩.৩: Product মডেল

```bash
php artisan make:model Product
```

**Product.php:**

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    use HasFactory;

    protected $fillable = ['name', 'category_id', 'brand_id', 'price', 'stock'];

    public function category()
    {
        return $this->belongsTo(Category::class);
    }

    public function brand()
    {
        return $this->belongsTo(Brand::class);
    }
}
```

---

## ধাপ ৪: সার্ভিস লেয়ার তৈরি

Service Layer তৈরি করে আমরা লজিকগুলো কন্ট্রোলারের বাইরে রাখব।

### ৪.১: CategoryService

```bash
php artisan make:service CategoryService
```

**CategoryService.php:**

```php
namespace App\Services;

use App\Models\Category;

class CategoryService
{
    public function getAllCategories()
    {
        return Category::all();
    }

    public function createCategory($data)
    {
        return Category::create($data);
    }
}
```

### ৪.২: BrandService

```bash
php artisan make:service BrandService
```

**BrandService.php:**

```php
namespace App\Services;

use App\Models\Brand;

class BrandService
{
    public function getAllBrands()
    {
        return Brand::all();
    }

    public function createBrand($data)
    {
        return Brand::create($data);
    }
}
```

### ৪.৩: ProductService

```bash
php artisan make:service ProductService
```

**ProductService.php:**

```php
namespace App\Services;

use App\Models\Product;

class ProductService
{
    public function getAllProducts()
    {
        return Product::with(['category', 'brand'])->get();
    }

    public function createProduct($data)
    {
        return Product::create($data);
    }
}
```

---

## ধাপ ৫: কন্ট্রোলার আপডেট

### ৫.১: CategoryController

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Services\CategoryService;

class CategoryController extends Controller
{
    protected $categoryService;

    public function __construct(CategoryService $categoryService)
    {
        $this->categoryService = $categoryService;
    }

    public function index()
    {
        return $this->categoryService->getAllCategories();
    }

    public function store(Request $request)
    {
        $data = $request->validate(['name' => 'required|string|max:255']);
        return $this->categoryService->createCategory($data);
    }
}
```

### ৫.২: BrandController

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Services\BrandService;

class BrandController extends Controller
{
    protected $brandService;

    public function __construct(BrandService $brandService)
    {
        $this->brandService = $brandService;
    }

    public function index()
    {
        return $this->brandService->getAllBrands();
    }

    public function store(Request $request)
    {
        $data = $request->validate(['name' => 'required|string|max:255']);
        return $this->brandService->createBrand($data);
    }
}
```

### ৫.৩: ProductController

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Services\ProductService;

class ProductController extends Controller
{
    protected $productService;

    public function __construct(ProductService $productService)
    {
        $this->productService = $productService;
    }

    public function index()
    {
        return $this->productService->getAllProducts();
    }

    public function store(Request $request)
    {
        $data = $request->validate([
            'name' => 'required|string|max:255',
            'category_id' => 'required|exists:categories,id',
            'brand_id' => 'required|exists:brands,id',
            'price' => 'required|numeric',
            'stock' => 'required|integer',
        ]);
        return $this->productService->createProduct($data);
    }
}
```

---

## ধাপ ৬: রাউট সেটআপ

`routes/web.php` ফাইলটি আপডেট করুন:

```php
use App\Http\Controllers\CategoryController;
use App\Http\Controllers\BrandController;
use App\Http\Controllers\ProductController;

Route::get('/categories', [CategoryController::class, 'index']);
Route::post('/categories', [CategoryController::class, 'store']);

Route::get('/brands', [BrandController::class, 'index']);
Route::post('/brands', [BrandController::class, 'store']);

Route::get('/products', [ProductController::class, 'index']);
Route::post('/products', [ProductController::class, 'store']);
```

---

## সার্ভার চালান

```bash
php artisan serve
```

---

এই উদাহরণের মাধ্যমে Service Layer ব্যবহার করে একটি পরিষ্কার এবং সংগঠিত Laravel E-Commerce অ্যাপ্লিকেশন তৈরি করা হয়েছে।
