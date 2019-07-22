一:define('APP_PATH', __DIR__ . '/../application/');
function define ($name, $value, $case_insensitive = false) {}
说明:定义一个常量

| 参数| 描述 | 
| ------ | ------ | ------ |
| name  |必需。规定常量的名称。 |
| value |必需。规定常量的值。 |
| case_insensitive |可选。规定常量的名称是否对大小写敏感。若设置为 true，则对大小写不敏感。默认是 false（大小写敏感）。。 |
- 在设定以后，常量的值无法更改.
- 常量名不需要开头的美元符号（$）
- 作用域不影响对常量的访问
- 常量值只能是字符串和数字

二:is_null($error = error_get_last()) 
function is_null ($var) {}
说明:检测变量是否为 NULL

| 参数| 描述 | 
| ------ | ------ | ------ |
|var  |必需。变量。 |
|返回值  |布尔类型 |

三:funtion isset ( mixed $var [, mixed $... ] ) {}
说明:isset() 函数用于检测变量是否已设置并且非 NUL

| 参数| 描述 | 
| ------ | ------ | ------ |
|var  |必需。要检测的变量。 |
|返回值  |布尔类型 |