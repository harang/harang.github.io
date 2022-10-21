---
title: "Site to Site VPN"
categories:
  - AWS
  - Cloud
---


ex)

서울 리전 VPC IP 10.100.0.0/16, Subnet IP 10.100.0.0/24, EC2 IP10.100.0.182

도쿄 리전 VPC IP 10.200.0.0/16, Subnet IP 10.200.0.0/24, Public IP 13.230.39.165, EC2 IP 10.200.0.86 (고객사라고 가정)

![/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled.png](/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled.png)

고객의 게이트웨이와 연결될 게이트웨이 생성한 뒤, 원하는 vpc에 연결합니다.

![/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%201.png](/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%201.png)

VPN을 연결할 서버의 Public IP 주소를 입력합니다.

![/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%202.png](/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%202.png)

Site to Site VPN 설정에 들어가서, 
만들어놓은 가상 프라이빗 게이트웨이 선택하고
만들어놓은 고객 게이트웨이 선택한 뒤
정적 선택시 (customer ip입력)

![/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%203.png](/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%203.png)

라우팅 전파 편집 클릭

![/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%204.png](/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%204.png)

라우팅 전파 편집 활성화(가상 프라이빗 게이트웨이가 라우팅 테이블에 라우팅을 자동으로 전파)

![/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%205.png](/assets/img/2022-05-11-site-to-site-vpn-configuration/Untitled%205.png)

구성 다운로드( 구성 다운로드시 벤더사, 벤더사의 장비, 버전을 기입해줘야 그 벤더사장비에 맞는 문법으로 문서가 구성됨)

ex)

Amazon Web Services
Virtual Private Cloud

AWS utilizes unique identifiers to manipulate the configuration of
a VPN Connection. Each VPN Connection is assigned an identifier and is
associated with two other identifiers, namely the
Customer Gateway Identifier and Virtual Private Gateway Identifier.

Your VPN Connection ID                  : vpn-0f2260c678dc6de0c
Your Virtual Private Gateway ID         : vgw-032ba8f32027527b3
Your Customer Gateway ID                : cgw-0b4c2544373b8150d

This configuration consists of two tunnels. Both tunnels must be
configured on your Customer Gateway, but only one of those tunnels should be up at any given time.

At this time this configuration has only been tested for Openswan 2.6.38 or later, but may work with earlier versions.

---

## IPSEC Tunnel #1

```
This configuration assumes that you already have a default openswan installation in place on the Amazon Linux operating system (but may also work with other distros as well)

1. Open /etc/sysctl.conf and ensure that its values match the following:
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.accept_source_route = 0
2. Apply the changes in step 1 by executing the command 'sysctl -p'
3. Open /etc/ipsec.conf and look for the line below. Ensure that the # in front of the line has been removed, then save and exit the file.
#include /etc/ipsec.d/*.conf
4. Create a new file at /etc/ipsec.d/aws.conf if doesn't already exist, and then open it. Append the following configuration to the end in the file:
#leftsubnet= is the local network behind your openswan server, and you will need to replace the <LOCAL NETWORK> below with this value (don't include the brackets). If you have multiple subnets, you can use 0.0.0.0/0 instead.
#rightsubnet= is the remote network on the other side of your VPN tunnel that you wish to have connectivity with, and you will need to replace <REMOTE NETWORK> with this value (don't include brackets).

conn Tunnel1
authby=secret
auto=start
left=%defaultroute
leftid=13.230.39.165
right=54.180.5.191
type=tunnel
ikelifetime=8h
keylife=1h
phase2alg=aes128-sha1;modp1024
ike=aes128-sha1;modp1024
auth=esp
keyingtries=%forever
keyexchange=ike
leftsubnet=<LOCAL NETWORK>
rightsubnet=<REMOTE NETWORK>
dpddelay=10
dpdtimeout=30
dpdaction=restart_by_peer

1. Create a new file at /etc/ipsec.d/aws.secrets if it doesn't already exist, and append this line to the file (be mindful of the spacing!):
13.230.39.165 54.180.5.191: PSK "XILjX_XM1tu0m9E9KH8MsA0JxPnX5F54"
```
---

## IPSEC Tunnel #2
```
This configuration assumes that you already have a default openswan installation in place on the Amazon Linux operating system (but may also work with other distros as well)

1. Open /etc/sysctl.conf and ensure that its values match the following:
net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.accept_source_route = 0
2. Apply the changes in step 1 by executing the command 'sysctl -p'
3. Open /etc/ipsec.conf and look for the line below. Ensure that the # in front of the line has been removed, then save and exit the file.
#include /etc/ipsec.d/*.conf
4. Create a new file at /etc/ipsec.d/aws.conf if doesn't already exist, and then open it. Append the following configuration to the end in the file:
#leftsubnet= is the local network behind your openswan server, and you will need to replace the <LOCAL NETWORK> below with this value (don't include the brackets). If you have multiple subnets, you can use 0.0.0.0/0 instead.
#rightsubnet= is the remote network on the other side of your VPN tunnel that you wish to have connectivity with, and you will need to replace <REMOTE NETWORK> with this value (don't include brackets).

conn Tunnel2
authby=secret
auto=start
left=%defaultroute
leftid=13.230.39.165
right=54.180.224.136
type=tunnel
ikelifetime=8h
keylife=1h
phase2alg=aes128-sha1;modp1024
ike=aes128-sha1;modp1024
auth=esp
keyingtries=%forever
keyexchange=ike
leftsubnet=<LOCAL NETWORK>
rightsubnet=<REMOTE NETWORK>
dpddelay=10
dpdtimeout=30
dpdaction=restart_by_peer

1. Create a new file at /etc/ipsec.d/aws.secrets if it doesn't already exist, and append this line to the file (be mindful of the spacing!):
13.230.39.165 54.180.224.136: PSK "uZgFivQfgaYcLJoTiuKaLotZpCIKtfxq"
```
---

## (OPTIONAL CONFIG) Tunnel Healthcheck and Failover
```
Openswan does not provide a built-in tunnel failover functionality. However, there are some third-party workarounds to this.

=== DISCLAIMER ===
Please be aware that AWS is in no way responsible for any of the use, management, maintenance, or potential issues you may encounter with the third-party workarounds. It is strongly recommended that you thoroughly test any failover solution prior to implementing it into your production environment

Additional Notes and Questions

- Amazon Virtual Private Cloud Getting Started Guide:
[http://docs.amazonwebservices.com/AmazonVPC/latest/GettingStartedGuide](http://docs.amazonwebservices.com/AmazonVPC/latest/GettingStartedGuide)
- Amazon Virtual Private Cloud Network Administrator Guide:
[http://docs.amazonwebservices.com/AmazonVPC/latest/NetworkAdminGuide](http://docs.amazonwebservices.com/AmazonVPC/latest/NetworkAdminGuide)
- XSL Version: 2009-07-15-1119716
```

해당 내용을 고객 서버의 라우터에서 설정합니다.
(터널링 정보)