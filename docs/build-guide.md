# 구축 순서

실제로 구축한 순서대로 정리했습니다. 단계마다 확인 명령을 돌려 보고 다음으로 넘어갔습니다.

- 장비 설정 파일: [configs](../configs)
- 주소 표: [design.md](design.md)

## 0. 설정 넣는 방법

- `configs/` 폴더의 txt 파일을 복사해 장비 CLI에 붙여넣기
- 라우터 처음 켤 때 `initial configuration dialog` 질문에는 `no`
- `crypto key generate rsa` 다음의 `1024`는 키 길이 질문에 대한 답. 붙여넣기가 꼬이면 이 두 줄만 직접 입력
- 설정 후에는 CLI 접속 시 로그인 필요 (계정은 [accounts.md](accounts.md))
- `show` 명령은 `enable`로 `#` 모드에 들어가서 실행

## 1. 장비 배치

| 종류 | 위치 | 모델 | 개수 | 이름 |
|---|---|---|---|---|
| 라우터 | Network Devices → Routers | 2911 | 4 | R1, R2, R3, ISP |
| 스위치 | Network Devices → Switches | 2960-24TT | 4 | HQ-SW1, SRV-SW1, BR-SW1, BR2-SW1 |
| 서버 | End Devices | Server-PT | 4 | DHCP-DNS, WEB, FTP-MAIL, Internet-Web |
| PC | End Devices | PC-PT | 9 | Sales-PC1·2, Dev-PC1·2, Admin-PC, Branch-PC1·2, Branch2-PC1·2 |
| 무선 AP | Network Devices → Wireless Devices | AccessPoint-PT | 1 | Guest-AP |
| 노트북 | End Devices | Laptop-PT | 1 | Guest-Laptop |

모델을 고른 이유

- 2911: R1에 기가비트 포트가 3개(G0/0~G0/2) 필요해서. 1941, 2901은 2개뿐
- 2960: L2 스위치로 충분. 3560 같은 L3 스위치는 이번 구성에서 쓰지 않음
- AccessPoint-PT: AP 1대라 단독형으로 충분. 가정용 공유기(WRT300N)는 기업망에 맞지 않음
- ISP와 Internet-Web은 실제 인터넷 대신 라우터 1대와 서버 1대로 흉내 냄

## 2. 모듈 장착

케이블을 꽂기 전에, 전원을 끈 상태에서 장착합니다.

- R1, R2, R3: HWIC-2T를 맨 왼쪽 슬롯(eHWIC 3)에 장착 → 포트 이름 `Serial0/3/0`, `Serial0/3/1`
  - 처음에는 `Serial0/0/0`이 생길 줄 알았는데 슬롯 번호를 따라 `0/3/x`가 됨. 세 대 모두 같은 슬롯에 꽂아 이름을 맞춤
- Guest-Laptop: 기본 유선 랜카드(PT-LAPTOP-NM-1CFE)를 빼고 무선 카드 WPC300N 장착 → `Wireless0` 생성 확인

## 3. 케이블 연결

| 케이블 | 장비 A | 장비 B |
|---|---|---|
| Straight | Sales-PC1, Sales-PC2 | HQ-SW1 Fa0/1, Fa0/2 |
| Straight | Dev-PC1, Dev-PC2 | HQ-SW1 Fa0/11, Fa0/12 |
| Straight | Admin-PC | HQ-SW1 Fa0/21 |
| Straight | Guest-AP Port 0 | HQ-SW1 Fa0/24 |
| Straight | HQ-SW1 G0/1 | R1 G0/0 |
| Straight | R1 G0/1 | SRV-SW1 G0/1 |
| Straight | DHCP-DNS, WEB, FTP-MAIL | SRV-SW1 Fa0/1, Fa0/2, Fa0/3 |
| Cross | R1 G0/2 | ISP G0/0 |
| Cross | ISP G0/1 | Internet-Web Fa0 |
| Straight | R2 G0/0 | BR-SW1 G0/1 |
| Straight | Branch-PC1, Branch-PC2 | BR-SW1 Fa0/1, Fa0/2 |
| Straight | R3 G0/0 | BR2-SW1 G0/1 |
| Straight | Branch2-PC1, Branch2-PC2 | BR2-SW1 Fa0/1, Fa0/2 |
| Serial DCE | R1 S0/3/0 | R2 S0/3/0 |
| Serial DCE | R1 S0/3/1 | R3 S0/3/0 |
| Serial DCE | R2 S0/3/1 | R3 S0/3/1 |

