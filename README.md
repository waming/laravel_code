## 使用版本
    laravel 5.5
## 步骤
1.去app/Providers/EventServiceProvider中大概19行出添加代码
``` php
protected $listen = [
        'App\Events\Event' => [
            'App\Listeners\EventListener',
        ],
        
        //此处为添加
        'Illuminate\Database\Events\QueryExecuted' => [
            'App\Listeners\SqlListener',
        ],
    ];
```

2.新建app\Listeners\SqlListener.php文件
``` php
<?php

namespace App\Listeners;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\LineFormatter;
use Monolog\Logger;
use Log;

class SqlListener
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  QueryExecuted  $event
     * @return void
     */
    public function handle(QueryExecuted $event)
    {
        $sql     = str_replace("?", "'%s'", $event->sql);

        //排除监听的job表
        $queue_driver = config('queue.default');
        $table_name   = config("queue.connections.{$queue_driver}.table");

        if ( false !== stristr( $sql, $table_name)) {
            return true;
        }

        $log_str = vsprintf($sql, $event->bindings);

        //执行时间毫秒
        $time = $event->time / 1000;

        $log_str .= "执行时间". $time . "s";
        Log::info($log_str);

        //$log     = new Logger('SQL');
        //$handle  = new StreamHandler(storage_path('logs/sql/sql-'.date('Y-m-d').'.log'), Logger::INFO);
        //$handle->setFormatter(new LineFormatter(null, null, true, true));
        //$log->pushHandler($handle);
        //$log->info($log_str);
    }
}
```
