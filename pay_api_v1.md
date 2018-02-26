# **Pay API** V1

## 개요

`Pay API`는 [`Chat Bot API`](./README.md)의 확장 API로서, 네이버페이 결제 버튼의 생성 요청 및 네이버페이 결제 관련 이벤트 수신 기능을 제공합니다. 
`Pay API`는 챗봇 API에 네이버페이 가맹 정보를 연동해야 이용할 수 있습니다.

 > **참고**
 > 
 > 네이버페이 가맹 정보 연동 방법은 [네이버 톡톡 파트너센터](https://partner.talk.naver.com/)의 **챗봇 API > API 설정** 메뉴에서 확인할 수 있습니다.
 
<br>


## 이벤트 명세

### pay_complete 이벤트
`pay_complete` 이벤트는 `사용자`가 [`compositeContent`](./README.md#compositecontent-object)의 `PAY` 타입 버튼을 눌러 네이버페이 간편 결제를 완료한 경우 발생하는 이벤트입니다.<br>
결제 승인 진행 여부는 `pay_complete` 이벤트에 대한 챗봇 응답의 HTTP 상태 코드로 결정됩니다. 각 HTTP 상태 코드에 따라 다음과 같이 결제 승인을 처리합니다.
* `200 OK`: 결제 승인을 진행합니다. 이후 결제 승인이 완료되면 실제 결제 처리(예: 계좌 이체나 카드 결제 등)가 되며 `pay_confirm` 이벤트가 전달됩니다.
* `200 OK`가 아닌 모든 HTTP 상태 코드(예: 301, 400, 404, 500...): 결제 승인을 진행하지 않습니다.<br>
결제 승인을 진행하지 않는 경우 사용자는 기본적으로 *어떠한 안내도 받지 못합니다*. 따라서 챗봇은 결제 승인을 진행하지 않는 건에 관해 적절한 이벤트를 전송하여 사용자에게 결제 승인을 진행하지 않는 이유를 안내해야 합니다. 예를 들어, `404` 상태 코드 응답과 함께 응답 본문으로 실패 안내를 담은 `textContent`를 첨부하는 것을 권장합니다.

결제 후 주문/예약 등을 구현하려면 `pay_complete` 이벤트가 아닌 `pay_confirm` 이벤트를 활용해야 합니다. `pay_complete`는 네이버페이 결제 절차의 완전한 종료가 아닌 사용자 결제에 대한 승인 진행 여부를 묻는 이벤트입니다. `pay_confirm` 이벤트는 결제 승인이 완료된 후 전달되는 이벤트입니다.

<br>

#### pay_complete 이벤트 구조
[결제 성공 시]
```javascript
{
    "event": "pay_complete",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    "options": {
        "paymentResult": { /* 결제 결과 정보 */
            "code" : "Success", /* 네이버페이 결제 결과 코드 */
            "paymentId" : "20170811D3adfaasLL", /* 네이버페이 결제 식별값.  */
            "merchantPayKey" : "bot-custom-pay-key-1234", /* PAY 버튼 데이터로 지정한 커스텀 결제 식별값 */
            "merchantUserKey" : "al-2eGuGr5WQOnco1_V-FQ",/* PAY 버튼 데이터로 지정한 커스텀 결제 사용자 식별값 */
	}
    }
}
```

[결제 실패 시]
```javascript
{
    "event": "pay_complete",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    "options": {
        "paymentResult": { /* 네이버페이 간편결제 결과 정보 */
            "code" : "Fail", /* 네이버페이 간편결제 결제창 호출 결과 코드 */
            "message" : "OwnerAuthFail", /* 네이버페이 간편결제 결제창 호출 결과 메시지 */
            "merchantPayKey" : "bot-custom-pay-key-1234", /* 판매자 결제 식별값 */
            "merchantUserKey" : "al-2eGuGr5WQOnco1_V-FQ" /* 판매자 사용자 식별값 */
	}
    }
}
```
* `paymentResult`의 `code`, `message` 필드값은 네이버페이 간편결제 API 결체창 호출 후 이동하는 `returnUrl`의 파라미터인 `resultCode`, `resultMessage`값을 그대로 반환합니다.
* 네이버페이 간편결제 결제창: https://developer.pay.naver.com/docs/api/payments#payments_window

<br>


### pay_confirm 이벤트
`pay_complete` 이벤트는 `챗봇`이 `pay_complete` 이벤트에 대해 `200 OK` 응답을 한 후 진행된 네이버페이 간편결제 결제 승인 결과를 전달하는 이벤트입니다. 네이버페이 간편결제 결제 승인 결과는 이벤트의 `options.paymentConfirmResult` 필드로 전달합니다.<br>
네이버페이 결제 승인은 연동 은행이나 사용자의 결제 수단 이슈 등 다양한 이유로 실패할 수 있습니다. 결제 승인 실패 시에도 `pay_confirm` 이벤트가 전달되며, 이때 `options.paymentConfirmResult.code`(JSON 응답 `{"options" : {"paymentConfirmResult": {"code": "값"}}}`)값은 `Fail`, 실패 사유는 `options.paymentConfirmResult.message`로 전달됩니다. 성공 시에는 `options.paymentConfirmResult.code` 필드값이 `Success`입니다. 따라서 `pay_confirm` 이벤트 수신 시 바로 주문/예약 등의 처리를 진행해서는 안 되며, `options.paymentConfirmResult.code`값에 따라 처리해야 합니다. 
* 네이버페이 간편결제 결제 승인 API: https://developer.pay.naver.com/docs/api/payments#payments_apply

<br>

#### pay_confirm 이벤트 구조
[결제 승인 성공 시]
```javascript
{
    "event": "pay_confirm",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    "options": {
        "paymentConfirmResult": { /* 결제 결과 정보 */
            "code" : "Success", /* 네이버페이 결제 승인 결과 코드 */
	    "message" : null, /* 네이버페이 결제 승인 결과 메시지 */
            "paymentId" : "20170811D3adfaasLL", /* 네이버페이 결제 식별값.  */
            "deatil" : {
		/* 네이버페이 간편결제 결제 승인 API 응답 본문의 detail 필드를 그대로 반환. */
	    }
	}
    }
}
```

[결제 승인 실패 시]
```javascript
{
    "event": "pay_confirm",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    "options": {
        "paymentConfirmResult": { /* 결제 결과 정보 */
            "code" : "Fail", /* 네이버페이 결제 승인 결과 코드 */
	    "message" : "잔액 부족", /* 네이버페이 결제 승인 결과 메시지 */
            "paymentId" : "20170811D3adfaasLL", /* 네이버페이 결제 식별값.  */
            "deatil" : {
		/* 네이버페이 간편결제 결제 승인 API 응답 본문의 detail 필드를 그대로 반환. */	    
	    }
	}
    }
}
```
<br>


## 메시지 데이터 명세

`PAY` 타입 버튼은 네이버페이 간편결제창을 호출하는 버튼으로서, [compsiteContent](./README.md#compositecontent-object)에 넣을 수 있습니다.

### `compositeContent`

##### `ButtonData` Object (`PAY` 타입)

| 필드 | 타입 | 필수 | 설명 |
|----|----|----|----|
|data|`PaymentInfo`|true|네이버페이 간편결제 결제 예약 파라미터|

#### `PaymentInfo` Object

`paymentInfo`와 네이버페이 간편결제 결제 예약 API 파라미터는 1:1로 대응합니다.([네이버페이 간편결제 결제예약 API](https://developer.pay.naver.com/docs/api/payments#payments_reserve) 참고) <br>
그러나 아래와 같은 몇 가지 차이가 있습니다.

* `modelVersion` 없음: 톡톡 결제 버튼은 model2 결제 방식만을 지원하므로 해당 값을 지정할 수 없습니다.
* `productName` 필수 아님: 생략 시 `productItems`의 첫 번째 요소의 `name` 필드로 대치합니다.
* `productName` 필수 아님: 생략 시 `productItems` 각 요소의 `count`의 총합으로 대치합니다.
* `taxScopeAmount` 필수 아님: 생략 시 `totalPayAmount`값으로 대치합니다. 따라서 전체 결제 금액이 과세 대상 금액이 됩니다.
* `taxExScopeAmount` 필수 아님: 생략 시 `totalPayAmount` - `taxScopeAmount`값으로 대치합니다.
* `merchantUserKey` 필수 아님: 생략 시 네이버 톡톡 챗봇 `user`값으로 대치합니다. 챗봇 사용자와 결제자 정보를 매핑할 수 있으므로 유용합니다.
* `webhookUrl` 없음: 네이버 톡톡 챗봇 결제에서 지원하지 않습니다.
* `returnUrl` 없음: 네이버 톡톡 챗봇 결제에서 지원하지 않습니다. 결제 최종 승인 처리는 [pay_complete 이벤트](#pay_complete-이벤트) 항목을 참고하십시오.

| 키 | 타입 | 필수 | 설명 |
|----|-----|----|----|
|merchantPayKey|string|true|가맹점 주문 내역을 확인할 수 있는 가맹점 결제 번호 또는 주문 번호를 전달해야 합니다.|
|merchantUserKey|string|false|가맹점의 사용자 키(개인 아이디와 같은 개인 정보 데이터는 제외하고 전달해야 합니다). 기본값은 네이버 톡톡 챗봇 사용자 키|
|productName|string|false|대표 상품명. 예: 장미의 이름 외 1건(X), 장미의 이름(O)|
|productCount|number|false|상품 수량. 예: A 상품 2개 + B 상품 1개의 경우 productCount 3으로 전달|
|totalPayAmount|number|true|총 결제 금액. 최소 결제 금액은 100원|
|deliveryFee|number|false|배송비. 기본값 0|
|taxScopeAmount|number|false|과세 대상 금액. 과세 대상 금액 + 면세 대상 금액 = 총 결제 금액|
|taxExScopeAmount|number|false|면세 대상 금액. 과세 대상 금액 + 면세 대상 금액 = 총 결제 금액|
|purchaserName|string|false|구매자 성명. 결제 상품이 보험인 경우에만 필수입니다. 그 외에는 전달할 필요가 없습니다.|
|purchaserBirthday|string|false|구매자 생년월일(yyyymmdd). 결제 상품이 보험인 경우에만 필수입니다. 그 외에는 전달할 필요가 없습니다.|
|productItems|`ProductItem`[]|true|productItem 배열|

#### `ProductItem` Object

`ProductItem` Object와 네이버페이 간편결제 결제 예약 API의 파라미터 `productItems`의 요소는 1:1로 대응합니다.

| 키 | 타입 | 필수 | 설명 |
|----|-----|----|----|
|categoryType|string|true|결제 상품 유형. 유형 상세 정보는 [개발 방법](#개발-방법)을 참고하십시오.|
|categoryId|string|true|결제 상품 유형 아이디. 유형 아이디 상세 정보는 [개발 방법](#개발-방법)을 참고하십시오.|
|uid|string|true|상품 고유 아이디|
|name|string|true|상품명|
|startDate|string|false|시작일(yyyyMMdd). 예: 20160701 결제 상품이 공연, 영화, 보험, 여행, 항공, 숙박인 경우 입력하는 것을 권장합니다. |
|endDate|string|false|종료일(yyyyMMdd). 예: 20160701 결제 상품이 공연, 영화, 보험, 여행, 항공, 숙박인 경우 입력하는 것을 권장합니다. |
|sellerId|string|true|파트너 사에서 하위 판매자를 식별하기 위해 사용하는 식별 키를 전달합니다. 영문 대소문자와 숫자만 허용합니다. |
|count|number|false|결제 상품 개수. 기본값은 1|

[`PAY` 타입 버튼 사용 예]
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    
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

## 네이버페이 결제 처리 구현 방법

네이버페이 결제 승인 진행 여부는 챗봇이 결정합니다. 챗봇은 상품의 재고 여부나 판매 가능 여부에 따라 결제 승인을 진행할지 결정해 `pay_complete` 이벤트에 대한 HTTP 응답으로 전달합니다
  * 항상 `200 OK` 응답으로 결제 승인을 처리하면 재고가 없는 상품에 대한 주문이 발생할 수 있으므로 주의해야 합니다.
  * 품절된 상품의 결제 승인을 거부하려면 `400`이나 `404`와 같은 오류 HTTP 상태 코드로 `pay_complete` 이벤트에 응답해야 합니다.

챗봇은 네이버페이 결제 승인 후 `pay_confirm` 이벤트를 수신합니다. 이 이벤트를 활용해 결제 완료 후 트리거될 로직을 구현할 수 있습니다.(예: 결제 후 예약/구매)<br>

이미 승인된 결제 건의 취소는 다음의 두 가지 방법으로 할 수 있습니다.
 * 네이버페이센터의 **결제 내역 관리**에서 취소할 수 있습니다.
 * 네이버페이 결제 취소 API를 사용해 취소할 수 있습니다. `paymentId`를 호출 파라미터로 사용하며, 이는 `pay_complete`의 이벤트 객체에 담겨 있습니다. 따라서 추후 취소 버튼 등을 구현하려면 `paymentId`를 반드시 보관해야 합니다.
    * 네이버페이 결제 취소 API: https://developer.pay.naver.com/docs/api/payments#payments_cancel<br>

> **참고**
>
> 챗봇 API에서는 별도로 결제 취소를 위한 API를 지원하지 않습니다.

### [예제] 결제 처리하기

```javascript
let express = reuiqre('express');
let app = express();

app.use(require('body-parser').json());

app.post('/', (req, res) => {
    // 네이버페이 결제창에서 결제가 완료되어 이벤트가 발생
    if (req.body.event == 'pay_complete') {
        // 상품의 재고 여부를 확인하여 결제 승인을 결정한다.
        if (hasStock(req.body.options.paymentResult.merchantPayKey)) {
            res.status(200); // 응답 본문을 전달할 필요가 없다.
        } else {
            res.status(404).json({
                "event": "send",
                "textContent": {
                    "text": "상품이 품절되어 결제를 취소합니다."
                }
            });
        }
    } else if (req.body.event == 'pay_confirm') {
        const result = req.body.event.options.paymentConfirmResult;
        if (result.code == 'Success') {
	    registOrder(result.paymentId, result.merchantUserKey, result.merchantPayKey); // 서비스의 주문로직 처리
	  res.json({
	        "event": "send",
	        "textContent": {
	            "text": "주문이 접수되었습니다."
 	        }
            });
	} else {
	   res.json({
	        "event": "send",
	        "textContent": {
	            "text": result.message // 네이버페이 간편결제 결제 승인 오류 메시지를 그대로 사용(예: 잔액 부족)
 	        }
            });
	}
   }
})
```
