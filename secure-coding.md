# 시큐어 코딩

<!-- toc -->

> [!NOTE] 
> 안전한 SW 를 만들려면 시큐어 코딩을 꼭 익혀야 하며 이를 실무에 손쉽게 적용하기 위해서는 프레임워크 도입을 권장합니다.

## SQL Injection

OWASP 에서 1위로 선정한 위협으로 막기가 어렵지 않고 공격 당했을 경우 파급력이 크므로 꼭 고려해야 할 취약점입니다.

만약 사용자 id와 암호를 입력받아서 로그인을 처리하는 페이지가 있을 경우 SQL Injection 을 고려하지 않을 경우 다음과 같이 코딩하게 됩니다.

```php
$sql="SELECT * FROM users WHERE userid='$userid' and password='$password'";
 
```

자바의 경우

```java

String sql = "SELECT * FROM users WHERE userid='" + request.getParameter("id") + "'" + " and password='" + request.getParameter("password") + "'";;
```

위와 같은 코드는 클라이언트가 보낸 문자열을 검증없이 사용하므로 공격자가 password 에 **' or '1' = '1** 라는 문자열을 넣으면 or 뒤의 문장이 참이므로 id와 암호를 몰라도 관리자로 로그인이 가능해져 버립니다.

![SQL Injection](https://www.lesstif.com/download/attachments/24445746/image2016-4-10%2011%3A47%3A0.png?version=1&modificationDate=1460255964000&api=v2 "SQL Injection")


이 공격을 막으려면 위와 같이 동적 쿼리를 사용하지 말고 준비된 문장(Prepared Statement)와 매개변수 바인딩(Bound Parameter) 을 사용해야 합니다.

PHP 의 경우 mysql_* 같은 직접 데이타베이스에 연결하는 함수보다는 PDO 를 사용하는 게 좋으며 Java 의 경우 JDBC 직접 사용보다는 MyBatis나 Hibernate 같은 프레임워크를 사용하는 것이 좋습니다.

PHP 라라벨의 경우 쿼리 빌더를 사용하여 아래와 같은 쿼리를 입력한 경우

```
DB::select('select * from users where id = :userid and password = :password', ['userid' => 'myid', 'password' => 'mypasswd']);
```

다음과 같이 변환되므로 안전하게 실행됩니다.


```php
$s = $dbh->prepare('SELECT * FROM users WHERE userid = :userid and password = :password') ;
$s->bindParam(':userid ', $userid );
$s->bindParam(':password', $password);
```

## 인증 및 세션 관리 취약점

인증이나 세션은 적절하게 보호되어야 하며 다음과 같이 관리되면 안 됩니다.

1. 사용자 인증 정보를 해시나 암호화로 보호하지 않음

1. 세션 ID 를 GET 파라미터로 전달하여 URL 에 노출 

	```
	http://example.com/user/login;jsessionId=39ju9jcsejcj3r
	```
1. 세션 타임 아웃이 설정되지 않았거나 너무 긴 경우

1. 세션에 추가되는 정보가 유출 가능한 경우. 예로 관리자로 로그인했을 경우 admin=1 이 설정

위 문제를 해결하려면 HTTPS 를 적용하고 세션 데이타를 암호화 하는 게 좋습니다.

라라벨의 경우 다음과 같이 자동으로 세션 값을 암호화해서 전달합니다.

```
HTTP/1.1 200 OK
Server: nginx/1.9.9
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
Cache-Control: no-cache
Date: Fri, 08 Apr 2016 17:40:42 GMT
Set-Cookie: XSRF-TOKEN=eyJpdiI6IlpONm1OKzBkSGw5WVBOeHZFRzc0WGc9PSIsInZhbHVlIjoiSzBnc01DVXlwVFwvUjc1NnA5YW93NllST3MrYUtYenZMSE5ERG9Ha0hcL2tFWEhMS3d5bjVtb1NBN296RW9EeG5EZ2t1RGdGUTVGZjZXQ2l5bm1wOUFTZz09IiwibWFjIjoiOWY2MjhlZGFlMDFkODYzMGZhMWQwM2Y3MTFiYTBkNGVhMGNhMzQzMmViOWVjNjY3ZmYwMWNmOTFlNTA3NGFjMyJ9; expires=Fri, 08-Apr-2016 19:40:42 GMT; Max-Age=7200; path=/
Set-Cookie: laravel_session=eyJpdiI6IldzcTFBbEpTTk9sZXJyUkRjbW9GblE9PSIsInZhbHVlIjoiNlp6RnVmdUx1eFwvVXVwY2xwVEE1bjN5eW9lbm9cL3BqXC9GVUNESTA1bmdcL1NHWkhYRDJ5U0JRbGtqTEFYV09QcVZuYUNTVFZzME56bkVZdUErNDlCNnZnPT0iLCJtYWMiOiIyODkyYzk5YmQ1MjBjNDQ2YTRlYWU3Y2I1Yjk1OTdhOWU5MWFmNmRlYzQ3ZWM1MDEwNTdhNWIzMjA1NGI0ZTBjIn0%3D; expires=Fri, 08-Apr-2016 19:40:42 GMT; Max-Age=7200; path=/; HttpOnly
``` 

## 크로스 사이트 스크립팅(XSS; Cross Site Scripting)

XSS 는 사용자의 입력값을 사용하여 동적인 내용을 표시하는 웹 페이지에서 외부 입력값에 대한 검증 없이 사용자에게 출력으로 내보내는 경우에 발생합니다.

즉 사용자가 글 쓰기가 가능한 게시판을 만들 경우 다음과 같이 게시글을 입력했을 때 이 글을 클릭한 사용자에게 바로 출력할 경우 자바스크립트가 실행되게 됩니다.

```
 <script>alert("Welcome");</script>
```

위와 같은 취약점을 XSS 라 하며 이를 사용하여 공격자는 악성코드를 유포하거나 사용자의 쿠키를 갈취할 수 있습니다.

사용자의 입력은 꼭 소독해서 사용해야 합니다.

![소독](
https://www.lesstif.com/download/attachments/24445478/image2015-6-13%2012%3A3%3A20.png?version=1&modificationDate=1434163785000&api=v2 "소독")

PHP 는 htmlentities() 함수를 사용하여 입력 값을 소독할 수 있으며 Java 의 경우 OWASP 의 [java encoder](https://github.com/OWASP/owasp-java-encoder) 라이브러리나 네이버의 [lucy-xss-filter](https://github.com/naver/lucy-xss-filter)를 사용하면 됩니다.

## 취약한 직접 객체 참조 

얼마전 모 서점에서 주문 번호만 입력하면 다른 이의 배송 정보와 결재 정보를 그대로 볼 수 있는 취약점이 있었습니다. <sup id="fnref1">[1](#footnote1)</sup>

이런 문제를 해결하려면 사용자가 직접 입력한 객체의 소유자를 검증하고 인증 여부를 확인해야 합니다.

```php
<?php
session_start();
if (!(isset($_SESSION['login']) && $_SESSION['login'] != '')) {
    header ("Location: login.php");
    return;
}

if (!isOwner($request)) {
    http_response_code(403);
    return;
}	
}
?>
```

위와 같은 코드를 app 로직마다 추가해야 하는데 실수로 빼 먹으면 보안 문제가 발생하므로 프레임워크를 도입하고 필터나 미들웨어같이 app 로직 전에 수행되는 기능을 사용하여 처리하는 게 좋습니다.

![라라벨 미들웨어 구성도](https://www.lesstif.com/download/attachments/24445339/image2015-11-13%2013%3A13%3A7.png?version=1&modificationDate=1447387938000&api=v2 "라라벨 미들웨어 구성도")

## 크로스 사이트 요청 변조(CSRF)

CSRF 는 인증된 사용자를 통해 공격자가 원하는 명령을 수행하게 하는 기법입니다.

공격자가 메일이나 사이트 게시글로 이미지나 링크를 전송하여 물품구매를 수행하거나 사이트 글을 변조하는 등의 작업을 수행할 수 있습니다.

CSRF 를 막기 위해서는 인증된 사용자라 하더라도 중요한 작업을 수행할 경우 다시 검증을 수행하도록 하는 방법입니다.

즉 POST/PUT/DELETE 등의 HTTP Method 수행시 FORM 등에 추측이 불가능한 임시 토큰을 삽입한 후에 서버에서 검증하면 됩니다.

프레임워크가 없이 이 공격을 막으려면 app 소스를 많이 수정해야 하므로 꼭 프레임워크 도입을 권장합니다.


라라벨의 경우 POST/PUT/DELETE HTTP Method 를 수행하면 자동으로 서버에서 CSRF 토큰을 검증하고 에러가 발생시 예외 처리하도록 동작하고 있으며 app\Http\Kernel.php 에 VerifyCsrfToken 미들웨어가 이 처리를 담당합니다.

```php
protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            \Illuminate\Session\Middleware\StartSession::class,
            \Illuminate\View\Middleware\ShareErrorsFromSession::class,
            \App\Http\Middleware\VerifyCsrfToken::class,
            \App\Http\Middleware\OwnerCheck::class,
        ],
    ];
```



## 컴포넌트의 취약점 점검

"알려진 취약점이 있는 컴포넌트 사용(Using Components with Known Vulnerabilities)" 은 
OWASP 의 2013년 Top 중 9위인 위협으로 컴포넌트, 라이브러리, 프레임워크등의 보안 취약점을 악용하여 공격에 노출되게 됩니다.

 * [Spring Remote Code Execution](https://gist.github.com/benelog/4582041)
 * [struts 2 xwork remote](http://blog.o0o.nu/2010/07/cve-2010-1870-struts2xwork-remote.html)
 * [PHP 7 Format string vulnerability](http://www.cvedetails.com/cve/CVE-2015-8617/)

Linux 는 yum 이나 apt-get 같은 패키지 관리자 기능을 제공하므로 운영 체제나 인프라 SW의 패치가 매우 쉬워졌지만 app 에서 사용하는 library 나 컴포넌트는 변경시 영향도때문에 교체가 쉽지 않습니다.

이 취약점을 예방하기 하려면 다음과 같은 방법이 있습니다.

1. 개발시 외부 컴포넌트는 패키지 관리자를 사용하여 관리합니다. 패키지 관리자는 사용하는 언어나 프레임워크에 따라 다르며 자바의 경우 maven, gradle, PHP 는 composer, ruby 는 bundle, python 는 pip 등이 있습니다.
패키지 관리자를 사용하면 특정 컴포넌트를 교체하기가 용이해 집니다.

1. 내/외부 컴포넌트들의 버전 및 의존성을 식별할 수 있도록 관리합니다. 빌드된 바이너리의 버전 관리는 [유의적 버전(Semantic Versioning)](http://semver.org/lang/ko/)을 따르는 것이 좋습니다.

1. 인터넷진흥원의 [보호나라 & KrCERT](https://www.krcert.or.kr/krcert/secNoticeList.do)나 [CVE 취약점 데이타베이스](https://cve.mitre.org/) 및 보안 메일링리스트등을 구독하여 최신 동향을 파악합니다.

1. 상용 repository manager(Sonatype nexus등) 에서는 repos 에 있는 라이브러리를 스캔하여 취약점 보고서를 생성하는 기능 제공하는 경우가 있으므로 여건이 된다면 이런 제품을 사용하는 것이 좋습니다.
![Sonatype nexus 의 취약점 보고서](https://www.lesstif.com/download/attachments/20775149/image2014-8-21%2023%3A53%3A3.png?version=1&modificationDate=1408632775000&api=v2 "Sonatype nexus 의 취약점 보고서")




<a name="footnote1" href="#fnref1">[1]</a> http://www.insight.co.kr/newsRead.php?ArtNo=56987
