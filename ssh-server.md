### 목차 

* [방화벽으로 ssh 포트 보호](#방화벽으로-ssh-포트-보호)
* [fail2ban 사용](#fail2ban-사용)
* [강화된 인증 사용](#강화된-인증-사용)
  * [공개키 인증](#공개키-인증)
  * [2단계 인증](#2단계-인증)
* [참고 자료](#참고-자료)

원격지 연결에 사용하는 텔넷(telnet) 이나 rlogin 프로토콜은 오래되었고 보안상 취약한 프로토콜로 이제는 사용하면 안 됩니다.

ssh 는 암호화를 사용하여 세션을 보호하고 암호, 공개키, 챌린지 응답등 다양한 인증 방식을 제공하고 있으며 포트 포워딩을 제공하므로 X-Windows가 필요해도 ssh 포트만 열어도 사용할 수 있습니다.

많은 리눅스 배포판들이 오래전부터 텔넷 서버를 제거하고 ssh 서버를 기본 원격 로그인 서버로 사용했습니다.

그러면 sshd를 더 안전하게 사용할 수 있는 방법을 알아 봅시다.

### 방화벽으로 ssh 포트 보호

ssh의 기본 포트인 22 번 포트는 모든 IP에 대해서 열지 말고 허용된 곳에서만 연결을 허용하도록 해야 합니다.

다음은 192.168.10 대역에서 ssh 연결을 허용하는 iptables 의 설정 예제입니다.

```
-A INPUT -s 192.168.10.0/24 -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
```

### fail2ban 사용

### 강화된 인증 사용

ssh의 암호 인증은 암호가 유출되므로 다른 방식의 인증을 사용하는 것이 좋습니다.
암호 인증을 사용하지 않으려면 /etc/ssh/sshd_config 에 다음 항목을 설정하고 *service sshd restart* 를 실행하면 됩니다.

```
PasswordAuthentication no
```

#### 공개키 인증

#### 2단계 인증

2단계 인증(2 factor authentication)은 안전합니다.

### 참고 자료
* [fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page)
* [google authenticator 를 사용하여 Linux ssh 에 OTP 적용하기](https://www.lesstif.com/pages/viewpage.action?pageId=24444948)
