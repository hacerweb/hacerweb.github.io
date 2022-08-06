---
layout: post
title:  "Laravel | Cấu trúc thư mục trong Laravel"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/file-structure.png
tags: [summer]
---
Khi tìm hiểu về một framework thì mình thường tìm hiểu tới các tổ chức của framework đó đầu tiên, nó giúp chúng ta hiểu hơn về framework, cách triển khai cũng như nhiều cái khác nữa. Laravel chia cấu trúc thư mục cũng khá tường minh và phân nhiệm vụ cũng khá rõ ràng, cấu trúc thư mục gốc của Laravel sẽ giống như sau:

```bash
Project
    - app/
        - Console/
            - Kernel.php
        - Exceptions/
            - Handler.php
        - Http/
            - Controllers/
            - Middleware/
            - Repositories/
            - Requests/
            - Kernel.php
        - Models/
        - Providers/
    - bootstrap/
    - config/
    - database/
    - public/
    - resources/
    - routes/
    - storage/
    - test/
    - vendor/
    - .env
    - artisan
    - composer.json
    - server.php
    - package.json
```

- `app/`: Thư mục này chứa hầu hết code của chúng ta trong ứng dụng.
- `bootstrap/`: Thư mục `bootstrap` chứa file `app.php` khởi động framework. Thư mục này cũng chứa một thư mục `cache` chứa các tệp được tạo khung để tối ưu hóa hiệu suất. Chúng ta thường không cần phải sửa đổi bất kỳ tệp nào trong thư mục này.
- `config/`: Như tên của nó đã khá rõ ràng, nó chứa các thư mục cấu hình của ứng dụng.
- `database/`: chứa các `seeds`, `migrations`, `factories`.
- `public/`: là một public dictionary, chứ file `index.php` là đầu vào cho mọi request. Các file `css`, `js` hay `images` đều được đặt ở đây. Thư mục này là công khai vì vậy mọi file bạn đặt trong đây đều có thể được truy cập từ bên ngoài.
- `resources/`: chứa các `views` được định nghĩa và các nội dung CSS, Javascript chưa được biên dịch.
- `routes/`: chứa toàn bộ các đường dẫn được định nghĩa trong ứng dụng, `web.php` định nghĩa các đường dẫn web, `api.php` định nghĩa các đường dẫn API.
- `storage/`: nơi đây chứa các file log ứng dụng của bạn, các file blade đã biên dịch, các file session, file cache, các file được gen bởi framework. Nơi đây cũng là nơi lưu trữ các file của lập trình viên.
- `tests/`: chứa các automated tests.
- `vendor/`: thư mục này chứa các thư viện cài thông qua `composer`.
- `.env`: đây là file cấu hình chung của hệ thống, thường file này sẽ nằm trong `.gitignore`, vì mỗi môi trường sẽ có cấu hình trong file `.env` là khác nhau. Thông thường thì có 1 file `.env.example` cho mỗi dự án làm khuôn mẫu trước, khi clone dự án về, việc đầu tiên là copy file `.env.example` ra file `.env` và cấu hình lại cho đúng.
- Các thư mục trong `app/` sẽ là nơi chúng ta làm việc nhiều nhất nên mình sẽ nói kỹ hơn vào các bài viết chi tiết nói tới từng phần nha.


Tham khảo: [https://laravel.com/docs/9.x/](https://laravel.com/docs/9.x/structure#the-storage-directory)