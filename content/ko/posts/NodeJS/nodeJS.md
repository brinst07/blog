---
author: "brinst"
title: "Node JS 와 Express JS"
date: 2021-03-21T09:22:01+09:00
draft: false
---

## Node Js란?

Node Js가 나오기 전에는 우리는 항상 앞단에서만 JS를 가지고 작업을 하였다.
하지만 Node JS가 나오고 나서부터는 서버사이드에도 JS를 사용할수 있게 되었다.

## Express JS

Express JS는 Node JS를 더 잘 활용할 수 있게 도와주는 Framework라고 할 수 있다.

### NodeJS 설치하기

Node.JS를 설치하는 방법은 여러가지가 있다.
아래의 링크를 방문하여 LTS 버전을 사용하면된다.
https://nodejs.org/ko/

### npm package 만들기

폴더를 만들고 terminal을 켜 npm init 명령어를 입력한다.
그러면 콘솔창에 정보들을 입력하는 칸이 나올텐데 본인이 원하는 정보를 입력하면 package가 구성되게 된다.
해당과정을 완료하면 package.json이라는 json파일이 만들어진다.

### express.js 설치하기

본인이 사용하고 싶은 IDE를 켜고(나는 WebStorm을 사용한다.) terminal을 켜서
npm install express --save
위 명령어를 입력한다.
정상적으로 완료되면 package.json의 dependencies에 express가 추가된것을 확인할수 있을것이다.
또한 node_modules폴더도 생겼을 것이다.

### nestjs 설치하기


### 서버켜보기

프로젝트 폴더에 index.js를 만들어준다.

```JS
const express = require('express')
const app = express()
const port = 5000

app.get('/',(req,res) => res.send('Hello World!'))

app.listen(port, () => console.log(`Example app Listeming on port ${port}!`))
```

package.json으로 가서 scripts안에
"start" : "node index.js"를 입력해준다.
이렇게 입력하고 terminal에서 npm run start를 입력해주면 끝!!

### MongoDB 연결

#### 사전작업

1. MongoDB 계정 생성
2. organization 생성
3. Cluster 생성(무조건 무료로 선택하기!!)
4. connection method 탭에서 주소 긁어오기

난 과거에 Azure api를 쓸때 40만원 정도의 cost가 발생한 적이 있었다....
처음이라 뭔지도 몰랐었고, 내가 git에 올린 api key를 다른 사람이 사용할 수도 있다는 생각 조차 하지 못했었다.
이러한 불상사를 발생하기 위해서 안전장치를 만들어보자

#### URI키 감추기

1. config 폴더를 만든다
2. config 폴더안에 dev.js, key.js, prod.js 세가지 파일을 만든다.
   - dev.js -> 내가 개발용으로 사용할 uri를 적는다.
   - key.js -> 분기처리를 통해 dev.js를 사용할지 prod.js를 사용할지 처리한다.
   - prod.js -> heroku나 배포 서비스를 사용할때 처리하는 부분이다.
3. key.js
   ```JS
      //이 부분에서 개발용인지 배포용인지 분기처리가 된다.
      if(process.env.NODE_ENV === 'production'){
         module.exports = require('./prod')
      }else {
         module.exports = require('./dev')
      }
   ```
4. dev.js
   ```JS
      module.exports ={
         mongoURI : 'uri~'
      }
   ```
5. prod.js
   ```JS
      module.exports = {
         mongoURI : process.env.MONGO_URI
      }
   ```
6. URI 적용하기
   위처럼 파일을 만들어서 적용하고 index.js에서 mongoDB와 연결시킨다.
   ```JS
      const mongoose = require('mongoose')
      const config = require('./config/key')
      mongoose.connect(config.mongoURI, {
         useNewUrlParser: true, useUnifiedTopology: true, useCreateIndex: true, useFindAndModify: false
      }).then(() => console.log('Mongo DB Connect....'))
      .catch(err => console.log(err))
   ```

### Model & Schema 만들기

- Model이란?
  Model이란 Schema를 감싸주는 역할을 말한다.
- Schema란?
  document의 구조 등을 의미하며, 또한 데이터베이스를 creating하고 querying하고, updating, deleting 하는 interface를 제공한다.
- Model & Schema 만들기

  ```JS
     const mongoose = require('mongoose')
     const Schema = mongoose.Schema

     const userSchema = mongoose.Schema({
        name: {
           type: String,
           maxlength: 50
        },
        email: {
           type: String,
           trim: true,
           unique: 1
        }......
     })
  ```

  다음과 같이 제작하면 된다.

### client와 server 통신시키기

만약 앞단이 존재한다면 그걸 눌러보면서 테스트 할 수 있지만, rest api 방식의 로직을 가지고는 테스트 할 수 있는 방법이 존재하지 않는다.
그래서 POSTMAN이라는 프로그램을 사용해서 테스트를 진행할 것이다.

먼저 npm install body-parser --save 명령어로 설치해준다.

백단의 로직을 짠후 , POSTMAN으로 테스트해본다.

```JS
app.post('/api/users/register', (req, res) => {
    //회원가입할때 필요한 정보들을 client에서 가져오면
    //그것들을 데이터 베이스에 넣어준다.
    const user = new User(req.body)
    console.log(req.body)
    user.save((err, userInfo) => {
        if (err) return res.json({success: false, err})
        return res.status(200).json({
            success: true
        })
    })
})
```
