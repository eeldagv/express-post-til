# [TIL] Express - Postman을 사용한 POST 메서드 테스트

# ✅ POST 
```
app.post("/test", (req, res) => {
  res.send("Hello POST!");
});
```
GET 메서드 코드에서 get을 post로 수정해서 실행시켜보면, 딱히 에러는 안 나지만 브라우저가 ‘POST’ 메서드로 요청이 왔음을 알지 못한다. 

POST는 ‘등록’을 요청하는 메서드이다. 예를 들어, ‘회원가입’은 그 사이트에 ‘회원’ 을 ‘등록’하는 것이고, 회원의 고유한 아이디, 비밀번호, 이름, 이메일 등의 ‘개인정보’를 가지고 가입을 하겠다는 뜻이다. GET 요청은 URL에 담아 보냈지만, POST는 앞서 말한 바와 같이 ‘개인정보’가 들어있기 때문에 모두가 볼 수 있는 URL 등에 담아 보내면 큰일난다. 때문에 숨겨서 보내야 한다. 어디에다 숨기느냐? 바로 ‘body’이다.

웹 브라우저는 POST 요청이 body에 숨겨져서 들어와야 한다는 걸 알고 있고, 데이터가 body 에 숨겨져서 와야만 POST 인 걸 알 수 있다. 때문에 URL에는 데이터를 숨길 곳이 없으므로 웹 브라우저는 URL로 받을 수 있는 GET 방식만 취급한다.

그러면 POST 방식은 어떻게 테스트 해야 할까? 그 때 사용하는 프로그램이 “포스트맨”이다.

## 💬 Postman
<img width="619" alt="image" src="https://github.com/user-attachments/assets/a8afde60-53a4-4eff-9951-4de7f39be622" />
포스트맨은 위와 같은 화면으로 구성되어있다. GET 방식인지 POST 방식인지 선택할 수 있고, 요청을 주고 받는 URL을 적을 수 있으며 그 밑의 윗부분은 Request body이고 아래는 Response body이다. 아까 브라우저에서 테스트 했을 때 에러가 났던 코드를 포스트맨에서 테스트하면, 의도한 바와 같이 Response body에 정상적으로 문자열이 보여진다.


그렇다면 body에 데이터를 어떻게 전달하고, 어떻게 데이터를 가져올까?

```
app.use(express.json());
```
json 미들웨어를 사용하면, req로 날아오는 body 값을 json 으로 읽을 수 있다. Express가 json을 자동으로 파싱해서 req.body에 담아주기 때문이다. 파서가 없으면 POST 요청을 json으로 보내도 undefined가 뜬다.

포스트맨에서 테스트 할 데이터를 작성해보자.

<img width="620" alt="image" src="https://github.com/user-attachments/assets/bd0e04e2-8470-4be4-a98a-7caca9cd69de" />
직접 작성해야 하기 때문에 raw 옵션을 선택하고, JSON 옵션을 선택한 뒤 보낼 데이터를 json 형식으로 작성한다.

```
app.use(express.json());
app.post("/test", (req, res) => {
  console.log(req.body);
  res.send(req.body);
});
```
그 후 콘솔창과 response body 부분에 보내서 어떻게 뜨는지 확인한다.

<img width="664" alt="image" src="https://github.com/user-attachments/assets/481900af-dc1a-4c15-940e-789b4e3318e9" />
<img width="621" alt="image" src="https://github.com/user-attachments/assets/ef794f7a-b74b-4dd8-adb5-5d89ccaac7d5" />
요청한 대로 아주 잘 뜨는 것을 확인할 수 있다.

```
app.use(express.json());
app.post("/test", (req, res) => {
  console.log(req.body.message);
  res.send(req.body.message);
});
```
<img width="621" alt="image" src="https://github.com/user-attachments/assets/12a4bc5f-b6d0-43a4-b0c6-82c7f913ba76" />


key 값으로 데이터만 보내도 잘 나오는 것을 확인할 수 있다.

하지만 애초에 JSON 형식으로 보내고 있기 때문에, response 할 때 send 말고 json 으로 작성해도 된다.

<img width="622" alt="image" src="https://github.com/user-attachments/assets/923967cf-6e36-4196-a501-7d3f9f478118" />
아주 잘 뜨고 있다.

##  POST 로 객체 등록하기
지난 시간에 유튜브 채널로 Map 을 만들고 API 설계 연습한 코드를 고도화 해보자.

기존에는 채널 3개에 대한 정보만을 반환할 수 있었고, 새로운 채널을 등록할 수는 없었다. 또한 3개 채널 대한 api만 존재하고, 나머지는 사용할 수 없었다.

이제 POST 활용해서 채널을 추가해보자.

API 설계(URL + Method)

