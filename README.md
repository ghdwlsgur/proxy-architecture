
코로나 19 팬데믹 때 많은 회사들이 시행했던 재택근무에 대해 팬데믹에서 벗어난 지금도 개발자들의 만족도가 높아지자 여전히 재택근무를 하고자 하는 개발자들이 많은 것 같다. 그렇다면 재택근무 환경을 위한 아키텍처는 어떤 것이 있을까 ? 하고 여러 AWS 아키텍처를 탐색하다 아래에 좋은 템플릿을 발견해서 실습하고자 한다. 하지만 프록시 서버를 직접 구축한 적도 없고 처음 보는 서비스들도 있기에 템플릿을 따라하며 겪을 난항들을 줄이고자 내가 몰랐던 지식들과 서비스들을 마이크로서비스 다루듯이 하나의 도메인으로 분류하여 마스터해보자.
먼저 Squid를 사용하여 프록시 서버를 구축해보자 ! 하... 요즘 인기있는 블로그들을 보니 문어체보다 구어체가 많이 사용돼서 구어체로 사용해보고자 하는데 어색하다...
#### [보안성 높은 재택근무 환경을 위한 AWS 아키텍처 구성하기](https://aws.amazon.com/ko/blogs/korea/improving-security-architecture-controls-for-wfh/)를 완벽하게 따라하기 위해서 사전 빌드업하기 
### Architecture
> ![](https://velog.velcdn.com/images/ragnarok_code/post/60f0e3ab-442f-484f-bf15-6fb40a9a6517/image.png)

- 가용영역이 서로 다른 EC2 2개를 각각 만들고 하나는 `리눅스`, 다른 하나는 `윈도우` OS로 생성
- `리눅스` 서버에 SSH로 접속 및 프록시 설정을 위해 TCP 22번 포트를 전체 허용 또는 내 IP로 제한 
- `3128` 포트는 스퀴드 프록시 서버에서 주로 포트로 윈도우에서 `3128` 포트로 접속하여 인터넷과 통신 해당 포트로 들어오는 IP CIDR 블록을 VPC CIDR 블록을 부여하여 동일 VPC내 모든 접속을 허용
- 다른 포트번호를 사용하고 싶다면 `/etc/squid/squid.conf` 파일에서 `http_port 3128` 수정
- RDP 프로토콜, `3389`번으로 윈도우 가상머신에 접속
- 리눅스 EC2를 프록시 서버로 사용할 것이므로 squid 설치 및 설정
### 브라우저별 프록시 설정 
> IE 
> - `Internet` - `Internet Options` - `LAN settings` - `Proxy Server` - `리눅스 EC2 private IP`, `port`: `3128`
![](https://velog.velcdn.com/images/ragnarok_code/post/3bc18559-7446-4793-8922-f7312c802004/image.png)

> Chrome
> - `설정` - `고급 설정 표시` - `프록시 설정 변경` - `프록시 서버 IP와 Port 설정`







### Squid 설치 및 설정
`sudo yum update -y`

`sudo yum install squid.x86_64`

`sudo chkconfig squid on`

`sudo service squid start`

### 특정 사이트 허용하기 1
`sudo vi /etc/squid/allowed_websites.txt`

```bash
.microsoft.com
.redhat.com
```

`sudo vi /etc/squid/squid.conf`

```bash
acl whitelist dstdomain "/etc/squid/allowed_websites.txt"
http_access allow whitelist
```

### 특정 사이트 허용하기 2
```bash
acl allow_dst dstdomain .facebook.com .naver.com .daum.net
http_access deny !allow_dst
http_access allow allow_dst 
```

### 특정 사이트 차단하기 3
```bash
acl deny_dst dstdom_regex \.google.com$ \.google.co.kr$
http_access deny deny_dst
```

### 특정 사이트 차단하기 4
```bash
acl deny_dst dstdomain .
.com .naver.com .daum.net
http_access deny deny_dst 
```

`sudo squid -k reconfigure`

>`sudo service squid stop`

>`sudo service squid start`

`sudo service squid restart`

특정 아이피를 차단 및 허용을 반복하는 와중에 한가지 특징을 발견했다. `.naver.com`를 차단한 상황에서 http`80`으로 접속할 때랑 https`443`으로 접속할 때의 차단 화면이 달랐다. 다른 도메인에서도 마찬가지였다.
### `http://www.naver.com` 접속화면
![](https://velog.velcdn.com/images/ragnarok_code/post/4cad118f-07da-4bcf-ab1b-87ebc56a2f90/image.png)

### `https://www.naver.com` 접속화면 
![](https://velog.velcdn.com/images/ragnarok_code/post/fbdfdfcb-bbf0-4bb6-9459-998f0356a359/image.png)

- squid 기본 설정에서는 http에서만 에러페이지가 호스팅되는 모양이다. 
- 다음번에 직접 https에서도 에러페이지를 호스팅 하도록 커스텀해봐야겠다.

스퀴드와 프록시 서버의 개념에 대해서 정리해보자.
### 스퀴드(Squid)란 ?
- 사전적인 의미로는 '오징어'
- 오픈 소스(GPL) 소프트웨어 프록시 서버이자 웹 캐시
- 반복된 요청을 캐싱함으로 웹서버의 속도를 향상시킴
- 네트워크 자원을 공유하려는 사람들에게 웹, DNS와 다른 네트워크 검색의 캐싱을 제공함
- 트래픽을 걸러줌으로써 안정성에 도움을 주는 등에 이르기까지 광범위하게 이용

### 프록시 서버(Proxy Server)
- 사전적인 의미로는 '대라인'
- 클라이언트가 자신을 통해서 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해주는 컴퓨터나 응용 프로그램
- 서버와 클라이언트 사이에서 중계기로서 대리로 통신을 수행하는 기능을 가리켜 '프록시', 그 중계 기능을 하는 것을 '프록시 서버'라고 부름

### 프록시 서버의 장점
- 프록시 서버에 요청된 내용들을 캐시에 저장 후, 캐시 안에 있는 정보를 이용함으로써 불필요하게 외부와의 연결을 하지 않아 전송 시간을 절약할 수 있음
- 외부와의 트래픽을 줄이게 됨으로써 네트워크 병목 현상을 방지하는 효과도 얻을 수 있음

### 프록시 서버 사용 목적
- 캐시를 사용하여 리소스로의 접근을 빠르게 하기 위해
- 원하지 않는 사이트 차단을 위해
- 인터넷 이용률을 기록, 검사를 위해
- 악의적인 의도로 바이러스, 악성 루머 전파 또는, 다른 정보들을 빼낼 목적으로
- IP 추적을 당하지 않을 목적으로
- 밖으로 나가거나, 안으로 들어오는 콘텐츠를 검사하기 위헤
- 우회하기 위해
