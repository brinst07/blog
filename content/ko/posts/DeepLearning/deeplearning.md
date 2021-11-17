---
author: "brinst"
title: "Deep Learning의 기초"
date: 2021-11-17T22:39:24+09:00
draft: false
---
## Deep Learning

- Limitations of explicit programming

  이런 상황에서는 이렇게 해라 지시하는 프로그램

  하지만 스팸 필터나, 자율 주행 같은 프로그램에서는 많은 상황이 발생할 수 있기 때문에 제약 상황이 발생

  따라서 1959년에 Arthur Samuel이란 사람이 프로그래머가 지시하는게 아닌 학습하는 프로그램을 제안함


## Supervised / Unsupervised learning

- Supervised learning

  레이블이 정해져 있는 예제로 학습

  ![supervised](/images/content/supervised.png)

    - 대부분의 ML
    - Image labeling : learing from tagged images
    - Email spam filter : learning from labeld email
    - Predicting exam score : learning from previous exam score and time spent
- Unsupervised learning(un-labed data)
    - 구글 뉴스를 그루핑
    - 비슷한 단어를 모으기

## Training Data Set

training data set으로 머신러닝을 시키면 모델(weight file)이 생김

이 모델에게 input 값을 주면 그 훈련값을 토대로한 output을 받음

## Types of supervised learing

- Predicting final exam socre based on time spent

  regression(0~100) ⇒ 범위가 매우 넓음

- Pass/non-pass based on time spent

  binary classification ⇒ pass fail 둘중에 하나를 고른다.

- Letter grade (A,B,C,E,F) based on time spent

  multi-label classification ⇒ binary classification 과 다르게 label이 많다.