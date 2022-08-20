---
layout: post
title:  "Laravel | Toàn bộ về middleware trong Laravel"
categories: [ Laravel ]
image: assets/images/middleware.jpg
---

## Middleware là gì?

Middleware là phần mềm trung gian, nó nằm ở giữa trong các request của người dùng và controller, có nhiệm vụ kiểm tra và lọc các yêu cầu. Dĩ nhiên là chúng ta hoàn toàn có thể kiểm tra và lọc ở controller nhưng tính tái sử dụng sẽ không cao khi chúng ta muốn sử dụng middleware cho những yêu cầu khác ở những route khác.

## Tạo một middleware

Tạo middleware qua câu lệnh `artisan`: 

```bash
php artisan make:middleware EnsureTokenIsValid
```

Middleware tạo ra sẽ được tìm thấy trong thư mục `app/Http/Middleware`, chúng ta viết code cho middleware chỉ cần viết trong hàm `handle`, khi hàm trả về: 

```php
return $next($request);
```

tức là truy vấn của người dùng đã thông qua middleware, nếu có lỗi ví dụ như là truy cập vào tài nguyên có quyền hạn không cho phép thì chúng ta có thể trả về như sau: 

```php
return response()->json(['message' => 'Unauthorized'], 401);
```

Middleware sau khi tạo xong giờ chúng ta sẽ đi đăng kí chúng tại `app/Http/Kernel.php` để chúng có thể được sử dụng. Chúng ta cũng có thể thấy ngay một số các `middleware` như sau:

- Đầu tiên là các `middleware` được khai báo trong biến `$middleware`, đây là các `middleware` dành cho mọi `route`, tức là bất kì `route` nào được khai báo thì mọi `request` tới các `route` này đều sẽ đi qua các `middleware` này.


```php
protected $middleware = [
    TrustProxies::class,
    CheckForMaintenanceMode::class,
    ValidatePostSize::class,
    TrimStrings::class,
    ConvertEmptyStringsToNull::class,
    HandleCors::class,
    TerminatingMiddleware::class
];
```

- Các `middleware` được khai báo trong biến `$middlewareGroups` để nhóm lại các middleware một cách dễ dàng hơn. Như chúng ta thường thấy một số các api thường được nhóm lại và sử dụng:


```php
Route::group([
    'middleware' => ['api', 'auth:api']
], function(){});
```

Như vậy là các route trong group này đều sẽ đi qua một số các middleware thuộc nhóm middleware `api`.

- Các `middleware` được khai báo trong `$routeMiddleware` được dùng cho từng route cụ thể mà người dùng chỉ định.

## Tính thời gian phản hồi của 1 request với Middleware và LARAVEL_START

Các bạn để ý thì trong file `public/index.php` thì có một hằng `LARAVEL_START` được định nghĩa: 

```php
define('LARAVEL_START', microtime(true));
```

Vì thế đây là câu lệnh đầu tiên được thực hiện khi một request được gửi tới từ phía máy khách, nó đánh dấu thời gian bắt đầu nhận request.

Vậy `LARAVEL_START` có thể sử dụng để tính thời gian phản hồi của một request hoặc là thời gian cho tới một thời điểm nào đó.

Để tính thời gian phản hồi chúng có có thể sử dụng kết hợp thêm hàm `terminate` của `Middleware`, tạo một `TerminatingMiddleware` như sau: 

```php
<?php  
  
namespace App\Http\Middleware;  
  
use App\Jobs\LogFileJob;  
use Closure;  
use Symfony\Component\HttpFoundation\Request;  
use Symfony\Component\HttpFoundation\Response;  
use Symfony\Component\HttpKernel\TerminableInterface;  
  
class TerminatingMiddleware implements TerminableInterface  
{  
    protected $startTime;  
  
    /**  
     * Handle an incoming request.     
     *     
     * @param \Illuminate\Http\Request $request  
     * @param Closure $next  
     * @return mixed  
     */    
    public function handle(\Illuminate\Http\Request $request, Closure $next)  
    {        
        return $next($request);  
    }  
  
    public function terminate(Request $request, Response $response)  
    {        
        $totalTimeRequest = microtime(true) - LARAVEL_START;
    }
}
```

sau đó thêm `TerminatingMiddleware` vào `$middleware` tại file `app\Http\Kernel.php` để bất cứ request nào cũng đi qua `Middleware` này: 

```php
protected $middleware = [  
    TrustProxies::class,  
    CheckForMaintenanceMode::class,  
    ValidatePostSize::class,  
    TrimStrings::class,  
    ConvertEmptyStringsToNull::class,  
    HandleCors::class,  
    TerminatingMiddleware::class  
];
```

Cuối cùng bạn có thể lưu `$totalTimeRequest` lại đâu đó như database hay log. Nhưng có một vấn đề các bạn cần để ý khi xử lý như những vấn đề liên quan tới hiệu năng khi mà chúng ta thêm vào trong quá trình trả ra response cho người dùng một đoạn xử lý.

