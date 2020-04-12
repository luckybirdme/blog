---
title: CodeIgniter 执行流程
date: 2020-04-01 07:02:11
tags: CodeIgniter
---

> 本文基于 CodeIgniter 3.0 版本

<!-- more -->

## 一. CodeIgniter 框架
### 1. 整体流程图
![](http://qiniucdn.luckybird.me/blog/img/2020/CodeIgniter_main.png)

- index.php 文件作为前端控制器，初始化运行 CodeIgniter 所需的基本资源；
- Router 检查 HTTP 请求，以确定如何处理该请求；
- 如果存在缓存文件，将直接输出到浏览器，不用走下面正常的系统流程；
- 在加载应用程序控制器之前，对 HTTP 请求以及任何用户提交的数据进行安全检查；
- 控制器加载模型、核心类库、辅助函数以及其他所有处理请求所需的资源；
- 最后一步，渲染视图并发送至浏览器，如果开启了缓存，视图被会先缓存起来用于 后续的请求


### 2. 代码执行过程
![](http://qiniucdn.luckybird.me/blog/img/2020/CodeIgniter_flow.png)

- system/core/CodeIgniter.php 是框架的核心文件，所有流程都在这里执行完毕。
- 红色框架就是 MVC 模式对应组件，也是我们应用开发主要修改的文件。 


## 二. 代码流程解读
### 入口文件，根目录下的 index.php
```php
# index.php

// 根据不同环境，定义错误级别
switch (ENVIRONMENT)
{
    case 'development':
		error_reporting(-1);
		ini_set('display_errors', 1);
	break;
}

// 定义应用目录
define('APPPATH', $application_folder.DIRECTORY_SEPARATOR);

// And here we go, 框架最核心的文件，所有请求处理，结果返回等逻辑，都在这个文件统筹完成。
require_once BASEPATH.'core/CodeIgniter.php';
```

### 核心文件，首先加载常量和通用函数
```php
# system/core/CodeIgniter.php

// 加载常量
require_once(APPPATH.'config/constants.php');

// 加载常用函数
require_once(BASEPATH.'core/Common.php');

```

### 常用函数 load_class，利用静态变量和引用返回，实现单实例共享模式
```php
# system/core/Common.php

// 函数定义返回引用
function &load_class($class, $directory = 'libraries', $param = NULL)
{
    // 定义静态变量，用于保存所有加载过的类
    // 代码解析时执行一次初始化，函数执行时赋值，函数执行后变量依然存在
	static $_classes = array();

	// 如果存在了就直接返回，避免多次加载
	if (isset($_classes[$class]))
	{
		return $_classes[$class];
	}
	// 标记加载过的类
	is_loaded($class);
	
	// 实例化对象
	$_classes[$class] = isset($param)
		? new $name($param)
		: new $name();
	// 引用返回，以便单实例共享，节省内存
	return $_classes[$class];
}
```

### 通过 composer 加载第三方类库，如果设置了话
```php
# system/core/CodeIgniter.php
// 读取 composer 配置
if ($composer_autoload = config_item('composer_autoload'))
{
	if ($composer_autoload === TRUE)
	{
	    // 根据 vendor 目录的 autoload.php 文件的 PSR-4 协议，加载第三方类库
		file_exists(APPPATH.'vendor/autoload.php')
			? require_once(APPPATH.'vendor/autoload.php')
			: log_message('error', '$config[\'composer_autoload\'] is set to TRUE but '.APPPATH.'vendor/autoload.php was not found.');
	}
}
```

### 加载基准测试类 Benchmark，用于记录框架执行时间，以及性能分析
```php
# system/core/CodeIgniter.php

// 加载基准测试类 Benchmark
$BM =& load_class('Benchmark', 'core');
// 标记开始执行的时间
$BM->mark('total_execution_time_start');
$BM->mark('loading_time:_base_classes_start');
```

### 加载钩子，在不同阶段执行钩子函数
```php
# system/core/CodeIgniter.php

// 加载钩子
$EXT =& load_class('Hooks', 'core');

// 执行系统加载前的钩子函数
$EXT->call_hook('pre_system');

```

### 定义不同阶段的钩子函数
```php
# application/config/hooks.php

// 系统加载前
$hook['pre_system'] = array(
        'class'    => 'MyClass',
        'function' => 'Myfunction',
        'filename' => 'Myclass.php',
        'filepath' => 'hooks',
        'params'   => array('beer', 'wine', 'snacks')
);

// 解析完路由后，返回缓存前
$hook['cache_override'] = array();

// 实例化 controller 前
$hook['pre_controller'] = array();

// 实例化 controller 后，但还没执行具体方法前
$hook['post_controller_constructor'] = array();

// 执行了 controller 的具体方法后
$hook['post_controller'] = array();

// 在输出返回结果前
$hook['display_override'] = array();

// 系统框架执行完毕后
$hook['post_system'] = array();

```

### 加载基础配置，比如文件 application/config/config.php
```php
# system/core/CodeIgniter.php

$CFG =& load_class('Config', 'core');
```

### 加载兼容性函数，比如哈希，多字节字符串
```php
# system/core/CodeIgniter.php

// 多字节字符串
require_once(BASEPATH.'core/compat/mbstring.php');
// 哈希
require_once(BASEPATH.'core/compat/hash.php');
```

### 解析请求的 URI，并确定路由到具体 controller 和 function
```php
# system/core/CodeIgniter.php

// 解析 URI
$URI =& load_class('URI', 'core');
// 确定路由
$RTR =& load_class('Router', 'core', isset($routing) ? $routing : NULL);
```

### 初始化 Output 类，如果设置了缓存，而且存在缓存，则直接返回
```php
# system/core/CodeIgniter.php

// 初始化 Output
$OUT =& load_class('Output', 'core');

// 如果设置了缓存，而且存在，则直接返回
if ($EXT->call_hook('cache_override') === FALSE && $OUT->_display_cache($CFG, $URI) === TRUE)
{
	exit;
}
```

### 初始化 Security 类，比如 CSRF 验证方法，XSS 过滤方法
```php
# system/core/CodeIgniter.php
// 初始化 Security
$SEC =& load_class('Security', 'core');

# system/core/Security.php
// CSRF
public function csrf_verify(){}
// XSS
public function xss_clean(){}
```

### 初始化 Input 类，进行安全过滤，以及获取输入参数 $\_GET, $\_POST
```php
# system/core/CodeIgniter.php
// 初始化 Input 类
$IN	=& load_class('Input', 'core');

# system/core/Input.php
// GET
public function get(){}
// POST
public function post(){}
```

### 加载核心控制器 controller，所有控制器将会继承它。
```php
# system/core/CodeIgniter.php

// 加载核心控制器
require_once BASEPATH.'core/Controller.php';

// 定义获取核心控制器的全局方法，方便后续程序直接获取和共享
function &get_instance()
{
	return CI_Controller::get_instance();
}
```

### 将所有类库都赋值给核心控制器，继承它的控制器以 $this->class->function 的方式调用所有类库的方法。
```php
# system/core/Controller.php

class CI_Controller {

	public function __construct()
	{
	    // 将之前加载过的所有常用类，都赋值给核心控制器，以便后续调用
	    // 继承了核心控制器的对象，都可以直接使用 $this->config->item(),$this->input->get()
		foreach (is_loaded() as $var => $class)
		{
			$this->$var =& load_class($class);
		}
		// 添加 Loader 到核心控制器
		// Loader 包括一些常用的加载函数
		// 比如加载视图 $this->load->view()
		$this->load =& load_class('Loader', 'core');
		// 自动加载一些指定的类库，在 application/config/autoload.php 定义这些类库
		$this->load->initialize();
	}
}

```

### 常用的加载函数，包括视图，模型，数据库等
```php
# system/core/Loader.php

class CI_Loader {
    
    // 加载视图, $this->load->view()
    public function view(){}
    // 加载模型, $this->load->model()
    public function model(){}
    // 加载数据库, $this->load->database()
    public function database(){}
}
```

### 加载应用的控制器
```php
# system/core/CodeIgniter.php

// 将路由的类名首字母改为大写，所以你编写的应用控制器首字母必须大写
$class = ucfirst($RTR->class);
// 具体请求方法
$method = $RTR->method;

// 应用控制器的文件是否存在
if (empty($class) OR ! file_exists(APPPATH.'controllers/'.$RTR->directory.$class.'.php'))
{
	$e404 = TRUE;
}
else
{
    // 加载应用的控制器
	require_once(APPPATH.'controllers/'.$RTR->directory.$class.'.php');
}

```

### 执行钩子函数 pre_controller，记录应用控制器执行的开始时间
```php
# system/core/CodeIgniter.php

// 钩子函数
$EXT->call_hook('pre_controller');

// 记录时间
$BM->mark('controller_execution_time_( '.$class.' / '.$method.' )_start');

```


### 实例化应用控制器，并执行具体的方法
```php
# system/core/CodeIgniter.php

// 实例化应用控制器
$CI = new $class();
// 执行具体方法
call_user_func_array(array(&$CI, $method), $params);

```

### 执行钩子函数，并返回内容
```php
# system/core/CodeIgniter.php

// 钩子函数
if ($EXT->call_hook('display_override') === FALSE)
{
    // 输出内容
	$OUT->_display();
}

```


### 将返回内容写到缓存，并输出到浏览器
```php
# system/core/Output.php

class CI_Output {

    
    public function _display($output = '')
    {
        // 将内容写入缓存
        if ($this->cache_expiration > 0 && isset($CI) && ! method_exists($CI, '_output'))
		{
			$this->_write_cache($output);
		}
		
		// 是否自定义了统一的输出函数
		if (method_exists($CI, '_output'))
		{
			$CI->_output($output);
		}
		else
		{
		    // 输出到浏览器
			echo $output;
		}
        
	}
}

```

