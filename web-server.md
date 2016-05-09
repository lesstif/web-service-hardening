# Web 서버

<!-- toc -->

## 3 Tier 아키텍처 

웹 서버는 인터넷에서 가장 중요한 인프라 SW중 하나로 HTTP 를 기반으로 컨텐츠를 사용자에게 전달하고 사용자의 입력을 받아서 처리합니다.

과거의 정적 컨텐츠에서 동적 컨텐츠로 변화되면서 다양한 웹용 언어와 프레임워크가 탄생하였고 이런 도구들은 웹 서버 위에서 바로 동작하는 경우가 많았습니다.

PHP를 처리하는 mod_php, Perl 스크립트를 처리하는 mod_perl, python 용 mod_python 등이 이런 확장 모듈입니다.

이런 모듈은 웹 서버를 바로 활용할 수 있으므로 설치와 설정이 쉽고 사용하기도 쉬운 장점이 있었지만 서버스의 규모가 커지고 보안이 중요시 되면서 다음과 같은 문제점이 나타났습니다.

1. 확장성 문제
 
 규모가 커질 경우 Web/WAS/DBMS 등  Tier 별로 확장 전략이 필요하지만 한 서버에서 Web/WAS 기능을 다 수행할 경우 Tier 별 확장이 어렵습니다.
 
