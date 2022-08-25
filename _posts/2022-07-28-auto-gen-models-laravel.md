---
layout: post
title:  "Laravel | Tự động sinh Models trong Laravel"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/models.jpg
tags: [laravel, autogencode, models]
---

## Models

Như đã được nói ở [Bài 1](https://hacerweb.github.io/structure-folder-laravel/), thì `Models` là nơi chứa các lớp đại diện cho cơ sở dữ liệu. Với Laravel, trừ các bảng là các bảng nối giữa 2 bảng có mỗi quan hệ (nhiều - nhiều) thì mỗi bảng sẽ được đại diện bằng một `class` ở trong thư mục `Models`. Việc tạo models cho project là việc khá máy móc và chán nản. Trong những trường hợp như này, một thư viện tự động sinh các model là khả thi và rất hữu ích. Dĩ nhiên là không ai cần phải tự viết, hầu như mọi thứ đã có hết trên internet, chỉ cần lên mạng gõ và mang về thư viện phù hợp nhất thôi. Trong bài viết này, mình sẽ giới thiệu về thư viện [https://github.com/reliese/laravel](https://github.com/reliese/laravel).

Cài đặt thư viện:

```bash
composer require reliese/laravel --dev
```

Thêm file `models.php` vào thư mục `config/` và xóa cache: 

```bash
php artisan vendor:publish --tag=reliese-models
php artisan config:clear
```

## Sử dụng thư viện

Sử dụng thư viện cũng khá là đơn giản, chạy command sau để sinh Models:

```bash
php artisan code:models
```

Models sinh ra sẽ được tìm thấy trong thư mục `app/Models`. Nếu chỉ muốn sinh các model từ các bảng được chỉ định thì sử dụng command:

```bash
php artisan code:models --table=users
```

## Custom lại thư viện

Nếu bạn muốn dừng ở việc là sử dụng thư viện thì có thể dừng ở đây rồi, còn nếu bạn muốn tìm hiểu thêm cách custom lại thư viện để phục vụ cho các mục đích cao hơn của bản thân thì có thể xem tiếp phần mình giới thiệu sau đây. Trong phần này mình muốn nói tới việc custom lại thư viện để có thể sinh được các models theo ý của mình.

Những gì khó nhất thì các tác giả đã làm rồi, tuy nhiên thì việc sinh mặc dù theo chuẩn chung nhưng đôi lúc lại không đúng với ý từng người. Ví dụ nếu muốn thêm vào các models một `constants` `ORDERABLE` để đánh đấu các trường được phép sắp xếp thì sao. Trong phần này mình cũng sẽ lấy việc thêm vào các model một `constants` là `ORDERABLE` để minh họa việc custom lại code của thư viện phục vụ đúng mục đích trong project của mình.

Đầu tiên chúng ta sẽ xem qua cấu trúc thư mục của thư viện này:

![](https://hacerweb.github.io/assets/images/reliese.png)

### Ý tưởng của thư viện

Một template sẽ được định nghĩa sẵn trong thư mục `Model/Templates/`, khi tạo một file, thư viện sẽ đọc file template này trước, sau đó phân tích cơ sở dữ liệu và khớp vào các vị trí tương ứng.

### Custom thư viện

Để custom một cách tùy thích thì chúng ta có thể sử dụng `comment` trong cơ sở dữ liệu. Trong cơ sở dữ liệu mỗi trường luôn có 1 ô `comment` để mô tả trường đó, chúng ta có thể tận dụng để dùng nó làm trường đánh dấu. Ví dụ trường `avatar` sẽ được lưu trữ như sau:

![](https://hacerweb.github.io/assets/images/comment_orderable.png)

Giờ hãy xem cách thư viện lấy và tạo ra các file models. Chúng ta sẽ đi từ file `CodeModelsCommand.php` trước, hàm `handle` là nơi khởi tạo `Factory` để tạo kết nối để xử lý sinh code.

Trong lớp `Factory`, để ý tới hàm `create()`, ở đây là nơi thư viện sẽ tạo model, tạo template, và khớp model vào template để sinh ra file model tương ứng.

model sẽ đại diện qua đối tượng `Models`, tại đây, chúng ta tạo thêm 3 phương thức nữa để kiểm tra `ORDERABLE`:

```php
/**
 * @param array $comment
 * @return bool
 */
public function isOrderable($comment){
    if($comment == null){
        return false;
    }

    if(array_key_exists('orderable', $comment)){
        if($comment['orderable'] == true){
            return true;
        }
    }

    return false;
}

/**
 * @return bool
 */
public function hasOrderable(){
    return ! empty($this->orderable);
}

/**
 * @return array
 */
public function getOrderable(){
    return $this->orderable;
}
```

Trong hàm `parseColumn()` sẽ là nơi để lấy và chuyển đối các dữ liệu lấy từ cơ sở dữ liệu, ở đây mình sẽ thêm một dòng để lấy ra ô `comment` dưới dạng array:

```php
$comment = (array) json_decode($column->comment);
```

Sau đó kiểm tra và gán vào một thuộc tính trong đối tượng `Model` để đánh dấu:

```php
if($this->isOrderable($comment)){
    $this->orderable[] = $propertyName;
}
```

Sau khi lấy được dữ liệu từ cơ sở dữ liệu bỏ vào đối tượng `Model`, tiếp theo là lấy ra template. Như mình đã nói ở ý tưởng của thư viện này ở phần trước, template sẽ để trong thư mục `Model/Templates/` và sau khi lấy được template thư viện sẽ khớp các dữ liệu trong `Model` với template, chính là dòng gọi hàm `$file = $this->fillTemplate($template, $model);` trong lớp `Factory`.

Chúng ta sẽ để ý tới phần xây dựng `body` trong hàm `body()`, ở đây chúng ta thêm vào như sau để có thể thêm `ORDERABLE` cho các model được sinh ra trong cơ sở dữ liệu: 

```php
if($model->getOrderable() != null){
    $body .= $this->class->constant('ORDERABLE', $model->getOrderable());
    $excludedConstants[] = $model->getOrderable();
}
```

Vậy là đã xong, chạy lại câu lệnh sinh code và nhìn lại thành quả:

```php
<?php

/**
 * Created by Reliese Model.
 */

namespace App\Models;

use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Model;

/**
 * Class Album
 * 
 * @property int $id
 * @property string|null $avatar
 * @property string|null $album_name
 * 
 * @property Collection|Photo[] $photos
 *
 * @package App\Models
 */
class Album extends Model
{
    const ORDERABLE = [
        'avatar',
        'album_name'
    ];
    protected $table = 'album';
    public $timestamps = false;

    protected $hidden = [
        'avatar',
        'album_name'
    ];

    protected $fillable = [
        'avatar',
        'album_name'
    ];

    public function photos()
    {
        return $this->hasMany(Photo::class, 'id_album');
    }
}

```

Nên nhớ là bạn phải lưu code bạn đã sửa lại này ở đâu đó, hoặc là tạo một thư viện mới fork từ repo của tác giả để dùng lại. Dựa trên ý tưởng của thư viện này, mình cũng tự tạo một thư viện sinh controller, các bạn có thể xem tại [https://github.com/trannguyenhan/autogencontroller](https://github.com/trannguyenhan/autogencontroller). Cài đặt thông qua `composer` như sau: 

```bash
composer require trannguyenhan/autogencontroller
```

Dĩ nhiên là không giống với sinh model, với controller thì mỗi người có một cách thiết kế khác nhau vì vậy các bạn có thể custom lại thư viện của mình để phù hợp với cách thiết kế của các bạn.