---
title: CodeIgniter 介绍
date: 2020-04-01 07:02:05
tags: CodeIgniter
---

> 本文基于 CodeIgniter 3.0 版本

<!-- more -->


## 一. CodeIgniter 简介
追求大道至简，合适小型项目

### 优点
- MVC 模式，设计逻辑清晰，文件分类明确
- 轻量级框架，代码体积小，模块可以按需加载
- 自带模板引擎，可以高效生成html页面
- 包括常用的类库，比如路由管理，会话管理
- 文档完善，配置简单，易于上手，学习成本低
- 免费的，MIT开源

### 缺点
- 没有 ORM 的能力
- 过于简单，扩展能力较弱

## 二. CodeIgniter 学习笔记
### 利用 NGINX 的 rewrite 去掉路径的 index.php
```sh
location / {
    index index.php;
    if (!-e $request_filename) {
         rewrite ^/(.*)$ /index.php?$1 last;
         break;
    }
}
```
### 控制器重写路由
```php
# application/controllers/Welcome.php

class Welcome extends CI_Controller {
    public function _remap($method){
        if ($method === 'some_method'){
            $this->$method();
        }
    }
}
```
### 控制器处理输出内容 
```php
# application/controllers/Welcome.php

class Welcome extends CI_Controller {
    public function _output($content){
        echo $this->filter($content);
    }
}

```

### 获得视图返回内容
```php
# application/controllers/Welcome.php
class Welcome extends CI_Controller {
    public function index(){
        $string = $this->load->view('myfile', '', TRUE);
    }
}
```

### 自动加载模型
```php
# application/config/autoload.php
$autoload['model'] = array('first_model', 'second_model');
```

### 加载辅助函数
```php
$this->load->helper('url');
$base_url = base_url();
```

### 扩展核心类库，比如核心控制器
```php
# application/core/MY_Controller.php

class MY_Controller extends CI_Controller {
}

# application/controller/Welcome.php

class Welcome extends MY_Controller {

    public function __construct(){
        parent::__construct();
    }

    public function index(){
        $this->load->view('welcome_message');
    }
}
```

### 定义多个相同时间执行的钩子函数
```php
# application/config/hooks.php

$hook['pre_controller'][] = array(
    'class'    => 'MyClass',
    'function' => 'MyMethod',
    'filename' => 'Myclass.php',
    'filepath' => 'hooks',
    'params'   => array('beer', 'wine', 'snacks')
);

$hook['pre_controller'][] = array(
    'class'    => 'MyOtherClass',
    'function' => 'MyOtherMethod',
    'filename' => 'Myotherclass.php',
    'filepath' => 'hooks',
    'params'   => array('red', 'yellow', 'blue')
);
```

### 自定义路由格式
```php
# application/config/routes.php

# www.example.com/product/1 => www.example.com/catalog/product_lookup_by_id/1
$route['product/(:num)'] = 'catalog/product_lookup_by_id/$1';
```

### 实现错误信息的页面
```php
# application/views/errors/html/error_general.php

show_error('something is wrong',501)
```


### 缓存只对于视图有效
```php
# application/controllers/Welcome.php

class Welcome extends MY_Controller {
    public function index(){
        // 3 分钟
        $this->output->cache(3);
        // 删除缓存
        // $this->output->delete_cache();
        $this->load->view('welcome_message');
    }
}

```

### 基准测试类的应用
```php
$this->benchmark->mark('code_start');

// Some code happens here

$this->benchmark->mark('code_end');

// 计算 code 运行耗时
echo $this->benchmark->elapsed_time('code_start', 'code_end');
```

### 支持 APC 缓存，即把PHP文件源码的编译结果缓存起来
```php
// 如果机器不支持 APC，则降级到基于文件的缓存
$this->load->driver('cache', array('adapter' => 'apc', 'backup' => 'file'));
if ( ! $foo = $this->cache->get('foo')){
    echo 'Saving to the cache!<br />';
    $foo = 'foobarbaz!';
    // Save into the cache for 5 minutes
    $this->cache->save('foo', $foo, 300);
}
echo $foo;
```

### XSS 安全过滤
```php
$data = $this->security->xss_clean($data);

```

### csrf 安全校验
```php
# application/config/config.php
$config['csrf_protection'] = TRUE;

# application/controllers/Welcome.php
$csrf = array(
    'name' => $this->security->get_csrf_token_name(),
    'hash' => $this->security->get_csrf_hash()
);

# application/views/blog.php

<input type="hidden" name="<?php echo $csrf['name'];?>" value="<?php echo $csrf['hash'];?>" />

```


### session 保存方式支持
- files
- database
```php
$config['sess_driver'] = 'database';
$config['sess_save_path'] = 'ci_sessions';
```
- redis
```php
$config['sess_driver'] = 'redis';
$config['sess_save_path'] = 'tcp://localhost:6379';
```
- memcached

### session 操作
```php
// 添加
// 单个值
$this->session->set_userdata('some_name', 'some_value');
// 数组形式
$newdata = array(
    'username'  => 'johndoe',
    'email'     => 'johndoe@some-site.com',
    'logged_in' => TRUE
);

$this->session->set_userdata($newdata);


// 删除
$this->session->unset_userdata('some_name');

// 销毁
session_destroy();
// or
$this->session->sess_destroy();
```


