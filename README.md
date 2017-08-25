# **Chat Bot API** V1

## 차례
* [챗봇용 파트너 계정 생성](#챗봇용-파트너-계정-생성)
* [`Webhook` 작성 방법](#webhook-작성-방법)
  * [`Webhook` 요구사항](#webhook-요구사항)
  * [**[STEP 1]** 네이버톡톡으로부터 이벤트 받기](#step-1-네이버톡톡으로부터-이벤트-받기)
  * [**[STEP 2]** echo서버(챗봇예제) 만들기](#step-2-echo서버챗봇예제-만들기)
* [`보내기 API` 작성 방법](#보내기-api-작성-방법)
  * [API 호출 테스트](#api-호출-테스트)
* [이벤트 명세서](#이벤트-명세서)
  * [이벤트 기본 구조](#이벤트-기본-구조)
  * [`open` 이벤트](#open-이벤트)
  * [`leave` 이벤트](#leave-이벤트)
  * [`friend` 이벤트](#friend-이벤트)
  * [`send` 이벤트](#send-이벤트)
* [메시지 타입 명세서](#메시지-타입-명세서)
  * [textContent](#textcontent)
  * [imageContent](#imagecontent)
  * [compositeContent](#compositecontent)
    * [`CompositeContent` Object](#compositecontent-object)
    * [`Composite` Object](#composite-object)
    * [`Image` Object](#image-object)
    * [`ElementList` Object](#elementlist-object)
    * [`ElementData` Object (`LIST` 타입)](#elementdata-object-list-타입)
    * [`Button` Object](#button-object)
    * [컴포지트 메시지 사용 가이드](#컴포지트-메시지-사용-가이드)
  * [퀵버튼](#퀵버튼)
* [ERROR 명세서](#error-명세서)
* 확장 API
  * [Profile API V1](/profile_api_v1.md)
  * [Pay API V1](/pay_api_v1.md)
  * [Image Upload API V1](/imageupload_api_v1.md)
<br>
<br>


## 챗봇용 파트너 계정 생성
1. [네이버톡톡 파트너센터](https://partner.talk.naver.com/) 접근
2. 회사 `단체아이디` 또는 대표성있는 `개인아이디`로 로그인
3. `내 계정관리` > `새로운 톡톡 계정 만들기` 클릭
4. `서비스 선택하기`에서 서비스 선택없이 `서비스 연결 나중에 하기` 클릭
5. 테스트 계정생성시 `개인`을 선택, 서비스 계정생성시 `사업자` 또는 `기관/단쳬`을 선택 후 `다음`
6. `대표이미지`, `프로필명`, `휴대폰 번호` 등 정보를 입력 후 `사용신청`
7. 생성된 계정은 `검수중` 상태이며, 검수가 완료되면 `사용중`상태로 변경되고 SMS로 알림 전송됨
8. `내 계정관리`의 등록된 계정 리스트에서 `계정관리`로 계정 홈 진입
9. 좌측 메뉴 `챗봇 API` 하위 `API 설정` 메뉴를 통해 이용약관동의 및 챗봇설정 (beta기간에는 등록신청 필요)
<br>

### 참고

챗봇 테스트 편의를 위해서 챗봇 이름에 대한 규칙을 안내드립니다.

실제 서비스를 제공할 때 챗봇 이름에 `테스트`라는 키워드를 붙이지 않는다고 가정하였습니다.
챗봇 이름으로 테스트 챗봇인지 실제 서비스를 제공하는 챗봇인지 구분이 힘들기 때문에 편의성을 위한 네이밍 규칙입니다.

연동하고 있는 챗봇 이름의 제일 앞에 `[테스트]`를 붙여서 테스트하는 것을 권장하며,
테스트가 끝난 뒤에는 `[테스트]`키워드를 제거하시면 됩니다.

* 테스트 진행시,
  * "[테스트] 네이버톡톡" 검색 -> 가맹점 검색에 노출되지 않음.
  * "네이버톡톡" 검색 -> 가맹점 검색에 노출되지 않음.

* 실제 서비스시,
  * 가맹점 이름 : "네이버톡톡" -> 가맹점 검색에 노출됨.

따라서 테스트 진행시에는 개발하는 계정을 미리 `친구 추가` 해두시거나 `즐겨찾기` 혹은 채팅 URL을 저장해두시고,
개발 및 테스트 진행하는 것을 권장합니다.

<br>
<br>

## `Webhook` 작성 방법
<br>

### `Webhook` 요구사항
* `Webhook`은 `TLS`기반의 통신을 지원해야 합니다.
  * 네이버톡톡과 연동하는 외부 서비스간의 메시지는 암호화없이 평문으로 전달되기 때문에 반드시 보호되어야 합니다.
  * 지원하는 TLS는 `TLSv1`, `TLSv1.1`, `TLSv1.2`입니다.
  * 정식 인증기관으로 부터 발급받은 유효한 인증서만 사용이 가능합니다.
* `Webhook` 사용을 위해 ACL등록이 필요한 경우 아래 아이피 리스트를 추가합니다.
  * 117.52.141.192/27 (117.52.141.193 ~ 117.52.141.222)
* 네이버톡톡에서 `Webhook` 호출시 설정값
  * Connection timed out : `3초`
  * Read timed out : `5초`
<br>

### **[STEP 1]** 네이버톡톡으로부터 이벤트 받기

#### 1. 네이버톡톡으로부터 이벤트를 받기위해 간단한 코드를 작성합니다.
[`node.js` 샘플]
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());
app.post('/', (req, res) => {
  console.log(req.body);
  res.sendStatus(200);
});

app.listen(8080);
```
<br>

[`Spring Boot` 샘플]
```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
@EnableAutoConfiguration
public class BotApplication {

  @PostMapping
  public void event(@RequestBody String body) {
    System.out.println(body);
  }

  public static void main(String[] args) throws Exception {
    SpringApplication.run(BotApplication.class, args);
  }
}
```
<br>

#### 2. localhost에서 접근 테스트를 합니다.
```bash
curl -X POST -H "Content-Type: application/json;charset=UTF-8" -d '{ "event": "test" }' "http://localhost:8080/"
```
* localhost에서 테스트 후 정식 인증서를 사용하여 https://your.domain/ 으로 접근이 가능 해야합니다.
<br>

#### 3. URL를 등록합니다.
* `파트너센터` > `챗봇 API` > `API 설정` 메뉴에서 `Webhook` 영역 `이벤트 받을 URL`에 위에서 만든 URL를 등록합니다.
<br>

#### 4. 연동이 완료되어 톡톡으로부터 이벤트를 수신합니다.
* https://talk.naver.com/{파트너아이디 (예)wc1234} 에 접근하여, 유저로서 이벤트를 발생시켜봅니다.
<br>

* [채팅창에 진입했을때 이벤트 로그]

    ```javascript
    { 
      "event": "open",
      "user": "al-2eGuGr5WQOnco1_V-FQ",
      "options": {
          "inflow": "list",
          "referer": "https://talk.naver.com/",
          "friend": false,
          "under14": false,
          "under19": false
      }
    }
    ```

* [채팅창에 `hello world`메시지를 보냈을때 이벤트 로그]

    ```javascript
    {
      "event": "send",
      "user": "al-2eGuGr5WQOnco1_V-FQ",
      "textContent": {
          "text": "hello world",
          "inputType": "typing"
      }
    }
    ```
<br>

> **[사용팁]**<br>
> 만약 `Webhook`를 통해 이벤트 수신이 안되면, `API 설정` > `Webhook` > `Event 선택`에서 수신받고자 하는 이벤트를 선택해 줍니다.<br>
<br>
<br>

### **[STEP 2]** echo서버(챗봇예제) 만들기
* 네이버톡톡에서 보내는 각종 이벤트를 판별하고 적절한 메시지로 회신합니다.
* 유저 메시지에 echo메시지로 회신합니다.

#### 1. echo서버 코드 작성
[`node.js` 샘플]
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const app = express();

app.use(bodyParser.json());
app.post('/', (req, res) => {

  // request logging
  console.log(req.body);

  // default response
  let response = {
    event: 'send', /* send message */
    textContent: {
        text: ''
    }
  };

  switch(req.body.event) {

    // 메시지 전송 이벤트 처리
    case 'send' :
      if(req.body.textContent) {
        // 유저가 보내는 메시지에 대해 echo로 전송
        response.textContent.text = 'echo: ' + req.body.textContent.text;
        res.json(response);

      } else {
        // 그외의 경우는 무반응
        res.sendStatus(200);
      }
      break;

    // 채팅창 오픈 이벤트 처리
    case 'open' :
      switch(req.body.options.inflow) {
        // 채팅리스트로부터 인입되었을때
        case 'list' :
          response.textContent.text = '리스트에서 눌러서 방문하셨네요.';
          res.json(response);
          break;

        // 링크로부터 인입되었을때
        case 'button' :
          response.textContent.text = '버튼을 눌러서 방문하셨네요.';
          res.json(response);
          break;

        // 유입경로가 없을때
        case 'none' :
          response.textContent.text = '방문을 환영합니다.';
          res.json(response);
          break;

        default:
          res.sendStatus(200);
      }
      break;

    // 친구 이벤트 처리
    case 'friend' :
      if(req.body.options.set == 'on') {
        // 친구 추가시
        response.textContent.text = '친구가되어주셔서 감사합니다.';
        res.json(response);

      } else if(req.body.options.set == 'off') {
        // 친구 철회시
        response.textContent.text = '다음번에 꼭 친구추가 부탁드려요.';
        res.json(response);
      }
      break;

    // 그외의 이벤트에 대해 무반응
    default:
      res.sendStatus(200);
  }

});

app.listen(8080);
```
<br>

#### 2. https://talk.naver.com/{파트너아이디} 에 접근하여, 각종 이벤트를 확인하고 학습합니다.
* request log를 보면서 어떤 이벤트가 발생되고, 어떻게 대응할지 고민해봅니다.
<br>
<br>

## `보내기 API` 작성 방법
* 보통 단순한 챗봇들은 `Webhook`만으로도 구현이 가능합니다.
* 그러나 유저의 질문에 단순 대답하는 형태가 아니고 상태변화나 사건발생에 따라 유저에게 메시지를 푸시해야한다면 `보내기 API`를 사용하여 챗봇에서 톡톡으로 이벤트를 전송해야합니다.
<br>

### API 호출 테스트
[`request`]
```bash
curl -X POST \
    -H "Content-Type: application/json;charset=UTF-8" \
    -H "Authorization: ct_wc8b1i_Pb1AXDQ0RZWuCccpzdNL" \
    -d '{ "event": "send", "user": "al-2eGuGr5WQOnco1_V-FQ", "textContent": { "text": "hello world" } }' \
    "https://gw.talk.naver.com/chatbot/v1/event"
```
* `Authorization`은 `API 설정` > `보내기 API`에서 `생성`버튼을 눌러 인증키를 받으실 수 있습니다.
* 만약 인증키 `Authorization`가 공개될 경우 `재설정`버튼을 눌러 인증키를 변경하실 수 있습니다.
<br>

[`response`]
```javascript
HTTP/1.1 200 OK

{
    "success": true,
    "resultCode": "00"
}
```
* 응답결과는 [`ERROR 명세서`](#error-명세서)를 참고해 주세요.
<br>
<br>

## 이벤트 명세서
<br>

### 이벤트 기본 구조
<br>

[`Request` 메시지 구조]
```javascript
POST / HTTP/1.1
Host: your.bot.co.kr
Accept: application/json
Content-Type: application/json;charset=UTF-8

{
    "event": "", /* 이벤트명 */
    "options": {
        /* 추가 속성 */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* 유저 식별값 */
}
```

* `event`에는 다양한 이벤트들을 식별할 수 있는 `이벤트명`이 들어옵니다.
* 특정 이벤트만으로 설명될 수 없는 추가적인 속성은 `options`를 통해 정보를 얻을 수 있습니다.
* 톡톡에서 1:1대화란 `파트너(partner)`와 `유저(user)`와의 대화입니다.
  * 챗봇은 `partner`에 해당되며, 챗봇을 사용하는 네이버유저는 `user`에 해당합니다.
  * `"user": "al-2eGuGr5WQOnco1_V-FQ"`는 유저식별값이며 특정 `네이버아이디`에 해당하는 변하지않는 절대값입니다.
<br>
<br>

[`Response` 메시지 구조]
```javascript
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

/* 응답과 동시에 이벤트를 전송할때 사용 */
{
  "event": "send",
  "textContent": {
      "text": "hello world"
  }
}
```

* 챗봇에서 이벤트전달에 대한 응답으로 `HTTP Status`를 `200`으로 보내야 성공으로 간주합니다.
  * `200`외 `HTTP Status`를 전달시 `톡톡`에서 실패로 간주하고 에러를 로깅합니다.
* 만약 이벤트에 대해 자동반응 메시지를 보내고 싶다면 `Header`에 `Content-Type: application/json;charset=UTF-8`를 포함하고 `body`에 전달하고자하는 이벤트를 추가합니다.
  * 이때 `user`속성은 무시되고 이벤트를 보냈던 유저에게 이벤트가 전달됩니다.
<br>

> **[사용팁]**<br>
> 유저의 메시지에 챗봇이 회신하는 형태는 `동기식`/`비동기식` 메시지 전송이 있습니다.
> * `동기식`은 유저의 메시지 이벤트에 응답메시지를 함께 주는 방법입니다.
> * `비동기식`은 유저의 메시지 이벤트에 바로 응답하고(`200 OK`), 좀 이따 `보내기 API`를 이용하여 톡톡에 직접 이벤트를 전송하는 방법입니다.
> * `동기식`은 단건 응답이거나 5초이내에 결과 메시지를 전송할 수 있는경우 사용할 수 있습니다.
> * `비동기식`은 여러건의 메시지로 응답을 줘야하거나 일정시간 경과 후 메시지를 전송할때 사용할 수 있습니다.
<br>
<br>


### `open` 이벤트
* 유저가 채팅창 진입시 유입정보와 함께 이벤트 `open`을 전송합니다.
* 유입정보는 `inflow`를 통해 어떤방식으로 유입되었는지를 구분해주고, `referer`를 통해 유입페이지 URL를 알 수 있습니다.
* 별도로 `from`파라미터를 사용하였다면 `from`를 통해 값을 전달 받을 수 있습니다.

[Request 전체]
```javascript
{
    "event": "open",
    "options": {
        "inflow": "button",
        "referer": "http://storefarm.naver.com/pqbdo/products/309672359",
        "from": "309672359",
        "friend": false, /* false: 친구아님,  true: 친구 */
        "under14": false, /* false: 만14세이상,  true: 만14세미만 */
        "under19": false /* false: 만19세이상,  true: 만19세미만 */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* 유저 식별값 */
}
```

[버튼이나 링크를 통해 유입된 경우 `options`]
```javascript
    "options": {
        "inflow": "button",
        "referer": "http://storefarm.naver.com/pqbdo/products/309672359",
        "from": "309672359"
    }
```

[톡톡 채팅리스트를 통해 유입된 경우 `options`]
```javascript
    "options": {
        "inflow": "list",
        "referer": "https://talk.naver.com/"
    }
```

[브라우저 주소줄에 URL를 직접입력하여 유입된 경우 `options`]
```javascript
    "options": {
        "inflow": "none"
    }
```

> **[사용팁]**<br>
> 유저와 대화를 나눌때 유입경로는 많은 정보를 제공해 줄 수 있습니다. 쇼핑몰이라면 `http://shopping.com/product/1234`로부터 유입된경우 `1234 상품구매를 원하시나요?`라고 유저의 의도를 파악해 대응이 가능합니다. 경우에따라 하나의 웹페이지에 여러개의 상품과 함께 버튼들을 노출하였다면 `referer`만으로는 상품을 구분할 수 없습니다. 그럴땐 `https://talk.naver.com/wc1234?from=1234`와 `https://talk.naver.com/wc1234?from=5678`처럼 `from` 파라미터를 이용하여 상품번호를 받아낼 수 있습니다. 톡톡은 웹 서비스이기때문에 어떤 웹 페이지든 버튼 또는 링크를 통해 챗봇과 대화를 시작할 수 있습니다. 여러 서비스를 통해 배너광고를 하는경우 어떤 배너로 유입이 많은지 그래서 어떤 배너에 집중할지 통계 데이터로도 활용할 수 있습니다. `"inflow": "list"`의 경우 톡톡 채팅리스트를 통한 유입으로 단지 과거 대화이력을 보기위해 `open`했는지 알수없기때문에 메시지 마지막 발생시간 또는 문맥에 따라 적절히 대응해야 합니다. `under14`는 `open`이벤트에 반드시 포함되는 `options`의 하위 속성입니다. 개인정보수집을 위한 약관동의시 만14세미만의 경우 `법정대리인`의 동의를 받아야합니다.(`개인정보보호법 제22조`) 챗봇이 개인정보를 수집한다면 `법정대리인` 동의 프로세스를 구현하거나 만14세미만 사용을 못하도록 안내하여야 합니다.
<br>
<br>

### `leave` 이벤트
* 유저가 채팅창 또는 채팅리스트에서 `나가기`버튼을 누를때 발생되는 이벤트입니다.

```javascript
{
    "event": "leave",
    "user": "al-2eGuGr5WQOnco1_V-FQ"
}
```
> **[사용팁]**
> * `leave` 이벤트의 `Response`에 `request`를 넣어도 메시지가 전송되지않고 항상 무시됩니다.
> * `leave` 이벤트는 일반 메신저 경험처럼 `채팅창`을 나간것으로 `채팅리스트`에는 존재하지 않지만, 챗봇이 유저에게 다시 메시지를 보낸다면 소환됩니다.
> * `leave` 이벤트를 통해 챗봇이 유저에게 적극적으로 이후 메시지를 보낼것인지 아니면 이쯤에서 더 이상 대화를 하지 않을것인지 선택할 수 있습니다.
> * 유저가 `채팅창`을 나갔다고해서 반드시 챗봇과 대화를 원하지않는다고 볼 수 없습니다.
<br>
<br>

### `friend` 이벤트
* 유저가 `친구추가` 또는 `친구철회`버튼을 누를때 발생되는 이벤트입니다.
* `options`하위 `set`의 `on`/`off`값으로 `친구추가`와 `친구철회`를 구분할 수 있습니다.

```javascript
{
    "event": "friend",
    "options": {
        "set": "on" /* on: 친구추가,  off: 친구철회 */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ"
}
```
> **[사용팁]**<br>
> `friend` 이벤트는 가급적 이벤트 수신을 끄거나 수신 받더라도 무응답하는것이 좋습니다. 톡톡에서 친구란 [파트너센터](https://partner.talk.naver.com/)의 `마케팅관리` > `단체메시지`를 이용해 메시지를 보낼 수 있는 대상자를 의미합니다. 챗봇이 `친구추가`에 대해 감사 메시지를 직접 보내는것보다 `파트너센터`의 `친추감사메시지`를 설정하여 다양한 메시지를 보내는것을 추천합니다.
<br>
<br>

### `send` 이벤트
* `send` 이벤트는 메시지를 보낼때 발생되는 이벤트 입니다.
* 그 동안 살펴봤던 `open`, `leave`, `friend` 이벤트는 `유저`만 발생시킬 수 있는 이벤트였지만, `send`는 `유저`나 `파트너` 모두 발생시킬 수 있는 이벤트입니다.
* `send`는 여러 메시지 타입을 제공하고 있습니다.
  * `textContent`: 텍스트 메시지
  * `imageContent`: 단수 이미지로 구성된 메시지
  * `compositeContent`: 텍스트와 이미지 그리고 버튼을 포함하는 복합 메시지
<br>

#### `send` 이벤트 구조
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "textContent": {}, /* 텍스트 보낼때 사용 */
    "imageContent": {}, /* 단수 이미지 보낼때 사용 */
    "compositeContent": {}, /* 복합 메시지 보낼때 사용 */
    
    "options": {
        "notification": true /* push를 보낼때 사용 */
    }
}
```
> **[주의사항]**<br>
> `send` 이벤트는 **(1)톡톡에서 챗봇으로 이벤트를 전송할때**, **(2)`1번`의 응답으로 이벤트를 전송할때**, **(3)챗봇에서 특정 사건에의해 이벤트를 톡톡으로 전송할때** 사용될 수 있습니다.<br>
> `send` 이벤트에는 `textContent`, `imageContent`, `compositeContent` 중 하나만 선택하여 전송해야합니다.<br>
> `options`의 `notification`는 push를 보내야하는지 여부를 나타냅니다.<br>
> * `notification`는 `send`이벤트를 보내는 방법중 `3번`에서만 사용할 수 있습니다.
> * `notification`는 `default`값이 `false`입니다. push를 보내지않는경우는 `options`영역을 채울 필요가 없습니다.
> * `notification`를 `true`로 전송하더라도 `유저`가 채팅창에 들어와있으면 메시지를 실시간으로 받을 수 있기 때문에 push가 보내지지 않습니다.
> * push가 보내지면 메시지와 함께 `네이버me`의 `알림`에 알림이 전송되고, 모바일기기에 설치된 `네이버앱`에 push가 전달됩니다.
> * 위의 내용만으로는 `notification`를 항상 `true`로 보내는것이 좋을것같습니다. 그러나 몇몇 상황에서 `notification`는 부작용을 일으킬 수 있습니다.
>   * (1) `유저`가 톡톡 채팅창에서 챗봇과 채팅중 챗봇이 제공하는 외부링크로 페이지를 전환하였다면 톡톡페이지를 이미 벗어났기때문에 push를 받게됩니다. 그러나 `유저`의 경험은 아직 챗봇과 대화중에 있는것으로 알고있습니다. 예를 들어 별도 페이지에서 주소를 입력하고 있다가 push를 받을 수 있습니다.
>   * (2) `유저`가 채팅창에 있다고 판단하는 근거는 `Web Socket`으로 서버와 연결되어있을때 입니다. 그러나 빠른 페이지 렌더링을 위해 `Web Socket`는 `Lazy-Connection`를 하기때문에 `open`이벤트와 동시에 `notification` `true`로 전송하게되면 `유저`가 채팅창에서 push를 받을 수 있습니다.
>   * 위와같은 이유로 `default(false)` 사용을 추천합니다. `유저`의 요청에 챗봇은 실시간으로 응답하기 때문에 `유저`가 채팅창을 떠난상태에서 답변이 전송되는 경우가 드뭅니다.
>   * 그러나 `(1)`, `(2)`에 해당하지 않으면서 `배송출발` 또는 `예약완료`처럼 특정 사건에의해 챗봇이 이벤트를 보내야한다면 `notification` 사용을 권장합니다.
<br>

## 메시지 타입 명세서
<br>

### `textContent`
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "textContent": {
        "text": "안녕하세요? 도미노피자 주문 챗봇입니다. 6가지 인기메뉴를 빠르게 주문해보세요!", /* 채팅창에 노출할 텍스트 */
        "code": "", /* compositeContent 에서 버튼클릭시 전달받는 코드값, code는 채팅에 노출되지 않습니다. */
        "inputType": "typing|button|sticker|inquiry|vphone" /* 수신시에만 받는 속성으로, 사용자가 어떤 매개체로 봇에게 입력하는지를 나타내는 값입니다. */
    }
}
```
> **[사용팁]**<br>
> `text`에 전송하고자하는 텍스트를 기입합니다.
> * 줄바꿈이 필요할때 `\n`를 삽입합니다.
> * 텍스트에 `010-1234-1234` 또는 `01012341234`처럼 전화번호가 들어가면 채팅창에 노출될때 자동으로 `telto:`가 삽입되어 모바일기기는 `전화걸기`가 가능하도록 해 줍니다.
> * 텍스트에 `https://talk.naver.com/`처럼 URL이 삽입된경우 자동으로 Anchor Tag(`<a>`)로 변환하여 노출해 줍니다.
> * 텍스트의 최대길이는 영문/한글 구분없이 1만자이내로 전송 해야합니다.
> * `code`는 `compositeContent`에 내장된 버튼 클릭시 어떤 버튼을 눌렀는지 확인하는 용도로 활용할 수 있습니다.
> * `compositeContent` 전송시 두개의 버튼에 각각 `A`와 `B`코드를 넣었다면, 유저가 버튼클릭시 `code`값을 확인하여 어떤 버튼을 눌렀는지 확인할 수 있습니다.<br>

> **[inputType 정보]**<br>
> * typing : 유저가 직접 입력창에 적고 `보내기`버튼으로 보낸 경우입니다.
> * button : 봇에서 보낸 여러 형태의 버튼을 직접 눌러서 응답한 경우입니다. `code`값이 같이 포함됩니다.
> * sticker : 스티커를 눌러서 전송한 경우입니다. 사용자가 스티커를 눌렀을 때 응답을 하실 수 있습니다.
> * inquiry : 우측 상단의 상담요청 레이어를 통해서, `상담 요청하기` 버튼을 누른 경우입니다.
> * vphone : 스티커 버튼의 우측에 위치한 `안심전화번호로 상담요청하기` 버튼을 누른 경우입니다. 안심전화번호와 해당 전화번호의 유효기간(yyyy-MM-dd 형태)이 제공됩니다.
>   * 예 : {"text":"050719003814,2017-11-03", "inputType":"vphone"}
<br>

### `imageContent`
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "imageContent": {
        "imageUrl": "http://blogfiles5.naver.net/20130918_119/city0080_137946683395507ioT_JPEG/6.jpg" /* 전송하고자하는 이미지 URL */
    }
}
```
> **[사용팁]**
> * `imageUrl`는 외부에서도 접근가능한 URL이여야 합니다. 만약 사내에서만 접근가능한 URL이면 전송에 실패하게 됩니다.
> * 주어진 URL로 `톡톡서버`가 접근해서 이미지 다운로드 받을때 `HTTP` 응답 헤더 `Content-Type` 값은 반드시 해당 이미지 유형과 일치해야 합니다.
> * `imageContent`를 톡톡에 전송하게 되면 (1)`톡톡서버`는 해당 URL로 1회접근하여 이미지를 다운로드 받습니다. (2)다운로드받은 이미지는 톡톡에 보관되고 톡톡용 이미지URL로 `유저`에게 이미지를 노출합니다. 
> * `imageContent`를 톡톡에 전송하고 성공 응답을 받았다면 더 이상 이미지 URL은 유지되지 않아도 됩니다.
> * `imageContent`에 사용하는 이미지 제약은 아래와 같습니다.
>   * 이미지는 최대 20MB 용량까지 가능합니다.
>   * 이미지 포맷은 JPG, JPEG, PNG, GIF 가 가능합니다.
>   * `바탕투명` 또는 `움직이는이미지` 모두 채팅창에서 표현이 가능합니다.
> * `imageContent`를 톡톡으로 전송할때 문제가 발생되었다면 이미지 처리에대한 `imageContent Error 코드표`을 참조합니다.
> * `imageContent` 전송 중 오류가 발생되었다면 `imageContent Error 코드표`을 참조합니다.
<br>

> **[성능향상을 위한 고려사항]**
> * `imageUrl`에 명시된 이미지 URL은 메시지가 전송될때마다 `톡톡`에서 이미지를 다운로드받아 사용합니다.
> * 자주 사용되는 이미지는 [Image Upload API](/imageupload_api_v1.md)를 사용하여 메시지 전송 속도를 극대화 할 수 있습니다.
<br>
<br>

### `compositeContent`

![composite_message](/composite_message.jpg)

* `compositeContent`는 여러 형태의 `구성요소`를 복합적으로 사용할 수 있는 메시지 입니다.
* 하나의 `composite`은 아래의 `구성요소`를 포함할 수 있습니다.
  * `image`: 한 개의 이미지를 노출하는 `이미지` 요소
  * `elementList`: `타이틀`+`설명1`+`설명2`+`버튼`+`썸네일`로 구성할 수 있는 `리스트` 요소
  * `title`: 조금 두꺼운 글자로 표현되는 `타이틀` 요소
  * `description`: `title`보다 흐린 글자로 표현되는 `설명` 요소
  * `buttonList`: 다양한 기능을 가진 `버튼` 요소 리스트
* `compositeContent`에 `composite`을 두 개 이상 넣어서 `캐로셀(Carousel)`를 구성할 수 있습니다.
* `구성요소`의 노출 순서는 변경할 수 없습니다.
<br>

[모든 `구성요소`를 포함한 사용 예]
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "compositeContent": {
        "compositeList":[
            {
                "title": "타이틀",
                "description": "설명",
                "image": {
                    "imageUrl": "http://shop1.phinf.naver.net/20170216_20/talktalk_14872437839327BN4b_PNG/menu_01.png"
                },
                "elementList": { 
                    "type": "LIST",
                    "data": [
                        {
                            "title" : "리스트 요소 타이틀",
                            "description" : "리스트 요소 설명1",
                            "subDescription" : "리스트 요소 설명2",
                            "image" : {
                                "imageUrl": "http://shop1.phinf.naver.net/20170216_20/talktalk_14872437839327BN4b_PNG/menu_01.png"
                            },
                            "button" : {
                                "type": "TEXT", /* 리스트 요소의 버튼은 TEXT와 LINK 타입만 허용된다 */
                                "data" : {
                                    "title": "요소버튼", /* 리스트 요소의 버튼 명 (최대 4자) */
                                    "code" : "code"
                                }
                            }
                        }
                    ]
                },
                "buttonList": [
                    {
                        "type": "TEXT", /* 텍스트형 버튼 - 유저가 클릭하면 해당 버튼의 텍스트가 전송된다*/
                        "data" : {
                            "title": "텍스트형 버튼", /* 버튼에 노출하는 버튼명 (최대 18자)*/
                            "code" : "code" /* code를 정의하는경우 유저가 보내는 send이벤트 textContent에 code가 삽입되어 전송됨 (최대 1,000자)*/
                        }
                    },
                    {
                        "type": "LINK", /* 링크형 버튼 - 유저가 클릭하면 해당 페이지를 연다 */
                        "data": {
                            "title": "링크형 버튼", /* 버튼에 노출하는 버튼명 (최대 18자)*/
                            "url": "https://dominos-bot.talk.naver.com/view/menu/1", /* 톡톡 PC버전 채팅창에서 링크 URL */
                            "mobileUrl": "https://dominos-bot.talk.naver.com/view/menu/1#nafullscreen" /* 톡톡 모바일버전 채팅창에서 링크 URL */
                        }
                    },
                    {
                        "type": "OPTION", /* 옵션형 버튼 - 2depth로 이루어진 버튼. 유저가 클릭하면 채팅창 하단에 버튼 목록이 노출된다. */
                        "data": {
                            "title" : "옵션형 버튼",
                            "buttonList" :[
                                {
                                    "type": "TEXT", /* 옵션형 버튼은 TEXT, LINK, PAY 타입만 허용된다 */
                                    "data" : {
                                        "title": "옵션-텍스트버튼", /* 옵션형 버튼에 노출하는 버튼명 (최대 10자) */
                                        "code" : "code"
                                    }
                                },
                                {
                                    "type": "LINK",
                                    "data" : {
                                        "title": "옵션-링크버튼",
                                        "url": "https://dominos-bot.talk.naver.com/view/menu/1",
                                        "mobileUrl": "https://dominos-bot.talk.naver.com/view/menu/1#nafullscreen"
                                    }
                                }
                            ]
                        }
                    },
                    {
                        "type": "PAY", /* 네이버페이 결제 버튼 */
                        "data": {
                            "payKey": "wc8bls20170718002252151YjE1NzQwMD" /* 페이 결제 키 */
                        }
                    }
                ]
            }
        ]
    }
}
```
<br>

#### `CompositeContent` Object

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `compositeList` | Composite[] | Y |  |

> **[주의사항]**
> * `compositeList`에는 `Composite`을 최대 10개까지 넣을 수 있습니다.
> * `compositeList`에 넣는 `Composite`은 `null`일 수 없습니다.
<br>

#### `Composite` Object

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | N | `타이틀` 요소 값 |
| `description` | string | N | `설명` 요소 값 |
| `image` | Image | N | `이미지` 요소 |
| `elementList` | ElementList | N | `요소 리스트` |
| `buttonList` | Button[] | N | `버튼` 요소 리스트 |

> **[주의사항]**
> * 하나의 `composite`에는 `title`, `description`, `elementList` 중 하나는 반드시 존재해야 합니다.
> * 하나의 `composite`에는 최소한 두 개의 요소가 존재해야 합니다.
> * `title` 최대 길이는 200자 입니다.
> * `description` 최대 길이는 1000자 입니다.
> * `title`과 `description`에 줄바꿈이 필요하면 `\n`를 삽입합니다.
> * `buttonList`에는 `버튼`을 최대 10개까지 넣을 수 있습니다.
> * `buttonList`에 넣는 `버튼`은 `null`일 수 없습니다.
<br>

#### `Image` Object

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `imageUrl` | String | Y | 이미지 URL |

> **[주의사항]**
> * 권장 이미지 사이즈는 가로 530px, 세로 290px 입니다. (비율 1.82:1)
> * 지원포맷 : JPG, JPEG, PNG, GIF
> * 위에서 설명한 [imageContent](#imagecontent)의 사용방법과 성능향상을 위한 팁이 동일합니다.
<br>

#### `ElementList` Object

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | 리스트 요소의 타입. 현재는 `LIST` 타입만 존재 |
| `data` | ElementData[] | Y | 리스트 요소의 데이터 |

> **[주의사항]**
> * `data`에는 `ElementData`를 최대 3개까지 넣을 수 있습니다.
> * `data`에 넣는 `ElementData`는 `null`일 수 없습니다.
<br>

#### `ElementData` Object (`LIST` 타입)

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `타이틀` |
| `description` | string | N | `설명1` |
| `subDescription` | string | N | `설명2` |
| `image` | Image | N | `이미지` |
| `button` | Button | N | `버튼` |

> **[주의사항]**
> * `title`, `description`, `subDescription`은 최대 100자까지 입력할 수 있습니다.
> * `title` 모바일(해상도 375, 아이폰6s) 기준 최대 10자까지 노출되고 그 이상은 줄임표시(...)됩니다.
>   * `title`은 1줄로 노출됩니다.
> * `description` 모바일(해상도 375, 아이폰6s) 기준 최대 25자까지 노출되고 그 이상은 줄임표시(...)됩니다. 
>   * 줄바꿈이 필요하면 `\n`를 삽입합니다.
>   * `description`은 최대 2줄 노출됩니다.
> * `subDescription`모바일(해상도 375, 아이폰6s) 기준 최대 13자까지 노출되고 그 이상은 줄임표시(...)됩니다. 
>   * `subDescription`은 1줄로 노출됩니다.
> * `버튼`은 `TEXT`와 `LINK`타입만 허용되며 `title`길이는 10자로 제한됩니다.
> * `image`를 입력안하면 기본 이미지가 노출됩니다.
<br>

#### `Button` Object
```javascript
{
    "type": "버튼타입",
    "data": {
        /* 데이터 */
    }
}
```

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | `버튼` 요소의 타입. `TEXT`, `LINK`, `OPTION`, `PAY` |
| `data` | ButtonData | Y | `버튼` 요소의 데이터 |
<br>

##### `ButtonData` Object (`TEXT` 타입)
```javascript
{
    "type": "TEXT",
    "data": {
        "title": "타이틀",
        "code": "코드"
    }
}
```

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `버튼`에 노출되는 텍스트. `유저`가 버튼을 클릭하면 전송되는 `텍스트` |
| `code` | string | N | `유저`가 버튼을 클릭하면 전송되는 `코드` |

> **[주의사항]**
> * `title`의 최대 길이는 18자 입니다.
> * `code`의 최대 길이는 1,000자 입니다.
<br>

##### `ButtonData` Object (`LINK` 타입)
```javascript
{
    "type": "LINK",
    "data": {
        "title": "버튼에 노출되는 텍스트",
        "url": "http://your-pc-url.com",
        "mobileUrl": "http://your-mobile-url.com"
    }
}
```

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `버튼`에 노출되는 텍스트 |
| `url` | string | Y | PC버전 채팅창에서 `버튼`을 클릭하면 이동할 페이지 URL |
| `mobileUrl` | string | Y | 모바일 버전 채팅창에서 `버튼`을 클릭하면 이동할 페이지 URL |

> **[주의사항]**
> * `title`의 최대 길이는 18자 입니다.
> * 모바일에서는 `현재 창`에서 링크를 열고 PC에서는 `새 창`으로 링크를 엽니다.
> * 모바일에서는 `현재 창`으로 이동한 페이지에서 다시 채팅창으로 돌아올 때는 반드시 `뒤로가기` 또는 `history.back()`을 이용해야 합니다.
<br>

##### `ButtonData` Object (`OPTION` 타입)
```javascript
{
    "type": "OPTION",
    "data": {
        "title": "버튼에 노출되는 텍스트",
        "buttonList": [
            {
                "type": "타이틀",
                "data": {
                    /* 버튼 데이터*/
                }
            }
        ]
    }
}
```

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `버튼`에 노출되는 텍스트 |
| `buttonList` | Button[] | Y | `버튼` 클릭시 채팅창 하단에 노출되는 `버튼`의 목록 |

> **[주의사항]**
> * `title`의 최대 길이는 18자 입니다.
> * `buttonList`에는 `버튼`을 최대 10개까지 넣을 수 있습니다.
> * `buttonList`에 넣은 `버튼`은 `null`일 수 없습니다.
> * `buttonList`의 `버튼`에는 `TEXT`, `LINK`, `PAY` 타입만 허용됩니다.
>   * `title`은 10자로 제한됩니다.
<br>

#### 컴포지트 메시지 사용 가이드

> **[사용팁]**<br>
> `compositeList`에 들어가는 `composite`항목은 아래와 같은 제약이 있습니다.
> * `title`, `description`, `elementList` 중 적어도 한개의 속성은 존재해야 합니다.
> * `title`, `description`, `elementList`, `image`, `buttonList` 중 적어도 두개의 속성이 존재해야 합니다.
> `image`는 위에서 설명한 [imageContent](#imagecontent)의 사용방법과 성능향상을 위한 팁이 동일합니다.<br>
> `buttonList`에 `button`의 타입은 `TEXT`, `LINK`, `OPTION`, `PAY`가 있습니다.
> * `TEXT`타입을 사용할때 `유저`가 버튼을 클릭하면 버튼에 노출된 `text`가 `유저`가 보낸 메시지처럼 챗봇으로 `send`이벤트가 전달됩니다.
>   * 보통 `text`를 분석하면 `유저`가 어떤 버튼를 눌렀는지 파악할 수 있고 다음 프로세스를 진행할 수 있습니다.
>   * 그러나 `text`를 분석하기 어렵다면 또는 챗봇이 `Stateless`라서 상태를 관리할 수 없다면 `code`를 활용할 수 있습니다.
>   * 예를들어 `성별`를 물어보고 다음질문에 `나이대`를 물어본다고 했을때, `유저`가 남자(`code=1`)를 선택하고 `나이대` code를 이전 성별선택까지 넣어서 `1-10`, `1-20`... 처럼 만들 수 있습니다.
>   * 30대를 선택한경우 code는 `1-30`이 되고 `유저`가 남자이면서 30대라는 사실을 알 수 있습니다.
>   * 봇이 `Stateful`이라면 `text`를 직접 분석하는것을 추천합니다. 왜냐면 유저가 `compositeContent`에서 버튼을 누르거나 직접 타이핑한 텍스트까지 모두 이해할 수 있기 때문입니다.
> * `LINK`타입을 사용하면 톡톡 외부 웹페이지를 활용할 수 있습니다.
>   * 톡톡 채팅창은 `PC버전`과 `모바일버전` 모두 제공하고 있습니다. 심지어 `유저`는 `모바일버전`을 통해 챗봇과 대화를 하다가 PC를 켜고 `PC버전`에서 대화를 이어갈 수 도 있습니다. 그래서 `LINK`타입은 `PC`/`모바일` URL를 별도로 설정 해줘야합니다.
<br>
<br>

> **[LINK 타입 버튼 사용시 주의사항]**<br>
> `모바일` 채팅창에서 `LINK` 타입 버튼을 클릭하면 `현재 창`에서 링크를 엽니다. `톡톡 채팅창`에서 `외부 웹페이지`로 이동하게 되는것입니다.<br>
> 이때 중요한 `주의사항`은 `외부 웹페이지`에서 `톡톡 채팅창`으로 돌아올때 `페이지 이동`을 사용하지 말고 꼭! `history.back()`으로 돌아와야 합니다.
> * `외부 웹페이지`에서 `톡톡 채팅창`으로 `페이지 이동`하게 된다면 `톡톡 채팅창`에서 `history.back()`를 누르게 되면 `채팅 리스트`로 가는것이 아니라 `외부 웹페이지`로 다시 돌아가게 됩니다.
> * 보통 `톡톡 채팅창`의 버튼을 통해 이전 페이지를 간다면 충분히 핸들링이 가능하지만, 안드로이드의 물리적 버튼을 통해 `history.back()`하는 경우 `채팅 리스트`로 안내할 수 없습니다.
> * `외부 웹페이지`에서 `history.back()`으로 `톡톡 채팅창`에 돌아오기 위해서는 `외부 웹페이지`는 여러 페이지로 이동할 수 없고 SPA (Single-Page Application) 방식으로 만들어져야 합니다. 그렇지 않으면 여러 페이지로 만들어지되 `외부 웹페이지`도 철저히 `history.back()`으로만 원점으로 돌아올수있도록 유도해야 합니다.
<br>
<br>

> **[외부 웹페이지 제작 가이드]**<br>
> `유저`로부터 대화식으로 받기 어려운 복잡한 form 데이터가 있을 수 있습니다.<br>
> 이때 별도의 `외부 웹페이지`를 통해 form 데이터를 받아낼 수 있습니다. 그러나 `외부 웹페이지`에서 `파트너`와 `유저`를 어떻게 식별할지 고민이 생깁니다.<br>
> `파트너`와 `유저` 식별값을 모두 Get 파라미터로 넘길 수 있지만, `유저` 식별값은 반드시 `비공개`를 유지해야하며 임의로 식별값을 맞출만큼 복잡하지도 않습니다.<br>
> 그래서 토큰(`token`)을 사용해 `유저` 식별값이 외부에 노출되지 않도록 노력해야 합니다.<br>
> 토큰을 사용하는 방법을 좀더 설명하면, `compositeContent`를 톡톡으로 전송할때 삽입하는 `외부 웹페이지` URL에 토큰을 파라미터로 넣어서 전달해야합니다.
> * `compositeContent`를 톡톡에 전송할때 이미 알고있는 `유저`의 식별값에 대응되는 충분히 큰 랜덤 문자에 `key-value`로 유저 식별값을 서버에 저장하고 key를 토큰으로 URL에 파라미터로 붙입니다.
> * `외부 웹페이지`가 호출될때 `Server-Side`에서 토큰을 `key-value`로 유저 식별값을 얻어 사용하면 됩니다.<br>
> 위와같이 토큰방식이 구현되면 비록 동일한 유저라도 `compositeContent`에 넣어주는 토큰은 매번 변경될 것이고, 임의로 예측할 수 없을만큼 긴 토큰을 사용한다면 더욱 안전하게 유저 식별값을 보호 할 수 있습니다.
<br>
<br>

### 퀵버튼

![btn_quick](/btn_quick.jpg)

* `compositeContent`의 `OPTION`타입 버튼과 유사한 기능이 `textContent`와 `imageContent`에도 제공되고 있습니다.
* 메시지를 전송하면 채팅창 하단에 `버튼`이 나열됩니다.
* `TEXT`, `LINK`, `PAY` 타입 버튼을 사용할 수 있습니다.
* 버튼 `title`의 글자 수는 10자로 제한됩니다.
<br>

#### `textContent`에 퀵버튼 적용
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "textContent": {
        "text": "텍스트",
        "code": "코드",
        "quickReply": { /* 퀵버튼 - 빠른응답 */
            "buttonList": [ /* 버튼 리스트 */
                {
                    "type": "TEXT", /* TEXT, LINK, PAY 타입의 버튼이 허용된다. */
                    "data": {
                        /* 버튼 데이터 */
                    }
                }
            ]
        }
    }
}
```
<br>

#### `imageContent`에 퀵버튼 적용
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "imageContent": {
        "imageUrl": "http://blogfiles5.naver.net/20130918_119/city0080_137946683395507ioT_JPEG/6.jpg", /* 전송하고자하는 이미지 URL */
        "quickReply": { /* 퀵버튼 - 빠른응답 */
            "buttonList": [ /* 버튼 리스트 */
                {
                    "type": "TEXT",
                    "data": {
                        /* 버튼 데이터 */
                    }
                }
            ]
        }
    }
}
```
<br>

#### `compositeContent`에 퀵버튼 적용
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "compositeContent": {
        "compositeList": [
            /* composite 데이터 */
        ],
        "quickReply": {
            "buttonList": [
                {
                    "type": "TEXT",
                    "data": {
                        /* 버튼 데이터 */
                    }
                }
            ]
        }
    }
}
```
<br>

## ERROR 명세서

[Error 예제]
```javascript
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "success": false,
  "resultCode": "02",
  "resultMessage": "request json 문자열 파싱 에러"
}
```
<br>

[Error 구조]

| key | 이름 | Type | 필수 | 설명 |
|:---:|:----:|:----:|:----:|------|
| `success` | 성공여부 | boolean | Y | API호출 성공여부, false시 resultCode에따라 대응한다. |
| `resultCode` | 결과코드 | string | Y | "00"코드는 성공이며, 그외 실패시 원인이되는 코드값을 반환한다. |
| `resultMessage` | 코드설명 | string | N | resultCode를 구체적으로 설명한다. |
<br>

[공통 Error 코드표]

| success | resultCode | resultMessage | 설명 |
|:-------:|:----------:|---------------|------|
| true | `00` | success | 성공 |
| false | `01` | Authorization 정보 에러 | Authorization 정보가 잘못되었거나 이미 만료된 정보를 사용한 경우 |
| false | `02` | request json 문자열 파싱 에러 | json 구조에 문제가 있거나, 필수값이 없을경우 |
| false | `99` | {실패에 대한 구체적 내역} | 처리중 발생될 수 있는 다양한 상세설명 |
<br>

[imageContent Error 코드표]

| success | resultCode | resultMessage | 설명 |
|:-------:|:----------:|---------------|------|
| false | `IMG-01` | 이미지 업로드 - 포맷 불일치 or 처리 실패 | 업로드하는 이미지 포맷이 JPG, JPEG, PNG, GIF 아니거나 그외의 경우 |
| false | `IMG-02` | 이미지 업로드 - 전송/처리 시간 초과 | 업로드하는 이미지의 다운로드 시간이 10초를 초과하는경우  |
| false | `IMG-03` | 이미지 업로드 - 사이즈 초과 | 업로드하는 이미지가 20MB를 넘는경우 발생 |
<br>
<br>
