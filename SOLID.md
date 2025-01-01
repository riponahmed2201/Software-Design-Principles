# SOLID প্রিন্সিপালস: 🏛️

- Single Responsibility: একটা কাজ একটা ক্লাস
- Open/Closed: মডিফাই নয়, এক্সটেন্ড করুন
- Liskov Substitution: চাইলেই রিপ্লেস করা যাবে
- Interface Segregation: ছোট ছোট ইন্টারফেস
- Dependency Inversion: ডিপেন্ড অন অ্যাবস্ট্রাকশন

# SOLID Principles with Examples in PHP Laravel

The SOLID principles are five key design principles that make your code more maintainable, scalable, and easier to understand. Each principle focuses on a particular aspect of good software design. Below is an in-depth explanation of each principle with practical examples using PHP and Laravel.

---

## 1. Single Responsibility Principle (SRP)

**Definition:** A class should have one, and only one, reason to change. In other words, a class should only have one job. (একটি ক্লাসের শুধুমাত্র একটি কারণেই পরিবর্তন হওয়া উচিত। অর্থাৎ, একটি ক্লাস কেবলমাত্র একটি কাজ করবে।)

### Example:

Imagine a user management system where you need to create a user and send a welcome email. Instead of doing everything in a single class, we split responsibilities:

```php
class UserService {
    public function createUser($data) {
        // Logic for user creation
        $user = User::create($data);
        return $user;
    }
}

class NotificationService {
    public function sendWelcomeEmail($user) {
        // Logic to send email
        Mail::to($user->email)->send(new WelcomeEmail($user));
    }
}

// Controller Example
$userService = new UserService();
$notificationService = new NotificationService();

$user = $userService->createUser($request->all());
$notificationService->sendWelcomeEmail($user);
```

Here, the `UserService` handles user creation, while the `NotificationService` handles email sending. This separation ensures each class has a single responsibility.

---

## 2. Open/Closed Principle (OCP)

**Definition:** A class should be open for extension but closed for modification. You should be able to add new functionality without altering existing code.(কোড পরিবর্তনের পরিবর্তে এক্সটেনশন এর মাধ্যমে নতুন ফিচার যোগ করা উচিত।)

### Example:

A payment processing system where different payment methods are used:

```php
interface PaymentMethod {
    public function pay($amount);
}

class CreditCardPayment implements PaymentMethod {
    public function pay($amount) {
        echo "Paid $amount using Credit Card.";
    }
}

class PaypalPayment implements PaymentMethod {
    public function pay($amount) {
        echo "Paid $amount using PayPal.";
    }
}

class PaymentService {
    public function processPayment(PaymentMethod $paymentMethod, $amount) {
        $paymentMethod->pay($amount);
    }
}

// Controller Example
$paymentService = new PaymentService();
$paymentService->processPayment(new CreditCardPayment(), 100);
$paymentService->processPayment(new PaypalPayment(), 200);
```

Adding a new payment method like `StripePayment` would only require creating a new class implementing `PaymentMethod`, without modifying existing code.

---

## 3. Liskov Substitution Principle (LSP)

**Definition:** Objects of a superclass should be replaceable with objects of a subclass without altering the correctness of the program. (একটি সাবক্লাস তার প্যারেন্ট ক্লাসের পরিবর্তে ব্যবহার করা যেতে হবে।)

### Example:

Consider a class hierarchy of shapes:

```php
class Rectangle {
    protected $width;
    protected $height;

    public function setWidth($width) {
        $this->width = $width;
    }

    public function setHeight($height) {
        $this->height = $height;
    }

    public function getArea() {
        return $this->width * $this->height;
    }
}

class Square extends Rectangle {
    public function setWidth($width) {
        $this->width = $width;
        $this->height = $width;
    }

    public function setHeight($height) {
        $this->width = $height;
        $this->height = $height;
    }
}

function printArea(Rectangle $shape) {
    $shape->setWidth(4);
    $shape->setHeight(5);
    echo $shape->getArea();
}

printArea(new Rectangle()); // Output: 20
printArea(new Square());    // Output: 16 (Problem: LSP violated)
```

The `Square` class breaks LSP because it does not behave like its superclass `Rectangle`. To fix this, avoid overriding behavior that changes the fundamental contract of the parent class.

---

## 4. Interface Segregation Principle (ISP)

**Definition:** A class should not be forced to implement interfaces it does not use. Instead of one large interface, create smaller, more specific interfaces. (একটি ক্লাসের জন্য অপ্রয়োজনীয় মেথডসমূহ ইন্টারফেসে থাকা উচিত নয়।)

### Example:

A worker management system:

```php
interface Worker {
    public function work();
}

interface Manager {
    public function manage();
}

class Developer implements Worker {
    public function work() {
        echo "Writing code.";
    }
}

class ProjectManager implements Manager {
    public function manage() {
        echo "Managing project.";
    }
}
```

Here, `Developer` is not forced to implement `manage()` and `ProjectManager` is not forced to implement `work()`. Each interface is specific to the role. (এখানে Developer শুধুমাত্র কাজ করে এবং ProjectManager শুধু প্রজেক্ট ম্যানেজ করে। দুইটি আলাদা ইন্টারফেস থাকার কারণে দায়িত্ব স্পষ্ট।)

---

## 5. Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details, and details should depend on abstractions. (হাই-লেভেল মডিউলসমূহ লো-লেভেল মডিউলের উপর নির্ভর করবে না, বরং অ্যাবস্ট্রাকশনের উপর নির্ভর করবে।)

### Example:

A notification system:

```php
interface MessageSender {
    public function send($message);
}

class EmailSender implements MessageSender {
    public function send($message) {
        echo "Sending email: $message";
    }
}

class SmsSender implements MessageSender {
    public function send($message) {
        echo "Sending SMS: $message";
    }
}

class Notification {
    protected $messageSender;

    public function __construct(MessageSender $messageSender) {
        $this->messageSender = $messageSender;
    }

    public function sendNotification($message) {
        $this->messageSender->send($message);
    }
}

// Controller Example
$notification = new Notification(new EmailSender());
$notification->sendNotification("Hello via Email!");

$notification = new Notification(new SmsSender());
$notification->sendNotification("Hello via SMS!");
```

The `Notification` class depends on the `MessageSender` abstraction, not specific implementations like `EmailSender` or `SmsSender`. This makes it easy to extend and maintain.

---

## Conclusion

Following the SOLID principles in Laravel or any other framework improves the quality of your code by making it:

- Easier to understand
- More maintainable
- Scalable for future changes

By adhering to these principles, you ensure that your codebase remains robust, clean, and adaptable to new requirements.
