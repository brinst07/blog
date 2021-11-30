---
author: "brinst"
title: "DeepLearning의 흐름"
date: 2021-11-17T22:39:24+09:00
draft: false
---
## 데이터 전처리

### 1. 모델에 대한 이미지 입력을 단순화하기 위해 이미지 데이터를 평평하게 만든다.
    
    (28x28) ⇒ (784)로 1차원으로 내림
    
### 2. 모델에 대해 이미지 입력 값을 더 쉽게 사용할 수 있도록 이미지 데이터를 정규화한다.
정수 값을 0과 1사이의 부동 소수점  값으로 변환하는 것을 정규화라고 한다.
가장 쉬운 방법은 모든 픽셀 값을 가장 큰 값으로 나누는 것이다. ⇒ x_train = x_train / x_train.max()
### 3. 모델에 대해 레이블 값을 더 쉽게 사용할 수 있도록 레이블 값을 분류한다.
![label](/images/content/label.png)

y 값에 대하여 여러개 값중 답이 존재할 때 이것을 마찬가지로 0과 1로 변경해줘야한다.

```python
num_categories = 3
y_train = keras.utils.to_categorical(y_train, num_categories)

values = [
    [1,0,0],
    [0,0,1],
    [0,1,0],
    [0,0,1]
]
```
    

## 모델 생성 및 학습

[tensorflow_tf.keras.layers.Dense](https://han-py.tistory.com/207)

### 1. 모델 인스턴스화
    
```python
from tensorflow.keras.models import Sequential

model = Sequential()
```

### 2. Input Layer 생성
    
model에 layer를 쌓는 과정이다.

units은 레이어의 뉴런 수를 지정한다.  → 출력 값의 크기

activation : 활성화 함수

relu는 아래 그래프 처럼 0보다 작은 값은 0으로 나타내고 0보다 크게 되면 직선형태의 값을 가지게 된다.

![relu](/images/content/relu.png)
    
input_shape : input shape을 지정한다.

```python
from tensorflow.keras.layers import Dense

model.add(Dense(units=512, activation='relu', input_shape(784,)))
```
    
### 3. Hidden Layer 생성
    
Hidden Layer를 추가하면 더 많은 매개 변수를 제공함으로써 정확한 학습 기대할 수 있다.
    
```python
model.add(Dense(units=512, activation='relu'))
```
    
### 4. Output Layer 생성
    
y 값을 살펴보면 총 10개의 값이 출력된다. 따라서 출력 layer에서는 units 값을 10개로 지정해준다.

또한 기존과 다르게 활성화 함수를 relu가 아닌 softmax를 사용하였다.

softmax는 입력받은 값을 출력으로 0~1사이의 값으로 모두 정규화하며 출력 값들의 총합은 항상 1이 되는 특성을 가진 함수이다.

relu와 차이점이 있다면, 0보다 작은 값은 0인 대신 그 이상의 값을 직선형태로 증가하는 형태를 보이는 반면, softmax는 0.1,0.5,0.4 이런 형태로 값이 존재하게 된다.

즉 값 중에서 제일 큰 값을 결과로 특정할 수 있게 되는 것이다.
    
```python
model.add(Dense(units = 10, activation = 'softmax'))
```

### 5. model 요약
    
위에서 모델에 layer를 쌓았는데, 그것을 확인할 수 있는 방법이다.

```python
model.summary()
```
    
### 6. 모델 컴파일하기
    
데이터로 모델을 training하기 전 마지막 단계로, 모델을 컴파일 하는 단계이다.

여기서는 모델이 훈련 중 얼마나 잘 수행되는지 확인하기 위해서 모델에 사용할 loss function을 지정한다.

[[ML101] #3. Loss Function](https://brunch.co.kr/@mnc/9)

loss function이란 간단하게 얘기하면, 데이터를 토대로 산출한 모델의 예측 값과 실제 값과의 차이를 표현하는 지표이다.

loss function의 함수값이 최소화 되도록 하는 weight와 bias를 찾는 것이 목표이다. ⇒ 학습

또한 모델이 훈련하는 동안 정확성을 추적하고자 한다고 명시한다.

    ```python
    model.compile(loss='categorical_crossentrophy', metrics=['accuracy'])
    ```
    
### 7. 모델 훈련하기
    
이제 훈련 및 검증 데이터와 모델을 준비했으니, 훈련 데이터로 모델을 훈려하고 검증데이터로 데이터를 검증하면 된다.
    
```python
history = model.fit(
    x_train, y_train, epochs=5, verbose=1, validation_data=(x_valid, y_valid)
)

tf.model.get_weights()
```

- epochs ⇒ 전체 데이터를 사용하여 학습을 거치는 횟수를 의미한다. epochs 값이 너무 작다면 underfitting, 너무 크다면 overfitting이 발생할 확률이 높다.
- verbose ⇒ 학습시에 진행상황을 알려주는 argument이다.
    
    ( 0 = slient, 1 = progress bar, 2 = on line per epoch) 
    
- validation_data ⇒ 검증 데이터
- tf.model.get_weights()로 해당 모델의 W(기울기) 와 bias(절편) 값을 구할 수 있다.

verbose를 사용했기에 다음과 같은 로그가 출력되는 것을 확인 할 수 있다.

```
Epoch 1/5
1875/1875 [==============================] - 4s 2ms/step - loss: 0.1933 - accuracy: 0.9426 - val_loss: 0.1238 - val_accuracy: 0.9661
Epoch 2/5
1875/1875 [==============================] - 4s 2ms/step - loss: 0.1017 - accuracy: 0.9736 - val_loss: 0.1087 - val_accuracy: 0.9745
Epoch 3/5
1875/1875 [==============================] - 4s 2ms/step - loss: 0.0803 - accuracy: 0.9801 - val_loss: 0.1226 - val_accuracy: 0.9764
Epoch 4/5
1875/1875 [==============================] - 4s 2ms/step - loss: 0.0692 - accuracy: 0.9830 - val_loss: 0.1370 - val_accuracy: 0.9776
Epoch 5/5
1875/1875 [==============================] - 4s 2ms/step - loss: 0.0640 - accuracy: 0.9866 - val_loss: 0.1534 - val_accuracy: 0.9753
```

compile 시에 merics=['accuracy'] 추적 옵션을 넣었기에 다음과 같이 accuracy가 출력되는 것을 확인 할 수 있다.

val_는 검증데이터에 대한 출력 값이다.