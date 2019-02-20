### 在app\Http\Kernel.php文件中添加
``` php
 protected $middleware = [
        \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
        \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
        \App\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
        \App\Http\Middleware\TrustProxies::class,
        \App\Http\Middleware\LogRequest::class, //添加此行代码
        \App\Http\Middleware\EnableCrossRequestMiddleware::class,
    ];
```

### 在app\Http\Middleware\中添加LogRequest.php文件，代码如下
``` php
<?php

/** 记录所有input的数据，并保存在日志中 */
namespace App\Http\Middleware;
use Closure;
use Log;

class LogRequest
{
	 /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        $data = [];
        $data['method'] = $request->method();
        $data['url']    = $request->url();
        $data['data']   = $request->all();

        Log::info(print_r($data, true)); //打印请求参数
        $response = $next($request);

        Log::info($response->content()); //打印响应内容
        return $response;
    }
}
```