- 시리얼은 먼저 클릭한 쪽이 DCE가 됨. `clock rate`를 넣은 쪽(R1 두 포트, R2 S0/3/1)을 먼저 클릭
- 스위치 포트 번호대로 VLAN이 나뉘므로 포트를 정확히 맞춤
- 라우터 쪽 선은 설정을 넣기 전까지 빨간색

## 4. 스위치

`configs/HQ-SW1.txt`, `SRV-SW1.txt`, `BR-SW1.txt`, `BR2-SW1.txt` 붙여넣기

HQ-SW1 확인

```
show vlan brief
show port-security
```

| VLAN | 포트 |
|---|---|
| 10 SALES | Fa0/1 – 10 |
| 20 DEV | Fa0/11 – 20 |
| 30 GUEST | Fa0/24 |
| 99 MGMT | Fa0/21 – 22 |

- Port Security는 Fa0/1~22에 최대 2개, 위반 시 shutdown
- AP 포트(Fa0/24)는 제외. 무선 기기가 늘면 MAC 수가 2개를 넘기 때문

## 5. 외부망 (ISP, Internet-Web)

- ISP: `configs/ISP.txt`
- Internet-Web: IP `198.51.100.10/24`, 게이트웨이 `198.51.100.1`, HTTP On, 첫 화면을 `External Internet Site`로 수정

## 6. 본사 라우터 R1

`configs/R1.txt` 붙여넣기

```
show ip interface brief
```

- G0/0.10, .20, .30, .99, G0/1, G0/2가 `up up`인지 확인 (시리얼은 R2, R3 설정 전이라 down)
- HQ-SW1 `show interfaces trunk`에서 허용 VLAN `10,20,30,99` 확인
- R1에서 `ping 203.0.113.1`로 ISP 연결 확인

## 7. 지사 라우터 R2, R3

`configs/R2.txt`, `configs/R3.txt` 붙여넣기. 1분쯤 지나면 OSPF 이웃이 맺어짐

```
show ip ospf neighbor
show ip route
show controllers serial0/3/1
```

- R1: 2.2.2.2, 3.3.3.3 FULL
- R3: 1.1.1.1, 2.2.2.2 FULL
- R2 라우팅 테이블에 `O*E2 0.0.0.0/0` (R1이 OSPF로 뿌린 기본 경로)
- R1, R2의 DCE 포트에서 `DCE` 확인

## 8. 서버팜

| 서버 | IP | Mask | Gateway | DNS |
|---|---|---|---|---|
| DHCP-DNS | 192.168.10.162 | 255.255.255.240 | 192.168.10.161 | 192.168.10.162 |
| WEB | 192.168.10.163 | 255.255.255.240 | 192.168.10.161 | 192.168.10.162 |
| FTP-MAIL | 192.168.10.164 | 255.255.255.240 | 192.168.10.161 | 192.168.10.162 |

DHCP 풀 (DHCP-DNS 서버)

| Pool | Gateway | Start IP | Mask | Max | 범위 |
|---|---|---|---|---|---|
| serverPool | 192.168.10.161 | 192.168.10.170 | 255.255.255.240 | 4 | .170 – .173 |
| SALES | 192.168.10.1 | 192.168.10.10 | 255.255.255.192 | 50 | .10 – .59 |
| DEV | 192.168.10.129 | 192.168.10.135 | 255.255.255.224 | 20 | .135 – .154 |
| GUEST | 192.168.10.225 | 192.168.10.230 | 255.255.255.224 | 20 | .230 – .249 |
| MGMT | 192.168.10.177 | 192.168.10.180 | 255.255.255.240 | 10 | .180 – .189 |
| BRANCH | 192.168.10.65 | 192.168.10.74 | 255.255.255.192 | 50 | .74 – .123 |
| BRANCH2 | 192.168.10.209 | 192.168.10.212 | 255.255.255.240 | 10 | .212 – .221 |

