# ২. DRY (Don't Repeat Yourself): 🔄

- কোড কপি-পেস্ট করছেন? থামুন!
- রিইউজেবল কম্পোনেন্ট বানান
- ইউটিলিটি ফাংশন লিখুন
- কনফিগারেশন সেন্ট্রালাইজ করুন

# DRY (Don't Repeat Yourself) in Laravel (Bangla)

Laravel একটি MVC (Model-View-Controller) ফ্রেমওয়ার্ক যা DRY principle মেনে কাজ করার জন্য বেশ সুবিধাজনক। DRY বলতে বোঝায় একই কোড বা লজিক বারবার না লিখে তা পুনঃব্যবহারযোগ্য করা। এটি কেবল কোডের মান উন্নত করে না, বরং ভবিষ্যতে কোড মেইনটেইন করাও সহজ করে।

---

## DRY Principle অনুসরণ করার কয়েকটি উপায়:

### ১. Reusing Query Logic: Model Scope ব্যবহার করুন

কোনো কোয়ারি বারবার লিখার পরিবর্তে, আমরা **Local Scopes** ব্যবহার করতে পারি।

#### Example:

**Model (Product.php)**

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Product extends Model
{
    // Local Scope for Active Products
    public function scopeActive($query)
    {
        return $query->where('status', 'active');
    }

    // Local Scope for Featured Products
    public function scopeFeatured($query)
    {
        return $query->where('is_featured', true);
    }
}
```

**Controller (ProductController.php)**

```php
<?php

namespace App\Http\Controllers;

use App\Models\Product;

class ProductController extends Controller
{
    public function index()
    {
        // Using the scopes
        $activeProducts = Product::active()->get();
        $featuredProducts = Product::featured()->get();

        return view('products.index', compact('activeProducts', 'featuredProducts'));
    }
}
```

---

### ২. Reusable Validation Rules

কোনো ফর্ম ভ্যালিডেশন বারবার না লিখে, **Form Request Classes** ব্যবহার করুন।

#### Command:

```bash
php artisan make:request ProductRequest
```

#### Request (ProductRequest.php)

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class ProductRequest extends FormRequest
{
    public function rules()
    {
        return [
            'name' => 'required|string|max:255',
            'price' => 'required|numeric|min:1',
            'description' => 'nullable|string',
        ];
    }
}
```

#### Controller (ProductController.php)

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\ProductRequest;
use App\Models\Product;

class ProductController extends Controller
{
    public function store(ProductRequest $request)
    {
        Product::create($request->validated());
        return redirect()->back()->with('success', 'Product created successfully!');
    }
}
```

---

### ৩. Blade Components এবং Includes

কোনো ভিউ ফাইল বারবার লেখার পরিবর্তে **Blade Components** এবং **Includes** ব্যবহার করুন।

#### Command:

```bash
php artisan make:component Alert
```

#### Component (Alert.php)

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public $type;

    public function __construct($type)
    {
        $this->type = $type;
    }

    public function render()
    {
        return view('components.alert');
    }
}
```

#### View (resources/views/components/alert.blade.php)

```html
<div class="alert alert-{{ $type }}">{{ $slot }}</div>
```

#### Usage in Blade:

```html
<x-alert type="success"> This is a success message. </x-alert>

<x-alert type="error"> This is an error message. </x-alert>
```

---

### ৪. Helper Functions

যদি একাধিক স্থানে একই লজিক বা কোড ব্যবহার করতে হয়, Helper Function তৈরি করে ব্যবহার করুন।

#### Helper Function তৈরি করুন:

**File:** `app/helpers.php`

```php
<?php

if (!function_exists('format_price')) {
    function format_price($price)
    {
        return number_format($price, 2) . ' ৳';
    }
}
```

#### Composer.json আপডেট করুন:

```json
"autoload": {
    "files": [
        "app/helpers.php"
    ]
}
```

#### Usage in Blade:

```php
{{ format_price($product->price) }}
```

---

## DRY ব্যবহার করার সুবিধা:

1. কোড ক্লিন এবং রিডেবল হয়।
2. কোডের পুনঃব্যবহারযোগ্যতা বৃদ্ধি পায়।
3. বাগ ফিক্স এবং আপডেট সহজ হয়।

Laravel ফ্রেমওয়ার্কে DRY principle অনুসরণ করা সহজ, এবং এটি আপনার অ্যাপ্লিকেশনকে আরো maintainable করে তোলে। 🎉
