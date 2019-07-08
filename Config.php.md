一:config底层源码

-   private static $config = [];//@var array 配置参数
-   private static $range = '_sys_'; @var string 参数作用域

- 设定配置参数的作用域
```
 /**
     * 设定配置参数的作用域
     * @access public
     * @param  string $range 作用域
     * @return void
     */
    public static function range($range)
    {
        self::$range = $range;

        if (!isset(self::$config[$range])) self::$config[$range] = [];
    }
```



- 解析配置文件或内容


```
 /**
     * 解析配置文件或内容
     * @access public
     * @param  string $config 配置文件路径或内容
     * @param  string $type   配置解析类型
     * @param  string $name   配置名（如设置即表示二级配置）
     * @param  string $range  作用域
     * @return mixed
     */
    public static function parse($config, $type = '', $name = '', $range = '')
    {
        $range = $range ?: self::$range;

        if (empty($type)) $type = pathinfo($config, PATHINFO_EXTENSION);

        $class = false !== strpos($type, '\\') ?
            $type :
            '\\think\\config\\driver\\' . ucwords($type);

        return self::set((new $class())->parse($config), $name, $range);
    }

```
- 加载配置文件（PHP格式）

```
 /**
     * 加载配置文件（PHP格式）
     * @access public
     * @param  string $file  配置文件名
     * @param  string $name  配置名（如设置即表示二级配置）
     * @param  string $range 作用域
     * @return mixed
     */
    public static function load($file, $name = '', $range = '')
    {
        $range = $range ?: self::$range;

        if (!isset(self::$config[$range])) self::$config[$range] = [];

        if (is_file($file)) {
            $name = strtolower($name);
            //返回文件的扩展名
            $type = pathinfo($file, PATHINFO_EXTENSION);

            if ('php' == $type) {
                return self::set(include $file, $name, $range);
            }

            if ('yaml' == $type && function_exists('yaml_parse_file')) {
                return self::set(yaml_parse_file($file), $name, $range);
            }

            return self::parse($file, $type, $name, $range);
        }

        return self::$config[$range];
    }

```

- 检测配置是否存在

```
/**
     * 检测配置是否存在
     * @access public
     * @param  string $name 配置参数名（支持二级配置 . 号分割）
     * @param  string $range  作用域
     * @return bool
     */
    public static function has($name, $range = '')
    {
        $range = $range ?: self::$range;

        if (!strpos($name, '.')) {
            return isset(self::$config[$range][strtolower($name)]);
        }

        // 二维数组设置和获取支持
        $name = explode('.', $name, 2);
        return isset(self::$config[$range][strtolower($name[0])][$name[1]]);
    }
```

- 获取配置参数 为空则获取所有配置

```
/**
     * 获取配置参数 为空则获取所有配置
     * @access public
     * @param  string $name 配置参数名（支持二级配置 . 号分割）
     * @param  string $range  作用域
     * @return mixed
     */
    public static function get($name = null, $range = '')
    {
        $range = $range ?: self::$range;

        // 无参数时获取所有

        if (empty($name) && isset(self::$config[$range])) {
            return self::$config[$range];
        }

        // 非二级配置时直接返回
        if (!strpos($name, '.')) {
            $name = strtolower($name);
            return isset(self::$config[$range][$name]) ? self::$config[$range][$name] : null;
        }

        // 二维数组设置和获取支持
        $name    = explode('.', $name, 2);
        $name[0] = strtolower($name[0]);

        if (!isset(self::$config[$range][$name[0]])) {
            // 动态载入额外配置
            $module = Request::instance()->module();
            $file   = CONF_PATH . ($module ? $module . DS : '') . 'extra' . DS . $name[0] . CONF_EXT;

            is_file($file) && self::load($file, $name[0]);
        }

        return isset(self::$config[$range][$name[0]][$name[1]]) ?
            self::$config[$range][$name[0]][$name[1]] :
            null;
    }
```

- 设置配置参数 name 为数组则为批量设置

