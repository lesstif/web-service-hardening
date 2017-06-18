# 웹 서버 견고하게 하기

<!-- toc -->

## 3 Tier 아키텍처 

웹 서버는 인터넷에서 가장 중요한 인프라 SW중 하나로 HTTP 를 기반으로 사용자가 요청한 컨텐츠를 보내주고 사용자의 입력을 받아서 처리하는 역할을 수행합니다.

웹이 발달하면서 동적 컨텐츠를 제공해야 하는 필요성이 대두되었고 이를 위해 다양한 웹용 언어와 프레임워크가 탄생하였고 이런 도구들은 웹 서버 위에서 바로 동작하는 경우가 많았습니다.

대표적으로 PHP를 처리하는 *mod_php*, Perl 스크립트를 처리하는 *mod_perl*, python 용 *mod_python* 등이 웹 서버에서 동작하는 확장 모듈입니다.

이런 모듈은 웹 서버를 바로 활용할 수 있으므로 설치와 설정이 쉽고 사용하기도 쉬운 장점이 있었지만 서비스의 규모가 커질 경우 확장이 힘들고 웹 서버가 해킹당하면 DBMS도 해킹 당하는 등의 문제점이 나타났습니다.

이런 문제를 해결하기 위해서는 HTTP를 처리하는 Web Server, 웹 애플리케이션을 실행하는 WAS(Web Application Server), 그리고 DBMS 로 각각의 계층으로 분리하는 **3 Tier** 방식의 아키텍처로 구성하고 웹 서버의 역할은 최소화하는 것이 좋습니다.
 
### 시스템 구성

