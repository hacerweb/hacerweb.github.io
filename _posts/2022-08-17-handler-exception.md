---
layout: post
title:  "Laravel | Xử lý ngoại lệ"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/handler_exception.png
tags: [featured, laravel, handler-exception]
---

Trong ứng dụng không thể tránh khỏi ít nhiều gặp các ngoại lệ, giờ chúng ta muốn khi gặp ngoại lệ thì sẽ có một thông báo phù hợp để người dùng biết được là lỗi tại sao mà hông làm lộ ra các thông tin cấu hình trên ứng dụng. Ví dụ, một exception hay gặp nhất là `NotFoundHttpException`. Khi người dùng gõ sai đường dẫn API, nếu debug mode là `true` thì Laravel sẽ hiển thị lỗi như sau:

![](https://hacerweb.github.io/assets/images/notfoundexceptiondebugtrue.png)

Còn nếu debug mode là `false` thì Laravel sẽ hiển thị lỗi như sau:

![](https://hacerweb.github.io/assets/images/notfoundexceptiondebugfalse.png)

Chúng ta thấy là khi debug mode là `true` thì nó sẽ hiển thị rõ lỗi cho chúng ta thấy, tuy nhiên chỉ có thể để debug mode là `true` khi ở môi trường dev hoặc local. Vì nếu để debug mode là `true` ở môi trường product thì sẽ bị lộ các thông tin cấu hình và điều đó dẫn tới hệ thống dễ bị các hacker xâm nhập. Nhưng như ảnh trên, khi debug mode là `false` nó sẽ có một kết quả chung chung mà người dùng không thể biết chính xác là gì. Chúng ta cần custom lại response trả ra để người dùng có thể biết được chính xác lôĩ xảy ra hoặc là người dùng biết được những gì mà chúng ta muốn người dùng biết.

Trong Laravel khi một `Exception` xảy ra, tất cả đều đi qua class `App\Exceptions\Handler`. Class này sẽ làm nhiệm vụ ghi log và hiển thị kết quả cho người dùng. Và chúng ta cũng có thể dễ dàng custom lại lỗi thông qua `class` này.

Ví dụ mình sẽ bắt và xử lý 2 ngoại lệ là không có quyền và đường dẫn không tồn tại như sau:

```php
<?php

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Validation\UnauthorizedException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Throwable;

class Handler extends ExceptionHandler
{
    /**
     * A list of the exception types that are not reported.
     *
     * @var array<int, class-string<Throwable>>
     */
    protected $dontReport = [
        //
    ];

    /**
     * A list of the inputs that are never flashed for validation exceptions.
     *
     * @var array<int, string>
     */
    protected $dontFlash = [
        'current_password',
        'password',
        'password_confirmation',
    ];

    /**
     * Register the exception handling callbacks for the application.
     *
     * @return void
     */
    public function register()
    {
        $this->reportable(function (Throwable $e) {
            //
        });

        $this->renderable(function (UnauthorizedException $e, $request){
            return response()->json([
                'message' => 'Không có quyền'
            ], 403);
        });

        $this->renderable(function (NotFoundHttpException $e, $request) {
            return response()->json([
                'message' => 'Không có API này'
            ], 404);
        });
    }
}
```

Kết quả sau khi exception đã được bắt và xử lý sẽ như sau:

![](https://hacerweb.github.io/assets/images/handler_exception_handler.png)

Với bất kì exception nào chỉ cần bắt và xử lý tương tự như trên, để xem chi tiết hơn bạn có thể truy cập vào link [GITHUB](https://github.com/hacerweb/laravel-example/blob/master/app/Exceptions/Handler.php).

Tham khảo: [https://laravel.com/docs/9.x/](https://laravel.com/docs/9.x/errors#reporting-exceptions), [https://toidicode.com/](https://toidicode.com/xu-ly-loi-trong-laravel-8-461.html)

