---
author: "brinst"
title: "JMeter"
date: 2021-01-22T22:39:24+09:00
draft: false
---

## JMeter란??
>Apach JMeter는 웨 애플리케이션에 중점을 둔 다양한 서비스의 성능을 분석하고 측정하기 위한 로드 테스트 도구로 사용 할 수 있는 Apache 프로젝트이다(Wikipedia 참조)

JMeter를 이용하면 웹 어플리케이션의 부하를 걸어 성능 테스트를 할 수 있다.
마침 회사에서 JMeter에 대한 세미나가 열려 참석 후 이에 대한 간단한 포스팅을 해보려고 한다.

### JMeter 설치
JMeter를 사용하기 위해선 일단 먼저 다운로드를 해야한다.
>[JMeter 홈페이지](https://jmeter.apache.org/download_jmeter.cgi)

위 페이지에 접속하게 되면 Binaries에 두가지 버전이 존재하는 것을 알 수 있는데,
zip은 윈도우에서 tgz는 리눅스 환경에서 사용하면 된다.

이 포스팅은 zip버전을 기준으로 설명하고자 한다.

압축을 해제하고 bin 폴더에 들어가보면 ApachJMeter Jar 파일을 볼수 있다.
![image](https://user-images.githubusercontent.com/60083557/105463231-154aa380-5cd3-11eb-8877-1ec9a6e9a344.png)

위 파일을 실행하면 JMeter 프로그램이 실행된다.


![image](https://user-images.githubusercontent.com/60083557/105463385-55aa2180-5cd3-11eb-80f6-d30d3e453da8.png)

위 화면이 출력되었다면 설치하는데 성공한 것이다

### JMeter 사용해서 테스트 준비하기
이제 JMeter를 설치했으니 테스트를 진행한다.
좌측 상단에 보면 테스트 계획에 하위 항목들을 추가하여 테스트를 진행하는 방식으로 진행된다. 

#### Thread Group
Thread Group이란, JMeter에서 다양한 하위 Element 항목들을 제어하기 위한 시작점이다. Thread Group을 생성하면 부하 테스트 수행 시 원하는 항목들도 변경하기 위한 설정 항목들이 나온다.

테스트 계획에 쓰레드 그룹을 추가해보자
![image](https://user-images.githubusercontent.com/60083557/105463699-cbae8880-5cd3-11eb-8d4f-69fb0d44a4c1.png)

쓰레드 그룹을 추가하면 다음과 같은 화면을 볼 수 있다.
![image](https://user-images.githubusercontent.com/60083557/105463902-1203e780-5cd4-11eb-8dff-2f928991001b.png)

다른 항목들은 한글로 해석되어 있기에 ramp up 시간만 간단하게 설명하자면 쓰레드들을 실행시키기 위한 시간을 의미한다고 한다.
만약 쓰레드를 30개 주고 ramp-up을 120초를 준다고 하면, 각 1개의 쓰레드가 4초 간격으로 동작한다.

#### Loop Count
한 Thread 당 sampler로 있는 테스트 시나리오를 몇 번 돌릴지를 설정

다음 작업으로 넘어기가전에 먼저 테스트하길 원하는 페이지를 실행하고, Proxy설정을 진행한다.

#### Proxy란?
Proxy란 `'대리'`라는 의미로, 네트워크 기술에서는 프로토콜에 있어서 대리 응답 등에서 친숙한 개념이다.
보안 분야에서는 주로 보안 상의 이유로 직접 통신할 수 없는 두 점 사이에서 통신을 할 경우 그 사이에 있어서 중게기로서 대리로 통신을 수행하는 기능을 가리켜 `'프록시'`, 그 중계 기능을 하는 것을 `'프록시 서버'`라고 부른다.

우리는 프록시 설정을 진행함으로써 JMeter를 중계기로 두고 테스트를 진행하게 되는 것이다.

윈도우 사용자를 기준으로 검색창에 프록시를 입력하면 프록시 설정 변경 메뉴가 출력된다.
![image](https://user-images.githubusercontent.com/60083557/105465223-fef21700-5cd5-11eb-9267-1bbcd320f9ea.png)

설정 메뉴로 진입하면 다음과 같은 메뉴가 출력된다.

하단처럼 주소에는 테스트 하고 싶은 주소 포트에는 원하는 포트를 입력해준다.

나는 로컬서버로 진행하고 있기 때문에 localhost를 입력해줬다.
![image](https://user-images.githubusercontent.com/60083557/105465380-2ba62e80-5cd6-11eb-81c7-5cce66e035a1.png)

설정을 완료하고 저장버튼을 눌러준다. 이렇게 프록시 설정이 완료 되었다.


#### HTTP(S) Test Script Recorder
Proxy를 설정하여 요청 URL을 수집하여 sampler를 자동으로 생성하기 위한 작업이다.
Recorder를 키고 내가 하고 싶은 테스트를 웹상에서 진행하게 되면 이 행동들이 녹화되어 똑같이 테스트된다고 이해하면 편하다.

![image](https://user-images.githubusercontent.com/60083557/105464287-a79f7700-5cd4-11eb-80c5-20b86792a2a8.png)

테스트 계획에 정상적으로 추가하면 다음과 같은 화면이 출력되는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/60083557/105464679-4af08c00-5cd5-11eb-9ba3-e476c2174aa5.png)

이어서 프록시 설정시 진행한 포트 번호와 일치하는지 확인해주고
타겟컨트롤러에서 테스트 계획생성 탭에서 테스트계획 > 쓰레드 그룹으로 설정해준다.

또한 하단의 Redirect들을 따라가기를 해제해준다.
![image](https://user-images.githubusercontent.com/60083557/105466017-0534c300-5cd7-11eb-82f2-ef436972fd3c.png)


설정이 완료되었다면 시작버튼을 누르고 테스트 원하는 작업을 녹화한다.

![image](https://user-images.githubusercontent.com/60083557/105466444-93a94480-5cd7-11eb-8d49-044a62a1cb46.png)

작업이 완료되었다면 중지버튼을 누르고 쓰레드 그룹을 열어보자
다음과 같이 많은 항목들이 생겼을 것이다.
이 sampler를 가지고 테스트를 수행하게 되는 것이다.

![image](https://user-images.githubusercontent.com/60083557/105466681-f26ebe00-5cd7-11eb-8a11-886edbc23500.png)

이제 테스트계획을 저장해보자
테스트 계획을 선택하고 저장버튼을 누르면 jmx파일로 저장되는 것을 확인 할 수 있다.
![image](https://user-images.githubusercontent.com/60083557/105466824-29dd6a80-5cd8-11eb-93e3-7d5fd0296193.png)

### JMeter로 테스트 해보기
위에서 jmx파일도 준비했고 설치도 완료했으니 이제 본격적으로 테스트를 진행해보도록 하자.
기본적으로 하나의 서버를 실행하면 다음 명령어를 사용하여 테스트하면 된다.
```
$(위치)jmeter.sh -n -t (jmx파일위치) -j (output위치)/output/run.log -l (output위치)/output/result.jtl -e -o (output위치)/output/html 
```

이 명령어를 수행하게 되면 jmeter.sh로 jmx을 실행하여 output파일들을 생성하게 됩니다.

result 파일들을 보며 테스트 결과를 확인할 수 있다.

Jmeter로 테스트를 할 때 dstat을 사용하면 cpu나 메모리등 하드웨어 부하에 대한 수치를 쉽게 살펴 볼수 있다.

#### dstat
1. 설치
`yum install dstat`

2. 사용<br>
`dstst`을 옵션없이 실행하면 -cdngy 옵션을 준것과 동일하며, CPU,disk,network,paging,system 정보를 갱신하며 보여준다.

3. 로그파일 파일로 저장<br>
`dstat --output dstat-log.csv -cdnpmrt`<Br>


먼저 dstat을 실행한 뒤에 run을 실행하여 jmeter로 result 파일으로는 응답속도를 테스트하고, dstat으로 하드웨어 부하량에 대하여 알아볼수 있다.

