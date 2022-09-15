---
layout: post
title:  "Laravel | Phân trang"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/pagination.png
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



Tham khảo: [https://laravel.com/](https://laravel.com/docs/5.4/pagination), [https://viblo.asia/](https://viblo.asia/p/tim-hieu-ve-pagination-trong-laravel-3P0lPMrn5ox)