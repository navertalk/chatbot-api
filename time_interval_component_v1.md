# **TIME-INTERVAL 컴포넌트**

## 개요
* `TIME-INTERVAL` 컴포넌트는 유저로부터 정해진 시간을 선택받아 전달하는 UI 컴포넌트입니다. 고정적인 시간대를 사용하는 사업자의 경우 유용합니다.
* 시작시간, 종료시간, 인터벌은 따로 설정하지 않으면 기본값으로 설정됩니다.
* 선택 불가능한 시간은 `options -> timeInterval-> disables`로 설정할 수 있습니다. 
* 아래 이벤트 명세는 [`button`](/README.md#button-object) 요소에서의 컴포넌트 작성 방법을 설명 합니다. `button`이 사용 가능한 위치는 [UI 컴포넌트](/ui_component_v1.md)를 참고해주세요. 

## 이벤트 명세서
```javascript
{
  "type": "TIMEINTERVAL",
  "data": {
    "title": "방문 시간 선택하기",
    "code": "code_for_your_bot",
    "options": {
      "timeInterval": {
        "start": "0900",
        "end": "2200",
        "interval": "15",
        "disables": "1000,1115-1130,1200,1400-1430"
      }
    }
  }
}
```

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | `TIMEINTERVAL` 고정값 |
| `title` | string | N | 버튼과 컴포넌트 타이틀에 노출되는 텍스트, 기본값은 '시간 선택하기'|
| `code` | string | N | 컴포넌트의 확인 버튼 클릭시 챗봇에게 전송되는 코드 |
| `options` | object | N | 컴포넌트의 추가 옵션 정보, 사용시 key는 `timeInterval` |
| `start` | string | N | 시작시간(HHMM), 5분 단위여야 하며 0000-2355 사이값만 가능, 기본값은 '0600'  |
| `end` | string | N | 종료시간(HHMM), 5분 단위여야 하며 0000-2355 사이값만 가능, 기본값은 '2355' |
| `interval` | string | N | 시간을 나누는 단위 분 값, 5분 단위여야 하며 5~720 사이값만 가능, 기본값은 '30' |
| `disables` | string | N | 시작-종료 시간내에서 선택 불가능한 시간을 지정. 단일시간(HHMM), 기간(HHMM-HHMM)을 콤마(,)로 연결, 복수개 나열 가능 |

> **[주의사항]**
> * `title`의 최대 길이는 18자 입니다.
> * `code`의 최대 길이는 1,000자 입니다. 
<br>

#### `compositeContent -> quickReply -> buttonList[] -> button`에 적용한 예시
```javascript
{
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "compositeContent": {
    "compositeList": [
      {
        "title": "톡톡 세탁소",
        "description": "회색은 방문이 불가능한 시간입니다.",
        "image": {
          "imageUrl": "https://post-phinf.pstatic.net/20160229_23/1456731493962khrE6_JPEG/%BF%CD%C0%CC%BC%C5%C3%F7.jpg_22.jpg?type=w1200"
        }
      }
    ],
    "quickReply": {
      "buttonList": [
        {
          "type": "TIMEINTERVAL",
          "data": {
            "title": "방문 시간 선택하기",
            "code": "code_for_your_bot",
            "options": {
              "timeInterval": {
                "start": "0900",
                "end": "2200",
                "interval": "15",
                "disables": "1000,1115-1130,1200,1400-1430"
              }
            }
          }
        }
      ]
    }
  }
}
```
 * 위 이벤트 전송시 사용자 화면

![image](/images/timeinterval-component-chat.png)

 * button 클릭시 열리는 view
 
[Mobile - webview]

![image](/images/timeinterval-component.png)

[PC - popup] - 모바일과 동일한 화면이 팝업으로 열립니다.

 * 컴포넌트에서 오전 > 9:30을 선택하고 `확인` 클릭시
   * 사용자 화면에는 `오전 9:30` 텍스트가 노출됩니다.
   * 챗봇에 전송 되는 데이터는 아래와 같습니다.
 ```javascript
 {
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "textContent": {
    "text": "09:30",
    "code": "code_for_your_bot",
    "inputType": "timeInterval"
  }
}
```
