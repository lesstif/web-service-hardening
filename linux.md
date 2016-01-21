## 패키지 최소 설치

리눅스를 설치할 때 보안을 위해 고려해야 할 사항은 **운영 체제의 사용 용도**(웹 서버, 애플리케이션 서버, DB 서버등)에 맞게 필요한 패키지만 설치하는 것입니다.

예로 DB 전용 서버라면 Web Server 나 Samba 서버같은 사용하지 않는 패키지를 설치하지 않도록 해야 합니다.

FTP나 Samba, Bind 같이 사용하지 않는 서버 프로세스를 구동하는 것은 보안상 문제를 일으킬 수 있으며 운영체제 업그레이드나 패치가 힘들어 지며 재부팅시 오래 걸리는 문제가 있습니다.

특히 웹 서버나 메일 서버같이 DMZ 에 설치되는 서버는 최소 설치후 해당 패키지만 설치하는 것이 좋습니다.

## 구동 프로세스 최소화
사용하지 않는데 부팅시 자동 구동되는 데몬 프로세스가 있는지 확인하고  있다면 중지하고 자동 구동을 끄는 것이 좋습니다.

```sh
# chkconfig --list
 
auditd          0:off   1:off   2:on    3:on    4:on    5:on    6:off
blk-availability        0:off   1:on    2:on    3:on    4:on    5:on    6:off
cgconfig        0:off   1:off   2:off   3:off   4:off   5:off   6:off
cgred           0:off   1:off   2:off   3:off   4:off   5:off   6:off
crond           0:off   1:off   2:on    3:on    4:on    5:on    6:off
exim            0:off   1:off   2:on    3:on    4:on    5:on    6:off
 ````
*0:off   1:off   2:on    3:on    4:on    5:on    6:off* 의 의미는 run level 이 0, 1 일때는 구동되지 않고 2,3,4,5 일때는 구동된다는 의미입니다.

자동 시작을 끄려면 *chkconfig* 명령어 뒤에 서비스 명을 입력하고 off 옵션을 주면 됩니다.
```sh
chkconfig mysqld off
```

우분투는 *sysv-rc-conf* 명령어로 프로세스를 확인할 수 있습니다

```sh
$ sudo sysv-rc-conf --list             
apparmor     S:on
beanstalkd   0:off      1:off   2:on    3:on    4:on    5:on    6:off
blackfire-ag 0:off      1:off   2:on    3:on    4:on    5:on    6:off
console-setu
cron        
cryptdisks  
cryptdisks-e 0:on       6:on    S:on
```

## 시스템을 최신 상태로 유지

운영중인 시스템은 패키지 관리자를 사용하여 시스템을 상시 업데이트하여 최신 상태로 유지해야 제로 데이 공격(zero-day attack) 등에 희생되지 않도록 해야 합니다.

RHEL은 yum 으로 시스템을 최신 상태로 유지할 수 있습니다. 
```sh
yum update
```

우분투는 아래 명령으로 패키지를 업데이트 할 수 있습니다.
```sh
apt-get update
apt-get upgrade
```


## X-Windows 삭제
이미 설치되어 운영을 하고 있는 서버라면 X-Windows 가 설치되어 있는지 확인하고 설치되어 있다면 삭제하는 게 좋습니다.

X-Windows 는 용량이 크고 의존성 있는 패키지가 많으므로 잦은 업데이트가 발생하게 되며 XDM 은 보안상 문제가 발생할 수 있습니다.

만약 X-Windows 가 없으면 명령어를 입력하지 못하는 운영자라면 계속 같이 일하는 것을 고민해 보기 바랍니다.
 

### 참고 자료
* [20 Linux Server Hardening Security Tips](http://www.cyberciti.biz/tips/linux-security.html)


