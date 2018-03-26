# **CALENDAR 컴포넌트**

## 개요
* `CALENDAR` 컴포넌트는 단일 날짜를 입력하는 UI 컴포넌트입니다. 사용자가 간편하게 날짜을 입력할 수 있으며, 챗봇에게는 데이터가 일정한 형식으로 전달되므로 처리가 수월해집니다.
* 날짜 선택 가능 기간을 별도로 설정하고자 할 때에는 `options -> calendar -> start/end/disables`를 사용할 수 있습니다.
  * 설정하지 않으면 달력 전체가 선택 가능합니다.
  * 선택이 불가능한 날짜는 회색으로 표시되며 선택이 되지 않습니다.
  * 선택 가능 기간을 설정하면 해당 기간 이외의 달력으로는 이동이 불가능합니다.
* 아래 이벤트 명세는 [`button`](/README.md#button-object) 요소에서의 컴포넌트 작성 방법을 설명 합니다. `button`이 사용 가능한 위치는 [UI 컴포넌트](/ui_component_v1.md)를 참고해주세요. 

## 이벤트 명세서
```javascript
{
  "type": "CALENDAR",
  "data": {
    "title": "방문 날짜 선택하기",
    "code": "code_for_your_bot",
    "options": {
      "calendar": {
        "placeholder": "방문 날짜를 선택해주세요.",
        "start": "20180301",
        "end": "20180430",        
        "disables": "1,20180309,20180315-20180316"
      }
    }
  }
}
```

| key | Type | 필수 | 설명 |
|:---:|:----:|:----:|------|
| `type` | string | Y | `CALENDAR` 고정값 |
| `title` | string | N | 버튼과 컴포넌트 타이틀에 노출되는 텍스트, 기본값은 '날짜 선택하기'|
| `code` | string | N | 컴포넌트의 확인 버튼 클릭시 챗봇에게 전송되는 코드 |
| `options` | object | N | 컴포넌트의 추가 옵션 정보, 사용시 key는 `calendar` |
| `placeholder` | string | N | 날짜 표시란에 초기에 보여지는 텍스트, 기본값은 '날짜를 선택해주세요.' |
| `start` | string | N | 선택 가능 기간 설정시 시작 날짜값 (YYYYMMDD) |
| `end` | string | N | 선택 가능 기간 설정시 종료 날짜값 (YYYYMMDD) |
| `disables` | string | N | 선택 가능 기간내에서 불가능한 날을 지정. 요일(0\~6까지의 숫자,일요일\~토요일에 해당함), 단일날짜(YYYYMMDD), 기간(YYYYMMDD-YYYYMMDD)을 콤마(,)로 연결, 복수개 나열 가능 |
| `isRange` | boolean | N | 연속 날짜 선택 여부, 기본값은 false |

> **[주의사항]**
> * `title`의 최대 길이는 18자 입니다.
> * `code`의 최대 길이는 1,000자 입니다.
> * 선택 가능 기간을 설정하실 때에는 `start`,`end` 두 값이 필수입니다. 하나만 입력시에는 무시됩니다. 
> * 연속 날짜 선택 설정시(isRange : true) 선택 불가능한 날짜(disables)를 연속 날짜 안에 포함하여 선택하면, 경고문을 보여주고 선택이 해제 됩니다.
<br>

### 1) 단일 날짜
#### `compositeContent -> compositeList -> buttonList[] -> button`에 적용한 예시
```javascript
{
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "compositeContent": {
    "compositeList": [
      {
        "title": "톡톡 레스토랑",
        "description": "파스타가 맛있는집",
        "image": {
          "imageUrl": "https://search.pstatic.net/common/?autoRotate=true&quality=95&src=http%3A%2F%2Fldb.phinf.naver.net%2F20171212_39%2F1513070642332Sre4X_JPEG%2FlLWrszsMNIW4RLx5R_or39IB.JPG.jpg&type=m1000_692"
        },
        "buttonList": [
          {
            "type": "CALENDAR",
            "data": {
              "title": "방문 날짜 선택하기",
              "code": "code_for_your_bot",
              "options": {
                "calendar": {
                  "placeholder": "방문 날짜를 선택해주세요.",
                  "start": "20180301",
                  "end": "20180430",
                  "disables": "1,20180309,20180315-20180316"
                }
              }
            }
          }
        ]
      }
    ]
  }
}
```
 * 위 이벤트 전송시 사용자 화면
 
![image](/images/calendar-component-chat.PNG)

 * button 클릭시 열리는 view
 
[Mobile - webview]

![image](/images/calendar-component.PNG)

[PC - popup] - 모바일과 동일한 화면이 팝업으로 열립니다.

 * 컴포넌트에서 3월 20일을 선택하고 `확인` 클릭시
   * 사용자 화면에는 `2018. 3. 20.(화)` 텍스트가 노출됩니다.
   * 챗봇에 전송 되는 데이터는 아래와 같습니다.
 ```javascript
 {
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "textContent": {
    "text": "20180320",
    "code": "code_for_your_bot",
    "inputType": "calendar"
  }
}
```

### 2) 연속 날짜
#### `compositeContent -> compositeList -> buttonList[] -> button`에 적용한 예시
```javascript
{
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "compositeContent": {
    "compositeList": [
      {
        "title": "톡톡 펜션",
        "description": "스위트 디럭스",
        "image": {
          "imageUrl": "https://ssl.phinf.net/naverbooking/20180309_45/1520573251611uJEvu_JPEG/fd120edc23805aa842b2a228596595cb.jpg?type=w1500"
        },
        "buttonList": [
          {
            "type": "CALENDAR",
            "data": {
              "title": "펜션 예약하기",
              "code": "code_for_your_bot",
              "options": {
                "calendar": {
                  "placeholder": "입실/퇴실일을 선택하세요.",
                  "start": "20180301",
                  "end": "20180430",
                  "disables": "1,20180309,20180315-20180316",
                  "isRange" : true
                }
              }
            }
          }
        ]
      }
    ]
  }
}
```
 * 위 이벤트 전송시 사용자 화면
 
![image](/images/calendar-component2-chat.png)

 * button 클릭시 열리는 view
 
[Mobile - webview] (날짜 선택 후)

![image](/images/calendar-component2.png)

[PC - popup] - 모바일과 동일한 화면이 팝업으로 열립니다.

 * 컴포넌트에서 3월 29일 ~ 4월 1일을 선택하고 `확인` 클릭시
   * 사용자 화면에는 `2018. 3. 29.(목) ~ 2018. 4. 1(일)` 텍스트가 노출됩니다.
   * 챗봇에 전송 되는 데이터는 아래와 같습니다.
 ```javascript
 {
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "textContent": {
    "text": "20180329-20180401",
    "code": "code_for_your_bot",
    "inputType": "calendar"
  }
}
```
