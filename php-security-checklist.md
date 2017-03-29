# PHP 보안 강화하기

<!-- toc -->

> **Info**  많은 웹 서비스들이 PHP 로 개발되었지만 긴급한 비즈니스의 요구사항에 밀려서 보안을 고려하지 못한 경우가 많고 이로 인해 여러 취약점을 갖고 있을수 있습니다.
> 이제 보안은 **"하면 좋고 안해도 그만"**인 선택 기능이 아니라 고객과 비즈니스를 보호하기 위해서 우선적으로 고려해야 할 핵심 구성 요소입니다.
> 이를 위해 PHP 로 웹 서비스를 만들고 운영할 때 알아두어야 할 보안 요구 사항을 정리해 봅니다.


## PHP version update & patch

PHP 는 웹 개발에 필요한 기능을 제공하는 도구로 출발했기 때문에 실용성이 높지만 언어의 일관성이 부족하고 보안에 취약한 문제가 있었습니다.

최신 버전은 성능 개선과 함께 언어적 결함과 발견된 보안 취약점을 해결했으므로 주기적으로 최신 버전으로 갱신 및 패치를 해주는 것이 좋습니다.

이를 손쉽게 하기 위해서는 PHP 소스를 직접 컴파일해서 설치하는 것보다는 yum 이나 apt 같은 패키지 매니저를 활용하여 설치하는 방법을 권장합니다.

RHEL이나 CentOS 6 의 경우 탑재된 PHP 버전이 낮으므로 외부 패키지 저장소를 권장하며 아래에 설명할 AddHandler 문제때문에 WebTatic 이 아닌 Remi 저장소를 추천합니다.

## register_globals 사용 금지

PHP 의 register_globals 기능은 오래전부터 심각한 보안 취약점을 갖고 있는 기능으로 악명이 높았으며 5.4 부터는 제거되었습니다.

register_globals 을 사용하고 있다면 소스의 해당 부분을 수정하고 PHP 버전을 업그레이드 하세요.


## Apache 의 AddHandler 설정

> **Danger**  매우 치명적인 보안 취약점이므로 설정을 확인하고 바로 수정하세요.
>

php 를 Apache http 모듈(mod_php)로 설치할 경우 아파치의 설정에 php 스크립트를 구동하기 위한 설정을 추가해야 합니다.
추가하는 방법중 AddHandler 지시자를 사용하는 방법이 있습니다.

```
AddHandler php5-script .php
AddType text/html .php
```

위와 같이 설정할 경우 파일명에 php 가 있을 경우 실행해 버리는 문제가 있으므로 공격자는 원격지에서 임의의 php 코드를 실행할 수 있습니다.
예로 웹 서버의 DocumentRoot 에 *info.php.txt* 있을 경우를 생각해 봅시다.

```sh
echo "<?php phpinfo();" > /var/www/html/info.php.txt
```

이제 브라우저에서 info.php.txt 를 호출하면 확장자가 .txt 이지만 php code 가 실행되는 것을 볼 수 있습니다.
만약 서비스에 파일 업로드 기능이 있다면 공격자는 이 기능을 사용하여 임의의 php 를 올리고 원격에서 실행할 수 있습니다.

좋은 해결책은 mod_php 를 사용하지 않고 **fastcgi 모듈인 php-fpm 을 사용하여 web server 의 스크립트 실행 기능을 차단**하는 것입니다.
mod_php 를 계속 사용해야 한다면 *AddHandler* 지시자를 삭제하고 *FilesMatch* 지시자와 *SetHandler* 를 사용해야 합니다.

```
<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

외부 저장소에서 PHP 를 설치할 경우 [WebTatic](https://webtatic.com/) 은 AddHandler 문제가 있기때문에 대신 [remi 저장소](http://rpms.famillecollet.com/) 사용을 권장합니다.

* [RHEL/CentOS 5,6,7 에 EPEL 과 Remi/WebTatic Repository 설치하기](https://www.lesstif.com/pages/viewpage.action?pageId=6979743#RHEL/CentOS5,6,7에EPEL과Remi/WebTaticRepository설치하기-Remirepository설치) 참고

## Database

웹 서비스의 많은 기능들이 DBMS 와 상호 작용을 통해 이루어 집니다. PHP 로 DBMS 를 다룰 경우 보안을 위해 고려해야 할 사항을 정리합니다.

### Prepared Statements 와 Bound Parameters 사용

client 에서 넘어온 파라미터를 바로 SQL 에 사용하면 SQL injection 취약점이 생길 수 있습니다. 이를 방지하려면 코드 작성시 client 가 전달한 파라미터를 binding 해서
SQL 로 해석되지 않도록 해야 합니다.

```php
// prepare and bind
$stmt = $conn->prepare("INSERT INTO users (firstname, lastname, email) VALUES (?, ?, ?)");
$stmt->bind_param("sss", $firstname, $lastname, $email);
```

### PDO 사용

PHP 에서 DBMS 를 다루기 위한 범용 인터페이스인 PDO 를 사용하면 기본적으로 bind parameter 를 사용하므로 실수에 의한 SQL Injection 취약점이 적어집니다.

## Framework 도입

이미 개발된 서비스는 어렵겠지만 신규로 개발시 프레임워크를 도입하는 것을 고려하는 것이 좋습니다. 
Laravel 등 PHP 프레임워크는 SQL Injection, XSS, CSRF 등 잘 알려진 보안 취약점에 대응하여 설계되었으므로 사용하는 것이 좋습니다.

## 참고 자료

* [OWASP's PHP Security Cheat Sheet](https://www.owasp.org/index.php/PHP_Security_Cheat_Sheet)
