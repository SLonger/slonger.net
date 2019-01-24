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
环境
 ubuntu	Ubuntu 16.04.5 LTS; 
 nginx nginx/1.10.3
 hugo  v0.53  
php-fpm7.0
git

vps server 配置(www-data 用户)

 1. vps server 获取 github repo  through ssh

``` md
mkdir /var/www/.ssh  
chown -R www-data:www-data /var/www/.ssh/
 ```

 生成ssh key
`sudo -Hu www-data ssh-keygen -t rsa`
www-data 用户主 home目录 /var/www 生成ssh ke在/var/www.ssh/id_rsa.pub    
添加 ssh key 到github 中。
假设下载 mysite 项目

```
cd /var/www
sudo chown -R www-data:www-data /var/www/mysite  
sudo -u www-data git clone git@github.com:[githubuser]/mysite.git /var/www/mysite 
```
2.  nginx 和php 脚本设置
	  php 脚本(test.php:最简单的，如果需要添加认证什么自行添加)
	  ``` php 
	    <?php
		exec("cd /var/www/mysite; git pull ");
		?>
	```
	添加访问权限  
    `chown www-data:www-data /var/www/script/test.php`
    
	 nginx 配置
	```
	 server {
        listen 80;
        listen [::]:80;
        server_name example.com;
        location ~ \.php$ {
            root /var/www/script;
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.0-fpm.sock;
        }
        location / {
                root /var/www/mysite/public/;
                index index.html;
                try_files $uri $uri/ =404;
        }
    }
    ```
       nginx 和php-fpm 运行用户都是 www-data 
3. github webhook 设置
	    添加 webhook url	http://[yoursite].com/test.php
	    Click  **Add webhook**

- 本地更新后 vps server 自动执行 /var/www/script/test.php 脚本 更新 github mysite repo
- http://example.com  nginx 访问 /var/www/mysite/public/ 显示静态内容
		 
Reference link
https://www.portent.com/blog/design-dev/github-auto-deploy-setup-guide.htm)
