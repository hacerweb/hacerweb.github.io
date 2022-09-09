---
layout: post
title:  "Biến cục bộ cho mọi request trong controller"
author: trannguyenhan
categories: [ Laravel ]
tags: [laravel, request]
image: assets/images/global.png
---

Trong một ứng dụng sẽ có những biến phải sử dụng đi sử dụng lại nhiều lần, ví dụ như trong một dự án đa ngôn ngữ thì chắc chắn phải luôn có một tham số `lang` truyền về để backend biết và trả ra nội dung có ngôn ngữ tương ứng. Vậy trong một controller việc lúc nào cũng lấy đi lấy lại tham số `lang` sẽ rất mất thời gian và khá là vô vị. Laravel cung cấp một hàm trong `helpers` giúp làm việc này dễ dàng hơn đó là hàm `request()`. Trong một controller sẽ luôn lấy giá trị `lang` được gửi về từ phía người dùng bằng cách như sau:

```php
private $lang = "en";
public function __construct(){
    if(!empty(request()->header('lang'))){
        $this->lang = request()->header('lang');
    }
}
```

Sau đó ở mỗi hàm chỉ cần gọi tới biến `$this->lang` để lấy ngôn ngữ mà client gọi: 

```php
<?php

namespace App\Http\Controllers\Api\V3\Blog;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class BlogController extends Controller
{
    private $lang = "en";
    public function __construct(){
        if(!empty(request()->header('lang'))){
            $this->lang = request()->header('lang');
        }
    }

    public function getBlog(){
        return \App\Blog::all()->map(function($item){
            return \App\Http\Mapper::toBlogApiV3($item, $this->lang);
        });
    }
}
```