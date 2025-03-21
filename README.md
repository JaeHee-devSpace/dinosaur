# 시나리오 : 大기업 서버 관리자 이현정의 고민


## 👨‍👨‍👦‍👦 팀원 소개  
| <img src="https://avatars.githubusercontent.com/u/87555330?v=4" width="200px"> | <img src="https://avatars.githubusercontent.com/u/82265395?v=4" width="200px"> | <img src="https://github.com/JaeHee-devSpace.png" width="200px"> | <img src="https://avatars.githubusercontent.com/u/115103394?v=4" width="200px"> |
| :---: | :---: | :---: | :---: |
| [김민성](https://github.com/minsung159357) | [구민지](https://github.com/minjee83) | [박재희](https://github.com/JaeHee-devSpace) | [이현정](https://github.com/nanahj) |


# 1. 배경

이현정 사원은 한 대기업의 IT 관리자다. 이 회사는 고객 데이터를 저장하는 내부 서버를 운영 중이며, 사내 직원들이 원격으로 접속할 수 있도록 SSH (Secure Shell)를 사용하고 있다.최근 들어 서버 로그를 확인하던 이현정 사원은 이상한 점을 발견했다.
```
Mar 17 02:05:21 server sshd[4582]: Failed password for root from 203.0.113.45 port 54721 ssh2
Mar 17 02:05:33 server sshd[4621]: Failed password for root from 203.0.113.45 port 54758 ssh2
Mar 17 02:05:47 server sshd[4681]: Failed password for root from 203.0.113.45 port 54794 ssh2
```
❗ **새벽 시간대에 알 수 없는 IP에서 반복적으로 로그인 시도를 하고 있었다.** 
<br/>
더욱 놀라운 것은, 이런 공격이 **하루에 수천 번씩** 발생하고 있었다는 점이다. 이 공격이 성공하면 해커는 **서버에 불법적으로 침입**하여 데이터를 유출하거나 시스템을 마비시킬 수 있다.

<br/>

**해결책: 자동화된 SSH 로그인 감지 및 알림 시스템**

<br/>


**SSH 로그인 로그를 자동으로 수집하고, 일정 횟수 이상 로그인 실패가 발생한 IP를 자동으로 슬랙에 알림을 보내주는 시스템**을 구축하기로 했다.

**(1) 자동 로그 수집**

- `cron`을 이용해 **SSH 로그인 실패 기록을 자동으로 저장**
- 이상 징후를 분석하여 **추후 보안 정책을 강화할 데이터로 활용**


**(2) 자동 알림 수신**

- 로그인 실패가 **3회 이상 발생한 IP를 슬랙 알림으로 수신**

# **구현**
## 1. SSH 로그인 로그 수집
   ```
   sudo cat /var/log/auth.log | grep "sshd"
   ```
## 2. 자동 로그 수집 스크립트 작성
(1) ssh_log_collector.sh 스크립트 작성
```
#!/bin/bash

# 설정
SLACK_WEBHOOK_URL="_"  # Slack Webhook URL
LOG_FILE="/home/admin1/test.log"
FAILURE_TRACK_FILE="/home/admin1/.login_failure_tracker"  # 실패 여부를 기록할 파일

# 로그인 실패 횟수 확인 (최근 1시간 이내의 실패 횟수)
FAILURE_COUNT=$(grep "Failed password" /var/log/auth.log | wc -l)

# 이전에 알림을 보냈는지 확인
if [[ -f $FAILURE_TRACK_FILE ]]; then
    LAST_FAILURE_COUNT=$(cat $FAILURE_TRACK_FILE)
else
    LAST_FAILURE_COUNT=0
fi

# 로그인 실패 3회 이상이고, 이전에 알림을 보지 않았다면
if [ $FAILURE_COUNT -ge 3 ] && [ $FAILURE_COUNT -ne $LAST_FAILURE_COUNT ]; then
    # 알림을 Slack으로 전송
    curl -X POST -H 'Content-type: application/json' --data '{"text":"3회 이상 로그인에 실패했습니다"}' $SLACK_WEBHOOK_URL

    # 로그 파일에 기록
    echo "$(date) - 3회 이상 로그인에 실패했습니다." >> $LOG_FILE

    # 실패 횟수를 기록 (다시 실패 이력 확인을 위한 파일)
    echo $FAILURE_COUNT > $FAILURE_TRACK_FILE
else
    echo "$(date) - 로그인 실패가 3회 미만이거나 이전에 알림을 보냈습니다." >> /home/admin1/check_login_failures.log
fi
```
👉 이 스크립트는 auth.log에서 SSH 로그인 실패 기록을 가져와서 3회 미만이라면 /var/log/check_login_failures.log에 저장하고 3회 이상이라면 /home/admin1/test.log 에 저장.



(2) 실행 권한 부여
```
chmod +x /home/admin1/check_login_failures.sh
```


(3) 실행 테스트
```
cat check_login_failures.log
```
![image](https://github.com/user-attachments/assets/b843199d-cefd-4473-b217-29a647619835)

```
cat test.log
```
![image](https://github.com/user-attachments/assets/01965261-c672-469c-b7f8-e7cdc222a4f4)



## 3. 자동 실행
```
* * * * * /bin/bash /home/admin1/check_login_failures.sh > /home/admin1/check_login_fail>
```

## 4. 알림 연동 스크립트 추가
slack api 주소 발급받아 **로그인 3회 실패** 알림 전송

```
 # 알림을 Slack으로 전송
    curl -X POST -H 'Content-type: application/json' --data '{"text":"3회 이상 로그인에 실패했습니다"}' $SLACK_WEBHOOK_URL
```

## 5. 결과

![image](https://github.com/user-attachments/assets/5ba4b9a8-d07d-4ffc-a7fa-03c1dc3625cd)



**기대효과**

✅ **서버 보안 강화:** 자동으로 공격을 탐지하고 차단하여 해킹 위험 감소 

✅ **관리 효율성 증가:** IT 관리자가 24시간 로그를 확인하지 않아도 됨

✅ **데이터 보호:** 고객 정보 및 중요 시스템을 안전하게 유지