- DNS 서버는 모두 192.168.10.162
- 첫 풀은 기본 serverPool을 고쳐 Save, 나머지는 Add. 여기서 Save를 누르면 선택된 풀을 덮어씀

DNS A 레코드

| Name | Address |
|---|---|
| www.company.com | 192.168.10.163 |
| ftp.company.com | 192.168.10.164 |
| mail.company.com | 192.168.10.164 |
| www.external.com | 198.51.100.10 |

나머지 서비스

- WEB: HTTP·HTTPS On, 첫 화면 `Company Intranet`
- FTP-MAIL: FTP 계정 dev1 (권한 전체), 메일 도메인 `company.com`, 계정 sales1·dev1·branch1
- 웹 서버와 헷갈리지 않게 DHCP-DNS, FTP-MAIL의 HTTP는 Off

## 9. 게스트 무선랜

Guest-AP Port 1과 Guest-Laptop Wireless0에 같은 값 입력

| 항목 | 값 |
|---|---|
| SSID | Company-Guest |
| 인증 | WPA2-PSK |
| 비밀번호 | Guest2026! |
| 암호화 | AES |

- 노트북 IP는 DHCP. AP와 노트북 사이에 무선 연결선이 생기면 연결된 것
- `ipconfig`로 192.168.10.230~249, 게이트웨이 .225 확인

## 10. PC

- PC 9대 모두 IP Configuration을 DHCP로 설정
- Sales-PC1, Dev-PC1, Branch-PC1에 메일 설정 (서버 `mail.company.com`, 계정은 [accounts.md](accounts.md))

## 11. 테스트

테스트 22개 결과는 [README](../README.md#테스트)에 정리했습니다. 아래 두 개는 장비 상태를 바꾸는 테스트라 순서와 복구 방법을 따로 적었습니다.

T22 WAN 장애 우회

1. R3에서 `interface serial0/3/1` → `shutdown`
2. Branch2-PC1에서 `tracert 192.168.10.74`
3. 경로가 `.209 → .201 → .74`(직통)에서 `.209 → .197 → .194 → .74`(본사 경유)로 바뀌는지 확인
4. `no shutdown`으로 복구

T15 Port Security 위반 (포트가 꺼지므로 맨 마지막에 실행)

1. Sales-PC1의 원래 MAC 주소를 적어 둠
2. MAC을 두 번 바꾸며 `ping 192.168.10.1`
3. 세 번째 MAC에서 Fa0/1이 `Secure-shutdown`이 되는지 확인 (`show port-security interface fa0/1`)
4. MAC을 원래대로 돌리고 HQ-SW1 Fa0/1에서 `shutdown` → `no shutdown`

## 문제가 생겼을 때 먼저 본 것

| 증상 | 확인할 것 |
|---|---|
| 라우터 포트가 빨간색 | `show ip interface brief`, `no shutdown` 여부 |
| PC가 169.254.x.x를 받음 | 게이트웨이의 `ip helper-address`, DHCP 풀 Gateway 값 |
| 본사 VLAN 간 ping 실패 | `show interfaces trunk`, 서브인터페이스 `encapsulation dot1Q` 번호 |
| OSPF 이웃이 안 보임 | 시리얼 양쪽 up/up, DCE 쪽 `clock rate`, `network` 와일드카드, passive 여부 |
| 인터넷만 안 됨 | `show ip nat translations`, G0/2의 `ip nat outside`, 기본 경로 |
| SSH 거부 | 접속한 PC가 관리 VLAN인지, RSA 키 생성 여부 |
| 포트가 갑자기 꺼짐 | Port Security 위반. `show port-security interface` 확인 후 shutdown → no shutdown |
| 노트북이 연결 안 됨 | AP와 노트북의 SSID, 비밀번호, 인증 방식이 한 글자까지 같은지 |
| 게스트 인터넷이 이름으로만 안 됨 | `nslookup`, GUEST_IN의 DNS 허용 줄 |