### 单元测试
```php
$this->load->library('unit_test');
$test = 1 + 1;
$expected_result = 2;
// 代码结果和期待结果比较
$this->unit->run($test, $expected_result);
// 结果类型判断
$this->unit->run('Foo', 'is_string');

```



### 数据库操作
- 配置多个数据库
```php
# application/config/database.php
$db['default'] = array(
    'dsn'   => '',
    'hostname' => 'localhost',
    'username' => 'root',
    'password' => '',
    'database' => 'database_name',
    'dbdriver' => 'mysqli',
    'dbprefix' => '',
    'pconnect' => TRUE,
    'db_debug' => TRUE,
    'cache_on' => FALSE,
    'cachedir' => '',
    'char_set' => 'utf8',
    'dbcollat' => 'utf8_general_ci',
    'swap_pre' => '',
    'encrypt' => FALSE,
    'compress' => FALSE,
    'stricton' => FALSE,
    'failover' => array()
);
// 第二个数据库配置
$db['other_database'] = array();
// 设置默认数据库
$active_group = 'default';


# application/controllers/Welcome.php
// 获取数据库连接
// 默认数据库连接
$this->db =  $this->load->database();
// 其他数据库连接
$other_database = $this->load->database('other_database',TRUE);
```

- 执行原始SQL
```php
# application/controllers/Welcome.php

// 执行原始SQL
// $this->db 指向默认数据库
$query = $this->db->query("select * from blog");
// 指定数据库
$query = $other_database->query("select * from blog");

// 以数组形式返回结果
foreach ($query->result_array() as $row){
    echo $row['title'];
    echo $row['name'];
}

// 以对象形式返回结果
foreach ($query->result() as $row){
    echo $row->title;
    echo $row->name;
}

// 返回一行结果
$row = $query->row();
if (isset($row)){
    echo $row->title;
    echo $row->name;
}
```

- 借助SQL生成器，可以防止SQL注入
```php
# application/config/database.php
// 设置开启 query builder
$query_builder = TRUE;

# application/controllers/Welcome.php
// 生成SQL，相当于  SELECT * FROM blog，并执行SQL
$query = $this->db->get('blog');  
// 返回结果
foreach ($query->result_array() as $row){
    echo $row['title'];
    echo $row['name'];
}

// 设置 where,limit,offset 条件
// SELECT * FROM blog where name='jack' LIMIT 20, 10
$query = $this->db->get_where('mytable', array('name' => $name), $limit, $offset);

// 更复杂的 where 条件
// where name='jack'
$this->db->where('name', $name);
$query = $this->db->get('blog');

// where name in ('')
$names = array('Frank', 'Todd', 'James');
$this->db->where_in('name', $names);
$query = $this->db->get('blog');

// like
$this->db->like('title', 'match');
$this->db->like('body', 'match');
// WHERE `title` LIKE '%match%' ESCAPE '!' AND  `body` LIKE '%match% ESCAPE '!'

// order by 
$this->db->order_by('title', 'DESC');
// Produces: ORDER BY `title` DESC

$this->db->order_by('title DESC, name ASC');
// Produces: ORDER BY `title` DESC, `name` ASC


// 插入数据
$data = array(
    'title' => 'My title',
    'name' => 'My Name',
    'date' => 'My date'
);
$this->db->insert('mytable', $data);
// Produces: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')

// 批量插入
$data = array(
    array(
        'title' => 'My title',
        'name' => 'My Name',
        'date' => 'My date'
    ),
    array(
        'title' => 'Another title',
        'name' => 'Another Name',
        'date' => 'Another date'
    )
);
$this->db->insert_batch('mytable', $data);
// Produces: INSERT INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date'),  ('Another title', 'Another name', 'Another date')


// 更新数据
$data = array(
    'title' => $title,
    'name' => $name,
    'date' => $date
);

$this->db->where('id', $id);
$this->db->update('mytable', $data);
// Produces:
//  UPDATE mytable
//  SET title = '{$title}', name = '{$name}', date = '{$date}'
//  WHERE id = $id

// replace into 语句
$data = array(
    'title' => 'My title',
    'name'  => 'My Name',
    'date'  => 'My date'
);
$this->db->replace('table', $data);
// Executes: REPLACE INTO mytable (title, name, date) VALUES ('My title', 'My name', 'My date')


// 删除数据
$this->db->delete('mytable', array('id' => $id));

```

- 一些常用的辅助函数
```php
// 最新插入的 AUTO_INCREMENT ID，相当于执行 PHP 函数 mysql_insert_id
$AUTO_INCREMENT_ID = $this->db->insert_id();

// 获取insert update 操作的影响行数
$affected_rows = $this->db->affected_rows();

// 获取最新执行的SQL语句 
$last_query = $this->db->last_query();

```


## 三. CodeIgniter 版本变化

1. CI3
- 支持 composer

2. CI4
- 支持命名空间
- 支持PHP7
- 自定义路由，方便设计 RESTfull API
