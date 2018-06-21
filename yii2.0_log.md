#Yii2.0日志记录

###日志配置

这里只列举$config里面关于日志的配置需要关注的点:

1. bootstrap数组中必须有log，log组件是bootstrap阶段加载的, 不写将不会有日志产生
2. flushInterval 代表logger object自身的缓冲区, 默认100条, 如果需要立即观察到日志产生, 需要自定义配置, 比如1
3. 对应在targets中自定义的对象, 每个target对象又有自身的缓冲区, 这个值就是exportInterval, 默认100条, 如果需要立即输出看到效果, 也要自定义大小, 比如1
4. 如果你在输出的日志中看到很多上下文变量信息, 就是因为logVars这个数组, 里面默认是['\_GET', '\_POST', '\_FILES', '\_COOKIE', \_SESSION','\_SERVER'], 不需要时直接赋值空数组就行了
5. categories我们用和命名空间对应的写法, categories其实只是一个标识, 在调用`Yii::info(\$msg,\ $categories);`一类方法时, $categories填入对应的字符串, 日志就进入了对应的target中, 实际使用时不可能记得那么多category, 所以直接用`__METHOD__`魔术变量代替, 这样对应于此函数所在命名空间就和配置中的对应上了, 再加上可以用`\*`的语法进行前缀匹配, 可以直接将一个命名空间中的所有日志导出到一个target中, 是一种巧妙的用法



```php
$config = [
    'bootstrap' => ['log'],
    'components' => [
        'log' => [
            'flushInterval' => 1,
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'levels' => ['error', 'warning','info'],
                    'categories' => ['app\modules\syslog\*'],
                    'logFile' => Q_ROOT.'/../logs/php_console_syslog_commands.log',
                    'exportInterval' => 1,
                    'logVars' => [],
                ],
            ],
        ],
];
```

