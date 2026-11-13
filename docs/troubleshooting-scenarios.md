# 장애 실습 계획

구축을 마친 뒤, 일부러 설정을 하나씩 망가뜨리고 원인을 찾아보려고 정리한 목록입니다. 아직 실행하지 않았고, 증상은 예상입니다. 해 보고 결과가 다르면 그대로 고쳐 적을 생각입니다.

- 원본을 복사한 파일에서 하고, `write memory`는 하지 않음
- 장애 전, 장애 중, 복구 후를 캡처
- 복구 후 관련 테스트를 다시 실행

| 순서 | 장애 | 주제 | 확인하려는 것 | 다시 할 테스트 |
|---|---|---|---|---|
| 1 | NAT 바깥 포트 설정 누락 | 인터넷 | 어디까지 되는지로 범위 좁히기 | T08 |
| 2 | 게스트 ACL의 DNS 허용 줄 삭제 | ACL | 너무 막으면 정상 서비스도 막힘 | T17, T18 |
| 3 | 트렁크에서 VLAN 20 누락 | 스위칭 | 한 부서만 안 될 때 | T03, T09 |
| 4 | WAN 포트에 passive-interface | OSPF | 포트는 정상인데 이웃만 없을 때 | T20, T21 |

## 1. 사내는 되는데 인터넷만 안 됨

장애 (R1)
```
configure terminal
interface g0/2
no ip nat outside
end
```

예상 증상: 사내 웹은 되고 `www.external.com`만 안 됨

확인 순서
1. 사내 서버 접속 → 정상
2. R1에서 `ping 203.0.113.1` → 정상이면 회선 문제는 아님
3. `show ip nat translations` → 변환 기록 없음
4. `show running-config interface g0/2` → `ip nat outside` 없음

복구: `interface g0/2` → `ip nat outside`

## 2. 게스트 인터넷이 이름으로만 안 됨

장애 (R1)
```
configure terminal
ip access-list extended GUEST_IN
no 20
end
```

예상 증상: 노트북에서 `www.external.com`은 안 되고 `198.51.100.10`은 됨

확인 순서
1. IP로는 되고 이름으로는 안 됨 → DNS 쪽으로 좁힘
2. `nslookup www.external.com` → 실패
3. `show access-lists` → DNS 허용 줄이 없고 40번 deny 카운트 증가

복구
```
ip access-list extended GUEST_IN
20 permit udp 192.168.10.224 0.0.0.31 host 192.168.10.162 eq 53
```

## 3. 개발 부서만 통신이 안 됨

장애 (HQ-SW1)
```
configure terminal
interface g0/1
switchport trunk allowed vlan remove 20
end
```

예상 증상: Dev-PC1과 Dev-PC2끼리는 되지만 게이트웨이와 다른 부서로는 안 됨. 영업은 정상

확인 순서
1. 같은 VLAN끼리 ping → 정상
2. 게이트웨이 `192.168.10.129`로 ping → 실패
3. `show interfaces trunk` → 허용 VLAN에 20 없음

복구: `interface g0/1` → `switchport trunk allowed vlan add 20`

## 4. 포트는 정상인데 OSPF 이웃이 사라짐

장애 (R3)
```
configure terminal
router ospf 1
passive-interface serial0/3/1
end
```

예상 증상: R2–R3 이웃이 사라지고 지사 간 통신이 R1을 거침. 통신은 계속 되기 때문에 알아채기 어려움

확인 순서
1. Branch2-PC1에서 `tracert 192.168.10.74` → `.197` 경유로 바뀜
2. `show ip interface brief` → S0/3/1은 `up up`
3. `show ip ospf neighbor` → 2.2.2.2 없음
4. `show ip protocols` → passive 설정 발견

복구: `router ospf 1` → `no passive-interface serial0/3/1`
