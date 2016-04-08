# 데이타 암/복호

<!-- toc -->



## 암호 알고리즘


### 대칭키 암호화

대칭키(Symmetric key) 방식은 암호문을 생성(암호화)할 때 사용하는 키와 암호문으로부터 평문을 복원(복호화)할 때 사용하는 키가 동일한 암호 시스템입니다.

하나의 키로 암호화와 복호화를 수행하므로 대칭키(Symmetric key) 또는 비밀키(Secret Key) 방식의 암호화라고 합니다.

주요 알고리즘으로는 예전에 유닉스의 패스워드 파일을 암호화하는데 사용했던 DES와 현재 미국의 표준 암호 알고리즘인 AES가 있으며 국내에서 개발한 SEED와 ARIA , 그리고 유럽에서 개발한 IDEA 알고리즘 등이 있습니다.

대칭키 방식은 하나의 키만 사용하므로 상대방과 대칭키 기반으로 암호화 통신을 할 경우 상대방도 사전에 같은 키를 갖고 있어야 합니다.

대칭키의 단점은 키를 상대방과 안전하게 공유하는 것은 굉장히 어렵다는 점입니다. 
가령 키를 보내는 중간에 유출될 우려가 있고 이로 인해 도감청 당할 수 있습니다.

그리고 여러 상대방과 통신할 경우 각각의 대칭키를 관리하기(생성, 전달, 유출시 폐기)가 어렵다는 단점이 있습니다.


### 공개키 암호화

공개키 방식(Public Key)의 암호는 암호학적으로 연관된 두 개의 키를 만들어서 하나는 자기가 안전하게 보관하고  다른 하나는 상대방에게 공개하는 식으로 이루어집니다.

이때 본인만 갖고 있는 키를 개인키(Private Key)라고 하며 상대방에게 공개하는 키는 공개키(Public Key) 라고 합니다.

암호화하는 키와 복호화하는 키가 다르므로 공개키 방식 암호화는 비대칭키(Asymmetric Key) 방식 암호화라고도 부릅니다.
 
대표적인 공개키 알고리즘으로는 전 세계적으로 많이 사용되는 RSA, Elgamal 등이 있으며  전자서명에 사용하는 DSA(Digital Signature Algorithm), KCDSA 등도 공개키 알고리즘에 해당합니다. (DSA, KCDSA 는 암호기능이 없고 전자 서명만 가능합니다.)

송신자는 상대방과 공개 키 기반의 암호화된 통신을 할 경우 다음 그림과 같은 절차를 거치게 됩니다.

