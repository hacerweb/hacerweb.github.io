---
layout: post
title:  "Laravel | Thêm file route tùy chỉnh"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/routes.png
tags: [route, laravel]
---
Mọi người chắc chắn đã quen với việc các route liên quan tới web thì viết trong `routes/web.php`, còn route liên quan tới api thì viết trong `routes/api.php`. Tuy nhiên, khi dự án lớn dần thì việc viết chung tất cả vào 1 file cũng không phải là cách hay, chúng ta có thể chia chúng ra thành từng file với chức năng riêng biệt để có thể dễ quản lý và sửa đổi hơn. Mình sẽ giới thiệu 2 cách để có thể làm việc này một cách dễ dàng hơn.

## Sử dụng `RouteServiceProvider`

File `RouteServiceProvider.php` mặc định chúng ta sẽ thấy ngay 2 file mặc định `web.php` và `api.php` được định nghĩa: 

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Route;
use Illuminate\Foundation\Support\Providers\RouteServiceProvider as ServiceProvider;

class RouteServiceProvider extends ServiceProvider
{
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Http\Controllers';

    /**
     * Define your route model bindings, pattern filters, etc.
     *
     * @return void
     */
    public function boot()
    {
        //

        parent::boot();
    }

    /**
     * Define the routes for the application.
     *
     * @return void
     */
    public function map()
    {
        $this->mapApiRoutes();

        $this->mapWebRoutes();

        //
    }

    /**
     * Define the "web" routes for the application.
     *
     * These routes all receive session state, CSRF protection, etc.
     *
     * @return void
     */
    protected function mapWebRoutes()
    {
        Route::middleware('web')
             ->namespace($this->namespace)
             ->group(base_path('routes/web.php'));
    }

    /**
     * Define the "api" routes for the application.
     *
     * These routes are typically stateless.
     *
     * @return void
     */
    protected function mapApiRoutes()
    {
        Route::prefix('api')
             ->middleware('api')
             ->namespace($this->namespace)
             ->group(base_path('routes/api.php'));
    }
}

```

Để định nghĩa thêm chúng ta cũng sẽ làm tương tự, ví dụ mình muốn tạo một file để lưu trữ các route của admin riêng thì sẽ tạo ra 1 function tương tự như 2 function chứa `web.php` và `api.php`:

```php
protected function mapAdminRoutes()
{
    Route::middleware('web')
        ->namespace($this->namespace)
        ->group(base_path('routes/admin.php'));
}
```

sau đó khai báo chúng vào trong hàm map:

```php
public function map()
{
    $this->mapAdminRoutes();
    $this->mapApiRoutes();
    $this->mapWebRoutes();
}
```

Đã xong, giờ tất cả các route định nghĩa trong file `routes/admin.php` của bạn sẽ đều được quét qua.

## Tự tạo hàm 

Nếu như bạn không thích việc dùng `RouteServiceProvider` thì có thể tham khảo cách sau đây, đầu tiên hãy tạo ra một hàm và đặt nó trong `Helpers.php` để có thể gọi ở bất cứ đâu như sau:

```php
<?php

if (!function_exists('includeRouteFiles')) {

    /**
     * Loops through a folder and requires all PHP files
     * Searches sub-directories as well.
     *
     * @param $folder
     */
    function includeRouteFiles($folder)
    {
        try {
            $rdi = new recursiveDirectoryIterator($folder);
            $it = new recursiveIteratorIterator($rdi);

            while ($it->valid()) {
                if (!$it->isDot() && $it->isFile() && $it->isReadable() && $it->current()->getExtension() === 'php') {
                    require $it->key();
                }

                $it->next();
            }
        } catch (Exception $e) {
            echo $e->getMessage();
        }
    }
}

```

Sau đó ở các file `api.php` hay `web.php` các bạn thêm vào như sau: 

```php
Route::group(['namespace' => 'Statistic', 'prefix' => '/statistic', 'as' => 'statistic.'], function () {
    includeRouteFiles(__DIR__ . '/Statistic/');
});
```

Khi này, hàm `includeRouteFiles` có nhiệm vụ tìm tất cả các file trong thư mục `Statistic/` và nạp toàn bộ nội dung của nó vào file này, điều này giống việc mặc dù bạn viết các file route Statistic ra file riêng nhưng thực chất chúng vẫn đang được viết trong file `api.php` (hoặc `web.php`). Việc này giúp chúng ta quản lý rõ ràng hơn các route, dễ dàng chia nhỏ để tìm kiếm một cách dễ dàng hơn.