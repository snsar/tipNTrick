
```php
// 1. Replace nested if statements with early returns
// COMPLEX ❌
function processUser($user) {
    if ($user) {
        if ($user->isActive) {
            if ($user->hasPermission('admin')) {
                return doAdminStuff();
            } else {
                return doRegularStuff();
            }
        } else {
            throw new Exception('User not active');
        }
    } else {
        throw new Exception('No user found');
    }
}

// SIMPLE ✅
function processUser($user) {
    if (!$user) {
        throw new Exception('No user found');
    }
    if (!$user->isActive) {
        throw new Exception('User not active');
    }
    
    return $user->hasPermission('admin') 
        ? doAdminStuff() 
        : doRegularStuff();
}

// 2. Use array methods instead of loops
// COMPLEX ❌
$activeUsers = [];
foreach ($users as $user) {
    if ($user->isActive && $user->age >= 18) {
        $activeUsers[] = $user->name;
    }
}

// SIMPLE ✅
$activeUsers = array_map(
    fn($user) => $user->name,
    array_filter($users, fn($user) => $user->isActive && $user->age >= 18)
);

// 3. Use null coalescing and optional chaining
// COMPLEX ❌
$displayName = '';
if ($user && $user->profile && $user->profile->displayName) {
    $displayName = $user->profile->displayName;
} else {
    $displayName = 'Anonymous';
}

// SIMPLE ✅
$displayName = $user?->profile?->displayName ?? 'Anonymous';

// 4. Use object destructuring
// COMPLEX ❌
function createReport($data) {
    $title = $data['title'];
    $date = $data['date'];
    $author = $data['author'];
    // ... use variables
}

// SIMPLE ✅
function createReport($data) {
    ['title' => $title, 'date' => $date, 'author' => $author] = $data;
    // ... use variables
}

// 5. Use switch expressions instead of complex if-else chains
// COMPLEX ❌
function getStatusText($status) {
    if ($status === 'pending') {
        return 'Waiting for review';
    } elseif ($status === 'approved') {
        return 'Ready to process';
    } elseif ($status === 'rejected') {
        return 'Changes needed';
    } else {
        return 'Unknown status';
    }
}

// SIMPLE ✅
function getStatusText($status) {
    return match($status) {
        'pending' => 'Waiting for review',
        'approved' => 'Ready to process',
        'rejected' => 'Changes needed',
        default => 'Unknown status'
    };
}

// 6. Use default parameters instead of checking
// COMPLEX ❌
function createUser($name, $role, $active) {
    $role = $role ? $role : 'user';
    $active = isset($active) ? $active : true;
    // ... create user
}

// SIMPLE ✅
function createUser($name, $role = 'user', $active = true) {
    // ... create user
}

// 7. Use method chaining for cleaner operations
// COMPLEX ❌
$text = $string;
$text = trim($text);
$text = strtolower($text);
$text = str_replace(' ', '-', $text);

// SIMPLE ✅
$text = str_replace(' ', '-', strtolower(trim($string)));


// 8. Sử dụng Enums thay vì magic strings/numbers
// PHỨC TẠP ❌
function setUserStatus($status) {
    if ($status === 'active' || $status === 'inactive' || $status === 'pending') {
        $this->status = $status;
    }
}

// ĐƠN GIẢN ✅
enum UserStatus {
    case Active;
    case Inactive;
    case Pending;
}

function setUserStatus(UserStatus $status) {
    $this->status = $status;
}

// 9. Sử dụng Builder Pattern cho object phức tạp
// PHỨC TẠP ❌
$query = new Query();
$query->table = 'users';
$query->select = ['id', 'name', 'email'];
$query->where = ['active' => 1];
$query->orderBy = 'created_at';
$query->limit = 10;

// ĐƠN GIẢN ✅
$query = (new QueryBuilder())
    ->table('users')
    ->select(['id', 'name', 'email'])
    ->where(['active' => 1])
    ->orderBy('created_at')
    ->limit(10)
    ->build();

// 10. Sử dụng Named Arguments cho code dễ đọc hơn
// PHỨC TẠP ❌
createUser("John Doe", true, "admin", null, "EN", true);

// ĐƠN GIẢN ✅
createUser(
    name: "John Doe",
    isActive: true,
    role: "admin",
    department: null,
    language: "EN",
    sendEmail: true
);

// 11. Sử dụng Collection Methods thay vì logic phức tạp
// PHỨC TẠP ❌
$total = 0;
$count = 0;
foreach ($orders as $order) {
    if ($order->status === 'completed') {
        $total += $order->amount;
        $count++;
    }
}
$average = $count > 0 ? $total / $count : 0;

// ĐƠN GIẢN ✅
$completedOrders = collect($orders)
    ->filter(fn($order) => $order->status === 'completed');
    
$average = $completedOrders->average('amount');

// 12. Sử dụng Guard Clauses
// PHỨC TẠP ❌
function processOrder($order) {
    if ($order->isPaid()) {
        if ($order->hasStock()) {
            if ($order->isValid()) {
                // Xử lý đơn hàng
                return true;
            }
        }
    }
    return false;
}

// ĐƠN GIẢN ✅
function processOrder($order) {
    if (!$order->isPaid()) {
        throw new OrderException("Order is not paid");
    }
    
    if (!$order->hasStock()) {
        throw new OrderException("Products out of stock");
    }
    
    if (!$order->isValid()) {
        throw new OrderException("Order is invalid");
    }
    
    // Xử lý đơn hàng
    return true;
}

// 13. Sử dụng Null Object Pattern
// PHỨC TẠP ❌
function getDiscount($user) {
    if ($user && $user->membership) {
        return $user->membership->discount;
    }
    return 0;
}

// ĐƠN GIẢN ✅
class NullMembership {
    public function getDiscount() {
        return 0;
    }
}

class User {
    public function getMembership() {
        return $this->membership ?? new NullMembership();
    }
}

// 14. Sử dụng State Pattern cho logic phức tạp
// PHỨC TẠP ❌
class Order {
    public function process() {
        if ($this->status === 'new') {
            // xử lý đơn hàng mới
        } elseif ($this->status === 'pending') {
            // xử lý đơn hàng đang chờ
        } elseif ($this->status === 'completed') {
            // xử lý đơn hàng hoàn thành
        }
    }
}

// ĐƠN GIẢN ✅
interface OrderState {
    public function process(Order $order);
}

class NewOrderState implements OrderState {
    public function process(Order $order) {
        // xử lý đơn hàng mới
    }
}

class PendingOrderState implements OrderState {
    public function process(Order $order) {
        // xử lý đơn hàng đang chờ
    }
}
```

Here are the key takeaways for simplifying code:

1. Early Returns
- Exit the function as soon as possible
- Reduces nesting and makes flow clearer
- Handle error cases first

2. Use Modern Language Features
- Null coalescing (??)
- Optional chaining (?->)
- Arrow functions
- Match expressions

3. Simplify Conditions
- Combine related conditions
- Use built-in functions instead of manual checks
- Consider using switch/match for multiple conditions

4. Smart Default Values
- Use default parameters
- Use null coalescing for fallbacks
- Reduces explicit checks in code

5. Functional Approaches
- Use array functions instead of loops where possible
- Chain operations for data transformations
- Reduces temporary variables

6. Destructuring
- Unpack arrays/objects directly
- Assign multiple variables at once
- Makes working with complex data structures cleaner

7. Smart Operators
- Use ternary for simple if/else
- Use null coalescing for fallbacks
- Use spread operator for arrays

Would you like me to explain any of these techniques in more detail or show examples in a different programming language?
