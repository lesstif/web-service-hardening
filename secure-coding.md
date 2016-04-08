# 시큐어 코딩

<!-- toc -->

보안 코딩 관련 자료

## Injection
OWASP 에서 1위로 선정한 위협입니다.


## XSS

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



