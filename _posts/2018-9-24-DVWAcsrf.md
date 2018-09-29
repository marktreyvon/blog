---

layout: post

title: "DVWA笔记3（CSRF）"

author: "markt"

---

> CSRF，全称`Cross-site request forgery`，即跨站请求伪造，是指利用受害者尚未失效的身份认证信息（`cookie`、会话等），诱骗其点击恶意链接或者访问包含攻击代码的页面，在受害人不知情的情况下以受害者的身份向（身份认证信息所对应的）服务器发送请求，从而完成非法操作。
> 可以这样理解CSRF：攻击者盗用了你的身份，以你的名义发送恶意请求，对服务器来说这个请求是完全合法的，但是却完成了攻击者所期望的一个操作，比如以你的名义发送邮件、发消息，盗取你的账号，添加系统管理员，甚至于购买商品、虚拟货币转账等。

代码用处不大统一贴到文末。

搞了半天，看了半天，原来基本上是让做代码审计的活。。。真要利用漏洞的话还得自己写伪造页面。。。

---

之前粗略地看过一遍余弦的《web前端黑客技术揭秘》，里面对前端的漏洞分类是XSS，CSRF，点击劫持。核心总的来说还都是围绕一个cookie。但是相较于之前的漏洞，就如书中所说，CSRF的社工色彩更浓厚一些，毕竟要靠伪装来进行欺骗。

而这种虽然算作是漏洞的漏洞以我目前的水平来看从挖掘的角度来讲是不好发现的。

再说回来，如果想解决传统的CSRF的话，关于跨域策略肯定是要下一番功夫的。别的方法在我看来，似乎都有点南辕北辙。打住，不说废话了。

[CSRF简单介绍及利用方法](https://wps2015.org/drops/drops/CSRF%E7%AE%80%E5%8D%95%E4%BB%8B%E7%BB%8D%E5%8F%8A%E5%88%A9%E7%94%A8%E6%96%B9%E6%B3%95.html)

	再总结一下刁钻猥琐的思路：
		1. 原本post出去的请求，可以试试get出去，如果后端是用request接收的话；
		2. 修改协议，从http到https之类；
		3. 通过修改Referer使其模仿目标的Referer来欺骗；
	    4. 利用 xxx.src='javascript:"HTML代码的方式"'; 可以去掉refer，IE8要带。

---

## low

一句话：很low，没有任何防护措施仅仅验证了两次密码是否相同以及过滤了字符串。

按照网上的说法，可以构造短网址来增强欺骗效果。

## medium

只增加了一个eregi()函数（用于对比俩字符串）对比server name 和 HTTP Referer。HTTP Referer用来表示url来源，如果是自己编写的html文件就是空。

有个限制比较多的办法就是先伪造请求网页，然后抓包修改HTTP Referer。但是很明显很难适用于现实情况。

好吧，网上似乎也没有更实际一点的应用方法。

## High

又增加了一个之前见过的user_token。这个token就很没意思，之前说暴力破解的时候已经说过，不再赘述。

但是仔细点就会发现，虽然都有同一种障碍，但是实际上由于一个是暴力破解（需要多次重复请求）另外一个却是跨站的请求伪造（重点是让别人点击并帮你发出你伪造的请求，重在利用受信任的cookie），所以解决方法还是有区别的。

那么再说回来如何利用传统的方式绕过呢？很遗憾在网上并没有找到。有人说利用XSS跨域获取目标的URL上的token，无异于南辕北辙，而且现在的浏览器似乎都有跨域的限制？

希望能在那几本书上找到原因。

## Impossible

新增了一个输入原来密码的操作。在攻击者不知道目标密码的前提下基本做到了impossible。

### Impossible CSRF Source

```php
<?php
if( isset( $_GET[ 'Change' ] ) ) {
	// Check Anti-CSRF token
	checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_curr = $_GET[ 'password_current' ];
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];
    
    // Sanitise current password input
    $pass_curr = stripslashes( $pass_curr );
    $pass_curr = mysql_real_escape_string( $pass_curr );
    $pass_curr = md5( $pass_curr );
    
    // Check that the current password is correct
    $data = $db->prepare( 'SELECT password FROM users WHERE user = (:user) AND password = (:password) LIMIT 1;' );
    $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
    $data->bindParam( ':password', $pass_curr, PDO::PARAM_STR );
    $data->execute();
    
    // Do both new passwords match and does the current password match the user?
    if( ( $pass_new == $pass_conf ) && ( $data->rowCount() == 1 ) ) {
        // It does!
        $pass_new = stripslashes( $pass_new );
        $pass_new = mysql_real_escape_string( $pass_new );
        $pass_new = md5( $pass_new );
    
        // Update database with new password
        $data = $db->prepare( 'UPDATE users SET password = (:password) WHERE user = (:user);' );
        $data->bindParam( ':password', $pass_new, PDO::PARAM_STR );
        $data->bindParam( ':user', dvwaCurrentUser(), PDO::PARAM_STR );
        $data->execute();
    
        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match or current password incorrect.</pre>";
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```

### High CSRF Source

```php
<?php

if( isset( $_GET[ 'Change' ] ) ) {
	// Check Anti-CSRF token
	checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];
    
    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = mysql_real_escape_string( $pass_new );
        $pass_new = md5( $pass_new );
    
        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysql_query( $insert ) or die( '<pre>' . mysql_error() . '</pre>' );
    
        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }
    
    mysql_close();
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```
### Medium CSRF Source

```php
<?php

if( isset( $_GET[ 'Change' ] ) ) {
	// Checks to see where the request came from
	if( eregi( $_SERVER[ 'SERVER_NAME' ], $_SERVER[ 'HTTP_REFERER' ] ) ) {
		// Get input
		$pass_new  = $_GET[ 'password_new' ];
		$pass_conf = $_GET[ 'password_conf' ];

        // Do the passwords match?
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = mysql_real_escape_string( $pass_new );
            $pass_new = md5( $pass_new );
    
            // Update the database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysql_query( $insert ) or die( '<pre>' . mysql_error() . '</pre>' );
    
            // Feedback for the user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with passwords matching
            echo "<pre>Passwords did not match.</pre>";
        }
    }
    else {
        // Didn't come from a trusted source
        echo "<pre>That request didn't look correct.</pre>";
    }
    
    mysql_close();
}

?>
```
### Low CSRF Source

```php
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = mysql_real_escape_string( $pass_new );
        $pass_new = md5( $pass_new );
    
        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysql_query( $insert ) or die( '<pre>' . mysql_error() . '</pre>' );
    
        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }
    
    mysql_close();
}

?> 
```