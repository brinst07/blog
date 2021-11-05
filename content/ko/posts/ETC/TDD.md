---
title: "TDD"
date: 2021-11-05T00:08:39+09:00
draft: false
---
# TDD
TDD란 Test Driven Development의 약자로 '테스트 주도 개발'이라고 한다. 반복 테스트를 이용한 소프트웨어 방법론으로, 작은 단위의 테스트 케이스를 작성하고 이를 통과하는 코드를 추가하는 단계를 반복하여 구현한다.

출처: https://wooaoe.tistory.com/33 [개발개발 울었다]

라고 한다.
이전부터 TDD에 대한 관심은 꾸준히 있었지만, 실제로 실무에 어떻게 도입해야하는지 전혀 감이 오질 않아 항상 공부를 미뤘던거 같다.
하지만 이번에 새로 진행하는 프로젝트에 TDD를 도입해보면 어떨까 생각이 들어 공부를 시작했다.

# TDD의 필요성
기존에 개발 흐름을 생각해보면 다음처럼 진행했던거 같다.
개발 -> 에러 -> 디버깅 -> 에러 해결 -> 개발 ... 무한반복
순수 코드를 개발하는 데는 시간이 적게 들어갈 수도 있지만, 이 에러를 잡기 위해 디버깅하는 시간 
그리고 에러를 해결하기 위해 다시 코딩하는 시간 또한 많이 소요되고 있다.
반면 TDD는 다음과 같이 진행된다.
단위테스트 -> 개발 -> 통합테스트
기존에 방식대로 개발했던 나를 포함한 개발자들은 상당히 이질감이 들 것이다.
테스트 짜는 시간에 코드를 잘 설계해서 짜면 끝날 일을 왜 두세번 하지?? 이렇게 생각할 수도 있다.
하지만 조금 깊게 생각해보면, 테스트 코드를 미리 짜두게 되면 발생하는 에러에 대해서 조금더 유연하게 대처할수 있고,
리팩토링시에도 훨씬 빠르게 접근할수 있다.

요약하자면 다음과 같다.
* 효율적인 코드작성 가능
* 에러에 유연하게 대처가능
* 리팩토링시 시간 단축

# Jest
현재 NodeJS 환경에서 TDD를 공부하고 있기 때문에, Jest를 활용한다.

# Mock function
Mock function을 사용하는 이유는 다음과 같다.
함수들 사이안에 의존성이 있는 함수가 존재하게 된다면, 해당 함수를 테스트하는데 문제가 발생할 것이다.
이런상황일때는 테스트 어렵게 된다.
그러므로 의존성 있는 함수를 Mock 함수로 선언하고 정상적으로 돌아간다는 가정하게 테스트를 진행하는 것이다.

# 테스트 코드 작성하기
테스트 코드는 다음과 같이 작성한다.
```javascript
const productController = require("../../controller/product");
const productModel = require('../../models/Product')
const httpMocks = require('node-mocks-http')
const newProduct = require('../data/new-product.json')

//단위테스트이기 때문에 의존성이 있는 것을 mock function으로 만들어준다.
productModel.create = jest.fn();

//전역으로 쓰일 변수이기 때문에 다음과 같이 선언해준다.
let req, res, next;
//각 test마다 공통적으로 쓰이는 부분이기에 beforeEach를 사용하여 처리한다.
beforeEach(() => {
    req = httpMocks.createRequest();
    res = httpMocks.createResponse();
    next = null;
})
//describe으로 테스트코드 그룹을 만들어준다.
describe("Product Controller Create", () => {

    beforeEach(() => {
        req.body = newProduct;
    })
    //테스트 코드를 생성한다.
    //it이나 test로 생성이 가능하다.
    it('should have a createProduct function', () => {
        expect(typeof productController.createProduct).toBe("function");
    });

    it('should call ProductModel.create', () => {
        req.body = newProduct;
        productController.createProduct(req, res, next);
        expect(productModel.create).toBeCalledWith(newProduct);
    })

    it('should return 201 response code', () => {
        productController.createProduct(req,res,next);
        expect(res.statusCode).toBe(201);
        expect(res._isEndCalled()).toBeTruthy();
    })

    it('should return json body in response', () => {
        //Mock function의 returnValue를 선언해준다.
        //이렇게하면 함수가 정상적으로 실행된다고 가정하에 정상적으로 리턴값을 받은 것처럼 테스트를 진행할 수 있다.
        productModel.create.mockReturnValue(newProduct)
        productController.createProduct(req,res,next);
        expect(res._getJSONData()).toStrictEqual(newProduct)
    })
});
```