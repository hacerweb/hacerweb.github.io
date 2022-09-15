---
layout: post
title:  "Laravel | Phân trang"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/pagination.png
tags: [featured, laravel, pagination]
---

## Phân trang cho web

Với Laravel có một vài cách để phân trang, đơn giản nhất Laravel đã cung cấp một hàm paginate của Query Builder hoặc trong Eloquent.  Mặc định, trang hiện tại được nhận biết thông qua giá trị `?page` trên query string trên HTTP request và nó được tự động nhận biết bởi Laravel và cũng được tự động thêm vào các link sinh ra bởi paginator.

Chúng ta có một ví dụ về danh sách user như sau, trong `Controller`:

```php
<?php

namespace App\Http\Controllers\Pagination;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class PaginationController extends Controller
{
    public function index()
    {
        $users = DB::table('users')->paginate(3); // DB::table('users')->simplePaginate(3);
        return view('pagination.user', ['users' => $users]);
    }
}

```

Trong file `.blade`:

```php
<!DOCTYPE>
<html lang="en">
<head>
    <title>Laravel Example</title>
</head>
<body>
<div>
    <div class="container">
        @foreach ($users as $user)
            {{$user->id}}. {{ $user->username }} <br />
        @endforeach
    </div>

    <div>{{ $users->links() }}</div>
</div>
</body>
</html>
```

Sử dụng `simplePaginate` nếu chỉ cần hiển thị hai link đơn giản "Next" và "Previous" trên pagination view. và dưới đây là kết quả thu được: 

![](https://hacerweb.github.io/assets/images/pagination2.png)

(Nhấn Previou, Next để sang và quay lại trang tiếp theo)

Ngoài ra có 1 số phương thức trong paginator có thể sử dụng như sau:

```php
$results->count()
$results->currentPage()
$results->firstItem()
$results->hasMorePages()
$results->lastItem()
$results->lastPage() (Not available when using simplePaginate)
$results->nextPageUrl()
$results->perPage()
$results->previousPageUrl()
$results->total() (Not available when using simplePaginate)
$results->url($page)
```

## Phân trang cho REST API

### Sử dụng Laravel paginator

Chúng ta có thể convert một đối tượng paginator sang `JSON` bằng cách `return` trực tiếp nó từ một `route` hay một `Controller`:

```php
Route::get('users', function () {
    return App\User::paginate();
});
```

Ví dụ về một kết quả trả về được tạo bởi paginator:

```json
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}
```

### Sử dụng take và skip

Ngoài ra, với API để mọi thứ linh hoạt hơn chúng ta có thể sử dụng `take` và `skip` để tự phân trang.

Chúng ta cần 2 tham số ở client truyền từ phía client về để xác định số item trong 1 trang và số trang, thường chúng sẽ được truyền trên url luôn và chúng ta có thể lấy nó ra như sau:

```php
$perPage = $request->query('per_page');
$page = $request->query('page');
```

Chúng ta có thể thêm vào các giá trị mặc định cho `$page` hoặc `$perPage`, hoặc nếu không có thế lấy hết tất cả bản ghi nếu tham số truyền về là `null`.

Tùy thuộc vào việc frontend sẽ hiển thị như nào mà chúng ta sẽ custom kết quả cho hợp lý. Nếu như frontend hiển thị số trang thì chúng ta phải truyền thêm về `total` là tổng số item để frontend có thể tính toán hiển thị số trang, và `total` phải tính trước khi phân trang như sau: 

```php
$count = User::query()->count();
$data = User::query()->take($perPage)->skip(($page - 1) * $perPage)->get();

return response()->json([
    'total' => $count,
    'data' => $data
]);
```

Kết quả sau khi gọi bằng postman sẽ như sau:

![](https://hacerweb.github.io/assets/images/pagination2-api.png)



Tham khảo: [https://laravel.com/](https://laravel.com/docs/5.4/pagination), [https://viblo.asia/](https://viblo.asia/p/tim-hieu-ve-pagination-trong-laravel-3P0lPMrn5ox)