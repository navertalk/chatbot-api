# Handover API (BETA)

## :exclamation:공지
핸드오버 API는 비동기 방식으로 비정상 요청에도 `200(OK)`으로 응답하고 있었습니다.
2018년 6월 4일 (월요일)부터는 동기 방식으로 변경되고, 정상적이지 않은 요청에 대해 오류 코드와 사유를 담아서 응답할 예정입니다.
기존 시나리오에서 핸드오버 오류 발생시, 치명적인 피해가 발생하지 않도록 예외 처리를 추가하는 것을 권장합니다.

## 개요
하나의 계정에서 챗봇과 톡톡 파트너센터 상담원이 협업하여 응대할 수 있는 기능을 제공합니다. 
챗봇이 사용자의 메시지에 응답하지 못하는 경우 Handover API를 이용해 실제 상담원과 연결하는 방식으로 보다 나은 사용자 경험을 제공할 수 있습니다. 

## 핸드오버 기능 활성화 

핸드오버 기능을 활성화하려면 [네이버 톡톡 파트너센터](https://partner.talk.naver.com/)의 **개발자도구 > API 설정**에서 **핸드오버 API**를 **ON**으로 설정합니다.<br>
![composite_message](/images/handover-switch.png)

## 파트너센터와 연동해 챗봇과 파트너가 실시간으로 응대

 파트너센터에 연동된 챗봇이 파트너와 협업해 사용자의 메시지에 좀 더 인터랙티브(interactive)하게 응대할 수 있습니다. 기본적으로 챗봇이 응대를 하고 있다가 파트너의 응대가 필요하면 파트너에게 대화의 주도권을 넘깁니다(`passThread`).

 `핸드오버`를 켠 상태에서 챗봇이 응대하고 있는 대화는 **완료된 상담** 목록에 표시됩니다. 그러나 파트너에게 `passThread`를 하면 해당 대화는 **대기 상담** 목록에 표시되며, 파트너는 자신에게 전달된 상담을 바로 응대할 수 있게 됩니다. 파트너가 응대한다는 것은 최소 1개의 메시지를 보낸다는 것이며, 이때 사용자가 보내는 메시지의 `standby` 속성이 true이므로 챗봇이 응답하지 않도록 구현해야 합니다. 또한 해당 대화는 **진행중 상담** 목록에 표시됩니다.

 파트너가 응대를 하다가 **상담 완료하기** 버튼을 누르면 파트너로부터 챗봇에게 `passThread`됩니다. 그럼 대화는 다시 **완료된 상담** 목록에 표시됩니다. 이때는 사용자가 보내는 메시지에 `standby` 속성이 없으므로 챗봇이 응답하도록 구현해야 합니다. 챗봇과 상담원의 응대 및 핸드오버 상황에 따른 상담상태는 다음과 같습니다.
 
- 최초로 상담이 인입되어 챗봇이 응대중일 때 : [완료]
- 챗봇이 상담원에게 핸드오버 하면 : [대기]
- 상담원이 응대를 시작하면(메시지 발송) : [진행중]
- 상담원이 상담을 마친 뒤, 상담완료 버튼을 누르면 : [완료]

> **참고**
>
> `standby`에 관한 자세한 내용은 [챗봇의 메시지를 수신할 때 standby 속성](#챗봇의-메시지를-수신할-때-standby-속성)을 참고하십시오.


## echo 이벤트 활용

다음은 대화의 주도권의 변화(두 번째 이벤트)에 따라서 `echo` 이벤트 메시지가 어떻게 달라지는지 설명합니다.
다음의 세 가지 이벤트가 챗봇에게 전달됩니다. <br>
첫 번째 이벤트는 대화의 주도권이 파트너에게 있을 때 파트너가 입력하는 메시지를 챗봇에게 전달합니다.<br>
두 번째 이벤트에서는 파트너가 응대를 하다가 **상담 완료하기** 버튼을 눌러서 상담을 종료하면 대화의 주도권이 챗봇에게 넘어갑니다.<br>
마지막으로 세 번째 이벤트에서는 챗봇에게 대화의 주도권이 넘어왔기 때문에 챗봇이나 파트너가 입력하는 메시지에는 `echo` 이벤트에 있는 `options` 항목의 `threadOwnerId`값이 첫 번째 이벤트와 다르게 설정됩니다. <br>

```json
// 파트너가 대화의 주도권을 가질 땐 threadOwnerId가 파트너를 의미하는 1이며, 메시지의 주체인 sourceId도 1입니다.
{
  "echoedEvent": "send",
  "event": "echo",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "textContent": {
    "text": "하이",
    "inputType": "typing"
  },
  "options": {
    "mobile": false,
    "sourceId": 1,
    "threadOwnerId": 1,
    "managerNickname": "구매자"
  }
}

// 파트너가 상담을 완료하면 챗봇에 전달되는 핸드오버 데이터
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "options": {
    "control": "passThread",
    "metadata": "{'managerNickname':'구매자','autoEnd':false}"
  }
}

// 대화의 주도권이 챗봇에게 넘어간 뒤 파트너가 대화를 입력했을 때 threadOwnerId는 챗봇의 시퀀스이며, 메시지의 주체인 sourceId는 파트너인 1입니다.
{
  "echoedEvent": "send",
  "event": "echo",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "textContent": {
    "text": "하이",
    "inputType": "typing"
  },
  "options": {
    "mobile": false,
    "sourceId": 1,
    "threadOwnerId": 10007,
    "managerNickname": "구매자"
  }
}
```


## 챗봇의 메시지를 수신할 때 `standby` 속성

`standby`는 사용자가 보낸 메시지의 속성입니다. 대화의 주도권이 파트너에게 있으면 `true`이고, 챗봇에게 있으면 생략됩니다. 그러나 챗봇이 보낸 메시지를 사용자가 클릭했을 때는 대화의 주도권이 파트너에게 있더라도 `standby`가 생략됩니다. 따라서 대화의 주도권과 관계 없이 챗봇이 응대해야 하는 시나리오를 진행할 수 있습니다.<br> 

챗봇이 파트너에게 주도권을 전달할 때는 타깃(targetId)을 `1`로 설정해 다음과 같이 보냅니다.

```json
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "options": {
    "control": "passThread",
    "targetId": 1
  }
}
```

대화의 주도권이 파트너에게 있으므로 챗봇이 수신하는 사용자의 메시지에는 아래와 같이 `standby` 속성이 true로 되어 있습니다.
 ```json
{
  "standby": true,
  "event": "send",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc8b1i",
  "textContent": {
    "text": "헬로",
    "inputType": "typing"
  },
  "options": {
    "mobile": false
  }
}
```

다음은 파트너가 **상담 완료하기** 버튼을 누르면 챗봇에게 전송되는 메시지입니다.

```json
{
  "event": "handover",
  "user": "al-2eGuGr5WQOnco1_V-FQ",
  "partner": "wc1234",
  "options": {
    "control": "passThread",
    "metadata": "{\"managerNickname\":\"파트너닉네임\",\"autoEnd\":false}"
  }
}
```

## 챗봇에서 대화의 주도권 가져오기
 
대화의 주도권이 파트너에게 있더라도 챗봇이 특정 시점에 대화의 주도권을 빼앗아 올 수 있습니다. 이때 `takeThread`를 이용합니다. 예를 들어, 시나리오에 `상담원과 대화 종료하기`라는 의도의 버튼을 추가하여, 사용자가 원하는 시점에 상담원과의 대화를 종료하도록 만들 수 있습니다. 이때 사용자가 해당 버튼을 클릭하면 `takeThread`를 이용해 챗봇이 대화의 주도권을 가져오도록 기능을 구현하면 됩니다.<br>

챗봇이 원하는 시점에 파트너에게 있는 대화 주도권을 가져올 때는 아래와 같이 보냅니다.
```json
{
    "event": "handover",
    "user": "al-2eGuGr5WQOnco1_V-FQ",
    "partner": "wc1234",
    "options": {
        "control": "takeThread",
        "metadata": ""
    }
}
```
