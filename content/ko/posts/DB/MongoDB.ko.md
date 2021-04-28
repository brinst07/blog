---
title: "MongoDB"
date: 2021-04-28T10:05:20+09:00 
draft: false
---

인프런에서 Node.js React.js강의를 듣고있는데 그 강의에서 MongoDB를 사용하여 개인적으로 공부할겸 포스팅을 진행하게 되었다. 이전에 내가 사용하던 mysql, oracle과 다른 nosql
시스템이다. 따라서 문법도 다르고 개념도 다르기에 익숙하지 않다는 단점이 있지만, 익숙해지면 쿼리문을 작성하지 않고 훨씬 빠르게 진행할 수 있을 것 같다.

## MongoDB란?

몽고DB는 크로스 플랫폼 도큐먼트 지향 데이터베이스 시스템이다. NoSQL 데이터베이스로 분류되는 몽고DB는 JSON과 같은 동적 스키마형 도큐먼트들을 선호함에 따라 전통적인 테이블 기반 관계형 데이터베이스 구조의
사용을 삼간다.

참조 : https://ko.wikipedia.org/wiki/%EB%AA%BD%EA%B3%A0DB

## MongoDB method~

### save

save는 MongDB에서 데이터를 만드는 메소드이다. save말고도 insert 등이 있다.

사용예제

```js
router.post('/', (req, res) => {
    //가져온 정보들을 db에 넣어준다.
    //가져온 정보를 가지고 새로운 객체를 만든다.
    const product = new Product(req.body)
    //그 정보를 가지고 DB에 저장한다.
    product.save((err) => {
        if (err) return res.status(400).json({success: false, err})
        return res.status(200).json({success: true})
    })

})
```

위 예제처럼 스키마에 body를 담아 save를 하는 것을 볼 수 있다.

### find

find는 말 그대로 조회 메소드이다. find({}) 또는 find()를 하면 컬렉션 내에 모든 다큐먼트들을 선택하여 가져온다.

사용예제

```js
//기본예제
Product.find({})

//모두 일치해야하는 경우는 쉼표로구분하여 작성한다.
Product.find({name: 'test', job: '개발자'})

//한가지만 일치해도되는 경우는 $or를 사용한다.
Product.find({$or: [{name: 'test'}, {job: '개발자'}]})

//실제예제
router.post('/products', (req, res) => {
    //product collection에 들어있는 모든 상품정보를 가져온다.
    //조건은 Product.find({price:..}) 이런식으로 들어간다.
    Product.find()
        .populate('writer')
        .exec((err, productInfo) => {
            if (err) return res.status(400).json({success: false, err})
            return res.status(200).json({success: true, productInfo})
        })

})
```

예제를 보면 대충 파악했듯이 find메소드안에 **json형태**로 데이터를 만들어서 사용하는 것을 알 수 있다.

#### 숫자비교 옵션

큰 것은 $gt, 작은것은 $lt, 크거나 같은것은 $gte, 작거나 같은 것은 $lte이다.

사용예제

```js
router.post('/products', (req, res) => {
    //product collection에 들어있는 모든 상품정보를 가져온다.
    //조건은 Product.find({price:..}) 이런식으로 들어간다.

    let limit = req.body.limit ? parseInt(req.body.limit) : 20
    let skip = req.body.skip ? parseInt(req.body.limit) : 0

    //조건식을 만들어줘야하기에 배열을 만들어준다.
    let findArgs = {}

    //이렇게 되면 배열안의 배열 즉, price나 continents가 key가 된다.
    for (let key in req.body.filters) {
        //배열에 값이 존재한다면 아래를 수행한다.
        if (req.body.filters[key].length > 0) {
            //조건식을 넣어주는 부분이다.
            if (key === 'price') {
                findArgs[key] = {
                    //값은 [0.199] 이런식으로 들어온다.
                    //greater than equal -> 이것보다 크거나 같고
                    $gte: req.body.filters[key][0],
                    //less than equal => 이것보다 작거나 같고
                    $lte: req.body.filters[key][1]
                }
            } else {
                //그게아니라 그냥 값으로 find때려버리기 때문에 값을넣어버림
                findArgs[key] = req.body.filters[key];
            }

        }
    }

    console.log(findArgs)

    Product.find(findArgs)
        .populate('writer')
        .skip(skip)
        .limit(limit)
        .exec((err, productInfo) => {
            if (err) return res.status(400).json({success: false, err})
            return res.status(200).json({success: true, productInfo, postSize: productInfo.length})
        })

})
```

#### 투사(projection)

투사(projection)이란 find와 findOne 메소드의 두 번째 인자로 넣어주는 것을 말한다. select 절에 컬럼을 쓰는 것과 같은 효과이다. projection 기능을 사용하면 용량도 절약할 수 있고,
보안에도 좋다.

```js
Product.find({name: 'Slime'}, {name: true, hp: true, _id: false})
```

### update
업데이트는 보통 **update**나 **FindAndModify**를 사용한다.

####update
```js
Product.update({name : 'test'}, { $set : { hp : 30}})
```
이렇게 사용하면 수행결과를 반환한다.
그리고 반드시 **$set**을 사용해야 해당 필드만 변경된다.
만약 $set을 넣지 않으면, 객체가 {hp : 30}으로 변경된다.

**$inc**를 사용하면 숫자를 올리거나 내릴 수 있다. 음수를 넣으면 내려가고 양수를 넣으면 올라간다.
```js
Product.update({name : 'test'}, { $inc : { hp : -5 }})
```

####FindAndModify
update 메소드와 달리 upsert와 remove까지 수행할 수 있다.
* upsert -> update + insert이다.
  수정할 대상이 없는 경우 insert 동작을 수행할 수 있게한다.
* upsert 기능을 사용하려면 메소드에 옵션으로 {upsert : true}를 넣어주면 된다.
```js
Product.findAndModify({query : {name : 'test'}, update : {$set : {att : 150}, new : true}})
```

### delete
MongoDB는 Rollback을 지원하지 않는다.
```js
//전체가 지워짐
Product.remove({})

//해당하는 이름의 도큐먼트만 지워짐
Product.remove({name : 'zedd'})
```