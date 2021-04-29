---
title: "Redux"
date: 2021-04-29T09:22:01+09:00
draft: false
---

## Redux란?
간단하게 말하자면 Redux는 state를 관리하는 Tool이다.
그렇다면 state는 무엇일까??

### State
- 자신이 들고 있는 값을 말한다.
- 읽기전용인 props와 비교해보자면, 쓰기전용이라고 볼수 있다.
- 부모컴포넌트에서 자식컴포넌트로 data를 보내는 것이 아닌 component안에서 데이터를 전달하려면 state를 사용해야한다.
  
  ex) 검색 창에 글을 입력할 때 글이 변하는 것은 state를 바꿈
- State는 props와 다르게 mutable하다.
- State가 변하면 re-render된다.
```js
state = {
	message : '',
	attachFile : undefined,
	openMenu : false
}
```

### Props
- properties의 줄임말
- 부모 컴포넌트가 자식 컴포넌트한테 전달하는 데이터로, (자식 입장에서) 읽기 전용이다.
- flow가 부모 컴포넌트에서 자식컴포넌트에 전달하는 flow임(반대 불가)
- 부모에서 자식에게 1이라는 값을 던져주면 이 1는 immutable임
```js
//자식 컴포넌트
<ChatMessages
	messages={messages}
	currentMember={member}
/>
```

