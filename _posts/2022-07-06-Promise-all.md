---
layout: post
title: Promise.all
author: vgihan
description: study Promise.all
tags: [JavaScript, promise, async]
categories: [JavaScript]
published: true
extensions:
  preset: gfm
---

자바스크립트에서 비동기 작업을 처리하는 방법으로는 분류하자면 세 가지가 존재한다. 첫 번째는 콜백함수를 호출하는 것이고, 두 번째는 Promise.then, 마지막은 문법적으로 처리할 수 있는 async await 키워드를 이용한 방법이다.

사실, async await 키워드를 가지고 비동기 작업을 처리하면 매우 간단하다. Promise를 반환하는 작업 앞에 await를 붙이고 해당 함수를 async function으로 만들어주면 되기 때문이다.

하지만, 아래와 같은 코드에서처럼 async await를 사용하게 되면 비효율적인 문제가 발생한다.

### Problem

a는 1초 후 1을 반환하는 비동기 함수이고, b는 2초 후 2를, c는 3초 후 3을 반환하는 비동기 함수이다. 비동기 처리를 위해 main 에서는 async await 키워드를 통해 a, b, c 함수의 실행 순서를 보장한다. 그렇기 때문에 총 실행 시간은 1000 + 2000 + 3000 = 6000ms 가 된다. a, b, c 는 서로를 필요로하지 않는 독립적인 값인데도 불구하고 앞의 결과를 기다려야 한다는 문제가 있다.

```jsx
const a = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(1);
    }, 1000);
  });
};
const b = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(2);
    }, 2000);
  });
};
const c = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(3);
    }, 3000);
  });
};
const main = async () => {
  const start = new Date();

  const _a = await a();
  const _b = await b();
  const _c = await c();

  console.log(_a);
  console.log(_b);
  console.log(_c);

  const end = new Date();

  console.log(end - start);
};

main();
```

### Solution

위와 같은 문제를 해결하려면 Promise.all 함수를 이용해야 한다. 해당 함수는 여러 개의 비동기 작업을 병렬적으로 (block 없이 한 번에 요청) 처리하기 때문에 async await를 통해 block 하는 것보다 더 빨리 결과 값을 얻을 수 있다.

```jsx
const main = async () => {
  const start = new Date();

  const [_a, _b, _c] = await Promise.all([a(), b(), c()]);

  console.log(_a);
  console.log(_b);
  console.log(_c);

  const end = new Date();

  console.log(end - start);
};
```

### Polyfill

그렇다면, Promise.all 함수는 어떻게 구현되어 있을까? 궁금했기 때문에 직접 구현해보기러 한다. 참고로 아래 코드는 공식적인 코드가 아니라 필자의 머릿속에서 나온 아주아주 간단한 코드인 것을 감안했으면 좋겠다.

```jsx
const all = (promises) => {
  return new Promise((resolve, reject) => {
    const results = Array.from({ length: promises.length }).map(() => null);
    let count = 0;
    promises.forEach((promise, i) => {
      promise.then((result) => {
        results[i] = result;
        if (++count >= promises.length) resolve(results);
      });
    });
  });
};
```

우선 all 함수의 인자로 promise의 배열을 전달 받았다. 그리고, 배열 내에 같은 인덱스에 결과 값이 전달이 되어야 하기 때문에, 인자로 전달 받은 배열과 같은 크기의 result 배열을 선언해준다.

그리고, 모든 작업이 끝나면 results 배열을 resolve 해주기 위해, count 변수를 하나 둔다. 하나의 promise가 resolve 될 때마다 결과 값을 results 배열의 같은 인덱스에 넣어주고 count 를 1 증가시킨다.

만약, count 값이 인자로 전달 받은 배열의 크기보다 크거나 같아지면, results 배열을 resolve 한다.

### Result

**async/await**

![image](https://user-images.githubusercontent.com/49841765/177481050-178a3a5a-7660-4c60-81d0-ba42f03f1b05.png)

**all**

![image](https://user-images.githubusercontent.com/49841765/177481078-af1e870f-f264-438c-a00e-e5c4d5d3f61b.png)
