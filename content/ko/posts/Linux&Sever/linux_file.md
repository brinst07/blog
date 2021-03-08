---
author: "brinst"
title: "리눅스 파일시스템과 권한"
date: 2021-02-08T22:39:24+09:00
draft: false
---

## 세미나 이유
당사의 패키지를 실행할때 실행해야 하는 서버는 총 4개로 이루어져있다.
이 서버를 실행할때 평소 root로 실행하기 때문에 문제가 생기지 않지만, 고객사마다 root를 주지않는 곳도 존재한다.
보안을 신경쓰는 곳에서는 root 권한을 주지 않고 일반계정으로 주는 추세이다.
각 서버를가 각각 다른 계정으로 실행될때 문제가 생겨 이에 대한 교육이 필요하여 세미나가 열리게 되었다.

## 유저 등록
```shell
useradd user1
```
위 명령어로 user1를 등록하면 /etc/passwd에 등록이 된다.
passwd에는 정말 passwd가 등록되지는 않는다. 그냥 이름일뿐..
user데이터를 가지고 있는 데이터베이스라고 생각하면된다.

passwd가 등록되어 있는 곳은 /etc/shadow에 들어있다.
root만 접근 가능하다.

passwd
![image](https://user-images.githubusercontent.com/60083557/107193228-f74ea400-6a31-11eb-8331-2b371f61eb64.png)
위 그림과 같이 user 정보가 작성되어있는 것을 알 수 있다.
passwd는 하단의 형식대로 작성된다.<br>
root(문자아이디):x(비밀번호):0(숫자아이디):0(그룹아이디):홈디렉토리:shell
<br>
shadow
![image](https://user-images.githubusercontent.com/60083557/107193400-29600600-6a32-11eb-90c4-ef5384903aae.png)
비밀번호가 암호화되어 작성되어있는 것을 확인할 수 있다.

## 그룹 등록
```shell
groupadd temp1
```
그룹정보는 /etc/group에 들어가 있다.
![image](https://user-images.githubusercontent.com/60083557/107193709-7c39bd80-6a32-11eb-84da-6010cafbc35b.png)

```shell
usermod -aG [그룹1,그룹2,그룹3] [계정명]
```
* -g : 그룹을 기본그룹으로 추가시 사용
* -G : 그룹을 보조 그룹으로 추가시 사용
* -a : 보충 그룹에 사용자를 추가하는데 사용되며 -G와 같이 사용되어야한다.
* 기본그룹(primary group)
   이 계정이 파일이나 뭘 만들 때 primary group 권한으로 만들어진다.

## 리눅스의 파일 체계
리눅스에서는 파일이든 디렉토리든 모두 파일로 관리 하고 있다.
* regular file -> 우리가 생각하는 파일
* directory file -> 폴더
* 디렉토리도 파일로 관리되기 때문에 폴더도 vim으로 열면 열린다.

## 파일의 권한
리눅스에서 ll 명령어를 입력하게 되면 맨 앞에 이상한 문자들이 있다.
![image](https://user-images.githubusercontent.com/60083557/107194432-5fea5080-6a33-11eb-8428-f17fd4a86410.png)

![image](https://user-images.githubusercontent.com/60083557/107194563-8c9e6800-6a33-11eb-8d89-f55575225b8b.png)
위 형식대로 만들어진다.

* 일반파일에서는 r이 읽기, w가 쓰기, x가 실행을 의미한다.
* 하지만 폴더에서는 다른 개념이다.
   * r : ls, ll를 하는 권한(폴더안에서 목록을 읽어오는 권한)
   * x : 접근 권한을 의미한다.(cd)
   * w : 파일을 추가하거나 삭제하거나 mv 할 수 있는 권한

## 권한주기
```shell
#other에 권한주기
chmod o+rwx user1

#other에서 모든 권한 삭제
chmod o-rwx user1

#그룹에 권한주기
chmod g+rwx user1
```
other에 권한을 주는 행위는 매우 위험한 행위이다.

## 동적으로 파일 권한 추가해주기
```
000|000|000|000 -> 2진수
special permission|7|7|7    => chomod 0+777 로도 가능하다. -> 폴더를 할때는 755가 국룰임 ㅎㅎ
set uid(4)u+s, set uid(2)u+g, set sticky bit(1)
```
* set uid
   * 보통 실행파일을 실행할 때, 실행한 유저 권한으로 실행되는 데, 파일을 만든 사람의 권한으로 실행이 된다.(폴더에 적용해도 별로 의미가 없음)
* set gid
   * 위와 같은 맥락임
   * usergroup이라는 권한을 가진 폴더에 user2 권한을 가진폴더가 들어가 mkdir를 하면 user2 권한을 가진 폴더가 생기게 된다.
   * chmod u+g usergroup을 하면 mkdir를 해도 usergroup 권한을 가진 폴더가 만들어진다.
* set sticky bit
   * world writable에서 사용하는 권한임
   * 리눅스에서 사용되는 /tmp 임시공간 폴더는 other에게 권한이 주어져있어 누구든지 접근가능하고 삭제 가능하지만 set sticky bit 되어있어 자신이 소유한 파일만 삭제할 수 있다.

## umask
* umask를 지정하면 새로 만들어지는 권한 default 값을 수정할 수 있다.
* default 권한
   * 폴더 : rwxrwxrwx
   * 파일 : rw-rw-rw-
* umask default
   * 0000
   * 뺄 권한을 작성해주면 된다.
   * 0100 -> user의 x권한 삭제
* unix에서는 쉘 상에서 실행되는 프로그램은 설정값이나 이런것들이 그대로 상속되기 때문에 초기화할때 umask를 적용하면 된다.
* umask는 bash_profile에 초기화 되면서 세팅되도록 한다.