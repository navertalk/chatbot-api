# **UI 컴포넌트**
* UI 컴포넌트는 사용자에게 날짜, 시간 등을 간편하게 입력하기 위해 제공되는 도구입니다.
* mobile에서는 webview로, pc에서는 popup으로 열립니다.
* `send` 이벤트의 여러 `content` 중 [`button`](/README.md#button-object)요소에는 모두 적용할 수 있습니다.
* `button` 요소 사용이 가능한 부분은 아래와 같습니다. 
  * `compositeContent -> compositeList -> elementList -> button`
  * `compositeContent -> compositeList -> buttonList[] -> button`
  * `compositeContent -> compositeList -> buttonList[] -> button("type": "OPTION") -> buttonList[] -> button`
  * `compositeContent/textContent/imageContent -> quickReply -> buttonList[] -> button`

