# **Pay API** V1

## 개요

* `Pay API`는 [`Chat Bot API`](./README.md)의 확장 API이며 네이버페이 결제버튼의 생성요청 및 네이버페이 결제관련 이벤트 수신 기능을 포함합니다.
* `Pay API`는 챗봇 API에 네이버페이 가맹정보가 연동이 완료되어야 이용할 수 있습니다.
  * 네이버페이 가맹정보 연동방법은 네이버톡톡 파트너센터 챗봇API 설정 메뉴에서 확인할 수 있습니다.
  

## 이벤트 명세

### pay_complete 이벤트
* `pay_complete` 이벤트는 `유저`가 [`compositeContent`](./README.md#compositecontent-object)의 `PAY`타입 버튼을 눌러 네이버페이 간편결제를 완료한 경우 발생하는 이벤트입니다.
* 결제승인 여부는 `pay_complete` 이벤트에 대한 챗봇 응답의 HTTP 상태코드로 결정됩니다. 각 HTTP 상태코드에 따라 다음과 같이 결제승인을 처리합니다.
  * `200 OK` : 결제승인되어 실제 결제처리(ex: 계좌이체나 카드결제 등)가 진행됩니다.
  * `200 OK`가 아닌 모든 HTTP상태코드 (ex: 301, 400, 404, 500, ..) : 결제승인되지 않습니다.
    * 결제승인 되지 않는 경우 사용자는 기본적으로 *어떠한 안내를 받지 못합니다.*. 따라서 결제승인하지 않는 건에 대하여 챗봇은 적절한 이벤트를 전송하여 사용자에게 결제승인되지 않는 이유를 안내해야합니다. 가령 `404` 상태코드 응답과 함께 응답본문으로 실패안내를 담은 `textContent`를 첨부하는 것을 권장합니다.

<br>

#### pay_complete 이벤트 구조
[결제성공 시]
```javascript
{
    "event": "pay_complete",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    "options": {
        "paymentResult": { /* 결제 결과 정보 */
            "code" : "Success", /* 네이버페이 결제 결과 코드 */
            "paymentId" : "20170811D3adfaasLL", /* 네이버페이 결제 식별값.  */
            "merchantPayKey" : "bot-custom-pay-key-1234", /* PAY 버튼 데이터로 지정한 커스텀 결제 식별값 */
            "merchantUserKey" : "al-2eGuGr5WQOnco1_V-FQ",/* PAY 버튼 데이터로 지정한 커스텀 결제 유저 식별값 */
	}
    }
}
```

[결제실패 시]
```javascript
{
    "event": "pay_complete",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    "options": {
        "paymentResult": { /* 네이버페이 간편결제 결과정보 */
            "code" : "Fail", /* 네이버페이 간편결제 결제창호출 결과코드 */
            "message" : "OwnerAuthFail", /* 네이버페이 간편결제 결제창호출 결과메시지 */
            "merchantPayKey" : "bot-custom-pay-key-1234", /* 판매자 결제 식별값 */
            "merchantUserKey" : "al-2eGuGr5WQOnco1_V-FQ" /* 판매자 유저 식별값 */
	}
    }
}
```
* `paymentResult`의 `code`, `message` 필드값은 네이버페이 간편결제API 결체창 호출 후 이동되는 `returnUrl`의 파라메터인 `resultCode`, `resultMessage` 값을 그대로 반환합니다.
* 네이버페이 간편결제 결제창 : https://developer.pay.naver.com/docs/api/payments#payments_window
<br>

## 메시지 데이터 명세

`PAY` 타입 버튼은 네이버페이 간편결제창 호출을 위한 버튼으로써, [compsiteContent](./README.md#compositecontent-object)에 넣을 수 있습니다.

### `compositeContent`

##### `ButtonData` Object (`PAY` 타입)

| 필드 | 타입 | 필수 | 설명 |
|----|----|----|----|
|data|`PaymentInfo`|true|네이버페이 간편결제 결제예약 파라메터|

#### `PaymentInfo` 오브젝트

`paymentInfo` 필드와 네이버페이 간편결제 결제예약API 파라메터는 1:1로 대응합니다: [네이버페이 간편결제 결제예약 API 문서](https://developer.pay.naver.com/docs/api/payments#payments_reserve) <br>
그러나 아래와 같은 몇 개의 차이가 있습니다.

* `modelVersion` 필드 제거 : 톡톡 결제 버튼은 model2 결제방식만을 지원하므로 해당값을 지정할 수 없습니다.
* `productName` 필드 필수 -> 선택 : 생략시 `productItems`의 첫번째 엘리멘트의 `name` 필드로 대치합니다.
* `productName` 필드 필수 -> 선택 : 생략시 `productItems` 각 엘리먼트의 `count`의 총 합으로 대치합니다.
* `taxScopeAmount` 필드 필수 -> 선택 : 생략시 `totalPayAmount` 값으로 대치합니다. 따라서 전체 결제금액이 과세대상금액이 됩니다.
* `taxExScopeAmount` 필드 필수 -> 선택 : 생략시 `totalPayAmount` - `taxScopeAmount` 값으로 대치합니다.
* `merchantUserKey` 필드 필수 -> 선택 : 생략시 네이버톡톡 챗봇 `user`값으로 대치합니다. 챗봇 유저와 결제자 정보를 매핑할 수 있으므로 유용합니다.
* `webhookUrl` 필드 제거 : 네이버톡톡 챗봇 결제에서 지원하지 않습니다.
* `returnUrl` 필드 제거 : 네이버톡톡 챗봇 결제에서 지원하지 않습니다. 결제 최종승인 핸들링을 위해서는 [pay_complete 이벤트](#pay_complete-이벤트) 항목을 참고하십시오.

| key | type | 필수 | 설명 |
|----|-----|----|----|
|merchantPayKey|string|true|가맹점 주문내역 확인 가능한 가맹점 결제번호 또는 주문번호를 전달해야 합니다.|
|merchantUserKey|string|false|가맹점의 사용자 키(개인 아이디와 같은 개인정보 데이터는 제외하여 전달해야 합니다). 기본값은 네이버톡톡 챗봇 유저키|
|productName|string|false|대표 상품명. 예: 장미의 이름 외 1건(X), 장미의 이름(O)|
|productCount|number|false|상품 수량 예: A 상품 2개 + B 상품 1개의 경우 productCount 3으로 전달|
|totalPayAmount|number|true|총 결제 금액. 최소 결제금액은 100원|
|deliveryFee|number|false|배송비. 기본값 0|
|taxScopeAmount|number|false|과세 대상 금액. 과세 대상 금액 + 면세 대상 금액 = 총 결제 금액|
|taxExScopeAmount|number|false|면세 대상 금액. 과세 대상 금액 + 면세 대상 금액 = 총 결제 금액|
|purchaserName|string|false|구매자 성명. 결제 상품이 보험인 경우에만 필수 값입니다. 그 외에는 전달할 필요가 없습니다.|
|purchaserBirthday|string|false|구매자 생년월일(yyyymmdd). 결제 상품이 보험인 경우에만 필수 값입니다. 그 외에는 전달할 필요가 없습니다.|
|productItems|`ProductItem`[]|true|productItem 배열|

#### `ProductItem` 오브젝트

`ProductItem` 오브젝트와 네이버페이 간편결제 결제예약API 파라메터 `productItems`의 엘리멘트는 1:1로 대응합니다.

| key | type | 필수 | 설명 |
|----|-----|----|----|
|categoryType|string|true|결제 상품 유형. 유형 상세 정보는 페이 개발가이드를 참고하십시오.|
|categoryId|string|true|결제 상품 유형 아이디. 유형 아이디 상세 정보는 페이 개발가이드를 참고하십시오.|
|uid|string|true|상품 고유 아이디|
|name|string|true|상품명|
|startDate|string|false|시작일(yyyyMMdd). 예: 20160701 결제 상품이 공연, 영화, 보험, 여행, 항공, 숙박인 경우 입력을 권장합니다. |
|endDate|string|false|종료일(yyyyMMdd). 예: 20160701 결제 상품이 공연, 영화, 보험, 여행, 항공, 숙박인 경우 입력을 권장합니다. |
|sellerId|string|true|파트너 사에서 하위 판매자를 식별하기 위해 사용하는 식별키를 전달 합니다. ( 영대소문자 및 숫자만 허용 ) |
|count|number|false|결제 상품 개수. 기본값 1|

[`PAY` 타입 버튼 사용예]
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    
    "compositeContent": {
        "compositeList":[
            {
                "title": "챗봇 판매상품 1234",
                "description": "가격 15000원",
                "image": {
                    "imageUrl": "http://shop1.phinf.naver.net/20170216_20/talktalk_14872437839327BN4b_PNG/menu_01.png"
                },
                "buttonList": [
                    {
                        "type": "TEXT",
                        "data" : {
                            "title": "상품정보보기",
                            "code" : "info"
                        }
                    },
                    {
                        "type": "PAY", /* 네이버페이 결제 버튼 */
                        "data": {
                            "paymentInfo": {
                                "merchantPayKey": "bot-pay-1234",
                                "totalPayAmount": 15000,
                                "productItems": [
                                  {
                                    "categoryType": "FOOD",
                                    "categoryId": "DELIVERY",
                                    "uid": "bot-product-1234",
                                    "name": "챗봇 판매상품 1234"
                                  }
                                ]
                            }
                        }
                    }
                ]
            }
        ]
    }
}
```
<br>

## 개발가이드

* 네이버페이 결제에 대한 승인은 챗봇이 결정합니다. 상품의 재고여부나 판매 가능여부에 따라 챗봇은 `pay_complete` 이벤트에 대한 HTTP 응답을 적절하게 주어야합니다.
  * 항상 `200 OK` 응답으로 결제 승인을 처리할 경우, 재고 없는 상품에 대한 주문이 발생할 수 있으므로 주의해야합니다.
  * 품절된 상품에 대한 결제 승인을 거부하고자하면 `400`이나 `404`와 같은 에러 HTTP 상태코드로 `pay_complete` 이벤트에 응답해야합니다.
* 이미 승인된 결제건의 취소는 두 가지 방법으로 할 수 있습니다.
  * 네이버페이 가맹센터에서 결제내역 관리 메뉴를 통해 할 수 있습니다.
  * 네이버페이 결제취소API를 사용하여 할 수 있습니다. `paymentId`를 호출파라메터로 사용하며, 이는 `pay_complete`의 이벤트 객체에 담겨있습니다. 따라서 추후 취소 버튼 등을 구현하기 위해서는 `paymentId`를 반드시 보관해야합니다.
    * 참고 - 네이버페이 결제취소 API : https://developer.pay.naver.com/docs/api/payments#payments_cancel
  * 챗봇API에서는 별도로 결제취소를 위한 API를 지원하지 않습니다.

### [예제1] 결제승인 처리하기

```javascript
let express = reuiqre('express');
let app = express();

app.use(require('body-parser').json());

app.post('/', (req, res) => {
    // 네이버페이 결제창에서 결제가 완료되어 이벤트가 발생
    if (req.body.event == 'pay_complete') {
        // 상품의 재고여부를 확인하여 결제승인을 결정한다.
        if (hasStock(req.body.options.paymentResult.merchantPayKey)) {
            res.status(200).json({
                "event": "send",
                "textContent": {
                    "text": "결제가 승인되었습니다."
                }
            });
        } else {
            res.status(404).json({
                "event": "send",
                "textContent": {
                    "text": "상품이 품절되어 결제를 취소합니다."
                }
            });
        }
    }
}
})
```
