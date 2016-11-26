---
layout: post
title: 简单健康咨询系统Web API的设计与实现
tags:
  - PHP
  - API
date: 2015/12/30
---


## 简介
本教程专为刚刚熟悉基本知识的、但还没有接触到太多高级主题的开发者而设计。本教程也是想要去探索PHP编程一个很好的开始。

## 目标
* [提供用户注册/登陆接口](#用户操作)
* [匿名用户可以浏览已有咨询](#咨询查看)
* [已注册用户可以发布、评论咨询](#咨询提交)

## 数据库设计


#### 表: users

| name     | type         | note                                 |
|:---------|:-------------|:-------------------------------------|
| uid      | mediumint(8) | 用户ID, 自动增长                     |
| username | char(15)     | 用户名                               |
| usertype | bit          | 用户类型, 0代表普通用户, 1代表管理员 |
| password | char(32)     | 密码                                 |


#### 表: ziunx

| name     | type             | note         |
|:---------|:-----------------|:-------------|
| zid      | mediumint(8)     | 咨询ID       |
| uid      | mediumint(8)     | 发布用户ID   |
| subject  | char(80)         | 标题         |
| message  | text             | 咨询正文     |
| dateline | int(10)          | 时间戳       |


#### 表: comment

| name    | type         | note     |
|:--------|:-------------|:---------|
| cid     | mediumint(8) | 评论ID   |
| zid     | mediumint(8) | 咨询ID   |
| uid     | mediumint(8) | 作者ID   |
| message | text         | 评论正文 |

#### 创建数据库

```sql

CREATE TABLE IF NOT EXISTS `users` (
  `uid` mediumint(8) unsigned NOT NULL auto_increment,
  `username` char(15) NOT NULL default '',
  `usertype` bit NOT NULL default 0,
  `password` char(32) NOT NULL default '',
  PRIMARY KEY  (`uid`),
  UNIQUE (`username`) 
)DEFAULT CHARSET=utf8;

CREATE TABLE IF NOT EXISTS `zixun` (
  `zid` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
  `uid` mediumint(8) unsigned NOT NULL DEFAULT '0',
  `subject` char(80) NOT NULL DEFAULT '',
  `dateline` int(10) unsigned NOT NULL DEFAULT '0',
  `message` mediumtext NOT NULL,
  PRIMARY KEY (`zid`)
)DEFAULT CHARSET=utf8;

CREATE TABLE IF NOT EXISTS `comment` (
  `cid` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
  `zid` mediumint(8) unsigned NOT NULL DEFAULT '0',
  `uid` mediumint(8) unsigned NOT NULL DEFAULT '0',
  `message` mediumtext NOT NULL,
  PRIMARY KEY (`cid`)
)DEFAULT CHARSET=utf8;

```

## 程序结构

	.
	|-- config.php //配置文件
	|-- common.php //公共文件
	|-- index.php  //入口文件
	|-- view.php   //咨询查看
	|-- user.php   //登陆/注册
	|-- submit.php //咨询提交

## 程序实现

### 引用第三方类库

* [mysql数据库操作类](https://github.com/NaturalWill/PHP-MySQLi-Database-Class)

### 配置文件

用于存储数据库连接的配置，加密密钥

```php
<?php

/**
 * config.php
 * 配置文件
 */

define('UC_KEY', 'U90ev285I6F7Deh0E7beK7Ueg6zfm9kfE3Q3R8a7W4v0Og1ccb1aH9Veb3YdEbt2'); //加密密钥

$_DBCONFIG = array(
                'host' => 'host',     //服务器地址
                'username' => 'username',    //用户
                'password' => 'password', //密码
                'db'=> 'databaseName', //数据库
                'port' => 3306, //服务器端口
                'prefix' => '', //表名前缀
                'charset' => 'utf8' //字符集
				);
?>

```

### 公共文件

用于引入配置文件，第三方类库，及定义公共函数

```php
<?php

/**
 * common.php
 * 公共文件
 */

require_once ('MysqliDb.php'); //引入mysql操作类
require_once ('config.php'); //引入配置文件
//程序目录
define('S_ROOT', dirname(__FILE__).DIRECTORY_SEPARATOR);

$_SGLOBAL = array();
//时间
$mtime = explode(' ', microtime());
$_SGLOBAL['timestamp'] = $mtime[1];
$_SGLOBAL['supe_starttime'] = $_SGLOBAL['timestamp'] + $mtime[0];

$_REQUEST['auth'] = rawurldecode(req('auth'));

$db = new MysqliDb($_DBCONFIG); //连接数据库

//获取$_GET或$_POST数据
function req($arg,$default=''){
  return empty($_REQUEST[$arg])?$default:$_REQUEST[$arg];
}

//字符串解密加密
function authcode($string, $operation = 'DECODE', $key = '', $expiry = 0) {

	$ckey_length = 4;	// 随机密钥长度 取值 0-32;
				// 加入随机密钥，可以令密文无任何规律，即便是原文和密钥完全相同，加密结果也会每次不同，增大破解难度。
				// 取值越大，密文变动规律越大，密文变化 = 16 的 $ckey_length 次方
				// 当此值为 0 时，则不产生随机密钥

	$key = md5($key ? $key : UC_KEY);
	$keya = md5(substr($key, 0, 16));
	$keyb = md5(substr($key, 16, 16));
	$keyc = $ckey_length ? ($operation == 'DECODE' ? substr($string, 0, $ckey_length): substr(md5(microtime()), -$ckey_length)) : '';

	$cryptkey = $keya.md5($keya.$keyc);
	$key_length = strlen($cryptkey);

	$string = $operation == 'DECODE' ? base64_decode(substr($string, $ckey_length)) : sprintf('%010d', $expiry ? $expiry + time() : 0).substr(md5($string.$keyb), 0, 16).$string;
	$string_length = strlen($string);

	$result = '';
	$box = range(0, 255);

	$rndkey = array();
	for($i = 0; $i <= 255; $i++) {
		$rndkey[$i] = ord($cryptkey[$i % $key_length]);
	}

	for($j = $i = 0; $i < 256; $i++) {
		$j = ($j + $box[$i] + $rndkey[$i]) % 256;
		$tmp = $box[$i];
		$box[$i] = $box[$j];
		$box[$j] = $tmp;
	}

	for($a = $j = $i = 0; $i < $string_length; $i++) {
		$a = ($a + 1) % 256;
		$j = ($j + $box[$a]) % 256;
		$tmp = $box[$a];
		$box[$a] = $box[$j];
		$box[$j] = $tmp;
		$result .= chr(ord($string[$i]) ^ ($box[($box[$a] + $box[$j]) % 256]));
	}

	if($operation == 'DECODE') {
		if((substr($result, 0, 10) == 0 || substr($result, 0, 10) - time() > 0) && substr($result, 10, 16) == substr(md5(substr($result, 26).$keyb), 0, 16)) {
			return substr($result, 26);
		} else {
			return '';
		}
	} else {
		return $keyc.str_replace('=', '', base64_encode($result));
	}
}

//判断当前用户登录状态
function checkauth() {
	global $_SGLOBAL;
	$auth = req('auth');
	if($auth) {
		$db = MysqliDb::getInstance();
		@list($password, $uid) = explode("\t", authcode($auth, 'DECODE'));
		$_SGLOBAL['uid'] = intval($uid);
		if($password && $_SGLOBAL['uid']) {
			$db->where('uid', $_SGLOBAL['uid']);
			if($user = $db->getOne('users')) {
				if($user['password'] == $password) {
					$_SGLOBAL['usertype'] = $user['usertype'];					
					$_SGLOBAL['username'] = $user['username'];
					return;
				}
			}
		}
	}
	showjson('to_login');
}


/**
 * 返回json数据
 * @param string  $code 错误代码，0代表正确，1代表错误
 * @param string  $message 错误信息
 * @param string  $data json数据
 */
function showjson($message, $code=1, $data=array()){
	ob_clean();	
	$r = array();
	$r['code'] = $code;
	$r['msg'] = $message;
	$r['data'] = $data;
	header('Cache-Control: no-cache, must-revalidate');
	header('Content-Type: text/json;'); 
	echo json_encode($r);
	exit();
}

?>

```

 

### 用户操作
提供用户注册/登陆接口

```php

<?php
/**
 * common.php
 * 用户操作
 */

//注册用户
function register(){
	$data = array();
	$data['username'] = req('username');
	$data['password'] = req('password');
	if($data['username'] && $data['password']){
		$db = MysqliDb::getInstance();
		$data['usertype'] = 0;
		$id = $db->insert ('users', $data);
		if($id) showjson('do_success', 0);
		showjson('username_exist');
	}
	showjson('register_error');
}
//验证用户名密码
function login(){
	$password = req('password');
	$username = req('username');
	$db = MysqliDb::getInstance();
	if($password && $username) {
		$db->where('username', $username);
		if($user = $db->getOne('users')) {
			if($user['password'] == $password) {
				$auth = authcode("$user[password]\t$user[uid]", 'ENCODE');
				showjson('do_success', 0, array("m_auth"=>rawurlencode($auth)));
			}
			showjson('password_error');
		}
	}
	showjson('login_error');
}

```

### 咨询查看

匿名用户可以浏览已有咨询

```php
<?PHP
/**
 * view.php
 * 咨询查看
 */
 
//列出咨询
function listzx(){
	$start = req('start',0);
	$perpage = req('perpage',0);
	if($start<0) $start=0;
	if(empty($perpage))	$perpage = 10;
	$db = MysqliDb::getInstance();
	$stats = $db->getOne ("comment", "count(*) as cnt");
	$data=array();
	$data['total'] = $stats['cnt'];
	$list = $db->rawQuery("SELECT z.*, u.username FROM zixun z LEFT JOIN users u ON z.uid=u.uid ORDER BY z.dateline DESC LIMIT $start,$perpage");
	$data['count'] = $db->count;
	$data['list'] = $list;
	if ($db->count >= 0){
		showjson('do_success', 0, array("zixun"=>$data));
	}
	showjson('list_error');
}
//显示咨询内容
function showzx(){
	$zid = req('zid');
	$start = req('start',0);
	$perpage = req('perpage',0);
	if($start<0) $start=0;
	if(empty($perpage))	$perpage = 30;
	if(empty($zid))	showjson('zid_not_exist');
	
	$db = MysqliDb::getInstance();

	$data = $db->rawQueryOne ("SELECT z.*, u.username FROM zixun z LEFT JOIN users u ON z.uid=u.uid WHERE z.zid='$zid'");
	
	if ($db->count > 0){
		$db->where ("zid", $zid);
		$stats = $db->getOne ("comment", "count(*) as cnt");
		$data['total'] = $stats['cnt'];
		
		//if($start>=$data['total']) $start=0;
		$comment = $db->rawQuery("SELECT c.*,s.username FROM comment c LEFT JOIN users s ON c.uid=s.uid WHERE c.zid='$zid' ORDER BY c.cid LIMIT $start,$perpage");
		$data['count']=$db->count;
		$data['comment']=$comment ;
		showjson('do_success', 0, array("zixun"=>$data));
	}
	showjson('show_error');
}
?>

```

### 咨询提交
已注册用户可以发布、评论咨询

```php

<?PHP
/**
 * common.php
 * 咨询提交
 */

//发布咨询
function zixun(){
	global $_SGLOBAL;
	checkauth();   //验证登陆
	
	$op=req('op');
	$db = MysqliDb::getInstance();
	
	if($op=='add'){

		$setarr = array(
			'uid' => $_SGLOBAL['uid'],
			'dateline' => $_SGLOBAL['timestamp']
		);
		$setarr['subject'] = req('subject');
		$setarr['message'] = req('message');
		if($setarr['subject']&&$setarr['message']){
			$id = $db->insert('zixun', $setarr);   //插入数据
			if ($id){
				showjson('do_success', 0, array("zid"=>$id));
			}
			showjson('submit_zixun_error');
		}
		showjson('subject_or_message_can_not_empty');
	}
	elseif($op=='del'){
		$zid=req('zid', 0);
		if(empty($zid)){
			showjson('non_normal_operation');
		}
		$db->where('zid', $zid);
		if($_SGLOBAL['usertype']==1){ //是否管理员
			
		}else{
			$db->where('uid', $_SGLOBAL['uid']);
		}
		$result=$db->delete('zixun');   //删除咨询
		if($result) {
			$db->where('zid', $zid);
			$db->delete('comment');  //删除对应的评论
			showjson('do_success',0);
		}
		else{
			showjson('zixun_not_exist');
		}
	}
	showjson('error_submit');
}
//评论咨询
function comment(){
	global $_SGLOBAL;
	checkauth();   //验证登陆
	
	$op=req('op');
	
	$db = MysqliDb::getInstance();
	if($op=='add'){

		$setarr = array('uid' => $_SGLOBAL['uid']);
		$setarr['message'] = req('message');
		$setarr['zid'] =req('zid',0);
		if($setarr['message']&&$setarr['zid']){
			$id = $db->insert('comment', $setarr);   //插入数据
			if ($id){
				showjson('do_success', 0, array("cid"=>$id));
			}
			showjson('submit_comment_error');
		}
		showjson('zid_or_message_can_not_empty');
	}
	elseif($op=='del'){
		$cid=req('cid', 0);
		if(empty($cid)){
			showjson('non_normal_operation');
		}
		$db->where('cid', $cid);
		if($_SGLOBAL['usertype']==1){ //是否管理员
			
		}else{
			$db->where('uid', $_SGLOBAL['uid']);
		}
		$result=$db->delete('comment');   //删除评论
		if($result) {
			showjson('do_success',0);
		}
		showjson('comment_not_exist');
	}
}

```

### 入口文件

程序的主入口

```php
<?PHP
require_once ('common.php'); //引入公共文件

$do=req('do');
$ac=req('ac');

//允许的方法
$acs = array('user', 'submit', 'view');
if(empty($ac) || !in_array($ac, $acs)) {
	showjson('error_ac');
}
include_once(S_ROOT.$ac.'.php');
if(function_exists($do)){
	call_user_func($do);
}
showjson('error_do');
?>

```

## 接口使用方法

查看[接口使用方法](https://github.com/NaturalWill/Simple-Healthcare-Consulting-System-API#%e6%8e%a5%e5%8f%a3%e4%bd%bf%e7%94%a8%e6%96%b9%e6%b3%95)。

## 项目地址

查看[项目地址](https://github.com/NaturalWill/Simple-Healthcare-Consulting-System-API)。




**版权**: 本文采用以下协议进行授权, [自由转载 - 非商用 - 非衍生 - 保持署名 | Creative Commons BY-NC-ND 3.0](http://creativecommons.org/licenses/by-nc-nd/3.0/deed.zh)。
