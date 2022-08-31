---
layout: post
title: D3 Icicle Chart Transition
author: vgihan
description: D3 라이브러리를 활용하여 Icicle Chart에 transition 추가하기
tags: [d3, react, transition]
categories: [d3]
published: true
extensions:
  preset: gfm
---

# icicle chart transition

icicle chart 다음 목표는 transition을 넣는 것이다. 클릭한 블럭만 transition을 넣는 것이 아니라, 다른 블럭도 함께 움직이는 transition이라 생각하기가 쉽지 않다.

**Node vs Selection**

transition을 부여하기 위해선, Node와 Selection을 구별할 줄 알아야한다. Node는 hierarchy 함수를 통해 데이터를 갖는 객체이고, Selection은 해당 데이터를 통해 실질적으로 눈에 보여지는 element 들이다. 그렇기 때문에, transition은 Node가 아니라 Selection에 부여할 수 있다.

![Selection](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/68fb60e5-636a-4c2c-b2fa-0693e1bd86be/Untitled.png)

Selection

![Node](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd9480f5-1e9e-43f2-a944-6eef0c23e490/Untitled.png)

Node

**Selection의 하위 Selection 참조**

Node는 결국 Tree 자료구조이기 때문에 하위 Node를 참조하는 것이 가능하다. 하지만, Selection은 element 이기 때문에 하위 데이터를 물고 있는 Selection을 참조하는 것이 불가능하다. 실제로 children 이라는 프로퍼티를 참조해도 데이터 상 하위 Selection이 아니라 실제 element 내부의 다른 태그를 반환받게 된다. (svg > g > rect)

**그러면 어떡하지?**

githru에서 보여지는 icicle chart transition을 주기 위해선, click event의 target과 해당 Selection의 하위 Selection 들에 transition을 부여해야한다. 하지만, 앞서 언급했듯 그것은 불가능하기 때문에, 차트를 구성하는 모든 Selection에 transition을 부여함으로써 적용할 수 있다.

**HierarchyRectangularNode**

HierarchyRectangularNode는 차트를 구성하는 element의 각 좌표 값을 제공한다. 하지만, HierarchyRectangularNode의 값이 수정된다고 해서 바로 결과가 View에 반영되진 않는다. HierarchyRectangularNode의 데이터를 수정하고, transition과 attr를 통해 View에 반영하는 작업을 거쳐야 비로소 View의 변화가 일어난다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/84f8ef6b-a49e-4ed3-9e73-ba05c450c431/Untitled.png)

**가장 중요한 건 좌표 수정 !**

이번 transition을 적용하는 데 있어서 가장 중요한 건 결국 좌표 값 계산이다. zoom-in 시 결정되는 결과의 좌표값을 계산해서 transition만 적용시켜주면 되기 때문이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/423022b6-a17c-4fca-b1ca-01c77aa3448a/Untitled.png)

**좌표 수정 시 주의할 점**

아래 코드와 같이 좌표를 수정했더니, 문제가 발생했다. target은 event target이다. target이 수정되면, target 이후에 작업이 실행되는 Node들은 target의 수정된 값을 참조하는 것이다. 예상했던 결과값을 얻지 못했다.

```jsx
cell.each((d) => {
  d.x0 = d.x0 - target.x0;
  d.x1 = d.x1 - target.x0 + height;
  d.y0 = d.y0 - target.y0;
  d.y1 = 2;
});
```

아래와 같이 미리 target 값을 복사한 후 작업하니 제대로 동작했다.

```jsx
const { x0, x1, y0, y1 } = target;
cell.each((d) => {
  d.x0 = d.x0 - x0;
  d.x1 = d.x1 - x0 + height;
  d.y0 = d.y0 - y0;
  d.y1 = 2;
});
```

**width transition**

위에서 언급했듯, width transition에 관여하는 좌표 값은 y0, y1 이다. 적절한 계산을 통해 y0, y1을 정해주어야 한다. 기존에 y0, y1을 관리하던 방식은 왼쪽부터 height수에 전체 width를 곱한 수를 사용하고 있었다. (ex 0, 500, 1000, 1500 …) 이후에 실제 width를 정해줄 때 해당 값에 실제 비율을 적용하여 화면에 보여주는 방식이었다. 그러므로 click target의 y0와 비교하여 얼마나 차이가 있는지만 계산해주면 된다. 아래는 각 Node의 y0 값을 target의 y0와의 차로 대입하는 식이다.

```jsx
d.y0 = d.y0 - y0;
```

**height transition**

height 에 관여하는 좌표는 x0, x1 이다. 적절한 x0, x1을 계산하여 transform y, height를 구해보자. 아래 icicle chart 에서 `src/2` rect를 클릭한다고 가정하자. 그러면, `src/2` `transition/transition.js` `util/options.js` 3개의 rect가 svg를 꽉 채울 것이다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cee19eb5-df9d-441d-880a-d4b2ee4d2c7e/Untitled.png)

편의를 위해 `src/2` 를 1번 블록, `transition/transition.js` 를 2번 블록, `util/options.js` 를 3번 블록이라하자.

1번 블록의 높이는 transition 이후 svg의 height 만큼 증가한다. 이 비율을 기억해두자 (`transition 이전 1번 블록의 높이` : `svg 높이`)

`3번 블록의 x0 - 1번 블록의 x0` 값이 transition 이후에 변경되는 값과의 비율은 앞의 비율과 같다.

정리하자면,

`transition 이전 1번 블록의 높이` : `svg 높이` = `block3's x0 - block1's x0` : `transition 이후 3번 블록의 x0 => d.x0` 이고,

`target.x1 - target.x0` : `height` = `d.x0(이전) - target.x0` : `d.x0(이후)` ,

`d.x0(이후)` = (`d.x0(이전) - target.x0`) \* `height` / `target.x1 - target.x0` 이라는 결과가 도출된다.

```jsx
d.x0 = (d.x0 - x0) * (height / (x1 - x0));
```

다음은 `x1` 을 구할 차례다. `x0` 을 구할 때와 동일하게 1번 블록의 transition 이전과 이후 비율과 비교하여 구할 수 있다.

`transition 이전 1번 블록의 높이` : `svg 높이` = `block2's x1 - block1's x0` : `transition 이후 2번 블록의 x1 => d.x1` ,

`target.x1 - target.x0` : `height` = `d.x1(이전) - target.x0` : `d.x1(이후)` ,

`d.x1(이후)` = (`d.x1(이전) - target.x0`) \* `height` / `target.x1 - target.x0`

```jsx
d.x1 = (d.x1 - x0) * (height / (x1 - x0));
```

**반영하기**

위치 정보를 나타내는 translate는 아래와 같다. `target.height+1`에 따라서 svg의 width가 몇 개로 나눠지는 지 정해지기 때문에, svg의 width 단위로 정해두었던 y0의 값을 구해줄 수 있다.
