---
layout: post
title:  "Laravel | Laravel permission"
author: trannguyenhan
categories: [ Laravel ]
image: assets/images/laravel-permission.png
tags: [featured, laravel, laravel-permission, spatie]
---

## Phân quyền người dùng

Trong một hệ thống được sử dụng bởi nhiều người với nhiều vai trò khác nhau như quản trị viên, đối soát viên, người dùng,... thì việc phân quyền là khá quan trọng. Việc phân quyền ở đây là để giới hạn lại quyền hạn của một hoặc một nhóm người dùng. Ví dụ như mỗi người dùng trừ quản trị viên thì chỉ nhìn được các dữ liệu của chính nó (cái này gọi là phân quyền theo chiều ngang), hay mỗi người dùng hoặc một nhóm người dùng sẽ có quyền truy cập vào một số các tài nguyên được chỉ định (cái này gọi là phân quyền theo chiều dọc). Phân quyền trong bài viết này được nói tới ở các phần sau là phân quyền theo chiều dọc.

Hiện nay, hầu hết các hệ thống đều sẽ yêu cầu có phân quyền. Vì thế công việc phân quyền sẽ phải làm đi làm lại trong từng dự án, với một hệ thống đơn giản, ít thay đổi, chúng ta có thể phân quyền bằng role (role đại diện cho một nhóm người dùng), mỗi một người dùng (user) sẽ có một role, và chúng ta chỉ định trước chỉ role nào được truy cập vào tài nguyên nào. Tuy nhiên, chúng ta thấy ngay cách phân quyền này có điểm yếu đó là việc thay đổi sẽ rất mất thời gian để chỉnh sửa. Ví dụ nếu hệ thống tự dưng muốn thêm 1 role nữa thì gần như sẽ phải cập nhật hết phần phân quyền của dự án, hay là role này bị thay đổi quyền chúng ta sẽ phải cập nhật trong code, việc này rất bất tiện.

Để giải quyết vấn đề này, mỗi màn hay mỗi chức năng chúng ta sẽ gán cho nó một permission (quyền/ sự cho phép) và sẽ chỉ định trực tiếp role nào được truy cập vào chức năng nào thông qua permission. Điều này có thể không cần thay đổi trong code mà chỉ cần cập nhật cơ sở dữ liệu. Bây giờ role-permission được sử dụng khá rộng rãi ở các hệ thống.

Trong laravel, cũng có rất nhiều thư viện hỗ trợ việc phân quyền bằng role và permission. Trong laravel 5 có thể mọi người đã quen với [Entrust](https://github.com/Zizaco/entrust), tuy nhiên Entrust lại không hỗ trợ cho phiên bản laravel > 5, vì thế chúng ta phải tìm kiếm một sự thay thế khác, và [Laravel-permission](https://spatie.be/docs/laravel-permission/v5/introduction) là một sự thay thế hoàn hảo. Laravel-permission là một thư viện được phát triển bởi [Spatie](https://spatie.be/).

## Cài đặt 

Trang chủ của spatie đã nói rất chi tiết về cài đặt cho laravel-permission, bạn có thể xem tại đây: [https://spatie.be/docs/laravel-permission/v5/installation-laravel](https://spatie.be/docs/laravel-permission/v5/installation-laravel).

## Sử dụng

Thêm trait `Spatie\Permission\Traits\HasRoles` vào `User` model của bạn: 

```php
use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;

class User extends Authenticatable
{
    use HasRoles;

    // ...
}
```

Việc gán permission cho role, gán role cho user, gán permission cho user,... đều đã được cung cấp đầy đủ và rất dễ sử dụng ([xem thêm](https://spatie.be/docs/laravel-permission/v5/basic-usage/basic-usage)):

```php
$role->givePermissionTo($permission);
$permission->assignRole($role);
$role->syncPermissions($permissions);
$permission->syncRoles($roles);
$role->revokePermissionTo($permission);
$permission->removeRole($role);
```

Tuy nhiên, cái mà sử dụng nhiều nhất không phải là việc gọi các hàm trên mà là về sử dụng chúng trong middleware, đây cũng là phần chính mà mình muốn nói tới trong bài viết này. Với mỗi route (mỗi route đại diện cho mỗi chức năng hoặc mỗi màn đối với web, mỗi api đối với REST API), chúng ta sẽ gán cho nó một permission như sau:

```php
<?php

Route::group(['prefix' => '/route/test'], function () {
    Route::post('listing', 'TestController@listing')->name('listing');
    Route::post('store', 'TestController@store')->name('create')
        ->middleware("permission:route-teset.add");
    Route::post('update', 'TestController@update')->name('edit')
        ->middleware("permission:route-teset.edit");
    Route::post('delete', 'TestController@delete')->name('delete')
        ->middleware("permission:route-teset.delete");
});
```

Sau đó, thêm các permission `permission:route-teset.add`, `permission:route-teset.edit`, `permission:route-teset.delete` vào cơ sở dữ liệu. Viết một controller để có thể gán các role cho permission trong `RoleController`:

```php
public function updatePermission(UpdatePermissionRequest $request)
{
    $result = $this->repository->updatePermission($request->only('id', 'permission_names'));
    return \App\Helpers::processCommonResponse($result);
}
```

Trong class `RoleRepository`:

```php
public function updatePermission($arr)
{
    $role = Role::find($arr['id']);
    if ($role != null) {
        $permissionNames = $arr['permission_names'];
        foreach($permissionNames as $permissionName){
            $role->givePermissionTo($permissionName);
        }

        return true;
    }

    return false;
}
```

Trong class `UpdatePermissionRequest` cần validate để chỉ nhận vào các `permission_names` đúng:

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UpdatePermissionRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules()
    {
        return [
            'id' => 'required|numeric',
            'permission_names' => 'exists:permissions,name'
        ];
    }
}

```

Sau khi xong, chắc nhiều bạn sẽ thắc mắc là sao không validate luôn id là `exists:roles,id`. Chúng ta cũng có thể thấy, về bản chất chúng cũng chỉ là các câu query, mà trong `RoleRepository` chúng ta thấy đã có một chỗ kiểm tra id và lấy ra model tương ứng là:

```php
$role = Role::find($arr['id']);
```

Vì thế, nếu chúng ta kiểm tra một lần id nữa ở `Request` thì chương trình sẽ phải truy vấn vào cơ sở dữ liệu 2 phần để làm 2 việc y hệt nhau, việc này là gây lãng phí và chúng ta có thể bỏ qua việc kiểm tra ở `Request`.

Vậy là giờ đây, admin có thể cập nhật quyền của nhóm người dùng ngay trên giao diện admin của họ, việc này rất là tiện lợi.