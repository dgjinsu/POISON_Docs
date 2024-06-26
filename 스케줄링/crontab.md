## 크론탭

- 리눅스나 유닉스 환경에서 실행 프로그램을 설정한 시간에 자동으로 실행할 수 있도록 설정해주는 프로그램이다.
- 실행 환경을 설정하고 유지 관리하기 위한 시간, 날짜 또는 간격을 지정하여 주기적으로 프로그램을 실행가능하다.
- 시스템 유지관리를 위한 자동화를 위해 이용되고 파일 다운로드나 정기적 이메일 다운로드를 위한 작업에도 많이 사용되고 있다.


### 크론탭 설치
- 크론탭 설치
```
$ sudo apt install cron
```

- 설치 확인
- 만약 아래에서 명령한 status를 보고 상태가 active 설치 완료, 만약 active가 아니라면 크론탭을 시작해줘야한다. 
```
$ sudo service cron status
```

- 크론탭 서비스 시작
```
$ sudo service cron start
```

### 크론탭 명령어
- 작업 할당
```
$ crontab -e
```
- 위 커맨드를 입력하면 vim과 같은 텍스트 편집기가 열리는데텍스트 편집기를 이용해 작업을 할당하면 된다.

- 할당된 작업 리스트 확인
```
$ crontab -l
```


### 작업 할당 방법

크론탭에서는 텍스트 입력을 통해 작업을 할당하는데 할당하는 포맷은 다음과 같다.
```
* * * * * source /home/dlit/work/pred_batch.sh
```
위 명령어는 매분 source /home/dlit/work/pred_batch.sh명령어를 실행하라는 뜻이다. 커맨드를 보면 별 표시가 5개 있는데 이 별 표시는 반복실행 주기를 의미한다. 별 다섯개는 순서대로 다음을 의미합니다.


```
   *          *          *         *          *
분(0-59)  시간(0-23)  일(1-31)   월(1-12)   요일(0-7)
```
즉 위와 같이 모두 별표시면 매분, 매시간, 매일, 매월, 매요일을 의미한다. 요일은 0, 7은 일요일을 의미하며 월요일은 1, 토요일은 6에 해당한다.

- 예시
```
23 15 17 5 * source /home/dlit/work/pred_batch.sh
```
- 위와 같은 코드는 5월 17일 오후 3시 23분에 해당 스크립트를 돌리라는 의미


### 테스트 작업
- 작업 등록
- ![image](https://github.com/dgjinsu/POISON_Docs/assets/97269799/51711112-6654-458c-a78a-613e6259dbdd)
- wq!를 입력하여 저장후 터미널 밖으로 나와 crontab: installing new crontab 문구가 나오면 정상적으로 설치된 것을 확인할 수 있다.
- sudo crontab -l 를 터미널에 입력하여 등록된 크론택이 정상적으로 리스트에 등록이 되었는지 확인해보면 아래와 같이 볼 수 있다.
- ![image](https://github.com/dgjinsu/POISON_Docs/assets/97269799/fed53dda-d13e-42f0-a75c-c5cd1c82c6d1)

### 로그 확인
- 크론탭이 정상적으로 실행되고 있는가의 로그를 확인하고 싶으면 아래와 같이 log기록을 추가하여 누적되는 기록을 확인한다.
```
* * * * * python3 /home/kevin/scheduler/batch.py >> /home/kevin/batch.log
```
- /home/kevin/batch.log에서 로그를 확인할 수 있다.
- python 코드는 아래와 같이 작성하였다.
``` python
with open("/home/kevin/scheduler/result.txt", "w") as file:
    for i in range(10):
        file.write("test\n")
```

<br><br>
- 크론탭 적용을 위해선 restart해줘야 한다.
- 이때 sudo를 붙이니 적용되지 않았다.
```
service cron restart
```
![image](https://github.com/dgjinsu/POISON_Docs/assets/97269799/28e1a57f-be70-4780-ba8b-08bbf938b904)

- 적용을 완료하고 나면 로그 파일이 생성된다.
![image](https://github.com/dgjinsu/POISON_Docs/assets/97269799/3962fb82-6462-4374-a2e3-2800a4a754a4)
- 크론탭이 실행 된 후엔 result.txt 결과 파일이 잘 생성된 것을 확인할 수 있었고 test로 내용 또한 채워져있었다.
![image](https://github.com/dgjinsu/POISON_Docs/assets/97269799/38a8b488-ae5f-4440-bfbe-8f6890ac217f)



### shell script 실행 후 다른 서버로 결과 파일 전송시켜주기
- 신규 사용자가 새롭게 POI에 대한 리뷰를 작성한다면 ai 모델을 전체 재학습 시켜줘야 한다.
- 따라서 ai 모델 학습 python 파일을 돌리고, 만들어진 결과를 스프링 서버 환경으로 전송시킬 생각이다.
- 위 과정이 정상적으로 동작하는지 테스트 해보려 한다.
<br><br>


- test.sh 쉘 스크립트 생성
```
# python 파일 실행
python3 /home/kevin/scheduler/batch.py

# 파이썬 스크립트 실행 후 생성된 result.txt 파일을 xxx 호스트로 전송
scp /home/kevin/scheduler/result.txt [user]@[ip]:/home/kevin
```

- crontab 스케줄러 등록
```
* * * * * bash /home/kevin/scheduler/test.sh >> /home/kevin/batch.log
```

- 여기까지 하면 crontab 작업은 끝났다.
- 만든 test.sh 를 직접 실행시키면 목적지 서버의 비밀번호를 입력해야만 정상적으로 전송이 가능하다. 하지만 crontab에선 비밀번호를 입력해줄 수 없기 때문에 공개키를 적용해줘야 한다.
