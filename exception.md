## 修改app\Exceptions\Handler.php文件
``` php
    /**
     * Render an exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Exception  $exception
     * @return \Illuminate\Http\Response
     */
    public function render($request, Exception $exception)
    {
        
        //自定义表单验证异常
        if ($exception instanceof ValidationException) {
            return $this->handleValidationExceptionToResponse($exception, $request);
        }
        
        if(config('app.debug')) {
            return parent::render($request, $exception);
        } else {
            Log::error($exception);
            return response()->json(['message' => '操作异常，请稍后重试', 'code' => 1000, 'result' => []], 200);
        }
    }

    /**
     * 验证失败，FormRequest
    */
    protected function handleValidationExceptionToResponse(ValidationException $e, $request)
    {
        $errors = $e->errors();
        return response()->json([
            'code'     => 9999,
            'message'  => array_shift($errors)[0],
            'result'   => [],
        ], $e->status);
    }
```
