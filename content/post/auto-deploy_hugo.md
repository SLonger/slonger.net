---
title: "git webhook 自动部署 hugo  静态网站到 vps"
date: 2019-01-10T00:00:00+08:00
lastmod: 2019-01-10T00:00:00+08:00
tags: ["preview", "Theme preview", "tag-3"]
categories: ["hugo", "vps", "githook"]

weight: 10
contentCopyright: MIT 
mathjax: true
autoCollapseToc: true

---

<!--more-->

环境
 ubuntu	Ubuntu 16.04.5 LTS; 
 nginx nginx/1.10.3
 hugo  v0.53  
php-fpm7.0
git

vps server 配置(www-data 用户)

1. vps server 获取 github repo  through ssh

```md
mkdir /var/www/.ssh  
chown -R www-data:www-data /var/www/.ssh/
```
 
 生成ssh key
`sudo -Hu www-data ssh-keygen -t rsa`
www-data 用户主 home目录 /var/www 生成ssh key在/var/www.ssh/id_rsa.pub    ;添加 ssh key 到github 中。
假设下载 mysite 项目

```
cd /var/www
sudo chown -R www-data:www-data /var/www/mysite  
sudo -u www-data git clone git@github.com:[githubuser]/mysite.git /var/www/mysite 
```
2.  php 脚本设置
	 (test.php:  chown www-data:www-data /var/www/script/)
	  ``` php 
	    <?php
			$target = '/var/www/example.net'; // 生产环境web目录
			$secret = "xxx"; // git wehook secret
			
			$json = file_get_contents('php://input'); //获取 github 发送的消息
			$content = json_decode($json, true); 
			
		   $signature = $_SERVER['HTTP_X_HUB_SIGNATURE']; // 获取github 的secret 加密消息
           if (!$signature) {
	           return http_response_code(404);
           }
         list($algo, $hash) = explode('=', $signature, 2);
         
         //计算签名
         $payloadHash = hash_hmac($algo, $json, $secret);

       // 判断签名是否匹配
      if ($hash === $payloadHash) {
		     $cmd = "cd $target && git pull";
		     $res = shell_exec($cmd);
		} else {
			exit();
		}
	?>
	```
	
  3. nginx 配置  (path: /etc/nginx/site-available/example.net)
  
  ```
server {
      listen       443 ssl;
      charset utf-8;
      server_name  example.net;

	  # add ssl and ssl setting 
      ssl on;
      ssl_certificate      /path/full_chain.pem;
      ssl_certificate_key  /path/private.key;
      ssl_session_cache    shared:SSL:1m;
      ssl_session_timeout  5m;
      ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

      location / {
         root /var/www/exmaple.net/public/;
         index index.html;
         try_files $uri $uri/ =404;
      }
      
     location ~ \.php$ {
         root /var/www/script;
         include snippets/fastcgi-php.conf;
         fastcgi_pass unix:/run/php/php7.0-fpm.sock;
     }
  }
```

nginx 和php-fpm 运行用户都是 www-data 
3. github webhook 设置
	    添加 webhook url	http://[yoursite].com/test.php
	     添加    secret  密码
Click  **Add webhook**

- 本地更新后 vps server 自动执行 /var/www/script/test.php 脚本 更新 github mysite repo
- http://example.com  nginx 访问 /var/www/mysite/public/ 显示静态内容
		 
Reference link
https://www.portent.com/blog/design-dev/github-auto-deploy-setup-guide.htm)
https://qq52o.me/2482.html
