# 시나리오 : 大기업 서버 관리자 이현정의 고민


## 👨‍👨‍👦‍👦 팀원 소개  
| <img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/82265395?v=4" width="200px"> | <img src="https://github.com/JaeHee-devSpace.png" width="200px"> | <img src="https://avatars.githubusercontent.com/u/115103394?v=4" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김민성](https://github.com/minsung159357) | [구민지](https://github.com/minjee83) | [박재희](https://github.com/JaeHee-devSpace) | [이현정](https://github.com/nanahj) |


## 1. 배경

이현정 사원은 한 대기업의 IT 관리자다. 이 회사는 고객 데이터를 저장하는 내부 서버를 운영 중이며, 사내 직원들이 원격으로 접속할 수 있도록 SSH (Secure Shell)를 사용하고 있다.최근 들어 서버 로그를 확인하던 이현정 사원은 이상한 점을 발견했다.
```
Mar 17 02:05:21 server sshd[4582]: Failed password for root from 203.0.113.45 port 54721 ssh2
Mar 17 02:05:33 server sshd[4621]: Failed password for root from 203.0.113.45 port 54758 ssh2
Mar 17 02:05:47 server sshd[4681]: Failed password for root from 203.0.113.45 port 54794 ssh2
```
❗ **새벽 시간대에 알 수 없는 IP에서 반복적으로 로그인 시도를 하고 있었다.**
더욱 놀라운 것은, 이런 공격이 **하루에 수천 번씩** 발생하고 있었다는 점이다. 이 공격이 성공하면 해커는 **서버에 불법적으로 침입**하여 데이터를 유출하거나 시스템을 마비시킬 수 있다.

**해결책: 자동화된 SSH 로그인 감지 및 차단 시스템**


**SSH 로그인 로그를 자동으로 수집하고, 일정 횟수 이상 로그인 실패가 발생한 IP를 자동으로 차단하는 시스템**을 구축하기로 했다.

**(1) 자동 로그 수집**

- `cron`을 이용해 **SSH 로그인 실패 기록을 자동으로 저장**
- 이상 징후를 분석하여 **추후 보안 정책을 강화할 데이터로 활용**
```
코드 추가
```

**(2) 자동 IP 차단**

- 로그인 실패가 **3회 이상 발생한 IP를 자동으로 차단**
- `iptables` 또는 `fail2ban`을 이용해 **실시간으로 보안 강화**

```
코드 추가
```

## 결과

✅ **서버 보안 강화:** 자동으로 공격을 탐지하고 차단하여 해킹 위험 감소 

✅ **관리 효율성 증가:** IT 관리자가 24시간 로그를 확인하지 않아도 됨

✅ **데이터 보호:** 고객 정보 및 중요 시스템을 안전하게 유지
