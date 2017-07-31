# **Profile API** V1

## 개요
* `Profile API`는 [`Chat Bot API`](/README.md)의 확장 API이며 네이버 회원 정보를 요청할 수 있습니다.
* `Profile API`는 `이벤트`형식 API이며 회원 정보 요청시 유저에게 `개인정보 제3자 제공 동의`하에 정보를 제공하게 됩니다.
* 제공 가능한 네이버 회원 정보는 다음과 같습니다.
  * `닉네임`
  * `휴대전화번호`
  * `주소`
<br>

## 이벤트 명세서
<br>

### 회원 정보 요청 `Request` 명세서 (챗봇 -> 톡톡)
```javascript
POST /chatbot/v1/event HTTP/1.1
Host: gw.talk.naver.com
Content-Type: application/json;charset=UTF-8
Authorization: ct_wc8b1i_Pb1AXDQ0RZWuCccpzdNL

{
    "event": "profile", /* 프로필 이벤트 */
    "options": {
        "field": "nickname", /* 요청할 회원 정보 필드 nickname | cellphone | address */
        "agreements": [ "cellphone", "address" ] /* 제공동의 필요시 함께 동의받을 필드들 */
    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* 유저 식별값 */
}
```

### 회원 정보 전달 `Request` 명세서 (톡톡 -> 챗봇)
```javascript
POST / HTTP/1.1
Host: your.bot.co.kr
Accept: application/json
Content-Type: application/json;charset=UTF-8

{
    "event": "profile", /* 프로필 이벤트 */
    "options": {
        /* nickname 를 요청한 경우 */
        "nickname": "네이버톡톡",
        
        /* cellphone 를 요청한 경우 */
        "cellphone": "01012341234",
        
        /* address 를 요청한 경우 */
        "address": {
            "roadAddr": "경기도 성남시 분당구 불정로 6 (정자동)", /* 도로명주소 */
            "detAddr": "톡톡부서", /* 상세주소 */
            "zipNo": "13561", /* 우편번호 */
            "rnMgtSn": "411353180030", /* 도로명코드 */
            "latitude": "37.3595316", /* 위도 */
            "longitude": "127.1052133" /* 경도 */
        },

        "result": "SUCCESS" /* 요청결과 SUCCESS | DISAGREE | CANCEL */

    },
    "user": "al-2eGuGr5WQOnco1_V-FQ" /* 유저 식별값 */
}
```
<br>

### `Profile API` 동작 흐름
* 챗봇에서 회원 정보가 필요할때 `profile` 이벤트를 톡톡에 전송합니다.
  * `profile` 이벤트 전송시 `field` 항목에 하나의 정보 필드만 요청할 수 있습니다.
* 톡톡은 챗봇으로부터 `profile` 이벤트를 받으면 곧바로 `SUCCESS` 응답을 줍니다. (`Non-Blocking`)
* 톡톡은 챗봇이 요청한 `field`에 대해 유저가 챗봇에게 `개인정보 제3자 제공 동의`를 하였는지 판단하여 동의 되어있다면 요청한 정보를 담아 `profile` 이벤트를 챗봇에게 전송합니다.
* 만약 `개인정보 제3자 제공 동의`가 되어있지 않다면 톡톡은 유저와 동의절차를 진행합니다.
  * 톡톡과 챗봇사이의 연결이 동의절차가 완료될때까지 일시적으로 중단됩니다.
  * 동의절차가 완료되기 전까지 유저 이벤트가 챗봇으로 전달되지 않습니다. 또한 챗봇이 보내는 이벤트에 대해서도 전달 되지않고 `Error 코드`로 응답이 옵니다.
