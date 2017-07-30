---
layout: post
title: Laravel Dusk 浏览器测试更优雅
date: 2017-03-9
tag: Laravel
---


>欢迎大家关注我的其他<a herf="http://blog.csdn.net/u014377963">CSDN博客</a>和<a href="http://www.jianshu.com/u/a9f9d36ab057">简书</a>，互相交流！

当你使用一个 Laravel 5.4 开始你的应用程序时，Laravel Dusk 给我们带来一个在浏览器测试的API，它给我们一个内置的 ChromeDriver ， 当然别的浏览器要使用的话，可以使用 Selenium 。】当你的环境支持 Laravel 5.4 时，第一步是安装一个 demo ，我们使用composer安装Laravel

```
composer create-project --prefer-dist laravel/laravel demo

```
### 安装 Laravel Dusk#

```
composer require laravel/dusk
```
![这里写图片描述](http://img.blog.csdn.net/20170308201700941?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在你的 Laravel 应用中注册 `DuskServiceProvider`，有两个方法

#### 方法一
我们可以在 `config/app.php`文件中 `providers`数组中注册，

```
App\Providers\RouteServiceProvider::class,
Laravel\Dusk\DuskServiceProvider::class,
```
![这里写图片描述](http://img.blog.csdn.net/20170308202304485?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这种方法会在 Laravel 中全局注册，如果不想要全局注册我们使用方法二。

#### 方法二#

在安装环境中的 `AppServiceProvider` 注册 `DuskServiceProvider`

```
namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\DuskServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        if ($this->app->environment('local', 'testing', 'staging')) {
           $this->app->register(DuskServiceProvider::class);
        }
    }
}

```
接下来我们安装 `DUSK#`

```
php artisan dusk:install
```
![这里写图片描述](http://img.blog.csdn.net/20170308202530441?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 接下来开始我们的测试#
#### 第一个测试#
首先我们完成Laravel的认证机制。

```
php artisan make:auth

```
![这里写图片描述](http://img.blog.csdn.net/20170308202656052?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们创建一个 `Dusk` 测试

```
php artisan dusk:make LoginTest
```

![这里写图片描述](http://img.blog.csdn.net/20170308202740272?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

上面这个命令会在 `tests\Browser` 目录下创建一个 `LoginTest` 类
![这里写图片描述](http://img.blog.csdn.net/20170308202844431?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
<?php
namespace Tests\Browser;
use Tests\DuskTestCase;
use Illuminate\Foundation\Testing\DatabaseMigrations;

class LoginTest extends DuskTestCase
{
    /**
     * A Dusk test example.
     *
     * @return void
     */
    public function testExample()
    {
        $this->browse(function ($browser) {
            $browser->visit('/')
                    ->assertSee('Laravel');
        });
    }

```
注意：登录的话需要用户，我们已经添加了测试用户。

 - 添加测试用户

	1.执行命令创建 `User` 表
	
		php artisan migrate
		
	2.使用 `tinker` 命令添加10条测试数据
	
		php artisan tinker
		factory(App\User::class, 10)->create();

	当然我们自然也可以自己注册。测试的话需要知道用户名和密码。

	邮箱： moocfans@moocfans.cn
	密码： moocfans
	![这里写图片描述](http://img.blog.csdn.net/20170308203205901?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

	我们在 `LoginTest` 类中添加一个验证用户登录成功并返回欢迎页的测试用例。
	```
		/**
	 * A Dusk test example.
	 *
	 * @return void
	 */
	public function test_I_can_login_successfully()
	{
	    $this->browse(function ($browser) {
	        $browser->visit('/login')
	                ->type('email', 'moocfans@moocfans.cn')
	                ->type('password', 'moocfans')
	                ->press('Login')
	                ->assertSee('You are logged in!');
	    });
	}
	```
	接下来我们开始测试

		php artisan dusk
如果你的数据库有正确的数据，则会返回下面的结果。

![这里写图片描述](https://dn-phphub.qbox.me/uploads/images/201703/03/5698/KXJLqlTE3K.gif)

注意 chrome版本需>54

### 失败的测试#
我们可以刻意的修改一个错误的测试用例， `PHPUnit` 给我们抛出的错误提示。我们先把登录密码改成 1

```
public function test_I_can_login_successfully()
    {
        $this->browse(function ($browser) {
            $browser->visit('/login')
                    ->type('email', 'moocfans@moocfans.cn')
                    ->type('password', '1')
                    ->press('Login')
                    ->assertSee('You are logged in!');
        });
    }
```
![这里写图片描述](https://dn-phphub.qbox.me/uploads/images/201703/03/5698/XF7wMxWKmY.gif)

用户名和密码不匹配。所以有错误提示。`Dusk` 会把错误截图直接放到 `\tests\Browser\screenshots` 中，以方便大家可以更加准确的查找错误。

![这里写图片描述](http://img.blog.csdn.net/20170308204022118?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


测试Ajax调用#

Dusk 当然也可以测试ajax调用。
有一个完美的测试用例，在github上 ajax测试demo
我们下载下来直接可以用。

创建一个新的测试用例的过程，创建测试用例


 php artisan dusk:make CreateTaskTest

![这里写图片描述](http://img.blog.csdn.net/20170308204116578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxNDM3Nzk2Mw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

然后编辑测试用例

```
  class CreateTaskTest extends DuskTestCase
{
    /**
     * A Dusk test example.
     *
     * @return void
     */
    public function test_I_can_create_task_successfully()
    {
        $this->browse(function ($browser) {

            $browser->visit('/tasks/create')
                    ->type('title', 'My Task')
                    ->press('Add Task')
                    ->pause(5000)
                    ->assertPathIs('/tasks');
        });
    }
}
```

 - 测试用例的执行过程

	1.输入标题
	2.单击“添加任务”按钮
	3.等待5秒
	4.然后重定向到任务页

![这里写图片描述](https://dn-phphub.qbox.me/uploads/images/201703/04/5698/eimbbhYvNA.gif)

我们也可以使用 `waitUntilMissing` 来执行 `Dusk` 的 API

```
<?php

namespace Tests\Browser;

use Tests\DuskTestCase;
use Illuminate\Foundation\Testing\DatabaseMigrations;

class CreateTaskTest extends DuskTestCase
{
    /**
     * A Dusk test example.
     *
     * @return void
     */
    public function test_I_can_create_task_successfully()
    {
        $this->browse(function ($browser) {

            $browser->visit('/tasks/create')
                    ->type('title', 'My Task')
                    ->press('Add Task')
                    ->waitUntilMissing('.btn-primary')
                    ->assertPathIs('/tasks');
        });
    }
}
```

更多的API请查看<a href ="http://d.laravel-china.org/docs/5.4/dusk#scoping-selectors"> Laravel 5.4 文档</a>

在看另外一个例子#

模式对话框绑定你的登录EMail
创建这个测试用例的过程。

登录
找到 链接 Support Email
单击 Support Email
看到你绑定的EMail
根据上面的过程，我们创建测试用例
首先，先创建一个名为 `SupportEmailsTest` 测试用例

```
php artisan dusk:make SupportEmailsTest

```
编辑测试用例

```
class SupportEmailsTest extends DuskTestCase
{
    /**
     * A Dusk test example.
     *
     * @return void
     */
    public function test_I_can_open_modal_for_support_emails()
    {
        $this->browse(function ($browser) {

            $user = factory(User::class)->create();

            $browser->loginAs($user)
                    ->visit('/tasks')
                    ->clickLink('Support Email')
                    ->whenAvailable('#modal-support', function ($modal) use($user) {
                        $modal->assertInputValue('#support-from', $user->email);
                    });
        });
    }
}
```

我们来执行这个测试用例

```
php artisan dusk tests/Browser/SupportEmailsTest.php

```

![这里写图片描述](https://dn-phphub.qbox.me/uploads/images/201703/04/5698/oF2YrvoQrI.gif)

### 页面#

`Dusk` 的 `Pages` 是功能强大的可重用的测试类。
让我们使用 `createtasktest` 创建页面重构。

```
php artisan dusk:page CreateTaskPage
```
创建的页面会存放在 tests/Browser/Pages 目录中

我们来编辑这个类

```
public function url()
{
    return '/tasks/create';
}
```
`url` 可以导航 `Dusk` 执行的地址。

```
public function assert(Browser $browser)
{
    $browser->assertPathIs($this->url());
}
```
assert 定义页面的 assertions，当使用 CreateTaskPage 时，这些 assertions 将会使用 assert 方法执行。
在上面的例子中，我们只是明确 Url 是正确的。

```
public function elements()
{
    return [
        '@addTask' => '.btn-primary',
    ];
}
```

`elements` 方法可以定义选择器。我们可以定义程序可读的名称选择器和重用他们的网页在不同的测试案例。在上面的示例中，我定义了添加任务按钮的选择器。
现在让我们修改 `createtasktest` 类并使用选择器：

```
class CreateTaskTest extends DuskTestCase
{
    /**
     * A Dusk test example.
     *
     * @return void
     */
    public function test_I_can_create_task_successfully()
    {
        $this->browse(function ($browser) {

            $user = factory(User::class)->create();

            $browser->loginAs($user)
                    ->visit(new CreateTaskPage)
                    ->type('title', 'My Task')
                    ->click('@addTask')
                    ->waitUntilMissing('@addTask')
                    ->assertPathIs('/tasks');
        });
    }
}
```

我们修改看了 `createtaskpage`。现在让我们重新运行我们的测试，看看是否一切正常：
和上面测试一样，因此图我就用了同一个。
![这里写图片描述](https://dn-phphub.qbox.me/uploads/images/201703/04/5698/oF2YrvoQrI.gif)


