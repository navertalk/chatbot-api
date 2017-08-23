# **Image Upload API** V1

## 개요
* `Image Upload API`는 [`Chat Bot API`](/README.md)의 확장 API이며 자주 사용되는 이미지를 미리 업로드할 수 있습니다.
* 일반적으로 `imageContent` 나 `compositeContent`에서 사용하는 `imageUrl`은 매번 이미지를 업로드하기 때문에 유저에게 **즉각적인** 메시지 전달이 불가능 합니다.
* 그러나 이미지를 미리 업로드 후 `imageId`를 통해 메시지를 전달하는 경우 텍스트 메시지와 같은 속도로 메시지가 전달됩니다.
* 특히, `compositeContent`의 `캐로셀(Carousel)`방식으로 이미지를 여러개 사용하는경우 속도차이는 더 극명합니다.
<br>

## 이벤트 명세서
<br>

### 이미지 업로드 요청 `Request` 명세서
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

### 이미지 업로드 요청 `Response` 명세서
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


## `imageId` 사용방법
* 기존에 `imageUrl`속성 대신 `imageId`속성으로 대체해서 사용하면 됩니다.
<br>

### `imageContent` 예제
```javascript
{
    "event": "send",
    "user": "al-2eGuGr5WQOnco1_V-FQ", /* 유저 식별값 */
    "imageContent": {
        "imageId": "9vNeSIoFKkIHYCFZVveXr-TAm0-x5UifM3kAxs7EaXFWZmxj2U6yb_g9BZUFQQrX1Pf11UgsKdhANmsH2subzi2sQzeMKEJDfUd1jwmvuWNJ_C_PqeN8t6q7PeO1CzKh" /* 전송하고자하는 이미지 Id */
    }
}
```
