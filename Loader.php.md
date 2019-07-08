**һ:ע�������ռ�,���ض������鸳ֵ.**
- composer
��autoload_static.php 
- �����ռ� 
```php
      /**
         * ע�������ռ�
         * @access public
         * @param  string|array $namespace �����ռ�
         * @param  string       $path      ·��
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
     * ��� PSR-4 �ռ�
     * @access private
     * @param  array|string $prefix  �ռ�ǰ׺
     * @param  string       $paths   ·��
     * @param  bool         $prepend Ԥ�����õ����ȼ�����
     * @return void
     */
    private static function addPsr4($prefix, $paths, $prepend = false)
    {

        if (!$prefix) {//���ǰ׺������
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
 PSR-4 �����ռ�ǰ׺����ӳ��

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
PSR-4 �ļ���Ŀ¼

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
�Զ����� extend Ŀ¼
 

       array (size=1)
          0 => string 'F:\tp5\extend' (length=13)
          
          
**��:�����ļ�·��,����Զ�����.**

```php
 /**
     * �����ļ�
     * @access private
     * @param  string $class ����
     * @return bool|string
     */
    private static function findFile($class)
    {
        // ���ӳ��
        if (!empty(self::$classMap[$class])) {
            return self::$classMap[$class];
        }

        // ���� PSR-4
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

        // ���� PSR-4 fallback dirs
        foreach (self::$fallbackDirsPsr4 as $dir) {
            if (is_file($file = $dir . DS . $logicalPathPsr4)) {
                return $file;
            }
        }

        // ���� PSR-0
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

        // ���� PSR-0 fallback dirs
        foreach (self::$fallbackDirsPsr0 as $dir) {
            if (is_file($file = $dir . DS . $logicalPathPsr0)) {
                return $file;
            }
        }

        // �Ҳ���������ӳ��Ϊ false ������
        return self::$classMap[$class] = false;
    }
  
 /**
     * �Զ�����
     * @access public
     * @param  string $class ����
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
            // �� Win �������ϸ����ִ�Сд
            if (!IS_WIN || pathinfo($file, PATHINFO_FILENAME) == pathinfo(realpath($file), PATHINFO_FILENAME)) {
                __include_file($file);
                return true;
            }
        }

        return false;
    }
   
/**
 * include
 * @param  string $file �ļ�·��
 * @return mixed
 */
function __include_file($file)
{
    return include $file;
}

/**
 * require
 * @param  string $file �ļ�·��
 * @return mixed
 */
function __require_file($file)
{
    return require $file;
}
```
��:������չ��(���ô�ԭ�����չ�����Լ�����)
```php
 foreach (self::$fallbackDirsPsr4 as $dir) {
            if (is_file($file = $dir . DS . $logicalPathPsr4)) {
                return $file;
            }
        
```