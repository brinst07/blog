---
author: "brinst"
title: "cron 과 shellscript"
date: 2021-01-14T22:39:24+09:00
draft: false
---

## cron이란?
특정한 시간, 특정 시간 마다 어떤 작업을 수행하게 해줘야 할때 사용하는 명령어이다.
스케줄러 같은 개념이라고 생각하면 된다.
임시파일이나, log 파일 같이 계속 냅두면 처치하기 곤란한 파일들을 cron을 사용하게 쉽게 처리할 수 있다.
log같은 파일은 00시에 catlina 파일을 현재 날짜로 복제하고 원본파일은 로그를 삭제하는 방식으로 진행한다면
log 파일이 계속 커지지 않고 날짜별로 분류해서 확인할 수 있을 것이다.
리눅스에서 이러한 작업들을 cron으로 진행한다고 보면 된다.

또한 백업을 진행해야 한다고 할때 보통 새벽에 진행하게 되는데, cron이 없다면 개발자가 새벽에 출근하여
이 작업을 진행해야 할것이다. 하지만 cron이 있다면 새벽에 출근하지 않아도 백업을 진행할 수 있다.

## cron의 동작방식, cron 실행 흐름
![image](https://user-images.githubusercontent.com/60083557/104570044-c8dce380-5694-11eb-940a-26b6fe623505.png)
cron파일이 데몬이기 때문에 부팅시 백그라운드로 실행된다.

cron 동작방식을 보면 cron 데몬(crond)가 crontab을 참조하고 있다.
cron 데몬은 어떤 task를 언제 수행할지 crontab에서 찾아서 실행한다.
cron 데몬은 시스템 스케줄러 정보뿐만 아니라 각각 사용자가 설정한 작업 예약 정보도 crontab에서 확인한다.

## cron의 정규 표현식
|필드명|값의 허용 범위|허용된 특수문자|
|--|--|--|
|초(Seconds)|0~59|, - * /|
|분 (Minutes)|0~59|, - * /|
|시 (Hours)|0~23|, - * /|
|일 (Day)|1~31|, - * ? / L W|
|월 (Month)|1 ~ 12 or JAN ~ DEC|, - * /|
|요일 (Week)|0 ~ 6 or SUN ~ SAT|, - * ? / L #|
|연도 (Year)|empty or 1970 ~ 2099|, - * /|

## Cron 표현식 - 특수문자
● * : 모든 값을 뜻합니다.

● ? : 특정한 값이 없음을 뜻합니다. 

● - : 범위를 뜻합니다. (예) 월요일에서 수요일까지는 MON-WED로 표현

● , : 특별한 값일 때만 동작 (예) 월,수,금 MON,WED,FRI 

● / : 시작시간 / 단위  (예) 0분부터 매 5분 0/5

● L : 일에서 사용하면 마지막 일, 요일에서는 마지막 요일(토요일)

● W : 가장 가까운 평일 (예) 15W는 15일에서 가장 가까운 평일 (월 ~ 금)을 찾음

● # : 몇째주의 무슨 요일을 표현 (예) 3#2 : 2번째주 수요일

## crontab 파일의 7필드
### m h dom mon dow user command
|필드|설정 값 및 내용|
|--|--|
|m|분을 나타내고, 0~59로 설정한다.|
|h|시을 나타내고, 0~23으로 설정한다.|
|dom|날을 나타내고, 1~31로 설정한다.|
|mon|월을 나타내고, 1~12로 설정한다.|
|dow|요일을 나타내고, 0~7로 설정한다. 0과 7은 일요일에 해당하고 1은 월요일임
|user|user-name 사용자 이름
|command|실행할 명령어를 기입한다. 명령어 앞에 사용자 이름을 써도 된다.|

* '*' : 각 필드 자리에 * 기호가 오면 해당 필드의 모든 값을 의미한다.
* '-' : 그 사이의 모든 값
* ',' : 지정한 모든 값을 의미(불규칙한 값 지정시 주로 사용)
* '/' : '/'는 연결된 설정 값 범위에서 특정 주기로 나눌 때 사용한다.


## Cron 예시
![캡처](https://t1.daumcdn.net/cfile/tistory/25FB583359776C5710)

```
01  *  *  *  * root run-parts /etc/cron.hourly 
#매일 매시 1분에 root 권한으로 /etc/cron.hourly 내 모든 스크립트를 실행한다.

02  4  *  *  * root run-parts /etc/cron.daily  
# 매일 새벽 4시 2분에 /etc/cron.daily 내 모든 작업을 실행한다.
# 2분으로 한 것은 매 1분에는 시간별 작업이 실행되기 때문이다.

22  4  *  *  0 root run-parts /etc/cron.weekly 
# 매주 일요일(0) 4시 22분에 주간 작업들을 실행한다.

42  4  1  *  * root run-parts /etc/cron.monthly 
# 매월 1일 4시 42분에 월간 작업들을 실행한다.
```

## CronMaker 참고 사이트
http://www.cronmaker.com/;jsessionid=node068maia8exxmw1ia839g15hrpk46275.node0?0

---
## Shell Script
* 스크립트 : 텍스트 형식으로 저장되는 프로그램으로서 한줄씩 순차적으로 읽어 실행되도록 작성된 프로그램
    * 일반적으로 인터프리트 방식으로 동작하는 컴파일되지 않은 프로그램
* Shell Script : 운영체제의 쉘 즉 bash, ksh, csh 등이 읽어 실행해주는 스크립트 언어

### 환경
* Linux 기반 시스템
* Bash shell(/bin/bash)

### 작성방법
```shell
#!/bin/bash

하단에 스크립트를 작성한다.
```

### 쉘 스크립트 실행방법
```
./shell_test.sh

sh shell_test.sh
```

### 기본 문법
* 출력
    ```shell
    #! /bin/bash

    echo "hello~ script"
    ```
위 쉘 스크립트를 실행하면 hello script 한줄을 실행한다.

* 주석 : #
* 변수선언
    * =를 이용하여 선언하고 $를 이용해서 사용
    * =는 공백없이 붙여써야한다.
    * 지역변수에는 local을 붙인다.
    * ""로 감싸서 사용하면 더 안전하다. 문자열에 공백도 포함해서 값을 이용 할 수 있기 때문에
    ```shell
        #!/bin/bash

        # 변수 선언
        test="abc"
        num=100

        #변수 사용하기 및 출력
        echo ${test}
        echo ${num}
        echo "${test}"
        echo "${num}"

        #지역변수
        local local_var="local var"

        #변수가 선언되지 않았을 때 기본값을 지정하며 사용하는 방법
        defalut_var=${default_var="temp"}

    ```

* 배열
    * 배열의 인덱스는 0부터 시작함
    * 배열이름[@]는 배열의 모든 원소를 의미한다.
    * 배열의 원소 추가 시는 += 연산자를 사용한다.
    * 배열에서 원소를 삭제시
        * /를 사용해 해당 문자열 부분이 있으면 삭제, 삭제하고자 하는 문자나 문자열이 포함되어 있는 부분을 삭제
        * unset을 이용해 삭제
    ```shell
        #!/bin/bash

        arr_test=("1","2")
        echo "${arr_test[1]}"

        echo "${arr_test[@]}"

        arr_test+=("3","4")

        remove_element=(3)

        arr_test=("${arr_test[@]/$remove_element}")

        for i in ${!arr_test[@]}; do
	        if [ ${arr_test[i]} = ${remove_element} ]; then
		    # Use unset
		    unset arr_test[i]
	    fi
        done
    ```
    
* 조건문
    * if[조건]; then... elif[조건]; then...else 를 사용한다.
    ```shell
        #!/bin/bash

        # Numeric if statement
        test_num=5

        if [ "${test_num}" -eq 2 ]; then
            echo "number is 2"
        elif [ "${test_num}" -eq 3 ]; then
            echo "number is 3"
        else
            echo "number is not 2 or 3"
        fi

        # Numeric if statement
        test_num=5

        if (( ${test_num} > 3 )); then
            echo "number is greater than 3"
        else
            echo "number is not greater than 3"
        fi

        # String if statement
        test_str="test"

        if [ "${test_str}" = "test" ]; then
            echo "test_str is test"
        else
            echo "test_str is not test"
        fi
    ```

* 반복문
    * while 문의 사용
    ```shell
        #!/bin/bash

        cnt=0
        while (( "${cnt}" < 5 )); do
            echo "${cnt}"
            (( cnt = "${cnt}" + 1 )) # 숫자와 변수의 연산은 (())가 필요합니다.
        done
    ```

* for문의 사용법
    ```shell
        #!/bin/bash

        arr_num=(1 2 3 4 5 6 7)

        # 배열에 @는 모든 원소를 뜻합니다.
        for i in ${arr_num[@]}; do
            printf $i
        done
        echo

        for (( i = 0; i < 10; i++)); do
            printf $i
        done
        echo
    ```

#### 쉘스크립트 활용
github 블로그를 만들게 되면 submodule과 blog repository에 둘다 commit을 해야만이 정상적으로 블로그에 연동되는 것을 확인할 수 있다.
하지만 add -> commit -> message 작성 -> push 이 작업을 두번이나 해주는 것이 매우 번거롭다.
따라서 이러한 작업을 쉘 스크립트를 사용하여 간단히 처리할 수 있다.

```shell
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo -t zzo

# Go To Public folder
cd public
# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Come Back up to the Project Root
cd ..


# blog 저장소 Commit & Push
git add .

msg="rebuilding site `date`"
if [ $# -eq 1 ]
  then msg="$1"
fi
git commit -m "$msg"

git push origin master

```
위 쉘스크립트 코드를 보면 우리가 실제로 사용하는 git 명령어인 것을 확인 할 수 있다.
따라서 쉘 스크립트와 cron을 사용하면 훨씬더 편하게 작업할 수 있는 것을 알 수 있다.