웹 서버 [DMZ](firewall.html#비무장지대dmz)에 위치시키고 수시로 보안패치와 버그패치를 해주어야 합니다.

 ![DMZ](https://cloud.githubusercontent.com/assets/404534/14389496/db118380-fded-11e5-8af7-9c2e0b2baec4.png "DMZ")

특히 웹 서버는 설치된 패키지가 적어야 패치 속도가 빠르므로 [사용하는 패키지만 최소로 설치해하고 사용하지 않는 패키지는 삭제하는 것이 좋으며 특히 컴파일러 같은 개발도구와 X-Windows 관련 패키지는 반드시 삭제](linux.html#미사용-패키지-삭제)하는 것이  좋습니다.

웹 서비스 포트(80, 443)를 제외하고는 웹 서버의 모든 포트를 막도록 Inbound 방화벽 규칙을 설정합니다.

또 웹 서버는 내부망에 연결이 가능한 서버이므로 웹 서버에서 내부망으로 연결하는 방화벽 규칙은 WAS 의 서비스 포트만 연결 가능하도록 하는 것이 좋습니다.

웹 서버에는 [SELinux](selinux.html)나 AppArmor 같은 강제 접근 통제 기능을 적용하는 것이 필요하며 특히 SELinux 는 웹 서버에 대해서 엄격하게 통제하므로 웹 서버에는 꼭 적용하는 것이 좋습니다.

예로 SELinux는 웹 서버는 연결 가능한 포트를 미리 정해두고 있으며 허용하는 포트는 다음 명령어로 확인 가능합니다.

```bash
# semanage port -l |grep http_port_t

http_port_t                    tcp      9001, 9004, 8000, 8080, 10080, 8001, 80, 81, 443, 488, 8008, 8009, 8443, 9000
```
위와 같은 제약을 통해 웹 서버가 해킹당해도 이를 통해 내부 네트워크에 ssh로 침입해서 2차 해킹을 시도하거나 또는 웹 서버를 경유지로 하여 스팸 메일을 보내거나 외부 서버에 연결하는 등의 피해를 최소화할 수 있습니다.
   
### Reverse Proxy 로 사용

이제 웹 서버는 HTML/CSS/JavaScript 같은 정적 컨텐츠를 전송하거나 또는 내부망에 있는 WAS 에 연결하여 웹 애플리케이션의 실행 결과를 받아서 사용자에게 전달하는 Reverse Proxy 로 역할을 제한하는 것이 필요합니다.

 ![Reverse Proxy](
 https://www.lesstif.com/download/attachments/20776817/image2014-7-19%2022%3A39%3A39.png?version=1&modificationDate=1405777029000&api=v2 "Reverse Proxy")

Reverse Proxy 로 사용할 경우 WAS가 사용하는 포트만 열어주면 되며 다른 포트는 모두 사용을 막는 것이 좋습니다.

| WAS | Default Port | 개발 언어 |
| --- | ---       | --- |
| tomcat | 8080(HTTP), 8009(AJP) | Java|
| jetty | 8080(HTTP) | Java|
| PHP-FPM | 9000 | PHP |
| unicorn | 8080 | ruby |

만약 사용하는 WAS 의 포트가 SELinux에 등록되어 있지 않다면 차단되어 서비스가 불가능하므로 포트를 추가해야 하며 다음 명령어는 웹 서버가 9876 포트에 연결할 수 있도록 허용합니다.

```bas
semanage port -a -p tcp -t http_port_t 9876
```

만약 *ValueError: Port tcp/9876 already defineㅇ* 와 같은 에러가 발생한다면 SELinux 포트 규칙이 이미 정의되어 발생한 것이며 정책을 추가하는 *-a* 옵션대신 변경하는  *-m* 옵션을 사용하면 됩니다.

물리적인 구성에 대해서 알아보았으니 그러면 웹 서버를 견고하게 하기 위한 설정 방법을 알아 봅시다.

## 민감한 데이타 웹 서버에 올리지 않기

각종 설정 파일이나 민감한 파일(sql, shell script) 등은 웹 서버의 Document Root 에 올리지 않아야 합니다.

특히 디렉터리 목록이 활성화되어 있고 index 파일이 없을 경우 폴더의 전체 구조가 노출되므로 주의해야 합니다.

## 보안 강화 HTTP 헤더 사용

HTTP 헤더에는 [보안을 강화하기 위한  여러 헤더](https://www.owasp.org/index.php/List_of_useful_HTTP_headers)가 있습니다.

* **X-Frame-Options** : [클릭 하이재킹](http://blog.ahnlab.com/ahnlab/tag/1058)을 방지하기 위한 헤더로 *deny*로 설정하면 iframe 에서 렌더링을 하지 않습니다. *sameorigin* 은 origin이 일치하지 않을 경우 렌더링을 하지 않습니다. 
* **X-Content-Type-Options**: *"nosniff"* 만 설정할 수 있으며 [잘못된 MIME 형식이 포함된 응답을 거부](https://msdn.microsoft.com/ko-kr/library/gg622941\(v=vs.85\).aspx)합니다.
* **X-XSS-Protection**: IE와 Chrome 브라우저가 지원하며 [특정 유형의 XSS(cross site script)공격](https://msdn.microsoft.com/ko-kr/library/dd565647\(v=vs.85\).aspx)을 차단해 줍니다.

apache httpd 는 Header 지시자로 보안 관련 헤더를 설정하면 됩니다. 

```
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
Header set X-XSS-Protection "1; mode=block"
```

nginx 는 add_header 지시자로 설정합니다.

```
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";
```

>**Warning**
iframe 을 사용하여 서비스하는 페이지가 있을 경우 "*X-Frame-Option: Deny*" 헤더가 설정되면 제대로 동작하지 않습니다.

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

**Server** 헤더를 보면 현재 사용하는 서버 제품(Apache)과 버전(2.2.24), 모듈(mod_ssl, PHP, mod_jk ..)과 버전등 굉장히 상세한 정보를 보여주고 있으며 공격자는 해당 제품과 버전의 취약점을 찾아서 공격할 수 있습니다.

>**Tip** 
[wappalyzer](https://wappalyzer.com/) 를 사용하면 크롬이나 파이어 폭스에서 서버에서 사용하는 기술을 확인할 수 있습니다.
 ![wappalyzer](https://cloud.githubusercontent.com/assets/404534/15269913/c64ac174-1a48-11e6-8dfd-0496bb2d3ec0.png "wappalyzer")

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

웹 서버의 컨텐츠 디렉터리에는 URL Re-writing 을 처리하는 *.htaccess* 나 워드프레스의 *wp-config.php*, git이나 subversion의 형상 관리 메타 정보(.git, .svn), 설정 파일(*.inc, *.ini, *.cfg, *.conf)등의 중요 정보가 있을 수 있습니다.

공격자는 이런 파일을 내려 받아서 서버의 정보를 파악할 수 있으므로 다음 설정으로 중요 파일을 보호할 수 있습니다.


**httpd 2.2, 2.4**

```
<DirectoryMatch .*\.(git|svn)/.*>
    Redirect 404 /
</DirectoryMatch>
    
<FilesMatch "^(htaccess|wp-config)">
    Redirect 404 /
</FilesMatch>

<FilesMatch "\.(inc|ini|conf|cfg)$">
    Redirect 404 /
</FilesMatch>

<FilesMatch "\.(xml|properties)$">
    Redirect 404 /
</FilesMatch>
```

**nginx**

```
location ~ /\.(ht|git|svn) {
    return 404;
}
location ~ /wp-conf* {
    return 404;
}
location ~ /.*\.(inc|ini|conf|cfg)$ {
    return 404;
}
location ~ /.*\.(xml|properties)$ {
    return 404;
}
```

> **Info** *Require all denied*(apache) 나 *deny all*(nginx) 를 사용할 경우 HTTP 403 Forbidden 응답이 가게되며 공격자는 이 경우 해당 컨텐츠가 있으므로 적절한 권한을 얻기위해 추가 공격을 실행할 수 있습니다. 그러나 HTTP 404 Not Found 응답을 받으면 컨텐츠가 없다고 생각할 것이므로 더 적절한 설정입니다.

## 관리자 서비스 접근 제한

웹 애플리케이션에서 설정할수도  있지만 웹 서버에서도 관리자 영역등 특정 패턴의 url 을 차단할 수 있습니다.

다음은 아파치 톰캣의 관리자 context 인 /manager 에 127.0.0.1과 192.168.0.0 대역을 제외하고는 접근을 차단하는 예제입니다.

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

## 같이 읽기
* [Ubuntu Server의 보안을 위해서 해야 할 것들]
  * [Part 1](https://davidhyk.github.io/blog/things-you-should-do-to-secure-ubuntu-part1)
  * [Part 2](https://davidhyk.github.io/blog/things-you-should-do-to-secure-ubuntu-part2)
* [Ubuntu 서버 14.04 초기설정 가이드](http://www.shako.net/blog/ubuntu-server-14-04-initial-setup-guide/)

## 참고 자료
* [apache ServerTokens Directive](https://httpd.apache.org/docs/2.4/mod/core.html#servertokens)
* [nginx RESTRICTING ACCESS](https://www.nginx.com/resources/admin-guide/restricting-access/)
* [Nginx Server Configs](https://github.com/h5bp/server-configs-nginx) - nginx 설정 모음
* [Apache Server Configs](https://github.com/h5bp/server-configs-apache) - apache httpd 설정 모음
* [htaccess](https://github.com/phanan/htaccess) - apache .htaccess Snippets
