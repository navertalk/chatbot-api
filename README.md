# **Chat Bot API** V1

## 차례
* [개요](#개요)
* [챗봇용 파트너 계정 생성](#챗봇용-파트너-계정-생성)
* [`Webhook` 작성 방법](#webhook-작성-방법)
  * [`Webhook` 요구 사항](#webhook-요구-사항)
  * [**[STEP 1]** 네이버 톡톡으로부터 이벤트 받기](#step-1-네이버-톡톡으로부터-이벤트-받기)
  * [**[STEP 2]** echo 서버(챗봇 예제) 만들기](#step-2-echo-서버챗봇-예제-만들기)
* [`보내기 API` 작성 방법](#보내기-api-작성-방법)  
* [이벤트 명세](#이벤트-명세)
  * [이벤트 기본 구조](#이벤트-기본-구조)
  * [`open` 이벤트](#open-이벤트)
  * [`leave` 이벤트](#leave-이벤트)
  * [`friend` 이벤트](#friend-이벤트)
  * [`send` 이벤트](#send-이벤트)
  * [`echo` 이벤트](#echo-이벤트)
  * [`action` 이벤트](#action-이벤트)
  * [`persistentMenu` 이벤트](#persistentmenu-이벤트)
* [메시지 타입 명세](#메시지-타입-명세)
  * [textContent](#textcontent)
  * [imageContent](#imagecontent)
  * [compositeContent](#compositecontent)
    * [`CompositeContent` Object](#compositecontent-object)
    * [`Composite` Object](#composite-object)
    * [`Image` Object](#image-object)
    * [`ElementList` Object](#elementlist-object)
    * [`ElementData` Object(`LIST` 타입)](#elementdata-objectlist-타입)
    * [`Button` Object](#button-object)
    * [Composite 메시지 사용 시 참고 사항](#Composite-메시지-사용-시-참고-사항)
  * [퀵 버튼](#퀵-버튼)
* [오류 명세](#오류-명세)
* 확장 API
  * [Profile API V1](/profile_api_v1.md)
  * [Pay API V1](/pay_api_v1.md)
  * [Image Upload API V1](/imageupload_api_v1.md)
  * [Handover API V1](/handover_v1.md)
* [UI 컴포넌트](/ui_component_v1.md)
  * [TIME 컴포넌트](/time_component_v1.md)
  * [CALENDAR 컴포넌트](/calendar_component_v1.md)
  * [TIME-INTERVAL 컴포넌트](/time-interval-component.md)
<br>
<br>

## 개요
네이버 톡톡 챗봇 플랫폼은 네이버 톡톡에서 봇을 만드는 데 필요한 도구를 제공하는 Chat Bot API를 제공합니다. Chat Bot API를 이용해 사용자에게 다양한 메시지를 보내고, 봇에 네이버 쇼핑, 페이 등 네이버의 서비스를 연결할 수 있습니다. 

다음 그림은 Chat Bot API의 작동 구조를 나타낸 것입니다.
![composite_message](/images/chatbotapi_structure.png)
* 사용자의 다양한 이벤트가 `챗봇 플랫폼`을 통해 챗봇에게 `Webhook`으로 전달됩니다.
* 챗봇은 `보내기 API`를 이용해 사용자에게 메시지 전달 이벤트를 보낼 수 있습니다.
* `챗봇 플랫폼`은 기본적인 `메시지 API`뿐만 아니라 `Profile API`, `Pay API`를 제공합니다.
<br>
<br>


## 챗봇용 파트너 계정 생성
Chat Bot API를 이용하려면 먼저 챗봇용 파트너 계정을 생성해야 합니다. 챗봇용 파트너 계정 생성 방법은 다음과 같습니다.
1. [네이버 톡톡 파트너센터](https://partner.talk.naver.com/)에 접속합니다.
2. 회사 `단체아이디` 또는 대표성 있는 `개인아이디`로 로그인합니다.
3. **내계정관리 > 새로운 톡톡 계정 만들기**를 클릭합니다.
4. **서비스 선택하기**에서 서비스를 선택하지 않고 **서비스 연결 나중에 하기**를 클릭합니다.
5. 테스트 계정을 생성하려면 **개인**을 선택하고, 서비스 계정을 생성하려면 **국내사업자**, **해외사업자** 또는 **기관/단체**를 선택한 후 **다음**을 클릭합니다.
6. **대표이미지**, **프로필명**, **휴대폰 번호** 등의 정보를 입력하고 **사용신청**을 클릭합니다.
 생성된 계정은 **검수중** 상태로 표시되며, 검수가 완료되면 **사용중** 상태로 변경되고 SMS로 알림이 전송됩니다.
7. **내계정관리**의 **등록된 계정** 목록에서 **계정관리**를 클릭해 계정 홈에 진입합니다.
8. **개발자도구** 아래의 **챗봇API 설정**을 클릭해 신청 후 이용약관에 동의하고 챗봇을 설정합니다.
<br>

> **테스트 진행 시 검색 제외 처리**<br>
> 
> **파트너센터 > 기본설정 > 프로필정보**에서 프로필명 앞에 `[테스트]`를 삽입하면 `톡톡 가맹점 찾기`에서 제외됩니다. 그러면 테스트를 진행하는 동안 검색에서 유입되는 사용자 방문을 차단할 수 있습니다.<br>
> (예) `톡톡챗봇` -> `[테스트] 톡톡챗봇`

<br>

## `Webhook` 작성 방법
사용자 이벤트를 챗봇에게 전달할 때 `webhook`을 이용합니다. `webhook` 작성 방법은 다음과 같습니다.<br>

### `Webhook` 요구 사항
* `Webhook`은 `TLS` 기반의 통신을 지원해야 합니다.
  * 네이버 톡톡과 연동하는 외부 서비스 간의 메시지는 암호화 없이 평문으로 전달되므로 반드시 보호되어야 합니다.
  * 지원하는 TLS는 `TLSv1`, `TLSv1.1`, `TLSv1.2`입니다.
  * 정식 인증 기관으로부터 발급받은 유효한 인증서만 사용할 수 있습니다.
* `Webhook`을 사용하기 위해 ACL을 등록해야 한다면 다음의 IP 주소 목록을 추가합니다.
  * 117.52.141.192/27(117.52.141.193 ~ 117.52.141.222)
* 네이버 톡톡에서 `Webhook` 호출 시 설정값은 다음과 같습니다.
  * Connection timed out: `3초`
  * Read timed out: `5초`
<br>

### **[STEP 1]** 네이버 톡톡으로부터 이벤트 받기

#### 1. 네이버 톡톡으로부터 이벤트를 받기 위해 간단한 코드를 작성합니다.
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
* localhost에서 테스트 후 정식 인증서를 사용하여 https://your.domain/ 에 접근할 수 있어야 합니다.
<br>

#### 3. URL을 등록합니다.
* **파트너센터 > 개발자도구 > 챗봇API 설정**에서 **Webhook** 영역의 **이벤트 받을 URL**에 위에서 만든 URL을 등록합니다.
<br>

#### 4. 연동이 완료되어 톡톡으로부터 이벤트를 수신합니다.
* https://talk.naver.com/{파트너아이디 (예) wc1234}에 접근하여 사용자로서 이벤트를 발생시켜 봅니다.
<br>

* [채팅창에 진입했을 때 이벤트 로그]

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

* [채팅창에 `hello world` 메시지를 보냈을 때 이벤트 로그]

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

> **참고**
>
> 만약 `Webhook`을 통해 이벤트가 수신되지 않으면 **API 설정 > Webhook > Event 선택**에서 수신하고자 하는 이벤트를 선택합니다.<br>

<br>

### **[STEP 2]** echo 서버(챗봇 예제) 만들기
다음과 같은 기능을 하는 echo 서버를 만듭니다.
* 네이버 톡톡에서 보내는 각종 이벤트를 판별하고 적절한 메시지로 회신합니다.
* 사용자 메시지에 echo 메시지로 회신합니다.

#### 1. 다음과 같이 echo 서버 코드를 작성합니다.
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
        // 사용자가 보내는 메시지에 대해 echo로 전송
        response.textContent.text = 'echo: ' + req.body.textContent.text;
        res.json(response);

      } else {
        // 그 외의 경우는 무반응
        res.sendStatus(200);
      }
      break;

    // 채팅창 오픈 이벤트 처리
    case 'open' :
      switch(req.body.options.inflow) {
        // 채팅리스트에서 인입되었을 때
        case 'list' :
          response.textContent.text = '목록에서 눌러서 방문하셨네요.';
          res.json(response);
          break;

        // 링크로부터 인입되었을 때
        case 'button' :
          response.textContent.text = '버튼을 눌러서 방문하셨네요.';
          res.json(response);
          break;

        // 유입 경로가 없을 때
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
        // 친구 추가 시
        response.textContent.text = '친구가 되어 주셔서 감사합니다.';
        res.json(response);

      } else if(req.body.options.set == 'off') {
        // 친구 철회 시
        response.textContent.text = '다음 번에 꼭 친구 추가 부탁드려요.';
        res.json(response);
      }
      break;

    // 그 외의 이벤트에 대해 무반응
    default:
      res.sendStatus(200);
  }

});

app.listen(8080);
```
<br>

#### 2. https://talk.naver.com/{파트너아이디} 에 접근하여 각종 이벤트를 확인하고 학습합니다.
* request log를 보면서 어떤 이벤트가 발생되고, 각 이벤트에 어떻게 대응할지 고민해 봅니다.
<br>
<br>

## `보내기 API` 작성 방법
보통 단순한 챗봇은 `Webhook`만으로도 구현할 수 있습니다. 그러나 사용자의 질문에만 대답하는 단순한 형태가 아니라 상태 변화나 사건 발생에 따라 사용자에게 메시지를 푸시해야 한다면 `보내기 API`를 사용하여 챗봇에서 톡톡으로 이벤트를 전송해야 합니다.
<br>

다음과 같이 API 호출 테스트를 합니다.

[Request]
```bash
curl -X POST \
    -H "Content-Type: application/json;charset=UTF-8" \
    -H "Authorization: ct_wc8b1i_Pb1AXDQ0RZWuCccpzdNL" \
    -d '{ "event": "send", "user": "al-2eGuGr5WQOnco1_V-FQ", "textContent": { "text": "hello world" } }' \
    "https://gw.talk.naver.com/chatbot/v1/event"
```
* 인증 키(`Authorization`)는 **API 설정 > 보내기 API**에서 **생성**을 클릭해 받을 수 있습니다.
* 만약 인증 키(`Authorization`)가 공개되었다면 **재설정**을 클릭해 인증 키를 변경할 수 있습니다.
<br>

[Response]
```javascript
HTTP/1.1 200 OK

{
    "success": true,
    "resultCode": "00"
}
```
* 응답 결과는 [`오류 명세`](#오류-명세)를 참고하십시오.
<br>
<br>

## 이벤트 명세


### 이벤트 기본 구조
이벤트의 기본 구조는 다음과 같습니다.

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
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* 사용자 식별값 */
}
```

* `event`에는 다양한 이벤트를 식별할 수 있는 `이벤트명`이 들어옵니다.
* 특정 이벤트만으로 설명할 수 없는 추가적인 속성은 `options`를 이용해 얻을 수 있습니다.
* 톡톡에서 1:1 대화란 `파트너(partner)`와 `사용자(user)`의 대화입니다.
  * 챗봇은 `partner`에 해당되며, 챗봇을 사용하는 네이버 사용자는 `user`에 해당합니다.
  * `"user": "al-2eGuGr5WQOnco1_V-FQ"`는 사용자 식별값이며 특정 `네이버 아이디`에 해당하는, 변하지 않는 절댓값입니다.
<br>
<br>

[`Response` 메시지 구조]
```javascript
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

/* 응답과 동시에 이벤트를 전송할 때 사용 */
{
  "event": "send",
  "textContent": {
      "text": "hello world"
  }
}
```

* 챗봇에서 이벤트 전달에 대한 응답은 `HTTP Status` `200`이어야 성공으로 간주합니다. `HTTP Status`가 `200` 이외의 값이면 `톡톡`에서 실패로 간주하고 오류를 로깅합니다.
* 만약 이벤트에 대해 자동 반응 메시지를 보내고 싶다면 `Header`에 `Content-Type: application/json;charset=UTF-8`을 포함하고 `body`에 전달하고자 하는 이벤트를 추가합니다. 이때 `user` 속성은 무시되고 이벤트를 보냈던 사용자에게 이벤트가 전달됩니다.
<br>

> **참고**<br>
> 
> 사용자의 메시지에 챗봇이 회신하는 형태에는 `동기식`/`비동기식` 메시지 전송이 있습니다.
> * `동기식`은 사용자의 메시지 이벤트에 응답 메시지를 함께 주는 방법입니다. `동기식`은 단건 응답이거나 5초 이내에 결과 메시지를 전송할 수 있는 경우 사용할 수 있습니다.
> * `비동기식`은 사용자의 메시지 이벤트에 바로 응답한 후(`200 OK`), `보내기 API`를 이용해 톡톡에 직접 이벤트를 전송하는 방법입니다. `비동기식`은 여러 건의 메시지로 응답을 전달해야 하거나 일정 시간 경과 후 메시지를 전송할 때 사용할 수 있습니다.

<br>


### `open` 이벤트
사용자가 채팅창에 진입할 때 유입 정보와 함께 `open` 이벤트를 전송합니다.
* 유입 정보는 `inflow`를 이용해 어떤 방식으로 유입되었는지 구분하고, `referer`를 이용해 유입 페이지 URL을 알 수 있습니다.
* 별도로 `from` 파라미터를 사용하였다면 `from`을 이용해 값을 전달받을 수 있습니다.

[Request 전체]
```javascript
{
    "event": "open",
    "options": {
        "inflow": "button",
        "referer": "http://storefarm.naver.com/pqbdo/products/309672359",
        "from": "309672359",
        "friend": false, /* false: 친구 아님,  true: 친구 */
        "under14": false, /* false: 만14세 이상,  true: 만14세 미만 */
        "under19": false /* false: 만19세 이상,  true: 만19세 미만 */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* 사용자 식별값 */
}
```

[버튼이나 링크로 유입된 경우 `options`]
```javascript
    "options": {
        "inflow": "button",
        "referer": "http://storefarm.naver.com/pqbdo/products/309672359",
        "from": "309672359"
    }
```

[톡톡 채팅리스트로 유입된 경우 `options`]
```javascript
    "options": {
        "inflow": "list",
        "referer": "https://talk.naver.com/"
    }
```

[브라우저 주소창에 URL을 직접 입력해 유입된 경우 `options`]
```javascript
    "options": {
        "inflow": "none"
    }
```

> **참고**
> 
> 사용자와 대화를 나눌 때 유입 경로는 많은 정보를 제공해 줄 수 있습니다. 
> * 쇼핑몰이라면 `http://shopping.com/product/1234`로부터 유입된 경우 `1234 상품 구매를 원하시나요?`라고 사용자의 의도를 파악해 대응할 수 있습니다. 
> * 경우에 따라 하나의 웹페이지에 여러 개의 상품과 함께 버튼을 노출하였다면 `referer`만으로는 상품을 구분할 수 없습니다. 그럴 땐 `https://talk.naver.com/wc1234?from=1234`와 `https://talk.naver.com/wc1234?from=5678`처럼 `from` 파라미터를 이용하여 상품 번호를 받아낼 수 있습니다.
> * 톡톡은 웹 서비스이므로 어떤 웹 페이지든 버튼이나 링크를 통해 챗봇과 대화를 시작할 수 있습니다. 여러 서비스에 배너 광고를 하는 경우 사용자가 주로 어떤 배너로 유입되는지, 그래서 어떤 배너에 집중할지 통계 데이터로도 활용할 수 있습니다.
> * `"inflow": "list"`의 경우 톡톡 채팅리스트를 통한 유입으로 단지 과거 대화 이력을 보기 위해 `open`했는지 알 수 없으므로 마지막 메시지 발생 시간 또는 문맥에 따라 적절히 대응해야 합니다. 
> * `under14`는 `open` 이벤트에 반드시 포함되는 `options`의 하위 속성입니다. 개인정보수집을 위한 약관 동의 시 만 14세 미만의 경우 `법정대리인`의 동의를 받아야 합니다.(`개인정보보호법 제22조`) 챗봇이 개인정보를 수집한다면 `법정대리인` 동의 프로세스를 구현하거나 만 14세 미만은 사용하지 못하도록 안내하여야 합니다.

<br>

### `leave` 이벤트
사용자가 채팅창 또는 채팅리스트에서 **나가기**를 누르면 발생하는 이벤트입니다.
* `leave` 이벤트의 `Response`에 `request`를 넣어도 메시지가 전송되지 않고 항상 무시됩니다.
* `leave` 이벤트는 일반 메신저 경험처럼 `채팅창`을 나간 것으로 `채팅리스트`에는 존재하지 않지만, 챗봇이 사용자에게 다시 메시지를 보내면 채팅방이 다시 생성되고 `채팅리스트`에도 표시됩니다.
* `leave` 이벤트를 이용해 챗봇이 사용자에게 적극적으로 이후 메시지를 보낼 것인지 아니면 이쯤에서 더 이상 대화를 하지 않을 것인지 선택할 수 있습니다.
* 사용자가 `채팅창`을 나갔다고 해서 챗봇과 대화하기를 원하지 않는다고 볼 수는 없습니다.


```javascript
{
    "event": "leave",
    "user": "al-2eGuGr5WQOnco1_V-FQ"
}
```
<br>
<br>

### `friend` 이벤트
사용자가 **친구추가** 또는 **친구철회**를 누르면 발생하는 이벤트입니다. `options`의 `set`를 `on`/`off`로 설정해 `친구추가`와 `친구철회`를 구분할 수 있습니다.

```javascript
{
    "event": "friend",
    "options": {
        "set": "on" /* on: 친구추가,  off: 친구철회 */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ"
}
```
> **참고**
> 
> `friend` 이벤트는 가급적 이벤트 수신을 끄거나 수신하더라도 응답하지 않는 것이 좋습니다. 톡톡에서 친구란 [파트너센터](https://partner.talk.naver.com/)의 **마케팅관리 > 단체메시지**를 이용해 메시지를 보낼 수 있는 대상자를 의미합니다. 챗봇이 **친구추가**에 대해 감사 메시지를 직접 보내는 것보다 **마케팅관리 > 친추감사메시지**를 설정하여 다양한 메시지를 보내는 것을 추천합니다.

<br>

### `send` 이벤트
`send` 이벤트는 메시지를 보낼 때 발생하는 이벤트입니다. 앞서 살펴봤던 `open`, `leave`, `friend` 이벤트는 `사용자`만 발생시킬 수 있지만, `send`는 `사용자`와 `파트너` 모두 발생시킬 수 있습니다.<br>
`send` 이벤트는 다음과 같은 메시지 타입을 제공합니다.
  * `textContent`: 텍스트 메시지
  * `imageContent`: 단수 이미지로 구성된 메시지
  * `compositeContent`: 텍스트와 이미지, 버튼을 포함하는 복합 메시지
<br>

#### `send` 이벤트 구조
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
    "textContent": {}, /* 텍스트를 보낼 때 사용 */
    "imageContent": {}, /* 단수 이미지를 보낼 때 사용 */
    "compositeContent": {}, /* 복합 메시지를 보낼 때 사용 */
    
    "options": {
        "notification": true /* push를 보낼 때 사용 */
    }
}
```

`send` 이벤트는 다음과 같은 경우에 사용될 수 있습니다. 
* 톡톡에서 챗봇으로 이벤트를 전송할 때
* 톡톡에서 챗봇으로 전송한 이벤트의 응답으로 이벤트를 전송할 때
* 챗봇에서 특정 사건에 의해 이벤트를 톡톡으로 전송할 때

`send` 이벤트는 `textContent`, `imageContent`, `compositeContent` 중 하나만 선택하여 전송해야 합니다.<br>
`options`의 `notification`은 push를 보내야 하는지 여부를 나타냅니다.<br>
* `notification`은 `send` 이벤트를 전송하는 경우 중 세 번째 경우에만 사용할 수 있습니다.
* `notification`은 `default`값이 `false`입니다. push를 보내지 않는 경우는 `options` 영역을 채울 필요가 없습니다.
* `notification`을 `true`로 전송하더라도 `사용자`가 채팅창에 들어와 있으면 메시지를 실시간으로 받을 수 있기 때문에 push를 보낼 수 없습니다.
* push를 보내면 메시지와 함께 `네이버me`의 `알림`에 알림이 전송되고, 모바일 기기에 설치된 `네이버 앱`에 push가 전달됩니다.

위의 내용만으로는 `notification`을 항상 `true`로 보내는 것이 좋을 것 같습니다. 그러나 다음과 같은 상황에서 `notification`은 부작용을 일으킬 수 있습니다.
* `사용자`가 톡톡 채팅창에서 챗봇과 채팅 중 챗봇이 제공하는 외부 링크로 페이지를 전환했다면 톡톡 페이지를 이미 벗어났기 때문에 push를 받게 됩니다. 그러나 `사용자`는 아직 챗봇과 대화 중에 있는 것으로 알고 있습니다. 예를 들어 별도 페이지에서 주소를 입력하고 있다가 push를 받을 수 있습니다.
* `사용자`가 채팅창에 있는지는 `Web Socket`으로 서버와 연결되어 있는지 여부에 따라 판단합니다. 그러나 빠른 페이지 렌더링을 위해 `Web Socket`은 `Lazy-Connection`을 이용하므로 `open` 이벤트와 동시에 `notification`을 `true`로 전송하면 `사용자`가 채팅창에서 push를 받을 수 있습니다.

위와 같은 이유로 `default(false)` 사용을 추천합니다. `사용자`의 요청에 챗봇은 실시간으로 응답하므로 `사용자`가 채팅창을 떠난 상태에서 답변이 전송되는 경우는 드뭅니다. 그러나 위의 두 가지 경우에 해당하지 않으면서 `배송출발` 또는 `예약완료`처럼 특정 사건에 따라 챗봇이 이벤트를 보내야 한다면 `notification`을 사용하는 것을 권장합니다.
<br>
<br>

### `echo` 이벤트 
`echo` 이벤트는 [파트너센터](https://partner.talk.naver.com/)에서 상담사가 사용자에게 보낸 메시지와 챗봇이 사용자에게 보낸 메시지를 수신하는 이벤트입니다. 이벤트명은 `echo`이며 사용자에게 보내고자 했던 원본 이벤트는 `echoedEvent` 속성에 담기게 됩니다.<br>
`echo` 이벤트를 수신하려면 **개발자도구> API 설정 > Event선택**을 클릭하고 **echo**를 활성화합니다.
<br>

#### `echo` 이벤트 구조 

```json
{
  "event": "echo",
  "echoedEvent": "send",
  "user": "5KcCQTARWKNKv1IOvXwYQw",
  "partner": "wc8b1i",
  "textContent": {
    "text": "명함을 보냈습니다.",
    "inputType": "nameCard"
  },
  "options": {
    "mobile": false
  }
}
```

> **주의**
>
> `echo` 이벤트는 챗봇이 보낸 메시지까지 다시 수신하므로 해당 이벤트가 발생했을 때 메시지를 챗봇 플랫폼으로 재전송하면 끊임 없이 메시지가 송수신됩니다. 따라서 해당 이벤트를 정확히 테스트한 후  실제 서비스에 반영해야 합니다.
<br>

### `action` 이벤트
`action` 이벤트는 대화창에서 추가적인 행위가 필요할 때 사용할 수 있는 이벤트입니다. `action` 이벤트는 `Webhook`의 응답 메시지 또는 `보내기 API`를 이용해 전송할 수 있습니다.<br>
`action` 이벤트의 `action` 정보는 다음과 같습니다.
  * `typingOn`: `작성중` 이미지( ![image](https://talk.naver.com/static/front/pc/img/typing.gif) )가 파트너 메시지로 보입니다. 추가 액션이 없다면 10초 동안 유지됩니다. 연속해서 보내면 새로 보낸 시점부터 유지 시간이 갱신됩니다.
  * `typingOff`: `작성중` 이미지가 대화창에서 사라집니다. 다른 메시지를 `send` 이벤트로 보내도 동일한 효과가 있습니다.
```javascript
{
    "event": "action",
    "user": "al-2eGuGr5WQOnco1_V-FQ",
    "options": {
        "action": "typingOn" | "typingOff"
    }
}
```
> **참고**
>
> `typingOn`,`typingOff` 액션은 대화창에서 메시지 수신이 늦어지는 경우 메시지가 처리 중이라는 것을 사용자에게 알릴 때 사용할 수 있습니다. 혹은 너무 빠른 메시지 응답보다 천천히 대화하는 경험을 주고 싶을 때에도 사용할 수 있습니다. 
> 일반적으로 `typingOff` 액션을 사용하기 보다는 `typingOn` 전송 후 메시지 `send` 이벤트를 전송합니다.

<br>

### `persistentMenu` 이벤트

`persistentMenu`(이하 고정 메뉴)는 사용자가 대화 중에 상시로 접근할 수 있는 메뉴입니다. 사용자에게 챗봇을 소개하거나 공지, 이벤트를 안내하는 데 사용할 수 있습니다.

![persistent_menu](/images/persistentMenu.png)

이 메뉴에는 사용자가 언제든지 실행할 수 있는 기능만 포함해야 합니다. 예를 들어, URL을 연결하거나 클라이언트에서 코드를 전송받아 처리할 수 있습니다. 설정 이벤트이므로 `사용자 식별값`은 필요하지 않습니다. 

#### `persistentMenu` 이벤트 구조
```json
{
	"event":"persistentMenu",
	"menuContent" : [{
		"menus":
		[{
			"type":"TEXT", 
			"data":{
				"title":"챗봇 안내",
				"code":"CHATBOT_GUIDE"
			}
		},{
			"type":"LINK",
			"data":{
				"title":"이벤트 페이지",
				"url":"http://your-pc-url.com/event",
				"mobileUrl":"http://your-mobile-url.com/event"
			}
		},{
			"type":"LINK",
			"data":{
				"title":"전화하기",
				"url":"tel:021234567"  
			}
		},{
			"type":"NESTED",
			"data":{
				"title":"공지사항",
				"menus":
				[{
					"type":"LINK",    
					"data":{
						"title":"교환/환불 안내",
						"url":"http://your-pc-url.com/guide",
						"mobileUrl":"http://your-mobile-url.com/guide"
					}
				}]
			}
		}]
	}]
}
```
<br>

#### `menuContent` List

`menuContent`는 리스트 형태로 받습니다. `menuContent`를 초기화하려면 빈 리스트([])를 보냅니다.

> **주의**
> * 개발 확장성을 위해 `List`로 두었습니다. 현재는 첫 번째 `menuContent`만 허용합니다. 
> * 비어 있는 List를 전송하면 고정 메뉴가 `삭제`됩니다. 
>
> ```json
>{
>	"event":"persistentMenu",
>	"menuContent" : []
>}
>```
<br>

#### `menuContent` Object

`menuContent`는 `persistentMenu` 이벤트에 넣을 수 있는 여러 가지 오브젝트를 담는 리스트입니다. 넣을 수 있는 오브젝트 항목은 다음과 같습니다.

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `menus` | Menu[] | Y | `메뉴` 목록. <br>- 최대 4개까지 추가할 수 있습니다.<br>- `menus`는 `null`일 수 없습니다.   |
<br>

#### `Menu` Object

`Menu` 오브젝트의 가장 기본적인 형태입니다. `type`에 구체적인 종류를 선언하고, `data`에는 각 종류에 맞는 데이터를 넣습니다.

```
{
    "type": "메뉴 타입",
    "data": {
        /* MenuData */
    }
}
```

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | `메뉴` 요소의 타입. `TEXT`, `LINK`, `NESTED` |
| `data` | MenuData| Y | `메뉴` 요소의 데이터 |
<br>

##### `MenuData` Object(`TEXT` 타입)

텍스트 타입 버튼과 유사합니다. `code` 항목에 식별하고자 하는 값을 담아서 챗봇 클라이언트에서 받을 수 있습니다.

```json
{
    "type": "TEXT",
    "data": {
        "title": "타이틀",
        "code": "CODE"
    }
}
```

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `메뉴`에 노출되는 텍스트. `사용자`가 메뉴를 클릭하면 채팅창에 나타나는 `텍스트`. <br>최대 길이는 20자입니다. |
| `code` | string | Y | `사용자`가 메뉴를 클릭하면 클라이언트에게 전송되는 `코드`.  <br>최대 길이는 1,000자입니다. |

<br>

##### `MenuData` Object(`LINK` 타입)

링크 타입 버튼과 유사합니다. 사용자의 환경(PC, 모바일)에 따라서 URL 주소를 새 창으로 표시합니다.

```json
{
    "type": "LINK",
    "data": {
        "title": "타이틀",
        "url": "http://your-pc-url.com",
        "mobileUrl": "http://your-mobile-url.com"
    }
}
```
| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `메뉴`에 노출되는 텍스트. 최대 길이는 20자입니다. |
| `url` | string | Y | PC 채팅창에서 `메뉴`를 클릭하면 이동할 페이지 URL. <br>`url`에 `tel:{전화번호}`값을 넣으면 모바일에서는 전화 연결이 가능합니다. |
| `mobileUrl` | string | N | 모바일 채팅창에서 `메뉴`를 클릭하면 이동할 페이지 URL. <br>모바일 URL이 따로 있을 경우, `mobileUrl`을 설정하면 모바일에서는 해당 URL로 연결됩니다. |

> **참고**
>
> * 모바일에서는 `현재 창`에서 링크를 열고 PC에서는 `새 창`으로 링크를 엽니다.
> * 모바일에서는 `현재 창`으로 이동한 페이지에서 다시 채팅창으로 돌아올 때 반드시 `뒤로가기` 또는 `history.back()`을 이용해야 합니다.
> * [LINK 타입 버튼 사용 시 주의 사항](#link-타입-버튼-사용-시-주의-사항)을 참고해 주세요.
<br>


##### `MenuData` Object(`NESTED` 타입)
`NESTED` 타입은 중첩된 내부 메뉴를 만듭니다. 내부 메뉴는 최상위 메뉴를 포함하여 최대 3단계까지 만들 수 있습니다.

```
{
    "type": "NESTED",
    "data": {
        "title": "타이틀",
        "menus": [ /* Menu[] */ ]
    }
}
```

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `메뉴`에 노출되는 텍스트. 최대 길이는 20자입니다. |
| `menus` | Menu[] | Y | 하위 메뉴 목록.  |
<br>

## 메시지 타입 명세
<br>

### `textContent`

챗봇이 사용자에게 텍스트 메시지만 보내고자 할 때는 `text` 항목에만 데이터를 넣어서 전달하면 됩니다. 챗봇에 메시지를 전달할 때도 같은 형식을 사용하는데, 이때는 `code` 항목에 챗봇이 넣었던 값이 전달됩니다. 
`inputType` 항목은 챗봇과 대화 중에 사용자가 톡톡의 부가 기능을 클릭했을 때, 해당 텍스트 속성이 어느 매개체(혹은 부가 기능)로 생성된 것인지에 관한 정보를 담습니다.


```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
    "textContent": {
        "text": "안녕하세요? 도미노 피자 주문 챗봇입니다. 6가지 인기 메뉴를 빠르게 주문해 보세요!", /* 채팅창에 노출할 텍스트 */
        "code": "", /* compositeContent에서 버튼 클릭 시 전달받는 코드값, code는 채팅에 노출되지 않습니다. */
        "inputType": "typing|button|sticker|inquiry|vphone" /* 수신 시에만 받는 속성으로, 사용자가 어떤 매개체로 봇에게 입력하는지를 나타내는 값입니다. */
    }
}
```

* `text`에 전송하고자 하는 텍스트를 기입합니다.
* 줄 바꿈이 필요할 때 `\n`을 삽입합니다.
* 텍스트에 `010-1234-1234` 또는 `01012341234`처럼 전화번호가 들어가면 채팅창에 노출될 때 자동으로 `telto:`가 삽입되어 모바일 기기에서는 바로 `전화걸기`를 할 수 있습니다.
* 텍스트에 `https://talk.naver.com/`처럼 URL이 삽입된 경우 자동으로 Anchor Tag(`<a>`)로 변환하여 노출합니다.
* 텍스트의 최대 길이는 영문/한글 구분 없이 1만자 이내로 전송해야 합니다.
* `code`는 `compositeContent`에 내장된 버튼 클릭 시 어떤 버튼을 눌렀는지 확인하는 용도로 활용할 수 있습니다.
* `compositeContent` 전송 시 두 개의 버튼에 각각 `A`와 `B` 코드를 넣었다면, 사용자가 버튼 클릭 시 `code`값을 확인하여 어떤 버튼을 눌렀는지 확인할 수 있습니다.<br>

* `inputType` 속성값의 정보는 다음과 같습니다.
  * typing: 사용자가 직접 입력창에 적고 `보내기` 버튼을 눌러 보낸 경우입니다.
  * button: 봇에서 보낸 여러 형태의 버튼을 직접 눌러서 응답한 경우입니다. `code`값이 같이 포함됩니다.
  * sticker: 스티커를 눌러서 전송한 경우입니다. 사용자가 스티커를 눌렀을 때 응답할 수 있습니다.
  * inquiry: 오른쪽 위의 상담 요청 레이어에서 `상담 요청하기` 버튼을 누른 경우입니다.
  * vphone: 스티커 버튼의 오른쪽에 위치한 `안심전화번호로 상담요청하기` 버튼을 누른 경우입니다. 안심전화번호와 해당 전화번호의 유효기간(yyyy-MM-dd 형태)이 제공됩니다.
    * 예: {"text":"050719003814,2017-11-03", "inputType":"vphone"}
<br>

### `imageContent`

`imageContent`는 이미지로만 구성된 말풍선입니다. 이미지의 크기는 고정되어 있으므로, 테스트를 거쳐  최적화된 이미지를 보내는 것이 사용자 경험에 좋습니다.

```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
    "imageContent": {
        "imageUrl": "http://blogfiles5.naver.net/20130918_119/city0080_137946683395507ioT_JPEG/6.jpg" /* 전송하고자 하는 이미지 URL */
    }
}
```
* `imageUrl`은 외부에서도 접근 가능한 URL이어야 합니다. 만약 내부에서만 접근 가능한 URL이면 전송에 실패하게 됩니다.
* 주어진 URL로 `톡톡 서버`가 접근해서 이미지를 다운로드할 때 `HTTP` 응답 헤더의 `Content-Type` 값은 반드시 해당 이미지 타입과 일치해야 합니다.
* `imageContent`를 톡톡에 전송하면 (1) `톡톡 서버`는 해당 URL로 1회 접근하여 이미지를 다운로드합니다. (2) 다운로드한 이미지는 톡톡에 보관되고 톡톡용 이미지 URL로 `사용자`에게  노출됩니다. 
* `imageContent`를 톡톡에 전송하고 성공 응답을 받았다면 더 이상 이미지 URL은 유지되지 않아도 됩니다.
* `imageContent`에 사용하는 이미지의 제약 사항은 아래와 같습니다.
   * 이미지는 최대 20MB 용량까지 사용할 수 있습니다.
   * 사용 가능한 이미지 포맷은 JPG, JPEG, PNG, GIF입니다.
   * `바탕투명` 또는 `움직이는이미지` 모두 채팅창에서 표현할 수 있습니다.
* `imageContent`를 톡톡으로 전송할 때 문제가 발생했다면 이미지 처리에 관한 "imageContent 오류 코드표"를 참고하십시오.
* `imageContent` 전송 중 오류가 발생했다면 "imageContent 오류 코드표"를 참고하십시오.
<br>

> **성능 향상의 위한 팁**
>
> `imageUrl`에 명시된 이미지 URL은 메시지가 전송될 때마다 `톡톡`에서 이미지를 다운로드해 사용합니다. [Image Upload API](/imageupload_api_v1.md)를 사용해 자주 사용되는 이미지를 미리 업로드하면 메시지 전송 속도를 높일 수 있습니다.

<br>

### `compositeContent`

`compositeContent`는 여러 형태의 `구성 요소`를 복합적으로 사용할 수 있는 메시지입니다.

![composite_message](/images/composite_message.jpg)

하나의 `composite`은 아래의 `구성 요소`를 포함할 수 있습니다.
  * `image`: 한 개의 이미지를 노출하는 `이미지` 요소
  * `elementList`: `타이틀`+`설명1`+`설명2`+`버튼`+`섬네일`로 구성할 수 있는 `리스트` 요소
  * `title`: 조금 굵게 표현되는 `타이틀` 요소
  * `description`: `title`보다 흐리게 표현되는 `설명` 요소
  * `buttonList`: 다양한 기능을 가진 `버튼` 요소 리스트

`compositeContent`에 `composite`을 두 개 이상 넣어서 `캐로셀(Carousel)`을 구성할 수 있습니다.<br>
`구성 요소`의 노출 순서는 변경할 수 없습니다.
<br>

[모든 `구성 요소`를 포함한 사용 예]
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
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
                                    "title": "요소버튼", /* 리스트 요소의 버튼명(최대 4자) */
                                    "code" : "code"
                                }
                            }
                        }
                    ]
                },
                "buttonList": [
                    {
                        "type": "TEXT", /* 텍스트형 버튼 - 사용자가 클릭하면 해당 버튼의 텍스트가 전송된다*/
                        "data" : {
                            "title": "텍스트형 버튼", /* 버튼에 노출하는 버튼명(최대 18자)*/
                            "code" : "code" /* code를 정의하는 경우 사용자가 보내는 send 이벤트 textContent에 code가 삽입되어 전송된다(최대 1,000자)*/
                        }
                    },
                    {
                        "type": "LINK", /* 링크형 버튼 - 사용자가 클릭하면 해당 페이지를 연다 */
                        "data": {
                            "title": "링크형 버튼", /* 버튼에 노출하는 버튼명(최대 18자)*/
                            "url": "https://dominos-bot.talk.naver.com/view/menu/1", /* 톡톡 PC 버전 채팅창에서 링크 URL */
                            "mobileUrl": "https://dominos-bot.talk.naver.com/view/menu/1#nafullscreen" /* 톡톡 모바일 버전 채팅창에서 링크 URL */
                        }
                    },
                    {
                        "type": "OPTION", /* 옵션형 버튼 - 2depth로 이루어진 버튼. 사용자가 클릭하면 채팅창 아래에 버튼 목록이 노출된다. */
                        "data": {
                            "title" : "옵션형 버튼",
                            "buttonList" :[
                                {
                                    "type": "TEXT", /* 옵션형 버튼은 TEXT, LINK, PAY 타입만 허용된다 */
                                    "data" : {
                                        "title": "옵션-텍스트버튼", /* 옵션형 버튼에 노출하는 버튼명(최대 10자) */
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

`CompositeContent`는 말풍선을 구성하는 여러 요소를 묶어서 표현합니다. 현재는 Composite 하나만 지원하며, 리스트 형태로 받게 되어 있습니다. 

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `compositeList` | Composite[] | Y |리스트 요소의 Composite. <br>- 최대 10개의 `Composite`를 추가할 수 있습니다.<br>- `Composite`는 `null`일 수 없습니다.  |

<br>

#### `Composite` Object

`Composite`는 여러 항목을 조합해서 만들 수 있는 말풍선입니다. 다음 사항에 유의해 주세요.
* 하나의 `composite`에 `title`, `description`, `elementList` 중 적어도 한 개의 속성은 존재해야 합니다.
* 하나의 `composite`에 `title`, `description`, `elementList`, `image`, `buttonList` 중 적어도 두 개의 속성이 존재해야 합니다.

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | N | `타이틀` 요소값. <br>- 최대 길이는 200자입니다. <br>- 줄 바꿈이 필요하면 `\n`을 삽입합니다. |
| `description` | string | N | `설명` 요소값. <br>- 최대 길이는 1000자입니다. <br>- 줄 바꿈이 필요하면 `\n`을 삽입합니다.|
| `image` | Image | N | `이미지` 요소 |
| `elementList` | ElementList | N | `요소 리스트` |
| `buttonList` | Button[] | N | `버튼` 요소 리스트. <br>- `버튼`을 최대 10개까지 넣을 수 있습니다.<br>- `버튼`은 `null`일 수 없습니다. |

<br>

#### `Image` Object

[imageContent](#imagecontent)의 사용 방법과 성능 향상을 위한 팁을 참고하십시오.

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `imageUrl` | String | Y | 이미지 URL |

> **참고**
>
> * 권장 이미지 크기는 가로 530px, 세로 290px입니다.(비율 1.82:1)
> * 지원되는 이미지 형식은 JPG, JPEG, PNG, GIF입니다.

<br>

#### `ElementList` Object

 Composite의 여러 요소 중 하나이며, Composite마다 작은 영역을 차지하고 있습니다. 리스트 형태로 여러 개의 요소를 받을 수 있습니다.

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | 리스트 요소의 타입. 현재는 `LIST` 타입만 존재합니다. |
| `data` | ElementData[] | Y | 리스트 요소의 데이터. <br>- `ElementData`를 최대 3개까지 넣을 수 있습니다. <br>- `ElementData`는 `null`일 수 없습니다. |

<br>

#### `ElementData` Object(`LIST` 타입)

`ElementData`는 Composite에서 이미지와 타이틀 사이에 위치하는 작은 영역입니다.
이 영역은 복잡한 내용보다는 Composite를 뒷받침하는 간략한 내용으로 구성하는 것을 권장합니다.


| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `타이틀`. <br>- 최대 100자까지 입력할 수 있습니다. <br>- 모바일(해상도 375, iPhone 6s) 기준으로 최대 10자까지 노출되고 그 이상은 줄임표(...)로 표시됩니다. <br>- `title`은 1줄로 노출됩니다. |
| `description` | string | N | `설명1`. <br>- 최대 100자까지 입력할 수 있습니다. <br>- 모바일(해상도 375, iPhone 6s) 기준으로 최대 25자까지 노출되고 그 이상은 줄임표(...)로 표시됩니다. <br>- 줄 바꿈이 필요하면 `\n`을 삽입합니다. <br>- `description`은 최대 2줄로 노출됩니다. |
| `subDescription` | string | N | `설명2`. <br>- 최대 100자까지 입력할 수 있습니다. <br>- 모바일(해상도 375, iPhone 6s) 기준으로 최대 13자까지 노출되고 그 이상은 줄임표(...)로 표시됩니다. <br>- `subDescription`은 1줄로 노출됩니다. |
| `image` | Image | N | `이미지`. 입력하지 않으면 기본 이미지가 노출됩니다. |
| `button` | Button | N | `버튼`. `TEXT`와 `LINK` 타입만 허용되며 `title` 길이는 10자로 제한됩니다. |

<br>

#### `Button` Object

사용자에게 버튼을 표현하기 위한 기본 구조는 아래와 같습니다. 버튼 타입별로 넣을 수 있는 데이터가 다르므로 유의하여 구성해야 합니다.

```javascript
{
    "type": "버튼 타입",
    "data": {
        /* 데이터 */
    }
}
```

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | `버튼` 요소의 타입. `TEXT`, `LINK`, `OPTION`, `PAY` |
| `data` | ButtonData | Y | `버튼` 요소의 데이터 |
<br>

##### `ButtonData` Object(`TEXT` 타입)

버튼 구성 시 챗봇에서 만든 텍스트와 코드입니다. 텍스트는 사용자가 볼 수 있는 값이고, 코드는 HTML 속성으로만 표현되는 값입니다. 
텍스트와 코드가 함께 전달되므로 코드값을 해석해 원하는 시나리오를 진행하는 것을 추천합니다.


```javascript
{
    "type": "TEXT",
    "data": {
        "title": "타이틀",
        "code": "코드"
    }
}
```

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `버튼`에 노출되는 텍스트. <br>- `사용자`가 버튼을 클릭하면 전송되는 `텍스트`입니다. <br>- 최대 길이는 18자입니다. |
| `code` | string | N | `사용자`가 버튼을 클릭하면 전송되는 `코드`. 최대 길이는 1,000자입니다. |

<br>

##### `ButtonData` Object(`LINK` 타입)

링크 타입의 버튼을 누르면 사용자의 환경에 따라 모바일 URL 혹은 URL의 페이지가 새로 표시됩니다. 제공할 페이지에서 모바일 버전과 PC 버전 모두 사용자의 환경에 맞게 제공하는 것을 추천합니다.

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

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `버튼`에 노출되는 텍스트. 최대 길이는 18자입니다. |
| `url` | string | Y | PC 버전 채팅창에서 `버튼`을 클릭하면 이동할 페이지 URL |
| `mobileUrl` | string | Y | 모바일 버전 채팅창에서 `버튼`을 클릭하면 이동할 페이지 URL |

> **참고**
>
> * 모바일에서는 `현재 창`에서 링크를 열고 PC에서는 `새 창`으로 링크를 엽니다.
> * 모바일에서는 `현재 창`으로 이동한 페이지에서 다시 채팅창으로 돌아올 때 반드시 `뒤로가기` 또는 `history.back()`을 이용해야 합니다.
<br>

##### `ButtonData` Object(`OPTION` 타입)

`OPTION` 타입의 버튼을 클릭하면 정의한 `퀵 버튼`이 제일 아래에 표시됩니다. 사용자가 버튼 영역 외의 영역을 클릭하면 사라집니다.

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

| 키 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `title` | string | Y | `버튼`에 노출되는 텍스트. 최대 길이는 18자입니다. |
| `buttonList` | Button[] | Y | `버튼` 클릭 시 채팅창 아래에 노출되는 `버튼`의 목록. <br>- `버튼`을 최대 10개까지 넣을 수 있습니다. <br>- `버튼`은 `null`일 수 없습니다. <br>- `버튼`에는 `TEXT`, `LINK`, `PAY` 타입만 허용됩니다. <br>- `title`은 10자로 제한됩니다. |

<br>

#### Composite 메시지 사용 시 참고 사항

* `buttonList`에 추가되는 `button`의 타입에는 `TEXT`, `LINK`, `OPTION`, `PAY`가 있습니다.
* `TEXT` 타입을 사용할 때 사용자가 채팅창에 적는 메시지와 버튼에 노출된 `text` 속성의 값은 동일한 형식으로 전달됩니다. 
   * 보통 `text`를 분석하면 `사용자`가 어떤 버튼을 눌렀는지 파악하고 다음 프로세스를 진행할 수 있습니다. 그러나 `text`를 분석하기 어렵거나 챗봇이 `Stateless`라서 상태를 관리할 수 없다면 `code`를 활용할 수 있습니다. 예를 들어, 사용자에게 `성별`과 `나이대`를 차례로 물어본다고 했을 때, `사용자`가 남자(`code=1`)를 선택한 경우 여기에 `나이대`(10대, 20대, 30대...)를 붙여 `1-10`, `1-20`, `1-30`...으로 코드를 만들 수 있습니다. 그런 다음 사용자가 30대를 선택하면 챗봇에는 `1-30`이라는 코드값이 전달되어 `사용자`가 남자이면서 30대라는 사실을 알 수 있습니다.
   * 봇이 `Stateful`이라면 `text`를 직접 분석하는 것을 추천합니다. 왜냐면 사용자가 `compositeContent`에서 버튼을 누르거나 직접 입력한 텍스트까지 모두 파악할 수 있기 때문입니다.
* `LINK` 타입을 사용하면 톡톡 외부 웹페이지를 활용할 수 있습니다.
   * 톡톡 채팅창은 `PC 버전`과 `모바일 버전`을 모두 제공하고 있습니다. 심지어 `사용자`는 `모바일 버전`에서 챗봇과 대화를 하다가 `PC버전`에서 대화를 이어갈 수도 있습니다. 그래서 `LINK` 타입은 `PC`/`모바일` URL을 별도로 설정해야 합니다.

<br>
<br>

##### LINK 타입 버튼 사용 시 주의 사항

`모바일` 채팅창에서 `LINK` 타입 버튼을 클릭하면 `현재 창`에서 링크를 엽니다. `톡톡 채팅창`에서 `외부 웹페이지`로 이동하게 되는 것입니다. 이렇게 `외부 웹페이지`로 이동했다가 `톡톡 채팅창`으로 돌아올 때는 `페이지 이동`을 사용하지 말고 **반드시 `history.back()`을 사용해야 합니다**.<br>
`외부 웹페이지`에서 `톡톡 채팅창`으로 `페이지 이동`을 할 때 `톡톡 채팅창`에서 `history.back()`을 누르면 `채팅 리스트`로 가는 것이 아니라 `외부 웹페이지`로 다시 돌아가게 됩니다. 보통 `톡톡 채팅창`의 버튼을 눌러 이전 페이지로 간다면 문제가 없지만, 안드로이드의 물리적 버튼을 눌러 `history.back()`하는 경우 `채팅 리스트`로 안내할 수 없습니다.<br>
`외부 웹페이지`에서 `history.back()`으로 `톡톡 채팅창`에 돌아오려면 `외부 웹페이지`는 여러 페이지로 이동할 수 없게 만들거나 SPA(Single-Page Application) 방식으로 만들어져야 합니다. 그렇지 않으면 여러 페이지로 만들되 `외부 웹페이지`도 철저히 `history.back()`으로만 이전 페이지로 돌아올 수 있도록 유도해야 합니다.
<br>
<br>

##### 토큰을 이용한 외부 웹페이지 제작
`사용자`로부터 대화식으로 받기 어려운 복잡한 폼 데이터는 별도의 `외부 웹페이지`를 통해 받아낼 수 있습니다. 이 경우 `외부 웹페이지`에서 `파트너`와 `사용자`를 어떻게 식별할지 고민이 생깁니다. `파트너`와 `사용자` 식별값을 모두 Get 파라미터로 넘길 수 있지만 `사용자` 식별값은 임의로 맞출 수 있을 정도로 단순하므로 반드시 `비공개`를 유지해야 합니다.
그래서 토큰(`token`)을 사용해 `사용자` 식별값이 외부에 노출되지 않게 해야 합니다. `compositeContent`를 톡톡으로 전송할 때 삽입하는 `외부 웹페이지` URL에 토큰을 파라미터로 넣어서 전달합니다.
* `compositeContent`를 톡톡에 전송할 때 이미 알고 있는 `사용자` 식별값에 대응되는 랜덤 문자를 `key`로, `사용자` 식별값은 `value`로 저장하고, `key`를 토큰값으로 사용해 URL에 파라미터로 추가합니다. 
* `외부 웹페이지`가 호출될 때 `Server-Side`에서 토큰을 `key`로 이용해 사용자 식별값을 얻을 수 있습니다.<br>

위와 같이 토큰 방식이 구현되면 비록 동일한 사용자라도 `compositeContent`에 넣는 토큰은 매번 변경될 것이고, 임의로 예측할 수 없을 만큼 긴 토큰을 사용한다면 더욱 안전하게 사용자 식별값을 보호할 수 있습니다.
<br>
<br>

### 퀵 버튼

퀵 버튼은 채팅 화면의 맨아래에 위치하는 버튼입니다. 따라서 퀵 버튼 아래에 다른 콘텐츠를 보내면 사라지게 됩니다. 또한 클릭 이후에는 사라지므로 한 번만 클릭할 수 있습니다.

![btn_quick](/images/btn_quick.jpg)

* `compositeContent`의 `OPTION` 타입 버튼과 유사한 기능이 `textContent`와 `imageContent`에도 제공되고 있습니다.
* 메시지를 전송하면 채팅창 아래에 `버튼`이 나열됩니다.
* `TEXT`, `LINK`, `PAY` 타입 버튼을 사용할 수 있습니다.
* 버튼 `title`의 글자 수는 10자로 제한됩니다.
<br>

#### `textContent`에 퀵 버튼 적용
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
    "textContent": {
        "text": "텍스트",
        "code": "코드",
        "quickReply": { /* 퀵 버튼 - 빠른 응답 */
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

#### `imageContent`에 퀵 버튼 적용
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
    "imageContent": {
        "imageUrl": "http://blogfiles5.naver.net/20130918_119/city0080_137946683395507ioT_JPEG/6.jpg", /* 전송하고자 하는 이미지 URL */
        "quickReply": { /* 퀵 버튼 - 빠른 응답 */
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

#### `compositeContent`에 퀵 버튼 적용
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
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

## 오류 명세

### 오류 예제
```javascript
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8

{
  "success": false,
  "resultCode": "02",
  "resultMessage": "request json 문자열 파싱 오류"
}
```
<br>

### 오류 구조

| 키 | 이름 | 타입 | 필수 | 설명 |
|:---:|:----:|:----:|:----:|------|
| `success` | 성공 여부 | boolean | Y | API 호출 성공 여부. 실패 시 resultCode에 따라 대응합니다. |
| `resultCode` | 결과 코드 | string | Y | "00" 코드는 성공이며, 그 외 실패 시 원인이 되는 코드값을 반환합니다. |
| `resultMessage` | 코드 설명 | string | N | resultCode를 구체적으로 설명합니다. |
<br>

### 공통 오류 코드표

| success | resultCode | resultMessage | 설명 |
|:-------:|:----------:|---------------|------|
| true | `00` | success | 성공 |
| false | `01` | Authorization 정보 오류 | Authorization 정보가 잘못되었거나 이미 만료된 정보를 사용한 경우 |
| false | `02` | request json 문자열 파싱 오류 | JSON 구조에 문제가 있거나 필숫값이 없을 경우 |
| false | `99` | {실패에 대한 구체적 내역} | 01, 02 외에 처리 중 발생할 수 있는 다양한 오류에 관한 상세 설명 |

<br>

### imageContent 오류 코드표

| success | resultCode | resultMessage | 설명 |
|:-------:|:----------:|---------------|------|
| false | `IMG-01` | 이미지 업로드 - 포맷 불일치 또는 처리 실패 | 업로드하는 이미지 포맷이 JPG, JPEG, PNG, GIF가 아니거나 그 외의 경우 |
| false | `IMG-02` | 이미지 업로드 - 전송/처리 시간 초과 | 업로드하는 이미지의 다운로드 시간이 10초를 초과하는 경우  |
| false | `IMG-03` | 이미지 업로드 - 크기 초과 | 업로드하는 이미지가 20MB를 넘는 경우 |
<br>
<br>
