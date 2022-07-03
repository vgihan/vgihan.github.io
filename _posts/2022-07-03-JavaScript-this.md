---
layout: post
title: JavaScript's this
author: vgihan
description: study JavaScript this
tags: [JavaScript, this]
categories: [JavaScript]
published: true
extensions:
  preset: gfm
---

js로 프로그래밍 할 때, this를 사용함에 있어서 큰 혼란을 겪었다. 같은 객체 안에서 this 키워드를 통해 내부의 메서드를 호출하거나 프로퍼티를 사용하려고 할 때, 의도한 대로 this가 결정되지 않아 어려움을 겪었고 이에 따라 필자는 this가 결정되는 원리부터 상황에 따른 this의 결정에 대해 정리하고자 한다.
JavaScript의 this는 함수 호출 시 함수가 어떻게 호출 되었는지 즉 함수 호출 패턴에 따라 값이 결정된다. 함수 호출 패턴은 여러가지 상황이 있을 수 있기 때문에 각 상황 별로 this가 어디에 바인딩 되는지 알아보고, 어떻게 해결 해야 하는지 살펴보자.

### 자바의 this vs 자바스크립트의 this

- **Java**

자바에서의 this는 객체 자신의 참조 값을 갖는다. 아래 예시를 보자.

```java
public class Person {
    private String name;
    public Person(name) {
    	this.name = name;
    }
}
```

자바에서 this는 Person으로 생성된 객체를 가리키기 때문에 주로 생성자에서 필드 변수와 파라미터를 구분하기 위해 쓰였다.

- **JavaScript**

자바스크립트의 경우 this는 자바와 달리 실행 시간에 결정되는데, 이로 인해 자바에서 든 예시의 Person 클래스로 생성된 객체 내부에서 작성한 this 키워드라도 Person 객체를 가리키지 않는 상황이 일어날 수 있다.

### this가 결정되는 내부 구조

- **실행 컨텍스트 (Execution Context)**

자바스크립트의 핵심 원리인 실행 컨텍스트 내부에서 this의 정보를 담고 있다. 함수를 호출할 때 하나의 실행 컨텍스트가 만들어져 스택에 쌓이게 된다. 실행 컨텍스트에는 변수와 함수를 담고 있는 객체를 가리키고 있는 Variable Object와 상위 스코프의 정보를 담고 있는 Scope Chain, 그리고 this의 정보를 담고 있다.

