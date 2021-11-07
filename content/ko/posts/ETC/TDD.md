---
title: "TDD"
date: 2021-11-05T00:08:39+09:00
draft: false
---
## TDD
TDD란 Test Driven Development의 약자로 '테스트 주도 개발'이라고 한다. 반복 테스트를 이용한 소프트웨어 방법론으로, 작은 단위의 테스트 케이스를 작성하고 이를 통과하는 코드를 추가하는 단계를 반복하여 구현한다.

출처: https://wooaoe.tistory.com/33 [개발개발 울었다]

라고 한다.
이전부터 TDD에 대한 관심은 꾸준히 있었지만, 실제로 실무에 어떻게 도입해야하는지 전혀 감이 오질 않아 항상 공부를 미뤘던거 같다.
하지만 이번에 새로 진행하는 프로젝트에 TDD를 도입해보면 어떨까 생각이 들어 공부를 시작했다.
이 포스팅은 JohnAhn 강사님의 따라하면서 배우는 TDD 개발을 듣고 작성한다.

## TDD의 필요성
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

## Jest
현재 NodeJS 환경에서 TDD를 공부하고 있기 때문에, Jest를 활용한다.

## Mock function
Mock function을 사용하는 이유는 다음과 같다.
함수들 사이안에 의존성이 있는 함수가 존재하게 된다면, 해당 함수를 테스트하는데 문제가 발생할 것이다.
이런상황일때는 테스트 어렵게 된다.
그러므로 의존성 있는 함수를 Mock 함수로 선언하고 정상적으로 돌아간다는 가정하게 테스트를 진행하는 것이다.

## 테스트 코드 작성하기
테스트 코드는 다음과 같이 작성한다.
### 단위테스트
```javascript
const productController = require("../../controller/product");
const productModel = require('../../models/Product')
const httpMocks = require('node-mocks-http')
const newProduct = require('../data/new-product.json')
const {Promise} = require("mongoose");

//단위테스트이기 때문에 의존성이 있는 것을 mock function으로 만들어준다.
//이 함수가 뭘 사용하는지 어떤것을 리턴하는지 추적이 가능해진다.
productModel.create = jest.fn();
productModel.find = jest.fn();
productModel.findById = jest.fn();
productModel.findByIdAndUpdate = jest.fn();

//전역으로 쓰일 변수이기 때문에 다음과 같이 선언해준다.
let req, res, next;

const productId = '618761fd1febbe2fe466a211';

//각 test마다 공통적으로 쓰이는 부분이기에 beforeEach를 사용하여 처리한다.
beforeEach(() => {
    req = httpMocks.createRequest();
    res = httpMocks.createResponse();
    next = jest.fn();
})

describe("Product Controller Create", () => {

    beforeEach(() => {
        req.body = newProduct;
    })

    it('should have a createProduct function', () => {
        expect(typeof productController.createProduct).toBe("function");
    });

    it('should call ProductModel.create', async () => {
        req.body = newProduct;
        await productController.createProduct(req, res, next);
        expect(productModel.create).toBeCalledWith(newProduct);
    })

    it('should return 201 response code', async () => {
        await productController.createProduct(req, res, next);
        expect(res.statusCode).toBe(201);
        //값이 무엇인지 신경 쓰지 않고 Boolean 컨텍스트에서 값이 참인지 확인
        expect(res._isEndCalled()).toBeTruthy();
    })

    it('should return json body in response', async () => {
        productModel.create.mockReturnValue(newProduct)
        await productController.createProduct(req, res, next);
        expect(res._getJSONData()).toStrictEqual(newProduct)
    })

    it('should handle errors', async () => {
        //에러메시지 선언
        const errorMessage = {message : 'description property missing'};
        //Promise.reject(reason) 메서드는 주어진 이유(reason)로 거부된 Promise 객체를 반환
        //따라서 아래의 코드는 errorMessage를 이유로 reject된 상황을 만들기위함이다.
        const rejectedPromise = Promise.reject(errorMessage);
        //위 상황으로 returnValue를 만들어준다.
        productModel.create.mockReturnValue(rejectedPromise);
        //아래 메소드를 실행하면 create에서 위의 에러가 발생할 것이다.
        await productController.createProduct(req,res,next);
        //해당 에러메시지가 발생하였는지 확인한다.
        expect(next).toBeCalledWith(errorMessage);
    });
});

describe("Product Controller Get", () => {
    it('should have a getProducts function', function () {
        expect(typeof productController.getProducts).toBe("function");
    });

    it('should call ProductModel.find({})', async () => {
        await productController.getProducts(req,res,next);
        expect(productModel.find).toHaveBeenCalledWith({})
    });

    it('should return 200 response',async function () {
        await productController.getProducts(req,res,next);
        expect(res.statusCode).toBe(200);
        //값이 무엇인지 신경 쓰지 않고 Boolean 컨텍스트에서 값이 참인지 확인
        expect(res._isEndCalled).toBeTruthy()
    });

    // it('should return json boyd in response',async function () {
    //     productModel.find.mockReturnValue()
    // });

    it('should handle errors', async function () {
        const errorMessage = {message : "Error finding product data"};
        const rejectedPromise = Promise.reject(errorMessage);
        productModel.find.mockReturnValue(rejectedPromise);
        await productController.getProducts(req,res,next);
        expect(next).toBeCalledWith(errorMessage);
    });
})

describe('Product Controller GetById', () => {
    it('should have a getProductById', function () {
        expect(typeof productController.getProductById).toBe("function");
    });
    it('should call productModel findById', async function () {
        req.params.productId = productId;
        await productController.getProductById(req,res,next);
        expect(productModel.findById).toBeCalledWith(productId)
    });
    it('should return json body and response code 200', async function () {
        productModel.findById.mockReturnValue(newProduct);
        await productController.getProductById(req,res,next);
        expect(res.statusCode).toBe(200)
        expect(res._getJSONData()).toStrictEqual(newProduct);
        expect(res._isEndCalled()).toBeTruthy();
    });
    it('should return 404 when item doesnt exist', async function () {
        productModel.findById.mockReturnValue(null);
        await productController.getProductById(req,res,next);
        expect(res.statusCode).toBe(404);
        expect(res._isEndCalled()).toBeTruthy();
    });
    it('should handle errors', async function () {
        const errorMessage = {message : 'error'};
        const rejectPromise = Promise.reject(errorMessage);
        productModel.findById.mockReturnValue(rejectPromise);
        await productController.getProductById(req,res,next);
        expect(next).toHaveBeenCalledWith(errorMessage);
    });
})

describe('Product Controller Update',() => {
    it('should have an updateProduct function', function () {
        expect(typeof productController.updateProduct).toBe("function");
    });

    it('should call productModel findByIdAndUpadte', async () => {
        //검색할 아이디를 선언한다.
        req.params.productId = productId;
        //업데이트할 객체와 option을 선언한다.
        //option은 리턴하는 객체가 업데이트 되기전 객체인지 새로운 객체인지를 결정하는 것이다. true를 입력하였으므로 업데이트 된 객체를 리턴한다.
        req.body = {name:"updated name"},{new : true};
        await productController.updateProduct(req,res,next);
        expect(productModel.findByIdAndUpdate).toHaveBeenCalledWith(productId,{name:"updated name"},{new : true})
    })
})
```

