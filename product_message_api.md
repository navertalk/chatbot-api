# 스마트스토어 상품 메시지 API 
## 개요 
스마트스토어에서 판매하고 있는 상품들을 챗봇의 메시지로 표현할 수 있게 되었습니다.
`스마트스토어 상품 메시지 API`를 통해서 생성되는 말풍선은
* 상세 보기 (상품 상세 페이지 이동)
* 바로 구매
* 장바구니 담기
3가지 기능을 수행할 수 있습니다.
## 화면 구성
스마트스토어에 등록된 상품을 그대로 표현하고 있으며, 상품 노출 정책도 동일하게 반영하였습니다.
아래는 특정 상품에 대한 예시입니다. 
### 스마트스토어
![buying_ui](/images/buying_ui_both_c.png)
### `displayType`이 `single`인 예
![single](/images/product_message_api_single_type.png)
### `displayType`이 `list`인 예
![list](/images/product_message_api_list_type.png)
## API
```json
{
  "event": "product",
  "options": {
    // 스토어에서 사용하는 상품번호를 single형은 15개, list형은 4개까지 넣을 수 있습니다.
    "ids": [
      1002324883,
      1002793763,
      2265658394,
      2299323502
    ],
    "displayType": "single" | "list"
  },
  "user": "zejoVy3F9c98gvc-v6PFlQ"
}
```
## 챗봇에서 구성 가능한 커스텀 버튼
```json
{
  "event": "product",
  "options": {
    "ids": [
      2000344432,
      2000344433,
      2000344434,
      2000344437
    ],
    "displayType": "single",
    // 퀵 리플라이 버튼은 메시지별 구성이 불가능하고, 대화의 제일 아래 쪽에 위치합니다. 기존 퀵 리플라이 버튼과 동일한 기능입니다.
    "quickReply": {
      "buttonList": [
        {
          "data": {
            "title": "퀵 다이알로그",
            "code": "DIALOGUE"
          },
          "type": "TEXT"
        },
        {
          "data": {
            "title": "퀵 베이직",
            "code": "BASIC"
          },
          "type": "TEXT"
        }
      ]
    },
    // 커스텀 버튼은 특정 상품 컴포짓에 덧붙일 수 있습니다.
    "customButtonList": [
      {
        "id": 2000344432,
        "buttonList": [
          {
            "type": "TEXT",
            "data": {
              "title": "텍스트형 버튼",
              "code": "code1"
            }
          }
        ]
      },
      {
        "id": 2000344433,
        "buttonList": [
          {
            "type": "TEXT",
            "data": {
              "title": "텍스트형 버튼",
              "code": "code2"
            }
          }
        ]
      }
    ]
  },
  "user": "zejoVy3F9c98gvc-v6PFlQ"
}
```
