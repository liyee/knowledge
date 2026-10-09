# PHP语言

1. 字符串函数区分：
    ```
    strstr()|strchr() 查找字符串的首次出现。strstr('https://www.PHP.php.net', 'php’)->php.net
    stristr() 查找字符串从另一字符串中第一次出现的位置开始到结尾的字符串（大小写不敏感）。strstr('https://www.PHP.php.net', 'php’)->PHP.php.net
    strrchr() 查找字符串在另一个字符串中最后一次出现。strrchr('https://www.PHP.php.net', 'php’)->p.net

    strpos() 返回字符串在另一字符串中第一次出现的位置（对大小写敏感）。strpos(‘https://www.PHP.php.net', 'php') ->16
    stripos() 返回字符串在另一字符串中第一次出现的位置（对大小写不敏感）。stripos('https://www.PHP.php.net', 'php’)->12

    strrpos() 查找字符串在另一字符串中最后一次出现的位置（对大小写敏感）。strrpos('https://www.PHP.php.net', 'php’)->16
    strripos()查找字符串在另一字符串中最后一次出现的位置（对大小写不敏感）。strripos('https://www.PHP.php.net', 'php’)->16
    ```
2. [php-fpm 讲解](https://blog.csdn.net/flymore96/article/details/125000344)
3. [详解Nginx的核心原理](https://blog.csdn.net/)
4. [PHP7和PHP8新特性大总结]：
 - https://blog.csdn.net/qq2942713658/article/details/117637745
 - https://www.runoob.com/php/php7-new-features.html
5. 在php版本进行升级后，我们能够明显感觉到，php的速度运行比较快了，这是由于其中的性能进行了一些调整，下面我们一起来探究php7比5运行快速的原因。
 - 减少变量存储字节，减少内存占用，提高变量运行速度。zval
 - 改进数组结构，将数组元素和hash映射表分配到相同的存储器中，减少存储器的占用，提高cpu缓存的命中率。zend_string
 - 改进函数调用机制，通过优化参数传递环节，减少部分指令，提高执行效率。