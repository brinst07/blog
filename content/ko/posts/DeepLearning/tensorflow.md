---
author: "brinst"
title: "Tensorflow"
date: 2021-11-17T22:39:24+09:00
draft: false
---
# 배워야 하는 이유

contributors가 제일 많고 사용하는 사람들도 제일 많음

# Tensorflow란?

Tesorflow is an open source software library for numerical computation using data flow graphs.

텐서블로는 data flow 그래프를 사용하여 수치적인 계산을 할 수 있는 오픈소스 라이브러리이다.

# Data Flow Graph

텐서 플로우는 **dataflow graph**형식을 따르는데, 이는 기존의 programming 방식과는 다르게 computation 순서가 각각의 operation간의 관계에 따라 결정되는 것이다. 이는 **Parallel compution**에 아주 유용한 개념인데, 각각의 **node**는 연산의 최소 단위를 말하며 **edge**는 연산에 사용되는 data를 표현한다. 예를 들어 두 개의 matrix를 곱해서 하나의 matrix 결과를 만드는 `tf.matmul`를 생각해보자. 이 operation에는 2개의 matrix input을 의미하는 2개의 incoming edge, 연산을 담당하는 node 그리고 matrix multiplication의 결과를 의미하는 1개의 outgoing edge가 존재한다.

Node → operation

edge → data

![tensors_flowing.gif](/images/content/tensors_flowing.gif)

# Install tensorflow

현재 내 맥북에는 python이 설치 되어있다.

~~python을 설치하면 pip도 설치되는 줄 알고 있었는데, 맥에서는 아닌 가보다.~~

위 경우는 파이썬 홈페이지에서 pkg를 직접 다운받아 설치한 경우 발생할 수 있는 문제라고 한다.

따라서 따로 pip를 설치해주자.

[ahndoori](https://blog.nachal.com/1530)

위 방법도 존재하고 아래의 방법 처럼 brew 로 재설치 하는 방법도 존재한다.

```bash
brew uninstall python3
brew install python3
```

pip 설정이 완료 되었으면 다음과 같이 명령어를 입력하여 tensorflow를 설치한다.

```bash
pip install --upgrade tensorflow
pip install --upgrade tensorflow-gpu
```

- 도커를 사용하는 방법도 존재한다. (docker 만세!)

    ```bash
    docker pull tensorflow/tensorflow:latest  # Download latest stable image
    docker run -it -p 8888:8888 tensorflow/tensorflow:latest-jupyter  # Start Jupyter server
    ```


텐서플로 버전 확인

```bash
import tensorflow as tf
tf.__version__
```

[[파이썬] Tensorflow 2.0 session / placeholder 오류 해결 방법](https://eclipse360.tistory.com/40)

# Install Tensorflow-GPU
기존 맥북 같은 경우는 tensorflow-gpu를 활성해주지 않았다.(gpu가 amd기 때문에...)
하지만 애플이 M1를 직접 제작하게 되면서, gpu를 지원해주려는 노력을 하고있다....
다음 과정을 따라하면 m1에서도 gpu를 활성화하여 tensorflow를 돌릴수 있다.

### conda install
https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
위의 링크에서 파일을 받고 다음과정으로 설치를 진행한다.

```bash
chmod +x ~/Downloads/Miniforge3-MacOSX-arm64.sh
sh ~/Downloads/Miniforge3-MacOSX-arm64.sh
source ~/miniforge3/bin/activate
```
이 후 원하는 환경을 만들어 activation 해준후 다음과정을 진행한다.


### install the tensorflow dependencies
```bash
conda install -c apple tensorflow-deps
```

### tensorflow version upgrade
```bash
# uninstall existing tensorflow-macos and tensorflow-metal
python -m pip uninstall tensorflow-macos
python -m pip uninstall tensorflow-metal
# Upgrade tensorflow-deps
conda install -c apple tensorflow-deps --force-reinstall
# or point to specific conda environment
conda install -c apple tensorflow-deps --force-reinstall -n my_env
```

### install base Tensorflow
```bash
python -m pip install tensorflow-macos
```

### install tensorflow-metal plugin
```bash
python -m pip install tensorflow-metal
```