* 톡톡과 유저사이에 동의절차가 완료되는 경우는 아래와 같습니다.
  * 유저가 동의한 경우 `"result": "SUCCESS"` 와 요청한 정보 필드 값을 담아서 `profile` 이벤트를 전송합니다.
  * 유저가 약관에 취소한 경우 `"result": "DISAGREE"` 만 담아서 `profile` 이벤트를 전송합니다.
    * `DISAGREE`의 경우 유저가 명확히 의사를 밝혔으므로 재차 요구하지 않는것이 좋습니다.
  * 유저가 정보 입력 중 취소한 경우 `"result": "CANCEL"` 만 담아서 `profile` 이벤트를 전송합니다.
    * `CANCEL`는 `cellphone`와 `address` 요청시에만 발생될 수 있습니다.
    * `cellphone`의 경우 유저가 `개인정보 제3자 제공 동의`를 하고 휴대폰번호를 변경하거나 `소유(점유)인증`과정 중 `취소`한 경우입니다.
    * `address`의 경우 유저가 `개인정보 제3자 제공 동의`를 하고 주소 선택 중에 `취소`한 경우입니다.
    * `CANCEL`이 `DISAGREE`와 다른점은 다음번에 다시 요청할때 동의절차가 생략되고 정보입력절차만 진행하게 됩니다.
  * 유저가 동의절차 중 아무런 응답을 하지 않으면 `타임아웃`이 발생될 수 있습니다.
    * 유저와 동의절차 과정중 마지막 메시지로부터 1분간 지체되면 `타임아웃`이 발생됩니다.
    * `타임아웃`발생시 챗봇으로 `profile` 이벤트가 전송되지 않으며 톡톡과 챗봇사이의 일시중지된 연결이 재개 됩니다.
      * 챗봇에서는 기대했던 회원 정보를 못받은 상태로 이벤트가 재개되었으므로 `profile` 이벤트를 다시 요청하거나 혹은 처음으로 진행하거나 판단하시면 됩니다.
<br>

### `개인정보 제3자 제공 동의`절차 최소화 방안
* 챗봇이 `nickname`, `cellphone`, `address` 정보가 필요할때 한번에 하나씩 요청할 수 있으므로 결국 유저는 3번의 동의절차가 필요합니다.
* 처음 `nickname`를 요청할때 `"agreements": [ "cellphone", "address" ]`를 통해 한번 동의에 3가지 필드를 포함시킬 수 있습니다.
* 만약 `nickname`이 동의 되어있고, 추가적으로 `"agreements": [ "cellphone", "address" ]`를 요청하였다면 유저는 동의절차없이 `nickname`만 전달됩니다.
  * 쉽게말하면 `nickname`이 동의 되어있지않아서 어차피 동의를 받는거라면 `agreements` 항목도 포함해서 동의 받는것입니다.
  * `nickname`이 이미 동의되어있을때는 `agreements`를 무시하는 이유는 현재 필요로하는 정보가 아니므로 다음번 `"field": "cellphone"`요청때 동의받을 수 있습니다.
<br>

### 정보 필드 설명

#### `nickname` 정보의 특징
* 네이버에서 유저가 직접 설정한 닉네임 정보입니다.
* 유저는 언제든지 닉네임을 변경할 수 있으므로 필요로할때마다 요청하셔야합니다.
* 닉네임 필드에 대해 한번 동의되어있으면 해당유저에게 다시 동의절차를 진행하지 않습니다.
* 경우에 따라 닉네임을 설정하지 않은 유저가 있습니다. 이럴경우 네이버아이디를 마스킹처리하여 제공하고 있습니다.
<br>

#### `cellphone` 정보의 특징
* 30일 이내 `소유(점유)인증`된 번호를 제공하고 있습니다.
* `소유(점유)인증`은 `본인인증`과 다르며 해당 번호로 랜덤번호 문자를 전송하여 인증하고 있습니다.
* 특정 유저가 해당 번호의 휴대폰을 소유하고 있다는것만 인증하는 것입니다.
* 인증된지 30일이 초과된 상태에서 `cellphone` 정보 제공 요청이 들어올때 다시 인증을 진행합니다.
* `소유(점유)인증`과 30일관리는 특정 챗봇마다 관리되는것이 아니고 톡톡에서 글로벌하게 관리합니다.
<br>

#### `address` 정보의 특징
* 톡톡은 유저마다 최대 5개의 주소를 보관하고 있습니다.
* `address` 정보 요청시 모든 주소리스트가 제공 되는것이 아니라 요청 시점에서 유저가 선택한 주소지 한개가 전달되는 것 입니다.
* 배달과 관련있는 챗봇이라면 한번 얻은 주소를 사용하지 마시고 매번 `address` 정보를 요청하여 현재 시점에 유저가 원하는 주소를 가져가 사용하셔야 합니다.
* 제공되는 `address` 속성명은 [행정안전부 주소 서비스](https://www.juso.go.kr/CommonPageLink.do?link=/addrlink/devAddrLinkRequestSample)를 따릅니다.
* 주소의 위치를 나타내는 좌표계는 `위도경도` 입니다. 네이버 지도 API와 연동될 수 있는 좌표계입니다.
* 주소는 `신주소`만 제공됩니다.
<br>
