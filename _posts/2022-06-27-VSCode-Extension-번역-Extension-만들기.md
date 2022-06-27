---
layout: post
title: VSCode Extension - 번역 extension 만들기
author: vgihan
description: make vscode translate extension
tags: [VSCode, Extension, translate]
categories: [VSCode]
published: true
extensions:
  preset: gfm
---

이전 시간에 vscode extension을 사용하는 방법에 대해 알아봤다. 이번 시간에는 파일의 텍스트를 드래그 하고 extension을 실행하면, 해당 문자를 번역 API를 이용하여 infomation message로 출력하는 extension을 만들어보자.

### 필요한 것

- extension을 실행하기 위한 방법 정의. 이전 시간에는 `Ctrl + Shift + P` 를 누르고 Hello World 를 입력하여 extension을 실행시켰음.
  - 이번 시간에는 단축키를 누른다던지 하는 이벤트를 통해 실행시키도록 해보자.
- cursor로 드래그한 내용을 vscode api로 text를 얻어올 수 있어야 함.
  - 보통 커서로 선택한 부분을 css에서는 selection이라고 표현하는데, 이것과 관련해서 찾아봐야할 것.
- vscode api로 text를 얻어왔다면, 번역하기위해 외부 Open API를 이용해야할 것.
  - open API를 사용할 수 있도록 설정해야함.
- vscode api 내부에서 외부 API 요청을 보낼 수 있는 방법을 찾아야 함. 기본적으로 fetch API가 동작할 것 같긴한데 찾아봐야할 것.
- CORS를 허용하지 않는다면, API를 Proxy 해줄 서버가 필요할 것. express로 간단히 진행하도록 한다.

### Activation Events

Extension을 실행 (Activate) 하기 위한 Event 이다. 여러가지 vscode에서 제공하는 이벤트들이 존재하는데, 해당 이벤트가 trigger 되면, extension이 활성화된다. 이벤트의 등록은 package.json의 activationEvents 프로퍼티에 등록할 수 있다.

- `onLanguage`

지정된 언어의 파일을 open 하면 extension이 활성화된다. 언어는 여러가지를 등록할 수 있고, 아래와 같이 package.json에 등록할 수 있다.

```json
"activationEvents": [
    "onLanguage:json",
    "onLanguage:markdown",
    "onLanguage:typescript"
]
```

⇒ JSON, md, ts 파일을 open 시 자동으로 extension이 활성화 된다.

- `onCommand`

`Ctrl + Shift + P` 를 입력하고 onCommand에 지정된 명령어를 입력하면 extension이 활성화된다. 이전 시간에 했던 Hello World 입력 시 extension이 활성화되던 코드이다. 아래와 같이 contributes 부분에 commands를 추가하고 commands와 onCommand 핸들러를 연결해주어야 한다.

```json
"activationEvents": [
    "onCommand:helloworld.helloWorld"
]
"contributes": {
  "commands": [
    {
      "command": "helloworld.helloWorld",
      "title": "Hello World"
    }
  ],
},
```

⇒ Hello World 입력 시 extension이 활성화된다.

- `*`

vscode를 키면 알아서 자동으로 extension이 시작된다. 이번 시간에 사용할 Activation Events 이다.

### VSCode Window Event

vscode 창에서 발생하는 event 들에 대해 핸들러를 연결해줄 수 있다. Text 드래그 시 해당 Text를 번역해주는 기능이 필요하므로, window에서 텍스트 드래그 이벤트를 찾아봤다. 그와 적절한 이벤트는 아래와 같이 `DidChangeTextEditorSelection`이다.

```jsx
vscode.window.onDidChangeTextEditorSelection((e) => {
  e.textEditor.document.getText();
});
```

event 객체에 text를 얻을 수 있는 함수가 존재해서 출력해보았다. 결과는 아래와 같다.

