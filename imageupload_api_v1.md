# **Image Upload API** V1

## 개요
`Image Upload API`는 [`Chat Bot API`](/README.md)의 확장 API로서, 자주 사용되는 이미지를 미리 업로드합니다. 일반적으로 `imageContent`나 `compositeContent`에서 사용하는 `imageUrl`은 매번 이미지를 업로드하므로 사용자에게 **즉각적으로** 메시지를 전달할 수 없습니다. 그러나 이미지를 미리 업로드하고 `imageId`를 이용해 메시지를 전달하면 텍스트 메시지와 같은 속도로 메시지가 전달됩니다.
특히, `compositeContent`의 `캐로셀(Carousel)` 방식으로 이미지를 여러 개 사용하는 경우 속도 차이는 더 극명합니다.
<br>

## 이벤트 명세

### 이미지 업로드 요청 `Request` 명세

주어진 URL로 톡톡 서버가 접근해서 이미지를 다운로드할 때 HTTP 응답 헤더의 Content-Type값은 반드시 해당 이미지 유형과 일치해야 합니다.

```javascript
POST /chatbot/v1/imageUpload HTTP/1.1
Host: gw.talk.naver.com
Content-Type: application/json;charset=UTF-8
Authorization: ct_wc8b1i_Pb1AXDQ0RZWuCccpzdNL

{
    "imageUrl": "http://shop1.phinf.naver.net/20170216_20/talktalk_14872437839327BN4b_PNG/menu_01.png"
}
```

<br>

### 이미지 업로드 요청 `Response` 명세
[성공]
```javascript
HTTP/1.1 200 OK

{
    "success": true,
    "resultCode": "00",
    "imageId": "9vNeSIoFKkIHYCFZVveXr-TAm0-x5UifM3kAxs7EaXFWZmxj2U6yb_g9BZUFQQrX1Pf11UgsKdhANmsH2subzi2sQzeMKEJDfUd1jwmvuWNJ_C_PqeN8t6q7PeO1CzKh"
}
```

[실패]
```javascript
HTTP/1.1 200 OK

{
    "success": false,
    "resultCode": "IMG-99",
    "resultMessage": "이미지 업로드 중 에러"
}
```
<br>
<br>

## `imageId` 사용 방법

다음과 같이 `imageUrl` 속성 대신 `imageId` 속성을 사용합니다.
<br>

### `imageContent` 예제
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 사용자 식별값 */
    "imageContent": {
        "imageId": "9vNeSIoFKkIHYCFZVveXr-TAm0-x5UifM3kAxs7EaXFWZmxj2U6yb_g9BZUFQQrX1Pf11UgsKdhANmsH2subzi2sQzeMKEJDfUd1jwmvuWNJ_C_PqeN8t6q7PeO1CzKh" /* 전송하고자 하는 이미지 Id */
    }
}
```