1. 보안 문제

 웹 서버는 [DMZ](firewall.html#비무장지대dmz)에 위치하므로 해킹에 대비하여 권한을 낮추고 내부 네트워크 접근을 최소화 해야 합니다. 
 
 웹 서버를 해킹하고 이를 통해 내부 네트워크에 침입해서 2차 피해를 입거나 또는 웹 서버를 경유지로 하여 스팸 메일을 보내거나 외부 서버에 연결하는 등의 공격이 많아 졌습니다.
 
 이런 이유로 SELinux 같은 보안 시스템은 DMZ에 위치하는 서비스들(Web, FTP, Mail)를 엄격하게 통제하며 미리 허가되지 않은 작업은 모두 차단합니다.
 
 ![DMZ](https://cloud.githubusercontent.com/assets/404534/14389496/db118380-fded-11e5-8af7-9c2e0b2baec4.png "DMZ")
 
 그러므로 웹 서버는 [DMZ](firewall.html#비무장지대dmz)에 위치시키고 정적 컨텐츠를 전송하거나 또는 내부 네트워크에 있는 Application Server 에 대해 Reverse Proxy 로 사용하도록 하는게 안정성과 보안 측면에서 좋습니다.
 
 ![Reverse Proxy](
 https://www.lesstif.com/download/attachments/20776817/image2014-7-19%2022%3A39%3A39.png?version=1&modificationDate=1405777029000&api=v2 "Reverse Proxy")

위와 같이 Web - WAS - DBMS 를 물리적(논리적)으로 분리한 것을 3 tier 아키텍처라고 합니다.
 
그러면 웹 서버를 견고하게 하기 위한 설정 방법을 알아 봅시다.

## 민감한 데이타 웹 서버에 올리지 않기

각종 설정 파일이나 민감한 파일(sql, shell script) 등은 웹 서버의 Document Root 에 올리지 않아야 합니다.

특히 디렉터리 목록이 활성화되어 있고 index 파일이 없을 경우 폴더의 전체 구조가 노출되므로 주의해야 합니다.

## 웹서버 디렉터리 목록 비활성화

특정 배포판에 포함된 웹 서버의 기본 설정은 브라우저가 웹 서버에 요청한 리소스가 디렉터리이고 해당 디렉터에 인덱스 파일이 없을 경우 모든 파일과 디렉터리 목록을 보여 주게 됩니다.

이렇게 서버의 모든 자원을 보여주는 것은 보안상 취약할 수 있으므로 웹 서버에 디렉터리의 인덱스를 처리할 확장자를 명시적으로 지정하고 없을 경우 목록을 보여주지 않도록 설정하는게 좋습니다.

apache httpd 의 경우 **DirectoryIndex** 에 인덱스 파일을 지정할 수 있습니다.

**Directory** 지시자에 아래와 같이 Indexes 설정이 있을 경우 목록 출력이 가능하므로 삭제하는 게 좋습니다.

```
DirectoryIndex index index.php index.html index.html.var

<Directory /path/to/directory>
   Options  Indexes
</Directory>
```

nginx 는 **location** 지시자에 autoindex 가 on 일 경우 요청 리소스가 목록을 출력하며 기본 설정은 off 이므로 명시적으로 설정할 필요가 없습니다.

```
index index.php index.html index.htm;

location / {
    autoindex off;
}
```

## server 정보 숨기기

운영하는 서버의 자세한 정보는 공격자에게 유용한 정보를 제공합니다. 다음과 같이 curl 명령어로 서버가 보내오는 HTTP 응답 헤더를 살펴 봅시다.

```
curl -I myhost.com
```

```
HTTP/1.1 200 OK
Date: Thu, 17 Mar 2016 05:54:33 GMT
Server: Apache/2.2.24 (Unix) mod_ssl/2.2.24 OpenSSL/0.9.8e-fips-rhel5 PHP/5.2.17 mod_jk/1.2.41
X-Powered-By: PHP/5.2.17
Content-Length: 1861
Content-Type: text/html
```

현재 사용하는 서버 제품(Apache)과 버전(2.2.24), 모듈(mod_ssl, PHP, mod_jk ..)과 버전등 굉장히 상세한 정보를 보여주고 있으며 공격자는 해당 제품과 버전의 취약점을 찾아서 공격할 수 있습니다.

### Server Token Off

apache httpd 는 httpd.conf 에 다음과 같이 설정하면 표시 정보를 최소화할 수 있습니다.

```
ServerTokens Prod
ServerSignature Off
```

nginx 는 nginx.conf 의 http 항목에 다음 내용을 추가합니다.

```
http {
	server_tokens off;
	
```

### X Powered By 헤더 제거

*X-Powered-By* 헤더는 서버 사이드에서 사용하는 기술(ASP.NET, PHP, JBoss 등)을 표시하는 비표준 헤더로 공격자에게 유용한 정보를 제공하므로 숨기는게 좋습니다.


apache httpd 의 경우 Header 지시자를 사용하면 브라우저로 전송하기 전에 모든 X-Powered-By 헤더를 삭제할 수 있습니다.

```
Header unset X-Powered-By
```

nginx 는 *fastcgi_hide_header* 모듈을 사용하여 숨길 헤더를 지정하면 됩니다.

```
http {
	fastcgi_hide_header X-Powered-By;
```

PHP 를 사용한다면 php.ini 의 다음 항목을 off 로 설정하면 됩니다.

```
expose_php = Off
```

## UTF-8 charset 사용

특별한 문제가 없다면 charset은 utf-8로 선언하고 사용하는 것이 서비스의 확장성 측면과 보안성 측면(*"UTF-7 XSS"* 같은 취약점 방지)에 좋습니다.

웹 서버에 기본 인코딩을 설정하면 HTTP의 *Content-Type* 헤더에 *charset=UTF-8* 을 자동으로 추가하므로 개별 컨텐츠마다 다음과 같이 *meta* 태그로 인코딩을 지정하지 않아도 되는 장점이 있습니다.

```html
<meta charset="utf-8">
```

웹 서버에 기본 인코딩을 설정하려면 **apache httpd**는 다음과 같이  *AddDefaultCharset* 지시자를 사용하여 설정하면 됩니다.

```
AddDefaultCharset utf-8
```

**nginx** 는 *charset* 키워드로 기본 인코딩을 설정할 수 있습니다.

```
http {
    charset utf-8;
```

> **Danger** 
웹 서버에 인코딩을 설정한 경우 브라우저는 개별 컨텐츠에 *meta* 키워드로 설정한 인코딩은 무시하므로 다른 인코딩(예: EUC-KR)을 사용하는 컨텐츠가 있을 경우 오작동할 수 있습니다.
자세한 내용은 아래의 블로그를 참고하세요.

* [Web Browser 가 Web Content 의 character set encoding 을 처리하는 순서 (HTTP Header charset과 meta charset)](https://www.lesstif.com/pages/viewpage.action?pageId=20775179)

## 폴더와 파일에 적절한 권한 부여

웹 컨텐츠 폴더와 파일은 적절한 권한이 부여해야 하며 폴더는 755(또는 775), 파일은 644(또는 664) 권한이 적절합니다.

```
chmod -R 755 /var/www/mycontents
```

특히 파일에 실행 권한을 주는 경우 웹 서버를 통해 실행할 경우 문제가 될 수 있으므로 주의깊게 설정해야 합니다.

> **Info** [SELinux](selinux.html)가 켜져 있을 경우 실행 속성을 주어도 *httpd_sys_script_exec_t* 컨텍스트가 설정되어 있지 않으면 웹 서버가 실행할 수 없습니다.

웹 서버는 루트로 구동해야 하므로 보안을 위해 구동후 별도의 계정으로 전환하며 배포판마다 전환하는 계정이 다릅니다.
그러므로 웹 서버가 쓰기 권한과 실행 권한을 가져야 하는 폴더와 파일은 웹 서버 구동후 전환 계정으로 설정하는 게 권한 문제를 방지할 수 있습니다.

예를 들어 ubuntu 에서 *PHP-FPM*을 도메인 소켓 방식으로 사용할 경우 nginx 구동 계정(www-data)의 소유이고 쓰기 권한이 있어야 정상적으로 서비스를 할 수 있습니다.

```sh
$ ls -l /var/run/php/php7.0-fpm.sock 
srw-rw---- 1 www-data www-data 0 May  9 06:18 /var/run/php/php7.0-fpm.sock
```

RHEL/CentOS 의 apache의 경우 */etc/httpd/conf/httpd.conf*에 구동 계정이 정의되어 있습니다.
 
```
User apache
Group apache
```

nginx 는 */etc/nginx/nginx.conf* 에 *user* 키워드로 구동 계정이 정의되어 있으며 ubuntu는 *www-data*를 RHEL/CentOS는 *nginx* 계정을 사용합니다.

```
user  nginx;
```

## 중요 파일 접근 차단

웹 서버의 컨텐츠 디렉터리에는 URL Re-writing 을 처리하는 *.htaccess* 나 워드프레스의 *wp-config.php*, 또 git이나 subversion의 형상 관리 메타 정보(.git, .svn)등의 중요 정보가 있을 수 있습니다.

공격자는 이런 파일을 내려 받아서 서버의 정보를 파악할 수 있으므로 다음 설정으로 중요 파일을 보호할 수 있습니다.

**httpd 2.2**

```
<DirectoryMatch .*\.(git|svn)/.*>
    Order deny,allow
    Deny From All
</DirectoryMatch>
    
<FilesMatch "^(htaccess|wp-config)">
    Order deny,allow
    Deny From All
</FilesMatch>

<FilesMatch "\.(inc|ini|conf|cfg)$">
    Order deny,allow
    Deny From All
</FilesMatch>
```

**httpd 2.4**

```
<DirectoryMatch .*\.(git|svn)/.*>
    Require all denied
</DirectoryMatch>
    
<FilesMatch "^(htaccess|wp-config)">
    Require all denied
</FilesMatch>

<FilesMatch "\.(inc|ini|conf|cfg)$">
    Require all denied
</FilesMatch>
```

**nginx**

```
location ~ /\.(ht|git|svn) {
    deny all;
}
location ~ /wp-conf* {
    deny all;
}
location ~ /.*\.(inc|ini|conf|cfg)$ {
    deny all;
}
```

## 관리자 서비스 접근 제한

웹 애플리케이션에도 추가되어야 하지만 웹 서버는 관리자 영역등 특정 url 을 차단할 수 있습니다.

다음은 아파치 톰캣의 관리자 context 인 /manager 에 접근을 차단하는 예제입니다.

**httpd 2.2**

```
<Location /manager>
    Order deny,allow
    Deny from all
    Allow from 127.0.0.1 192.168.0.0/24
</Location>
```

**httpd 2.4** 는 다음과 같이 설정합니다.

```
<Location /manager>
    Require host  127.0.0.1 192.168.0.0/24
</Location>
```

**nginx**

```
location /manager {
    satisfy any;

    allow 192.168.0.0/24;
    deny  all;
}
```

## 참고 자료
* [apache ServerTokens Directive](https://httpd.apache.org/docs/2.4/mod/core.html#servertokens)
* [nginx RESTRICTING ACCESS](https://www.nginx.com/resources/admin-guide/restricting-access/)