```
 public static function set($name, $value = null, $range = '')
    {
        $range = $range ?: self::$range;

        if (!isset(self::$config[$range]))
            self::$config[$range] = [];

        // 字符串则表示单个配置设置
         if (is_string($name)) {
            if (!strpos($name, '.')) {
                self::$config[$range][strtolower($name)] = $value;
            } else {
                // 二维数组
                $name = explode('.', $name, 2);
                self::$config[$range][strtolower($name[0])][$name[1]] = $value;
            }

            return $value;
        }

        // 数组则表示批量设置
        if (is_array($name)) {
            if (!empty($value)) {

                self::$config[$range][$value] = isset(self::$config[$range][$value]) ?
                    array_merge(self::$config[$range][$value], $name) :
                    $name;

                return self::$config[$range][$value];
            }

            return self::$config[$range] = array_merge(
                self::$config[$range], array_change_key_case($name)
            );
        }

        // 为空直接返回已有配置
        return self::$config[$range];
    }

```
- 重置配置参数

```
/**
     * 重置配置参数
     * @access public
     * @access public
     * @param  string $range 作用域
     * @return void
     */
    public static function reset($range = '')
    {
        $range = $range ?: self::$range;

        if (true === $range) {
            self::$config = [];
        } else {
            self::$config[$range] = [];
        }
    }
```
二:加载文件

```

        if (is_file(APP_PATH . $module . 'init' . EXT)) {
            include APP_PATH . $module . 'init' . EXT;
        } elseif (is_file(RUNTIME_PATH . $module . 'init' . EXT)) {
            include RUNTIME_PATH . $module . 'init' . EXT;
        } else {
            // 加载模块配置
            $config = Config::load(CONF_PATH . $module . 'config' . CONF_EXT);
            var_dump(CONF_PATH . $module . 'config' . CONF_EXT);
            // 读取数据库配置文件
            $filename = CONF_PATH . $module . 'database' . CONF_EXT;
            Config::load($filename, 'database');

            // 读取扩展配置文件
            if (is_dir(CONF_PATH . $module . 'extra')) {
                $dir   = CONF_PATH . $module . 'extra';
                $files = scandir($dir);
                foreach ($files as $file) {
                    if ('.' . pathinfo($file, PATHINFO_EXTENSION) === CONF_EXT) {
                        $filename = $dir . DS . $file;
                        Config::load($filename, pathinfo($file, PATHINFO_FILENAME));
                    }
                }
            }

            // 加载应用状态配置
            if ($config['app_status']) {
                Config::load(CONF_PATH . $module . $config['app_status'] . CONF_EXT);
            }

            // 加载行为扩展文件
            if (is_file(CONF_PATH . $module . 'tags' . EXT)) {
                Hook::import(include CONF_PATH . $module . 'tags' . EXT);
            }

            // 加载公共文件
            $path = APP_PATH . $module;
            if (is_file($path . 'common' . EXT)) {
                include $path . 'common' . EXT;
            }
```

三:使用及其说明
- 配置文件一般都是自动加载.顺序如下图

|  惯例配置   | 应用配置  |扩展配置|场景配置|模块配置|动态配置|
|  ----  | ----  |
| thinkphp/convention.php  | application/config.php |application/extra|application/office.php|application/当前模块名/config.php|Config::set('配置参数','配置值');|
- 使用set方法动态设置参数，例如：
```
Config::set('配置参数','配置值');
// 或者使用助手函数
config('配置参数','配置值');
```
也可以批量设置，例如：
```
Config::set([
    '配置参数1'=>'配置值',
    '配置参数2'=>'配置值'
]);
// 或者使用助手函数
config([
    '配置参数1'=>'配置值',
    '配置参数2'=>'配置值'
]);
``
- 读取配置参数
```
//读取所有的配置参数：
Config::get()
//读取单个参数
Config::get('配置参数1');
//读取二级配置，可以使用：
echo Config::get('配置参数.二级参数');
```
- 判断是否存在某个设置参数
```
Config::has('配置参数2');
```
- 配置作用域

```
//可以使用range方法切换当前配置文件的作用域，例如：
Config::range('test');
// 导入my_config.php中的配置参数，并纳入user作用域
Config::load('my_config.php','','user'); 
// 解析并导入my_config.ini 中的配置参数，读入test作用域
Config::parse('my_config.ini','ini','test'); 
// 设置user_type参数，并纳入user作用域
Config::set('user_type',1,'user'); 
// 批量设置配置参数，并纳入test作用域
Config::set($config,'test'); 
// 读取user作用域的user_type配置参数
echo Config::get('user_type','user'); 
// 读取user作用域下面的所有配置参数
dump(Config::get('','user')); 
dump(config('',null,'user')); // 同上
// 判断在test作用域下面是否存在user_type参数
Config::has('user_type','test'); 

```
