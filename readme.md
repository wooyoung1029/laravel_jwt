# Laravel 中使用 JWT 认证的 Restful API
##  安装 
### 创建新的项目

    composer create-project --prefer-dist laravel/laravel jwt

### 安装 tymon/jwt-auth 扩展包

    composer require tymon/jwt-auth:dev-develop --prefer-source

##  配置
### 发布配置文件

    php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"

### 生成 JWT 密钥
    
    php artisan jwt:secret


### 注册中间件

    protected $routeMiddleware = [
        ....
        'auth.jwt' => \Tymon\JWTAuth\Http\Middleware\Authenticate::class,
    ];
    
### 设置路由（api）
    Route::post('login', 'ApiController@login');
    Route::post('register', 'ApiController@register');
    
    Route::group(['middleware' => 'auth.jwt'], function () {
        Route::get('logout', 'ApiController@logout');
    
        Route::get('user', 'ApiController@getAuthUser'); 
         
    });

### 更新 User 模型

    <?php
    
    namespace App;
    
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Tymon\JWTAuth\Contracts\JWTSubject;
    
    class User extends Authenticatable implements JWTSubject
    {
        use Notifiable;
    
        /**
         * The attributes that are mass assignable.
         *
         * @var array
         */
        protected $fillable = [
            'name', 'email', 'password',
        ];
    
        /**
         * The attributes that should be hidden for arrays.
         *
         * @var array
         */
        protected $hidden = [
            'password', 'remember_token',
        ];
    
        /**
         * Get the identifier that will be stored in the subject claim of the JWT.
         *
         * @return mixed
         */
        public function getJWTIdentifier()
        {
            return $this->getKey();
        }
    
        /**
         * Return a key value array, containing any custom claims to be added to the JWT.
         *
         * @return array
         */
        public function getJWTCustomClaims()
        {
            return [];
        }
    }

### JWT 身份验证逻辑
    它将在 app/Http/Requests 目录下创建 RegisterAuthRequest.php 文件。将下面的代码黏贴至该文件中。
    <?php
    
    namespace App\Http\Requests;
    
    use Illuminate\Foundation\Http\FormRequest;
    
    class RegisterAuthRequest extends FormRequest
    {
        /**
         * 确定是否授权用户发出此请求
         *
         * @return bool
         */
        public function authorize()
        {
            return true;
        }
    
        /**
         * 获取应用于请求的验证规则
         *
         * @return array
         */
        public function rules()
        {
            return [
                'name' => 'required|string',
                'email' => 'required|email|unique:users',
                'password' => 'required|string|min:6|max:10'
            ];
        }
    }

### 创建一个新的 ApiControlle
    
     代码参照App\Http\Controllers\ApiControlle

### 执行下面命令，来往 user 表中填充刚刚添加好的 seed 数据。

	php artisan db:seed --class=UsersTableSeeder

### 启动测试服务（localhost:8000）

    php artisan serve
    

### 访问
    用户注册： http://127.0.0.1:8000/api/register 
    
    用户登录： http://127.0.0.1:8000/api/login
    结果返回
    {
        "success": true,
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC8xMjcuMC4wLjE6ODAwMFwvYXBpXC9sb2dpbiIsImlhdCI6MTU2MzkzMjgwMywiZXhwIjoxNTYzOTM2NDAzLCJuYmYiOjE1NjM5MzI4MDMsImp0aSI6ImxFVGJLZ1hCblFoMkk0dkUiLCJzdWIiOjU1LCJwcnYiOiI4N2UwYWYxZWY5ZmQxNTgxMmZkZWM5NzE1M2ExNGUwYjA0NzU0NmFhIn0.qIToKn3BgSb9-pg7YT8aUpPTnHPPynxkNMxsYsioZrQ"
    }

    后续请求其他接口是header中添加词token 即可完成登录token认证
    
## jwt说明
    https://blog.csdn.net/qq_34001735/article/details/97107838

    参照：
    https://bbs.huaweicloud.com/blogs/06607ea7b53211e7b8317ca23e93a891
    