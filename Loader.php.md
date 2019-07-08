**一:注册命名空间,给特定的数组赋值.**
- composer
见autoload_static.php 
- 命名空间 
```php
      /**
         * 注册命名空间
         * @access public
         * @param  string|array $namespace 命名空间
         * @param  string       $path      路径
         * @return void
         */
        public static function addNamespace($namespace, $path = '')
        {
    
            if (is_array($namespace)) {
                foreach ($namespace as $prefix => $paths) {
                    self::addPsr4($prefix . '\\', rtrim($paths, DS), true);
                }
            } else {
                self::addPsr4($namespace . '\\', rtrim($path, DS), true);
            }
        }
   
/**
     * 添加 PSR-4 空间
     * @access private
     * @param  array|string $prefix  空间前缀
     * @param  string       $paths   路径
     * @param  bool         $prepend 预先设置的优先级更高
     * @return void
     */
    private static function addPsr4($prefix, $paths, $prepend = false)
    {

        if (!$prefix) {//如果前缀不存在
            // Register directories for the root namespace.
            self::$fallbackDirsPsr4 = $prepend ?
            array_merge((array) $paths, self::$fallbackDirsPsr4) :
            array_merge(self::$fallbackDirsPsr4, (array) $paths);

        } elseif (!isset(self::$prefixDirsPsr4[$prefix])) {
            // Register directories for a new namespace.
            $length = strlen($prefix);
            if ('\\' !== $prefix[$length - 1]) {
                throw new \InvalidArgumentException(
                    "A non-empty PSR-4 prefix must end with a namespace separator."
                );
            }

            self::$prefixLengthsPsr4[$prefix[0]][$prefix] = $length;
            self::$prefixDirsPsr4[$prefix]                = (array) $paths;

        } else {
            self::$prefixDirsPsr4[$prefix] = $prepend ?
            // Prepend directories for an already registered namespace.
            array_merge((array) $paths, self::$prefixDirsPsr4[$prefix]) :
            // Append directories for an already registered namespace.
            array_merge(self::$prefixDirsPsr4[$prefix], (array) $paths);
        }

    }
    
```





- $prefixLengthsPsr4
 PSR-4 命名空间前缀长度映射

>     't' => 
>         array (size=3)
>           'think\composer\' => int 15
>           'think\' => int 6
>           'traits\' => int 7
>       'a' => 
>         array (size=1)
>           'app\' => int 4
>       'b' => 
>         array (size=1)
>           'behavior\' => int 9

- $prefixDirsPsr4[]
PSR-4 的加载目录

>     array (size=5)
>       'think\composer\' => 
>         array (size=1)
>           0 => string 'F:\tp5\vendor\composer/../topthink/think-installer/src' (length=54)
>       'think\' => 
>         array (size=2)
>           0 => string 'F:\tp5\thinkphp\library\think' (length=29)
>           1 => string 'F:\tp5\vendor\composer/../../thinkphp/library/think' (length=51)
>       'app\' => 
>         array (size=1)
>           0 => string 'F:\tp5\vendor\composer/../../application' (length=40)
>       'behavior\' => 
>         array (size=1)
>           0 => string 'F:\tp5\thinkphp\library\behavior' (length=32)
>       'traits\' => 
>         array (size=1)
>           0 => string 'F:\tp5\thinkphp\library\traits' (length=30)

- $fallbackDirsPsr4[]
自动加载 extend 目录
 

       array (size=1)
          0 => string 'F:\tp5\extend' (length=13)
          
          
**二:查找文件路径,完成自动加载.**

```php
 /**
     * 查找文件
     * @access private
     * @param  string $class 类名
     * @return bool|string
     */
    private static function findFile($class)
    {
        // 类库映射
        if (!empty(self::$classMap[$class])) {
            return self::$classMap[$class];
        }

        // 查找 PSR-4
        $logicalPathPsr4 = strtr($class, '\\', DS) . EXT;
        $first           = $class[0];

        if (isset(self::$prefixLengthsPsr4[$first])) {
            foreach (self::$prefixLengthsPsr4[$first] as $prefix => $length) {
                if (0 === strpos($class, $prefix)) {
                    foreach (self::$prefixDirsPsr4[$prefix] as $dir) {
                        if (is_file($file = $dir . DS . substr($logicalPathPsr4, $length))) {
                            return $file;
                        }
                    }
                }
            }
        }

        // 查找 PSR-4 fallback dirs
        foreach (self::$fallbackDirsPsr4 as $dir) {
            if (is_file($file = $dir . DS . $logicalPathPsr4)) {
                return $file;
            }
        }

        // 查找 PSR-0
        if (false !== $pos = strrpos($class, '\\')) {
            // namespace class name
            $logicalPathPsr0 = substr($logicalPathPsr4, 0, $pos + 1)
            . strtr(substr($logicalPathPsr4, $pos + 1), '_', DS);
        } else {
            // PEAR-like class name
            $logicalPathPsr0 = strtr($class, '_', DS) . EXT;
        }

        if (isset(self::$prefixesPsr0[$first])) {
            foreach (self::$prefixesPsr0[$first] as $prefix => $dirs) {
                if (0 === strpos($class, $prefix)) {
                    foreach ($dirs as $dir) {
                        if (is_file($file = $dir . DS . $logicalPathPsr0)) {
                            return $file;
                        }
                    }
                }
            }
        }

        // 查找 PSR-0 fallback dirs
        foreach (self::$fallbackDirsPsr0 as $dir) {
            if (is_file($file = $dir . DS . $logicalPathPsr0)) {
                return $file;
            }
        }

        // 找不到则设置映射为 false 并返回
        return self::$classMap[$class] = false;
    }
  
 /**
     * 自动加载
     * @access public
     * @param  string $class 类名
     * @return bool
     */
    public static function autoload($class)
    {

        if (!empty(self::$namespaceAlias)) {
            $namespace = dirname($class);
            if (isset(self::$namespaceAlias[$namespace])) {
                $original = self::$namespaceAlias[$namespace] . '\\' . basename($class);
                if (class_exists($original)) {
                    return class_alias($original, $class, false);
                }
            }
        }

        if ($file = self::findFile($class)) {
            // 非 Win 环境不严格区分大小写
            if (!IS_WIN || pathinfo($file, PATHINFO_FILENAME) == pathinfo(realpath($file), PATHINFO_FILENAME)) {
                __include_file($file);
                return true;
            }
        }

        return false;
    }
   
/**
 * include
 * @param  string $file 文件路径
 * @return mixed
 */
function __include_file($file)
{
    return include $file;
}

/**
 * require
 * @param  string $file 文件路径
 * @return mixed
 */
function __require_file($file)
{
    return require $file;
}
```
三:加载扩展类(利用此原理可扩展我们自己的类)
```php
 foreach (self::$fallbackDirsPsr4 as $dir) {
            if (is_file($file = $dir . DS . $logicalPathPsr4)) {
                return $file;
            }
        
```