### 통합테스트
```javascript
//통합테스트를 위하여 supertest 사용
const request = require('supertest')
//통합테스트를 위한 server.js를 import한다.
const app = require('../../server');
const newProduct = require('../data/new-product.json');
const {response} = require("express");

const productId = '618761fd1febbe2fe466a211';


it('POST /api/products', async () => {
    //superttest에 server.js를 넣어줌
    const response = await request(app)
        .post('/api/products')
        .send(newProduct);
    expect(response.statusCode).toBe(201)
    expect(response.body.name).toBe(newProduct.name)
    expect(response.body.description).toBe(newProduct.description)
});

it('should error', async function () {
    const errorProduct = {
        "name" : "oh",
        "price" : 15
    };

    const response = await request(app)
        .post('/api/products')
        .send(errorProduct);
    expect(response.statusCode).toBe(500)
});

it('should return 500 on POST /api/products', async function () {
    const response = await request(app)
        .post('/api/products')
        .send({name : "name"})
    expect(response.statusCode).toBe(500)
    expect(response.body).toStrictEqual({message : 'Product validation failed: description: Path `description` is required.'})

});

// it('GET /api/products', async () =>{
//     const response = await request(app).get('/api/products');
//     expect(response.statusCode).toBe(200)
//     expect(Array.isArray(response.body)).toBeTruthy()
//     expect(response.body[0].name).toBeDefined();
//     expect(response.body[0].description).toBeDefined();
// });

it('GET /api/product/:productId', async function () {
    const response = await request(app).get(`/api/products/${productId}`)
    expect(response.statusCode).toBe(200)
    console.log(response)
});

it('GET id doesnt exist /api/products/:productId', async function () {
    //id값을 많이 바꾸면 mongodb자체에서 없는 id값이라고 404를 주기때문에 살짝만 바꿔줘야함
    const response = await request(app).get('/api/products/618761fd1f12be2fe466a211')
    expect(response.statusCode).toBe(404)
});
```