- 개별 조회 : GET /channel/:id → 변수 id로 key 값을 받아 map에서 객체를 찾아서, 그 객체의 정보를 뿌려줌
    - req : [params.id](http://params.id) 값 던져줌 → map에 저장된 key 값
    - res : req에서 전달받은 id를 기준으로, map에서 id로 객체를 조회해서 전달
- 채널 등록 : POST /channel
    - req : body ← ‘channelTitle, sub = 0, videoNum = 0’ 이라는 신규 채널 정보 전달, Map 에 저장까지 해줌
    - res : “channelTitle 채널이 개설되었습니다.” 라는 메세지 전달

즉 등록 = “Map에 저장(set)”

```
app.use(express.json());
app.post("/channel", (req, res) => {
  db.set(4, req.body);
  res.json({
    message: `신규 채널 '${db.get(4).channelTitle}' 이(가) 개설되었습니다.`,
  });
});
```

<img width="621" alt="image" src="https://github.com/user-attachments/assets/45ec2d3f-6abe-4191-85d8-ffc32430afec" />


/channel 에서 작성해둔 데이터를 post 하면 db에 key 값이 4인 새로운 채널 정보 객체가 set 되면서, response body 에는 채널이 개설되었다는 메세지가 뜬다.

<img width="621" alt="image" src="https://github.com/user-attachments/assets/a0463e02-6538-4f41-91b4-f83583e9a297" />



key 값을 받아오는 변수 id에 4를 넣어서 get 요청을 하면, 아까 post한 객체 정보를 잘 전달하는 것을 확인할 수 있다.

등록할 때마다 key 값이 4인 value 에 데이터가 덮어씌워지면 큰일날테니, 해당 코드도 고도화를 해보도록 하자.

등록이 될 때마다 id 값이 1이 추가되고, 1이 추가된 id 값에 데이터가 등록되어야 한다. 그렇다면 key 값이 되는 id 를 변수로 저장을 해두고, 등록할 때마다 값이 1씩 올라가게 해 주어야 한다.

```
// 데이터 세팅
let db = new Map();
var id = 1;

db.set(id++, youtuber1);
db.set(id++, youtuber2);
db.set(id++, youtuber3);

// API 설계
app.use(express.json());
app.post("/channel", (req, res) => {
  db.set(id++, req.body);
  res.json({
    message: `신규 채널 '${db.get(id - 1).channelTitle}' 이(가) 개설되었습니다.`,
  });
});
```
우선 먼저 id는 1로 시작한다. 그래야 key 값이 1인 객체에 처음 채널 정보가 저장이 된 후에 id 값이 1이 증가한 상태로 저장이 된다. id++ 은 후위 연산이기 때문이다. 이 때 변수를 var 를 써야 하는 이유는, 만약 let으로 post 함수 바깥에서 변수 선언을 해버리면면 함수에서 해당 변수를 갖다 쓸 수 없기 때문이다.

데이터를 set 할 때 id++ 이기 때문에 연산이 된 뒤에 밑으로 전달되므로, 다시 -1 연산을 해주어야 방금 등록한 맞는 데이터가 res body에 정확히 뿌려지게 된다. 

## 💬 데이터 전체조회
이제 개별조회 말고 전체조회를 해보자.

개별조회는 각 데이터에 대한 id 값을 url에 받았지만 전체조회는 그럴 필요가 없다. 대신 ‘channels’ 라는 복수형의 이름으로 해 주면 직관적이고 좋다.

req를 따로 던질 건 없고, map 자체를 전체 조회해서 전달해주면 된다.

```
app.get("/channels", (req, res) => {
  res.json(db);
});
```
그런데 이렇게 작성하고 포스트맨에서 테스트하면 res body에 빈 {} 만 나오게 된다. 왜일까?

json 함수로 데이터를 보내면 내부적으로 JSON.stringify 함수를 사용해서 json 문자열로 변환을 하는데, 이 JSON.stringify 함수는 오직 ‘객체’ 형태의 데이터만 처리할 수 있다. 하지만 Map 은, 객체 ‘형태’로 데이터를 저장하기는 하지만 엄밀히 말하자면 ‘객체’는 아니다. 때문에 Map 을 json 으로 전달하려면 객체로 변환하는 과정이 필요하다. 따라서,
```
app.get("/channels", (req, res) => {
  res.send(Object.fromEntries(db));
});
```
Object.fromEntries 함수를 사용하여 Map 을 일반 객체로 변환해주는 과정이 필요하다.
<img width="622" alt="image" src="https://github.com/user-attachments/assets/608f167a-0e17-4e77-a2c6-f3210bd366ed" />
다시 포스트맨으로 get 요청을 해보면 정상적으로 전달하는 것을 확인할 수 있다.