### Redux를 사용하는 이유...
![image](https://user-images.githubusercontent.com/60083557/116488524-8353c780-a8cd-11eb-85b6-e1f4426e2f80.png)
A component에서 사용하는 comment라는 정보를 B컴포넌트에서 사용하고자 한다면 component를 타고 타고 해서 정보를 받아야한다.
하지만 리덕스를 사용하게 되면 **store**에 저장하고 필요한 곳에서 꺼내 쓰면 되는 것이다.

### Redux Data Flow
![image](https://user-images.githubusercontent.com/60083557/116488735-10971c00-a8ce-11eb-83a1-87f7288a93f8.png)
Redux는 위와 같은 flow로 동작한다.
특징은 **단방향**으로 data flow가 진행된다는 것이다.

### Redux 세팅하기
Client 폴더로 이동하여 다음 dependency를 설치한다.

{{< box >}}
npm install redux react-redux redux-promise redux-thunk --save
{{< /box >}}

#### Redux-promise, Redux-thunk
- Redux를 잘 쓸수 있게 도와주는 middleware이다.
- 기본적인 Redux Store는 반드시 객체 형식으로 된 action만을 받을 수 있다.
- 하지만 항상 plain object 형태로 오는 것이 아니다.(Promise형태로 올수 도 있고, Function 형태로 올 수도 있음)
- 따라서 위 두 dependency는 위 두가지 형태를 받을수 있게 해주는 것
  - Redux-promise ⇒ promise
  - Redux-thunk ⇒ function

#### Redux 연결하기
위에서 Redux를 설치해줬다면, Redux를 연결하는 작업 또한 필요하다.
client의 index.js로 이동하여 다음과 같이 작성한다.
```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';
import 'antd/dist/antd.css'
import {Provider} from "react-redux";
import {applyMiddleware, createStore} from "redux";
import promiseMiddleware from 'redux-promise'
import ReduxThunk from 'redux-thunk'
import Reducer from './_reducers/index'

//middleware가 빠진다면 createStore만 넣으면 되지만, Promise와 Function까지 Store에서 허용해줘야하기때문에 아래와 같이 설정해준다.
const createStoreWithMiddleware = applyMiddleware(promiseMiddleware, ReduxThunk)(createStore)

ReactDOM.render(
//Provider로 App을 감싸면 연결해주는 작업이 끝난다.
//반드시 Store를 설정해줘야하는데 위에서 만든 createStoreWithMiddleware안에 Reducer와 extension을 넣어준다.
    <Provider
        store={createStoreWithMiddleware(Reducer,
            window.__REDUX_DEVTOOLS_EXTENSION__ && window.__REDUX_DEVTOOLS_EXTENSION__()
        )}
    >
        <App/>
    </Provider>,
    document.getElementById('root')
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();
```

#### combineReducer 설정
reducer를 만들게 되면 여러개가 생길 것이고 이 만들어진 reducer를 관리해줘야한다.
이때 combineReducer를 사용하여 관리한다.
reducer 파일을 import해서 넣어주면 된다.
```js
import {combineReducers} from "redux";
import user from './user_reducer'

const rootReducer = combineReducers({
    user
})

export default rootReducer;
```


#### Action
* 무엇이 일어났는지 설명하는 객체
* 상태를 알려준다....
```js
{
  type : '액션의 종류를 한번에 식별할 수 있는 문자열 혹은 심볼',
  payload : '액션의 실행에 필요한 임의의 데이터'
}

//42번 게시물에 좋아요 버튼을 눌렀다..
{type : 'LIKE_ARTICLE', articleId : 42}
//3번 id, 이름이 Mary인 사람이 Fetch_user_success 했다.
{type : 'FETCH_USER_SUCCESS', response : {id : 3, name : 'Mary'}}
```
* Store에 있는 State는 리액트 컴포넌트가 접근할 수 없다. 반드시 **Action**을 통해서 접근해야함
  1. Store에 뭔가 하고 싶은 경우 Action을 발행
  2. Store에 문지기가 Action의 발생을 감지하면, State가 갱신된다.

#### Reducer
* state가 어떻게 변할지 묘사하는 function이다.
* 이전 상태와 action을 합쳐 새로운 state를 만드는 조작이다.
* 이전 state와 action object를 받은 후 next state를 return한다.

{{<box>}}
  (previousState, action) => nextState
{{</box>}}

#### Store
* 여러가지 State를 감싸고 있는 역할을 한다.
* State는 기본적으로 여기서 집중관리 된다. 커다란 JSON의 결정체 정도의 이미지이다.
* 다음은 STORE의 예제이다.
```js
{
    // 세션과 관련된 것
    session: {
        loggedIn: true,
        user: {
            id: "114514",
            screenName: "@mpyw",
        },
    },

    // 표시중인 타임라인에 관련된 것
    timeline: {
        type: "home",
        statuses: [
            {id: 1, screenName: "@mpyw", text: "hello"},
            {id: 2, screenName: "@mpyw", text: "bye"},
        ],
    },

    // 알림과 관련된 것
    notification: [],
}
```

#### Store에 있는 값 활용하기
Store에 저장되어 있는 값을 가져오려면 Hook을 사용해야 한다.
```js
import { useSelector } from 'react-redux';

const { id, text } = useSelector((state: RootState) => state.reducer1);
```
참고자료 : https://react-redux.js.org/api/hooks

### Redux 활용예제

{{< tabs Dispatch Action Node Reducer >}}
{{< tab >}}

#### Dispatch(action)

  ```javascript
import React, {useState} from 'react';
import {Button, Descriptions} from "antd";
import {useDispatch} from "react-redux";
import {addToCart} from "../../../../_actions/user_actions";

function ProductInfo(props) {
  const dispatch = useDispatch();

  const clickHandler = () => {
    //필요한 정보를 Cart 필드에다가 넣어준다.
    dispatch(addToCart(props.detail._id))
  }

  return (
          <div>
            <Descriptions title="Product Info">
              <Descriptions.Item label="Price">{props.detail.price}</Descriptions.Item>
              <Descriptions.Item label="Sold">{props.detail.sold}</Descriptions.Item>
              <Descriptions.Item label="View">{props.detail.views}</Descriptions.Item>
              <Descriptions.Item label="Description">{props.detail.description}</Descriptions.Item>
            </Descriptions>

            <br/>
            <br/>

            <div style={{display: 'flex', justifyContent: "center"}}>
              <Button size="large" shape='round' type='danger' onClick={clickHandler}>
                Add to Cart
              </Button>
            </div>
          </div>
  );
}

export default ProductInfo;
  ```

{{< /tab >}}
{{< tab >}}

#### Action

```js
export function addToCart(id){
    let body = {
        productId : id
    }

    const request = axios.post(`${USER_SERVER}/addToCart`,body)
        .then(response => response.data);

    return {
        type: ADD_TO_CART,
        payload: request
    }
}
```
{{< /tab >}}
{{< tab >}}

#### Node.js(백단)

```js
router.post("/addToCart", auth, (req, res) => {
    //먼저 User Collection에 해당 유저의 정보를 가져오기
    //auth 때문에 req.user._id를 가져올수 있다.
    User.findOne({_id: req.user._id},
        (err, userInfo) => {
            //가져온 정보에서 카트에다 넣으려 하는 상품이 이미 들어 있는지 확인
            let duplicate = false;
            userInfo.cart.forEach((item) => {
                if (item.id === req.body.productId) {
                    duplicate = true
                }
            })

            if (duplicate) {
                //상품이 이미 있을때
                //내부 객체를 사용할때는 다음과 같이 ''로 사용하여 문자로 만들어줘야한다.
                //{new: true} 는 update된 결과값을 받기위해 설정해주는 옵션이다.
                User.findOneAndUpdate({_id: req.user._id, 'cart.id': req.body.productId},
                    {$inc: {'cart.$.quantity': 1}},
                    {new: true},
                    (err, userInfo) => {
                        if (err) return res.status(400).json({success: false, err})
                        return res.status(200).send(userInfo.cart)
                    })
            } else {
                //상품이 이미 있지 않을때때
                User.findOneAndUpdate({_id: req.user._id},
                    {
                        $push: {
                            cart: {
                                id: req.body.productId,
                                quantity: 1,
                                date: Date.now()
                            }
                        }
                    },
                    {new: true},
                    (err, userInfo) => {
                        if (err) return res.status(400).json({success: false, err})
                        return res.status(200).send(userInfo.cart)
                    }
                )
            }
        })


});
```
{{< /tab >}}
{{<tab>}}
#### Reducer
```js
import {
    LOGIN_USER,
    REGISTER_USER,
    AUTH_USER,
    LOGOUT_USER, ADD_TO_CART,
} from '../_actions/types';


export default function (state = {}, action) {
    switch (action.type) {
        case REGISTER_USER:
            return {...state, register: action.payload}
        case LOGIN_USER:
            return {...state, loginSucces: action.payload}
        case AUTH_USER:
            return {...state, userData: action.payload}
        case LOGOUT_USER:
            return {...state}
        case ADD_TO_CART:
						//이렇게 해주는 이유는 redux 기존 정보에 cart 정보를 추가해주기 위해서이다.
            return {
                ...state, userData: {
                    ...state.userData,
										//위쪽 즉 action -> 백단 리턴값이 action.payload이다.
                    cart: action.payload
                }
            }
        default:
            return state;
    }
}
```
{{</tab>}}
{{< /tabs >}}

