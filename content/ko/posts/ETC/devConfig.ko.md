---
title: "DevConfig"
date: 2021-11-05T15:54:06+09:00
draft: fasle
---
# 개발환경 세팅하기
이번에 M1 Macbook을 개발용으로 구매하게 되어, M1 Mac으로 개발환경을 세팅하는 방법을 정리한다.

# HomeBrew
terminal에서 다음을 입력하여 homebrew를 설치한다.
```shell
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
설치가 종료 되면 다음 두줄을 실행한다.
```shell
$ echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/<USER_ID>/.zprofile
$ eval "$(/opt/homebrew/bin/brew shellenv)"
```
처음에 >> 이후의 부분을 지워서 정상적으로 세팅이 안되었었는데, 반드시 >> /Users/brinst/... 이런식으로 User_id를 넣어줘야한다.

# iterm2
https://iterm2.com/ 에서 iterm2를 설치한다.

## iterm2 세팅(ohmyzsh, powerlevel10K)
다음 명령어를 입력하여 oh-my-zsh를 설치한다.
```shell
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

터미널을 이쁘게 바꿔줄 powerlevel10k도 설치해준다.
```shell
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
이후에 재시작하면 테마설정 창이 뜰것이다.
만약 뜨지 않는다면 p10k configure 명령어로 설정을 시작한다.
