# ৩. KISS (Keep It Simple, Stupid): 💫

- জটিল সলিউশন = বাগি সলিউশন
- সহজ কোড = মেইনটেইনেবল কোড
- কমপ্লেক্স ফিচার = সাপোর্ট নাইটমেয়ার
- সিম্পল = স্কেলেবল

# KISS নীতিমালা Laravel-এ

**KISS** (Keep It Simple, Stupid) নীতিমালা কোড এবং ডিজাইনে সরলতার উপর গুরুত্ব দেয়। এটি অপ্রয়োজনীয় জটিলতা এড়ানোর পরামর্শ দেয় যাতে আপনার কোড সহজে বোঝা, রক্ষণাবেক্ষণ এবং সম্প্রসারণযোগ্য হয়।

এই গাইডে, আমরা Laravel অ্যাপ্লিকেশনে KISS নীতিমালা কীভাবে প্রয়োগ করা যায় তা প্রদর্শন করবো।

---

## উদাহরণ: একটি সহজ টাস্ক ম্যানেজমেন্ট অ্যাপ্লিকেশন

আমরা একটি মৌলিক টাস্ক ম্যানেজমেন্ট সিস্টেম তৈরি করবো, যেখানে থাকবে:

- টাস্ক যোগ করা
- টাস্ক দেখা
- টাস্ক সম্পন্ন হিসেবে চিহ্নিত করা

### ধাপ ১: মডেল সেট আপ করা

`Task` মডেল এবং মাইগ্রেশন ফাইল জেনারেট করতে নিচের কমান্ডটি চালান:

```bash
php artisan make:model Task -m
```

`database/migrations` ফোল্ডারে মাইগ্রেশন ফাইলটি সম্পাদনা করুন:

```php
// database/migrations/xxxx_xx_xx_create_tasks_table.php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateTasksTable extends Migration
{
    public function up()
    {
        Schema::create('tasks', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->boolean('is_completed')->default(false);
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::dropIfExists('tasks');
    }
}
```

মাইগ্রেশন চালান:

```bash
php artisan migrate
```

### ধাপ ২: কন্ট্রোলার সেট আপ করা

কন্ট্রোলার তৈরি করুন:

```bash
php artisan make:controller TaskController
```

কন্ট্রোলারটি সম্পাদনা করুন:

```php
// app/Http/Controllers/TaskController.php
namespace App\Http\Controllers;

use App\Models\Task;
use Illuminate\Http\Request;

class TaskController extends Controller
{
    public function index()
    {
        $tasks = Task::all();
        return view('tasks.index', compact('tasks'));
    }

    public function store(Request $request)
    {
        $request->validate(['title' => 'required|max:255']);

        Task::create($request->only('title'));

        return redirect()->back();
    }

    public function update(Task $task)
    {
        $task->update(['is_completed' => !$task->is_completed]);

        return redirect()->back();
    }
}
```

### ধাপ ৩: রাউট সেট আপ করা

`routes/web.php` ফাইলটি সম্পাদনা করুন:

```php
use App\Http\Controllers\TaskController;

Route::get('/', [TaskController::class, 'index']);
Route::post('/tasks', [TaskController::class, 'store']);
Route::patch('/tasks/{task}', [TaskController::class, 'update']);
```

### ধাপ ৪: ভিউ তৈরি করা

`resources/views/tasks/index.blade.php` ফাইলটি তৈরি করুন:

```blade
<!DOCTYPE html>
<html>
<head>
    <title>টাস্ক ম্যানেজার</title>
</head>
<body>
    <h1>টাস্ক ম্যানেজার</h1>

    <form method="POST" action="/tasks">
        @csrf
        <input type="text" name="title" placeholder="নতুন টাস্ক" required>
        <button type="submit">যোগ করুন</button>
    </form>

    <ul>
        @foreach ($tasks as $task)
            <li>
                <form method="POST" action="/tasks/{{ $task->id }}">
                    @csrf
                    @method('PATCH')

                    <label>
                        <input type="checkbox" onchange="this.form.submit()" {{ $task->is_completed ? 'checked' : '' }}>
                        {{ $task->title }}
                    </label>
                </form>
            </li>
        @endforeach
    </ul>
</body>
</html>
```

### ধাপ ৫: মডেল ম্যাস অ্যাসাইনমেন্টের জন্য সেট আপ করা

`Task` মডেলটি সম্পাদনা করুন:

```php
// app/Models/Task.php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Task extends Model
{
    use HasFactory;

    protected $fillable = ['title', 'is_completed'];
}
```

---

## সারাংশ

নূন্যতম এবং সরল কোড দিয়ে, আমরা একটি টাস্ক ম্যানেজমেন্ট সিস্টেম তৈরি করেছি যা KISS নীতিমালার সাথে সামঞ্জস্যপূর্ণ। এই উদাহরণটি অপ্রয়োজনীয় বিমূর্ততা এড়িয়ে যায় এবং কোড সহজে পড়া এবং রক্ষণাবেক্ষণের উপযোগী রাখে।

---

## KISS নীতিমালার সুবিধা

- **সহজে বোঝা যায়**: নতুন ডেভেলপাররা দ্রুত কোডবেস বুঝতে পারে।
- **রক্ষণাবেক্ষণযোগ্যতা**: সরল কোড বাগপ্রবণ কম এবং সহজে ডিবাগ করা যায়।
- **সম্প্রসারণযোগ্যতা**: পরিষ্কার এবং সরল কোড উল্লেখযোগ্য রিফ্যাক্টরিং ছাড়াই সম্প্রসারণ করা যায়।

সরলতা বজায় রেখে, আপনি নিশ্চিত করতে পারেন যে আপনার Laravel অ্যাপ্লিকেশনটি কাজ করার জন্য আনন্দদায়ক থাকে।
