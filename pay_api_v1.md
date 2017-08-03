# **Pay API** V1

## 개요

* `Pay API`는 [`Chat Bot API`](./README.md)의 확장 API이며 네이버페이 결제버튼의 생성요청 및 네이버페이 결제관련 이벤트 수신 기능을 포함합니다.
* `Pay API`는 챗봇 API에 네이버페이 가맹정보가 연동이 완료되어야 이용할 수 있습니다.
  * 네이버페이 가맹정보 연동방법은 네이버톡톡 파트너센터 챗봇API 설정 메뉴에서 확인할 수 있습니다.
  

## 이벤트 명세

### pay_complete 이벤트
* `pay_complete` 이벤트는 `유저`가 [`compositeContent`](./README.md#compositecontent-object)의 `PAY`타입 버튼을 눌러 네이버페이 간편결제를 완료한 경우 발생하는 이벤트입니다.
* 결제승인은 `pay_complete` 이벤트에 대한 챗봇응답의 HTTP 상태코드로 결정됩니다. 각 HTTP 상태코드에 따라 다음과 같이 결제승인을 처리합니다.
  * `200 OK` : 결제승인되어 실제 결제처리(ex: 계좌이체나 카드결제 등)가 진행됩니다.
  * `200 OK`가 아닌 모든 HTTP상태코드 (ex: 301, 400, 404, 500, ..) : 결제승인되지 않습니다.
    * 챗봇의 상태에 따라 `500`(ex: 서버오류 시), `400`(ex: 문제가 있는 결제 내용일 때), `404`(ex: 해당 결제 대상 상품을 찾을 수 없음)을 사용하여 톡톡이 결제취소상황을 자세하게 알 수 있도록 하는 것이 바람직합니다.
    * 결제승인 되지 않는 경우 사용자는 기본적으로 *어떠한 안내를 받지 못합니다.*. 따라서 결제승인하지 않는 건에 대하여 챗봇은 적절한 이벤트를 전송하여 사용자에게 결제승인되지 않는 이유를 안내해야합니다. 가령 `404` 상태코드 응답과 함께 응답본문으로 실패안내를 담은 `textContent`를 첨부하는 것을 권장합니다.

<br>

#### pay_complete 이벤트 구조
[결제성공 시]
```javascript
{
    "event": "pay_complete",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    "paymentResult": { /* 결제 결과 정보 */
        "code" : "Success", /* 네이버페이 결제 결과 코드 */
        "paymentId" : "20170811D3adfaasLL", /* 네이버페이 결제 식별값.  */
        "merchantPayKey" : "bot-custom-pay-key-1234", /* PAY 버튼 데이터로 지정한 커스텀 결제 식별값 */
        "merchantUserKey" : "al-2eGuGr5WQOnco1_V-FQ",/* PAY 버튼 데이터로 지정한 커스텀 결제 유저 식별값 */
    }

}
```

[결제실패 시]
```javascript
{
    "event": "pay_complete",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    "paymentResult": { /* 네이버페이 간편결제 결과정보 */
        "code" : "Fail", /* 네이버페이 간편결제 결제창호출 결과코드 */
        "message" : "OwnerAuthFail", /* 네이버페이 간편결제 결제창호출 결과메시지 */
        "merchantPayKey" : "bot-custom-pay-key-1234", /* 판매자 결제 식별값 */
        "merchantUserKey" : "al-2eGuGr5WQOnco1_V-FQ" /* 판매자 유저 식별값 */
    }
}
```
* `paymentResult`의 `code`, `message` 필드값은 네이버페이 간편결제API 결체창 호출 후 이동되는 `returnUrl`의 파라메터인 `resultCode`, `resultMessage` 값을 그대로 반환합니다.
* 네이버페이 간편결제 결제창 : https://developer.pay.naver.com/docs/api/payments#payments_window
<br>

## 메시지 데이터 명세

`PAY` 타입 버튼은 네이버페이 간편결제창 호출을 위한 버튼으로써, [compsiteContent](./README.md#compositecontent-object)에 넣을 수 있습니다.

### compositeContent

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
        if (hasStock(req.body.paymentResult.merchantPayKey)) {
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
