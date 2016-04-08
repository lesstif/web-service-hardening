# 네트워크와 방화벽 설정
<!-- toc -->

## 비무장지대(DMZ)

네트워크에서 비무장지대는 조직의 내부 네트워크와 (일반적으로 인터넷인) 외부 네트워크 사이에 위치한 서브넷을 의미합니다.

![DMZ](https://cloud.githubusercontent.com/assets/404534/14389496/db118380-fded-11e5-8af7-9c2e0b2baec4.png "DMZ")

외부에 서비스를 제공하면서 내부 네트워크를 보호해야 하는 임무를 띄고 있으며 웹 서버나 메일 서버같이 Well Known 포트를 사용하는 서비스가 위치하게 됩니다.

DMZ 영역의 서버는 외부에서 바로 연결 가능하므로 SELinux, 침입 탐지, 방화벽등으로 보호해야 하며 DMZ 서버가 해킹 당했을 경우를 가정해서 내부 네트워크로 연결은 허용된 정책(WAS 에 Reverse Proxy 로 연결등)만 빼고 모두 차단해야 합니다.

특히 2차 방화벽은 내부 네트워크를 보호해야 하므로 1차 방화벽과 다른 제품을 사용해야 침입자를 곤한하게 할 수 있습니다. 

> **Caution** DMZ와 내부 네트워크를 분리하고 1, 2차 방화벽을 적용해도 동일 제품이라면 하나의 방화벽과 큰 차이가 없으며 같은 자물쇠를 2개 끼워 놓은 것은 1개보다 별로 안전하지 않은 것과 마찬가지입니다.  

## iptables

Linux 커널 2.2 에는 ipchains 이라는 패킷 필터링/방화벽 프레임워크가 구현되어 있었고 2.4부터는 더 유연하고 다양한 기능을 가진 netfilter 로 기능이 교체 되었습니다.

iptables 는 netfilter 프레임워크의 최상단에 위치하는 사용자 레벨의 프로그램으로 관리자는 이를 이용하여 리눅스 서버로 들어오고 나가는 패킷을 필터링하거나 포트 포워딩을 설정할 수 있으며 방화벽으로도 사용할 수 있다.

## firewalld

RHEL 7 부터는 방화벽을 관리하는 데몬이 firewalld 로 변경되었고 방화벽 설정은 iptables 명령어대신 firewall-cmd (콘솔), firewall-config(X-Windows) 명령어를 사용해야 함.

## 웹 서버 방화벽 설정

### RHEL/CentOS 6 

root 로 실행해야 합니다.

1. /etc/sysconfig/iptables 에 다음과 같이 80, 443 정책을 추가합니다.

	```
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
	-A INPUT -p tcp -m state --state NEW -m tcp --dport 443 -j ACCEPT
	```

1. 또는 lokkit 유틸리티로 다음 내용을 추가합니다.

	```
	lokkit -s ssh -s http -s https
	```

1. 정책을 다시 로드합니다.

	```
	service iptables restart
	```

### RHEL/CentOS 7

1. firewall 명령어로 웹 서버가 사용하는 포트를 방화벽 정책에 등록합니다.

	```
	firewall-cmd --permanent --zone=public --add-service=http
	firewall-cmd --permanent --zone=public --add-service=https
	```

1. 정책을 다시 로드합니다.

	```
	firewall-cmd --reload
	```