![text_1](https://user-images.githubusercontent.com/49841765/175907522-f660cb03-f25c-43dc-a722-1c090469a23b.gif)

문제가 세 가지 존재한다.

- 문제 1: 드래그 할 때 이벤트가 활성화 되는 것 뿐만 아니라, 텍스트를 입력할 때도 활성화된다.
- 문제 2: selection된 text가 아닌, 전체 텍스트 내용을 보여준다.
- 문제 3: 이벤트가 너무 빨리 발생한다.

문제 3은 디바운스를 적용하면 되고, 문제 1과 2를 해결해보자. 문제 1,2를 해결한 코드는 다음과 같다.

```tsx
vscode.window.onDidChangeTextEditorSelection((e) => {
  if (e.kind !== vscode.TextEditorSelectionChangeKind.Mouse) {
    return;
  }
  const start = e.selections[0].start;
  const end = e.selections[0].end;
  if (start.isEqual(end)) {
    return;
  }
  const text = e.textEditor.document.getText(new vscode.Range(start, end));
  vscode.window.showInformationMessage(text);
});
```

알고보니 `DidChangeTextEditorSelection` 이벤트에는 종류가 세 가지 존재했다.

- 첫째는 `Keyboard` 로, 키보드 입력 시 발생하고,
- 둘째는 `Mouse` 로, 마우스 클릭이나 드래그와 같은 마우스 이벤트 시 발생하고,
- 셋째는 `Command` 로, 명령어 입력시 발생하는 이벤트가 존재한다.

우리는 selection 시 이벤트가 활성화 되기를 원하기 때문에, event 종류가 Mouse인 이벤트가 아니면 핸들러를 return 하도록 했다.

또한, 마우스를 드래그해서 selection 되도록 하는 이벤트가 아닌, 일반적인 마우스 클릭에도 핸들러가 동작했기 때문에, selection의 start 지점과 end 지점이 일치하면 역시 return 하도록 했다.

그리고 getText 함수의 인자로 출력할 Text의 범위를 지정해줄 수 있었는데, vscode API의 Range를 인자로 전달하여 해당 기능을 동작하도록 만들 수 있었다. 결과는 아래와 같다.

![text_2](https://user-images.githubusercontent.com/49841765/175907531-d6c247e6-7878-4408-a171-6e2a1c3772bc.gif)

텍스트 입력 시 출력하는 것이 아니고, selection 시 선택한 부분의 Text만 출력되는 것을 확인할 수 있다.

### 번역

selection Text도 얻어왔으니 이제 진짜 번역 기능을 만들 차례이다. 번역 API를 이용하여 만들지만, API를 얻어오는 작업은 VSCode Extension 만들기라는 주제에서 벗어나므로 생략하겠다. API는 카카오의 Translate API를 활용하였다.

**fetch**

API 요청을 위해서 브라우저에서는 fetch API를 이용하여 API 요청 기능을 만든다. VSCode API에서도 시도한 결과, fetch API를 사용하지 못했다. 구글링을 통해 알아낸 결과, node-fetch 라는 라이브러리를 통해 외부 API 요청을 구현할 수 있다고 한다. `npm install node-fetch@2` 명령을 통해 node-fetch 라이브러리를 추가해주자.

**code**

아래와 같이 API 코드를 작성해준다. 물론, 사용하는 API에 따라 API 요청 함수는 다르게 작성될 수 있다. node-fetch 라이브러리를 import하여 fetch 함수를 사용할 수 있도록 만들어주자.

```jsx
import fetch from "node-fetch";

const translate = async (word: string, src: string, target: string) => {
  try {
    const res = await fetch(
      `https://dapi.kakao.com/v2/translation/translate?query=${word}&src_lang=${src}&target_lang=${target}`,
      {
        headers: {
          Authorization: `KakaoAK ${API_KEY}`,
        },
      }
    );
    return res.json();
  } catch (e) {
    console.error(e);
  }
};
```

**결과**

selection 시 오른쪽에 번역된 내용이 Infomation Message 로 출력되는 모습이다. 하지만, 드래그 시 이벤트가 연속적으로 빠르게 발생한다. 디바운스를 적용해보자.

![text_3](https://user-images.githubusercontent.com/49841765/175907536-eda49c0f-ee9c-47c5-951d-8f9f5364cc64.gif)

아래와 같이 디바운스 함수를 작성하여 적용했다. 핸들러가 호출되고 500ms 이내에 다시 한 번 핸들러가 호출된다면, 이전에 등록했던 핸들러를 취소하고 가장 최근에 등록한 핸들러를 실행하도록 한다.

```jsx
let timer: NodeJS.Timeout;

const debounce =
  (callback: Function) =>
  (...arg: any[]) => {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      callback(...arg);
    }, 500);
  };

export default debounce;
```

디바운스 적용 후에는 아래와 같이 한 번만 출력되는 것을 확인할 수 있다.

![text_4](https://user-images.githubusercontent.com/49841765/175907539-e9a4e7bb-a393-4bec-9ad3-f1c5ad6d3ddd.gif)

### 결론

매우 간단한 VSCode Extension을 만들어보았다. 카카오 API는 이미 다른 앱에 적용해놓은 것이 있어서 시간이 많이 걸리지 않아 30분 ~ 1시간 정도 걸려서 만들 수 있었던 것 같다. 영어로 된 주석 코드를 읽는데 이용해볼 수 있을 것 같다.
