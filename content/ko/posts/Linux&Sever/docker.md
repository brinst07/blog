---
author: "brinst"
title: "docker log"
date: 2021-11-17T22:39:24+09:00
draft: false
---

## docker 이미지 파일로 저장하기
```shell
docker save -o <image> <filename>
```

## docker 이미지 파일 로드하기
```shell
docker load < <filename>
```
그 후 해당 이미지를 run 명령어로 실행하면 된다.

## docker log 찍기
docker container의 로그를 보려면 다음 명령어를 입력한다.
```shell
docker logs --tail 10 <containerID>
```
위와 같이 입력하면 —tail 10 이 옵션 때문에 해당하는 컨테이너의 로그에서 뒤에서 10줄만 출력해준다.
즉 실시간으로 조회가 안된다.

만약 실시간으로 로그를 보고 싶다면 다음과 같이 입력한다.
```shell
docker logs -f <containerID>
```
-f 옵션 때문에 실시간으로 조회가 가능하다.

## docker 마운트하기
업무를 보다가 도커폴더와 서버폴더를 마운트 할일이 생겨서 알게된 명령어를 정리한다.
```shell
docker run -v <서버경로>:<도커경로> ~~
```
위와 같이 입력하면 된다.