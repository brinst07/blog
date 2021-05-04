---
title: "SpreadOperator"
date: 2021-05-04T09:06:17+09:00
draft: false
---

## 전개연산자란?

전개 연산자는 나열되는 자료를 추출하거나, 연결할 때 사용한다.

ES6 이전에는 배열의 일부 요소나 객체의 일부분을 연결하려면 각 내장 함수를 이용해야 했다.

ES6에서 전개 연산자의 도입으로 간단하게 자료를 추출하거나 연결할 수 있게 되었다.

사용은 배열이나 객체의 이름 앞에 마침표 3개 ...를 붙여주면 된다.

쉽게 생각하면 그냥 객체나 배열 안의 값을 빼서 새로운 객체나 배열에 넣어준다고 생각하면 된다.

## 전개연산자 없을때 VS 전개연산자 있을때

### 배열에서의 전개연산자

```jsx
전개연산자가 없을때

var arr1 = ['one', 'two'];
var arr2 = ['three', 'four'];
var result = arr1.concat(arr2);
// 또는
var result = [arr1[0], arr1[1], arr2[0], arr2[1]];
// 또는
var result = [].concat(arr1, arr2);
console.log(result);
```

```jsx
전개연산자가 있을 때

var arr1 = ['one', 'two'];
var arr2 = ['three', 'four'];
var result = [...arr1, ...arr2];
console.log(result);
```

위 두개의 코드는 오른쪽의 같은 결과를 출력한다.

기존의 배열을 합치려면 concat이나 slice등 배열 리터럴 함수를 사용해야 했다. 하지만 전개연산자를 사용한다면 위와같이 간단하게 해결할 수 있다.

### 객체에서의 전개연산자

```jsx
var obj1 = { one: 1, two: 2 };
var obj2 = { three: 3, four: 4 };
var result = {
  one: obj1.one,
  two: obj1.two,
  three: obj2.three,
  four: obj2.four,
};
// 또는
var result = Object.assign({}, obj1, obj2);
// 또는
var result = Object.assign(obj1, obj2);
console.log(result);
```

```jsx
var obj1 = { one: 1, two: 2 };
var obj2 = { three: 3, four: 4 };
var result = { ...obj1, ...obj2 };
console.log(result);
```

### 함수에서의 전개연산자

```jsx
function func() {
  var arr = Array.prototype.slice.call(arguments);
  console.log(arr);
  console.log(arr[0]);
}

func(1, 2, 3, 4);
// (4) [1, 2, 3, 4]
// 1
```

```jsx
function func(...args) {
  var arr = [...args];
  var [one, ...others] = args; // 구조 분해 할당
  console.log(arr);
  console.log(one);
}

func(1, 2, 3, 4);
// (4) [1, 2, 3, 4]
// 1
```

함수에서 인자로 넘겨줄때 인자들은 arguments라는 함수 내부의 배열로 저장된다.

넘어온 인자의 수를 제한하지 않거나 얼마나 받을지 예상할 수 없는 상황에는 arguments를 사용하게 되는데 이 또한 전개 연산자로 변환이 가능하다.

## 전개연산자 사용예제
```js
  const fetchMovies = (endpoint) => {
        fetch(endpoint)
            .then(response => response.json())
            .then(response => {
                //받은 movies array를 state안에 넣는 부분이다.
                // setMovies([...response.results])면 기존 영화들이 날아가고 새로운 영화가 그자리를 채우게 된다.
                //이렇게 해주면 배열에안에 기존것과 새로운 것이 같이 존재하게 된다.
                //deepcopy를 하기위해 전개연산자를 사용해서 배열에 넣어줬다.
                setMovies([...Movies, ...response.results])
                console.log(...Movies)
                setMainMovieImage(response.results[0])
                setCurrentPage(response.page)
            })
    }
```

