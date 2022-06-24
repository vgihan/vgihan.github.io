---
layout: post
title: VSCode Extension 만들기
author: vgihan
description: study Cross Origin Resource Sharing
tags: [vscode, extension, frontend]
categories: [extension]
published: true
extensions:
  preset: gfm
---

vscode에 다양하고 신기한 앱들이 많은 걸보고 나도 만들어보고 싶단 생각이 들 찰나, 오픈소스 컨트리뷰톤에 vscode extension 개발을 주제로 하는 팀이 있어서 신청했다. 떨어질지도 모르지만 그냥 재밌을 것 같으니 Hello World 앱부터 따라 만들어본다 !

**참고 링크**

[Your First Extension](https://code.visualstudio.com/api/get-started/your-first-extension)

### install

yeoman과 vscode extension generator를 설치한다. vscode extension generator가 실질적인 역할을 수행하는 것 같고, yeoman은 빌드를 도와주는 소프트웨어 인듯하다.

```python
npm install -g yo generator-code
```

### Initial Project

아래 명령을 실행해서 프로젝트의 기본 틀을 만들어준다. 명령을 실행하면 몇 가지 질문 창이 나오는 데, 다음과 같이 입력해준다.

```python
yo code
```

```python
# ? What type of extension do you want to create? New Extension (TypeScript)
# ? What's the name of your extension? HelloWorld
### Press <Enter> to choose default for all options below ###

# ? What's the identifier of your extension? helloworld
# ? What's the description of your extension? LEAVE BLANK
# ? Initialize a git repository? Yes
# ? Bundle the source code with webpack? No
# ? Which package manager to use? npm

# ? Do you want to open the new folder with Visual Studio Code? Open with `code`
```

### Debug

프로젝트가 존재하는 vscode Window에서 `F5` 키를 누르면 컴파일과 함께 디버깅 모드로 전환되면서 새로운 Window가 하나 열리게 된다. 해당 Window에서 `Ctrl+Shift+P` 를 누르면 Command Palette 라고 하는 검색창 비슷하게 Window 상단에 열리는 데, 여기에 Hello World를 입력하면 다음과 같은 결과가 나온다.

![image](https://user-images.githubusercontent.com/49841765/175525523-b556f220-ff9a-42f9-91ce-f21bb560643f.png)

하단에 ‘Hello World from helloworld!’ 라고 하는 문구를 포함한 작은 박스를 확인할 수 있다 !

### Develop

어떻게 위와 같은 결과가 나온 걸까? 프로젝트로 다시 돌아가보자.

**박스 내부 문구 수정하기 (vscode API)**

src/extension.ts 파일 내부에 들어가면, 아래와 같은 코드를 확인할 수 있다.

```jsx
vscode.window.showInformationMessage("Hello World from helloworld!");
```

이 문구를 수정하면, 위 박스에 포함되는 문구가 달라진다.

![image](https://user-images.githubusercontent.com/49841765/175525651-e88d5547-ae8a-4c16-b981-48e65452e245.png)

**extension 실행 커맨드 수정하기 (Activation Event)**

또한, Command Palette (`Ctrl+Shift+P` 누르면 열리는 검색 창)에서 해당 extension을 실행하기 위해 입력하는 커맨드도 수정할 수 있다. package.json에서 `contributes > commands > title` 을 확인하자.

```jsx
"activationEvents": [
      "onCommand:helloworld.helloWorld"
],
"contributes": {
	"commands": [
		{
			"command": "helloworld.helloWorld",
			"title": "Hello World"    // 이곳을 수정하면 된다 !
		}
	]
},
```

이러한 결과가 나오는 과정은 다음과 같다.

- package.json > activationEvents에 `onCommand:helloworld.helloworld` 를 추가하여 커맨드 입력 시 작업을 수행하도록 한다.

### Extension File Structure

extension 프로젝트의 구조는 다음과 같다. 실질적인 내용은 src > extension.ts에서 일어나는 것 같고, .vscode 폴더 내부에서는 빌드와 관련된 내용을 config 하는 모양이다.

vscode extension 프로젝트의 root에는 반드시 manifest 로써 package.json이 존재해야한다고 한다. manifest란 애플리케이션이 실행되는데 있어서 필수적으로 필요한 파일이라고 한다.

```jsx
├── .vscode
│   ├── launch.json     // Config for launching and debugging the extension
│   └── tasks.json      // Config for build task that compiles TypeScript
├── .gitignore          // Ignore build output and node_modules
├── README.md           // Readable description of your extension's functionality
├── src
│   └── extension.ts    // Extension source code
├── package.json        // Extension manifest
├── tsconfig.json       // TypeScript configuration
```

manifest의 필드에 대한 자세한 애용은 아래 링크로부터 확인할 수 있다.

[Extension Manifest](https://code.visualstudio.com/api/references/extension-manifest)

### Entry File (extension.ts)

Entry File에서는 activate, deactivate 두 가지 함수를 export 하고 있다. activate 함수는 package.json에 등록한 이벤트가 발생했을 때 실행되는 것이고, deactivate 함수는 extension이 deactivate 하기 전 실행되는 것으로 clean up 작업을 수행할 수 있다.

```jsx
import * as vscode from "vscode"; // vscode API를 제공한다

/* 등록한 이벤트 트리거 시 작동 */
export function activate(context: vscode.ExtensionContext) {
  console.log('Congratulations, your extension "helloworld" is now active!');

  // 이벤트를 등록할 수 있다.
  // helloworld.helloworld 라는 이벤트에 핸들러를 등록한 것.
  let disposable = vscode.commands.registerCommand(
    "helloworld.helloWorld",
    () => {
      vscode.window.showInformationMessage("Hi I'm vgihan !");
    }
  );

  context.subscriptions.push(disposable);
}

/* extension deactivate 시 작동 */
export function deactivate() {}
```

### 갖고 놀기

extension으로 여러가지 기능을 수행할 수 있을 것이다. 아무거나 사용해보면서 감각을 익히자 !

**window**

showInformationMessage 함수 말고도, window에서 사용할 수 있는 다른 API들이 많이 존재했다.

- `showWarningMessage`

```jsx
vscode.window.showWarningMessage("Oops! Warning Message ~ !");
```

⇒ Warning 메시지 박스를 화면에 나타낸다.

- `showInputBox`

```jsx
vscode.window.showInputBox({ title: "Hello !" });
```

⇒ 사용자의 Text Input을 받을 수 있는 창을 상단에 열어준다.

- `showOpenDialog`

```jsx
vscode.window.showOpenDialog();
```

⇒ File Open 작업을 수행할 수 있다.

- `createWebviewPanel`

```jsx
vscode.window.createWebviewPanel("view", "hello !");
```

⇒ 새로운 패널을 열어준다. git graph와 같은 extension이 해당 함수를 통해 사용자에게 그래프 view 를 제공하는 것 같다.

![image](https://user-images.githubusercontent.com/49841765/175525733-00608b29-4bc7-4c68-9e7f-d2743e104db3.png)

- `createTreeView`

```jsx
vscode.window.createTreeView("vgihan", {
  treeDataProvider: new TreeDataProvider(),
});
```

⇒ vscode.TreeDataProvider 인터페이스를 구현하여 사용자의 커스텀 TreeDataProvider를 정의할 수 있다.

⇒ 아래 stackoverflow 페이지의 코드를 실행해보았다.

⇒ [https://stackoverflow.com/questions/56534723/simple-example-to-implement-vs-code-treedataprovider-with-json-data](https://stackoverflow.com/questions/56534723/simple-example-to-implement-vs-code-treedataprovider-with-json-data)

![image](https://user-images.githubusercontent.com/49841765/175525761-4d1f39d1-fd8a-4892-80bb-1a73b1987e15.png)

---

📣 시간 나는대로 더 작성하겠습니당
