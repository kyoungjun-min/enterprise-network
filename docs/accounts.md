# 실습용 계정

실습용 비밀번호입니다.

## 네트워크 장비 (콘솔, SSH)

| 장비 | Username | Password | enable 비밀번호 |
|---|---|---|---|
| R1, R2, R3 | admin | Admin123 | Cisco123 |
| HQ-SW1, SRV-SW1, BR-SW1, BR2-SW1 | admin | Admin123 | Cisco123 |
| ISP | 없음 (로그인 설정 안 함) | 없음 | 없음 |

## SSH 접속

관리 VLAN의 Admin-PC에서만 접속됩니다.

```
ssh -l admin 192.168.10.177
```

| 장비 | 접속 IP |
|---|---|
| R1 | 192.168.10.177 |
| R2 | 192.168.10.65 |
| R3 | 192.168.10.209 |
| HQ-SW1 | 192.168.10.178 |
| SRV-SW1 | 192.168.10.174 |
| BR-SW1 | 192.168.10.66 |
| BR2-SW1 | 192.168.10.210 |

## FTP (ftp.company.com)

| Username | Password | 권한 |
|---|---|---|
| dev1 | 1234 | 쓰기, 읽기, 삭제, 이름 변경, 목록 |
| cisco | cisco | Packet Tracer 기본 계정 |

## 이메일 (mail.company.com)

| PC | 이메일 주소 | User Name | Password |
|---|---|---|---|
| Sales-PC1 | sales1@company.com | sales1 | 1234 |
| Dev-PC1 | dev1@company.com | dev1 | 1234 |
| Branch-PC1 | branch1@company.com | branch1 | 1234 |

## 게스트 Wi-Fi

| SSID | 인증 | 암호화 | 비밀번호 |
|---|---|---|---|
| Company-Guest | WPA2-PSK | AES | Guest2026! |
