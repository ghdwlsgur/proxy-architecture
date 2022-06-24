
### Architecture
> ![](https://velog.velcdn.com/images/ragnarok_code/post/60f0e3ab-442f-484f-bf15-6fb40a9a6517/image.png)

- 가용영역이 서로 다른 EC2 2개를 각각 만들고 하나는 `리눅스`, 다른 하나는 `윈도우` OS로 생성
- `리눅스` 서버에 SSH로 접속 및 프록시 설정을 위해 TCP 22번 포트를 전체 허용, 또는 내 IP로 제한 
- `3128` 포트는 스퀴드 프록시 서버에서 주로 사용하는 포트로 윈도우에서 `3128` 포트로 접속하여 인터넷과 통신 해당 포트로 들어오는 IP CIDR 블록을 VPC CIDR 블록을 부여하여 동일 VPC내 모든 접속을 허용
- RDP 프로토콜, `3389`번으로 윈도우 가상머신에 접속
> - `Internet` - `Internet Options` - `LAN settings` - `Proxy Server` - `리눅스 EC2 private IP`, `port`: `3128`
![](https://velog.velcdn.com/images/ragnarok_code/post/3bc18559-7446-4793-8922-f7312c802004/image.png)


- 리눅스 EC2를 프록시 서버로 사용할 것이므로 squid 설치 및 설정



### Squid 설치 및 설정
sudo yum update -y

sudo yum install squid.x86_64

sudo chkconfig squid on

sudo service squid start

sudo cat /etc/squid/squid.conf

sudo vi /etc/squid/allowed_websites.txt

```bash
.microsoft.com
.redhat.com
```

sudo cat /etc/squid/allowed_websites.txt

sudo vi /etc/squid/squid.conf


![](https://velog.velcdn.com/images/ragnarok_code/post/517c3c88-1a69-4f33-9a4a-847a14500472/image.png)



sudo squid -k reconfigure

sudo service squid stop

sudo service squid start