![](https://images.velog.io/images/vgihan/post/aa5bdeef-e652-4cdc-8c79-18a61bae42e0/image.png "sadfklsf")

- **함수 호출 시 실행 컨텍스트의 생성 과정**

순서는 아래와 같고, 함수가 호출되어 하나의 실행 컨텍스트가 스택에 쌓이게 될 때 this가 한 번 결정되는 구조이다.

> 1. 스코프 체인 연결
> 2. VO 연결
> 3. 함수 실행 패턴에 따른 this 결정

### 함수 실행 패턴

this는 함수가 호출됐을 때마다 스택에 실행 컨텍스트가 쌓이고, 실행 컨텍스트마다 this가 결정되기 때문에 함수 호출 패턴에 따라 어디에 this가 바인딩 될지 결정된다. 그렇다면, 함수 호출 패턴에는 어떤 것이 있을까? 함수 호출 패턴은 아래와 같이 크게 4가지로 나뉜다고 한다.

> 1. 함수 호출
> 2. 메소드 호출
> 3. 생성자 함수 호출
> 4. apply/call/bind 호출

#### 함수 호출

기본적인 함수의 호출은 전역 객체를 가리키고 있다. 여기서 전역 객체란, 프로그램이 실행된 후 가장 먼저 EC 스택에 쌓이게 되는 Global Execution Context의 VO가 가리키고 있는 Global Object를 의미한다. `function fooName() {}` 와 같이 선언된 함수들은 기본적인 함수 호출에 해당하는데, 이는 일반적인 전역 스코프에 선언된 함수 뿐만 아니라 내부 함수, 콜백 함수로 전달된 함수 호출도 이에 해당한다. 아래 예시를 보자.

```javascript
function test() {
  console.log(this); // 전역 객체
}
```

=> 전역 스코프에서 선언된 함수 호출이므로 전역 객체를 가리킨다.

```javascript
const obj = {
  name: "gihan",
  foo: function () {
    console.log(this); // obj 객체
    function inner() {
      console.log(this); // 전역 객체
    }
  },
};
```

=> 이후 설명되는 메서드 호출에서는 자신을 메서드로 갖고 있는 객체를 가리키는데, 메서드 내부에서 호출한 내부 함수의 this 마저도 전역 객체를 가리킨다.

```javascript
setTimeout(function () {
  console.log(this); // 전역 객체
}, 100);
```

=> 콜백 함수 역시 마찬가지로 전역 객체가 바인딩된다.

#### 메서드 호출

함수가 객체의 프로퍼티로 존재한다면, 이는 메서드라고 불리며 this는 해당 객체를 가리키게 된다.

```javascript
const person = {
  name: "gihan",
  foo: function () {
    console.log(this); // person 객체
  },
};
```

#### 생성자 함수 호출

생성자 함수를 알아보기 전에, new 키워드에 대해 알 필요가 있다. 자바와 같은 다른 객체 지향 언어에서 new 키워드는 객체를 동적으로 만들어내는 역할을 수행한다. 자바스크립트에서 new 키워드는, 함수를 생성자 함수로써 동작하도록 하여 함수 호출 시 새로운 객체를 만들어 낸다.

```javascript
const a = test(); // undefined
const b = new test(); // {}
```

자바스크립트에서 함수는 return value를 명시하지 않을 시 undefined를 리턴한다. 하지만, 생성자 함수로써 new 키워드 이후에 함수 호출 시 해당 함수는 undefined 가 아닌 객체를 리턴하는데, 어떤 객체를 리턴하는지는 생성자 함수 동작 방식의 순서로 알아볼 수 있다.
(생성자 함수 호출은 함수 호출에 앞서 new 키워드가 필수적으로 동반된다)

**1. 빈 객체 생성 및 this 바인딩**

> 생성자 함수 호출 이후, 함수 내부에서 빈 객체를 생성하고 this를 해당 객체에 바인딩한다.

**2. this를 통한 프로퍼티, 메서드 정의**

> 생성자 함수 내부에서 this 키워드를 통해 해당 객체의 프로퍼티와 메서드를 정의할 수 있다.

**3. 객체 return**

> 프로퍼티와 메서드를 정의한 후, 별도의 return value를 명시하지 않으면 프로퍼티와 메서드를 정의한 해당 객체를 반환한다. 하지만, return value를 명시하면 해당 객체를 반환하지 않는다. 따라서 생성자 함수의 용도로 사용할 함수는 return value를 명시해선 안된다.

다음은 생성자 함수를 통한 객체의 생성과 this 바인딩의 예시이다.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
  this.introduce = function () {
    console.log(this);
  };
}
const gihan = new Person("gihan", 23);
gihan.introduce();
// result : Person {name: 'gihan', age: 23, introduce: ƒ}
```

#### apply/call/bind 호출

this는 함수 호출 패턴에 따라 결정된다고 했는데, 앞서 살펴본 일반 함수, 메서드, 생성자 함수 호출은 자바스크립트에서 자동으로 this를 바인딩 해주는 방식이다. 이번에 살펴볼 apply/call/bind 함수는 개발자가 직접 this를 바인딩하여 사용할 수 있다. 아래는 예시로 사용할 test 함수와 obj 객체이다.

```javascript
function test(param1, param2) {
  console.log(param1, param2);
  console.log(this);
}
const apple = {
  name: "apple",
};
const banana = {
  name: "banana",
};
```

**- apply**
첫 번째 인자로 this를 바인딩 할 객체를, 두 번째 인자로 실제 함수의 인자들을 묶어 만든 배열을 넣어준다. 즉, apply 함수는 호출 시 this를 지정한 객체에 바인딩하여 즉시 호출하는 함수이다.

```javascript
test.apply(apple, ["a", "b"]); // a b {name: 'apple'}
test.apply(banana, ["a", "b"]); // a b {name: 'banana'}
```

**- call**
apply와 같은 역할을 수행하지만, 실제 함수의 인자를 배열이 아닌 각각 하나의 인자로 넘길 수 있다.

```javascript
test.call(apple, "a", "b"); // a b {name: 'apple'}
test.call(banana, "a", "b"); // a b {name: 'banana'}
```

**- bind**
인자로 this를 바인딩 할 객체를 넣는다. bind의 리턴 값은 this를 바인딩 시킨 함수 자체를 반환하기 때문에, 바인딩 한 함수를 저장시켜놓고 다시 호출하는 코드가 필요하다. 커링 방식으로도 동작할 수 있다.

```javascript
const appleCall = test.bind(apple);
const BananaCall = test.bind(banana);
appleCall("a", "b"); // a b {name: 'apple'}
BananaCall("a", "b"); // a b {name: 'banana'}
```

### [4] 화살표 함수(Arrow Function)의 this 바인딩

일반적으로 this는 함수 호출 패턴에 의해 실행 시간에 동적으로 결정되지만, 화살표 함수의 경우 this 바인딩이 조금 특이하다. 화살표 함수에서는 this를 무조건 상위 정적 스코프에 바인딩한다. 이를 Lexical this라고 한다. 아래 예시를 보자.

```javascript
const obj = {
  name: "gihan",
  normal: function () {
    console.log(this);
  },
  arrow: () => {
    console.log(this);
  },
};
obj.normal(); // {name: 'gihan', normal: ƒ, arrow: ƒ}
obj.arrow(); // Window {0: global, 1: global, 2: Window, 3: global, window: Window, …}
```

arrow 메서드는 메서드 호출임에도 불구하고 window 전역 객체가 출력된 것을 확인할 수 있다. 이는 동적으로 this가 결정되는 것이 아니라 항상 상위 스코프에서 this가 정적으로 결정되기 때문이다.