![공개키 방식 암호화](https://www.lesstif.com/download/attachments/18219486/image2014-7-29%2023%3A33%3A40.png?version=1&modificationDate=1406644244000&api=v2 "공개키 방식 암호화")

1. 송신자는 수신자의 공개키를 구한다.

2. 송신자는 수신자의 공개키로 평문을 암호화 한다.

3. 송신자는 암호화된 메시지를 상대방에게 전달한다. 메시지는 암호화되어 있으므로 전달 도중에 유출되거나 도청되도 암호문으로부터 원문을 알아내기가 어렵다.

4. 수신자는 자신의 비밀키로 암호화된 메시지를 해독하여 평문을 얻는다. 



공개키로 암호화한 메시지는 수신자의 개인키로만 해독할 수 있으므로 보안 채널을 구성할 때 안전하게 상대방에게 키를 전달할 수 있는 엄청난 장점이 있습니다.

하지만 공개키 암호화는 대칭키에 비해 큰 단점이 있습니다.

그것은 대칭키 방식에 비해 속도가 비교가 되지 않을 정도로 느리다는 점입니다.

그래서 전자 서명이나 간단한 메시지 암호화에는 사용할 수 있지만 실시간 암호화 통신등에는 속도때문에 사용이 힘듭니다.

이로 인해 암호화 통신 프로토콜에서는 암호화에 사용할 대칭키(해당 세션에서만 사용하므로 세션 키 라고도 합니다.)를 상대방의 공개키로 암호화해서 안전하게 전달하는 용도로 사용하며, 실제 암호화 통신은 안전하게 전달받은 세션키를 통해 이루어 집니다.

SSL/TLS 또한 이같은 방식으로 동작합니다.

### 키 교환(Key Exchange)

암호화 통신을 하기 위한 키를 상대방과 교환하는 것은 공개키 방식 암호화가 발명되기 전에는 매우 어려운 일이었습니다.

HTTPS 를 사용할 경우 웹 서버에는 RSA 기반의 공개키 인증서를 설치합니다.

SSL/TLS 에서는 상대방과 키를 교환하기 위한 몇 개의 알고리즘이 있고 가장 간단한 RSA 인증서 기반 키 교환은 다음과 같이 동작합니다.

1. 암호화 통신에 사용할 알고리즘 결정

1. 클라이언트는 암호화 통신에 세션 키를 랜덤 함수를 사용하여 생성

1. 클라이언트는 서버의 인증서에서 공개키를 추출

1. 서버의 공개키를 사용하여 2에서 생성한 세션 키를 암호화하여 서버에 전달

1. 서버는 개인키를 사용하여 데이타를 해독한 후 세션 키를 추출

1. HTTPS 통신 시작

![인증서내 공개키](https://cloud.githubusercontent.com/assets/404534/14382662/68894a3c-fdca-11e5-8976-f6e0de604d76.png "인증서내 공개키")


### 암호화 해시 함수

암호화 해시 함수(Cryptographic hash function)는 해시 함수의 일종으로 해시 값으로부터 원래 값을 유추하기 어렵게 설계된 함수입니다.

임의이 입력이 있을 경우 정해진 길이의 출력을 하는 일방향 함수(One Way function)를 의미하며 동일한 입력에 대해 동일한 출력을 내야 하며 다른 값인데 동일한 출력을 내는 경우(해시 충돌)가 극히 드물어야 합니다.

유명한 알고리즘으로는 MD5(보안 문제때문에 암호용으로는 사용 되지 않음), SHA1(현재는 안전하지 않음), SHA-2(SHA256, SHA384, SHA512) 등이 있습니다.

## 비밀번호 암복호

* 안전한 해시 함수 사용
* Salt 첨가.

### PBKDF

PBKDF2(Password-Based Key Derivation Function 2) 는 사용자에게 문자열을 입력받아서 이를 기반으로 안전한 대칭키와 Initial Vector 를 생성하는 방법입니다.

단방향 함수이므로 결과로 부터 원문을 유추할 수 없으므로 사용자 암호에 적용하기 좋은 알고리즘입니다.

PHP 는 hash_pbkdf2 를 사용하면 됩니다.


```php


```



### bcrypt

bcrypt 는 비밀번호 해시에 사용하기 위해 만들어진 알고리즘으로 OpenBSD 에 기본 탑재되어 있습니다.

PHP 는 password_hash() 의 두 번째 파라미터를 PASSWORD_BCRYPT 로 지정하면 됩니다.

```php
<?php

echo password_hash("strong_password", PASSWORD_BCRYPT)."\n";
```

## 키 관리

데이타를 안전하게 관리하기 위해서 가장 중요한 부분중 하나는 암호화한 키를 어떻게 안전하게 보관하는지 입니다.

키를 데이타 파일로 저장할 경우 키와 데이타만 손에 넣으면 모두 해독 가능한 위험이 있습니다.

신용 카드 정보등 아주 중요한 데이타를 보관해야 한다면 HSM(Hardware Security Module) 처럼 키를 장비에서 생성하고 외부에 유출되지 않는 안전한 장비 도입을 검토해 볼 필요가 있습니다.

![Luna HSM](http://www.tssl.com/tsslweb/wp-content/uploads/2014/11/product_safenet_luna_sp2.png "Luna HSM")

유명한 HSM 제품중 하나인 LunaHSM 은 [Amazon Web Service에 CloudHSM](https://aws.amazon.com/ko/cloudhsm/) 에 적용되어 있습니다.


## 데이타 암호화


### Java


Java 는 JCE/JCA 라는 암복호를 하기 위한 표준이 있고 이를 구현한 JCE Provider 가 필요합니다.

권장하는 JCE 프로바이더는 [Bouncy Castle](https://www.bouncycastle.org/java.html) 입니다.

> **Tip** 기본 JDK 의 JCE 정책은 미국외에서는 강력한 키 길이(AES256, RSA2048)를 사용할 수 없어서 *"java.lang.SecurityException: Unsupported keysize or algorithm parameters"* 또는 *"java.security.InvalidKeyException: Illegal key size"* 에러가 발생합니다.
 
이를 해결하려면 Oracle 에서 "Unlimited Strength Jurisdiction Policy Files" 파일을 다운로드 받아서 $JRE/lib/security/ 에 복사해야 합니다.

**AES 256 암호화**

```java
String input = "Hello World";

KeyGenerator gen = KeyGenerator.getInstance("AES");
gen.init(256); 

// 사용할 대칭키
SecretKey key = gen.generateKey(); 

Cipher c = Cipher.getInstance("AES/CBC/PKCS5Padding", "BC");
				
IvParameterSpec    spec = new IvParameterSpec(iv);
		
c.init(Cipher.ENCRYPT_MODE, key, spec);
		
// 암호화된 데이타
byte[] encData = c.doFinal(input.getBytes());

```



### PHP 

PHP는 mcrypt 를 사용하여 대칭키 암호를 할 수 있습니다.
