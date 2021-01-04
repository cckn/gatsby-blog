---
title: Frontend에서 TDD를 이용해 견고한 JS 소프트웨어 만들기
date: "2021-01-04T22:40:00.000Z"
description: "TDD를 이용한 견고한 JS 소프트웨어 만들기"
---

> - 본 글은 김정환님의 강의 [견고한 JS 소프트웨어 만들기](https://www.inflearn.com/course/tdd-%EA%B2%AC%EA%B3%A0%ED%95%9C-%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4-%EB%A7%8C%EB%93%A4%EA%B8%B0/)를 수강 후 정리 및 복습하고자 작성한 포스트입니다.
> - 강의 내용과 제가 이해한 방향이 다소 다를 수 있습니다.

## Frontend JS에서 TDD를 사용할 수 있을까??

### TDD 좋아

![picture 2](https://i.imgur.com/RGDP6st.png)

TDD를 이용해 테스트코드를 쭉쭉 뽑아내고 한층한층 단단한 벽돌을 쌓아가며 개발하는 모습은 나의 오랜 워너비였다.

개인적으로 NodeJS 프로젝트를 진행하며 조금씩 적용해본 TDD는 아주 달콤했다.

쌓여가는 TestCode들과 코드를 저장할 때마다 자동 테스트가 수행되며 초록색으로 도배되는 모습.

리팩토링하면서도 사이드 이펙트가 일어날까 겁내지 않아도 됐다.

오랜만에 소스코드를 봐도 테스트를 몇 번 돌려보면 다시 개발에 착수할 수 있었다.

### 프론트엔드에서는 TDD 못써?

그래서 프론트엔드에서도 TDD를 사용하려고 했었는데...

장점보다 어려운 점들이 많았다.
![picture 3](https://i.imgur.com/Ad48gL6.png)

왜 그랬을까?

## 테스트하기 어려운 소스코드들

강의에서는 이야기한다.

> 프론트엔드 테스트는 어렵다.
> 테스트하기 어렵게 작성했기 때문에

### 테스트하기 어렵게 작성된 소스코드

```html
<button onclick="counter++; countDisplay()">증가</button>
<span id="counter-display">0</span>

<script>
  let counter = 0

  function countDisplay() {
    const el = document.getElementById("counter-display")
    el.innerHTML = counter
  }
</script>
```

이 코드의 문제점은 뭘까?

#### 관심사의 분리가 이뤄지지 않았다

```html
<button onclick="counter++; countDisplay()">증가</button>
```

이 라인은 버튼을 만들면서 변수를 증가시키고 view까지 담당하고 있다.
관심사의 분리가 전혀 이뤄지지 않았다.

#### 전역변수를 더럽히고 있다

`counter` 변수와 `countDisplay` 함수가 전역으로 선언되어 있다.

요즘 누가 모듈 안쓰고 이렇게 쓰겠냐만은...

#### 재사용이 어렵다

```js
const el = document.getElementById("counter-display")
```

관심사의 분리도 안되어 있고 이렇게 elementId를 지정해서 받다보니 재사용이 어렵다.

### 그럼 어떻게 코드를 짜야 테스트하기 용이할까?

#### 우선 모듈 방식부터 짚고 가자

임의 모듈 패턴 방식을 사용해서 전역변수를 어지럽히지 않고 모듈 형식으로 구현한다.

```js
var App = App || {}

App.Person = initName => {
  let name = initName
  return {
    getName: () => name,
    setName: newName => {
      name = newName
    },
  }
}
```

이렇게 구현한 임의 모듈은 다음과 같이 사용한다.

```js
const person = App.Person("jone")
console.log(person.getName()) // jone
person.setName("doe")
console.log(person.getName()) // doe
```

#### TDD 방식의 사고 - 의존성 주입

모듈들은 단일 책임의 원칙을 위해 의존성을 주입받아 사용해야한다.

아래 소스코드는 clickCounter, updateEl을 주입받아 사용한다.

결합도는 낮추고 응집도는 높이자

```js
var App = App || {}

App.ClickCountView = (clickCounter, updateEl) => {
  return {
    updateView() {
      updateEl.innerHTML = clickCounter.getValue()
    },

    increaseAndUpdateView() {
      clickCounter.increase()
      this.updateView()
    },
  }
}
```

## 테스트 방법

### 의존성 주입 여부 테스트

의존성이 주입됐는지는 어떻게 테스트할 수 있을까?

주입된 변수가 없다면 에러를 일으키고 그 에러를 테스트하자.

```js
// ClickCounterView.spec.js

describe("의존성 주입 테스트", () => {
  it("ClickCounter를 주입하지 않으면 에러를 던진다", () => {
    const clickCounter = null
    const updateEl = document.createElement("span")
    const actual = () => App.ClickCountView(clickCounter, updateEl)
    expect(actual).toThrowError()
  })

  it("updateEl를 주입하지 않으면 에러를 던진다", () => {
    const clickCounter = App.ClickCounter()
    const updateEl = null
    const actual = () => App.ClickCountView(clickCounter, updateEl)
    expect(actual).toThrowError()
  })
})
```

```js
// ClickCountView.js

var App = App || {}

App.ClickCountView = (clickCounter, updateEl) => {
  if (!clickCounter) throw new Error()
  if (!updateEl) throw new Error()

  return {
    // ...
  }
}
```

`ClickCounterView.spec.js`의 `actual`과 `expect(actual).toThrowError()` 를 주의하자.

### 호출여부 테스트

B 함수에서 A 함수를 호출한다고 B 함수의 테스트코드에 A 함수를 검증해야할까?

각 함수가 호출하는 관계에 대해 일일히 테스트코드를 작성 하면 테스트가 그만큼 피곤해진다.

그럼 B 함수의 테스트에서는 A 함수를 어떻게 테스트해야할까?

B 함수를 호출할 때 A 함수가 호출되는지는 테스트할 수 있다면??

그럴 때 필요한 테스트 함수가 `spyOn`이다.

```js
// ClickCounterView.spec.js

describe("increaseAndUpdateView()는", () => {
  it("ClickCounter의 increase 를 실행한다", () => {
    spyOn(clickCounter, "increase")
    view.increaseAndUpdateView()
    expect(clickCounter.increase).toHaveBeenCalled()
  })

  it("updateView를 실행한다", () => {
    spyOn(view, "updateView")
    view.increaseAndUpdateView()
    expect(view.updateView).toHaveBeenCalled()
  })
})
```

`spyOn`는 `spyOn(객체명, "메서드명")`과 같이 사용한다.

spy를 심어두면 해당 객체의 메서드를 감시하고

`expect(view.updateView).toHaveBeenCalled()`를 확인할 수 있다.

```js
// ClickCountView.js

var App = App || {}

App.ClickCountView = (clickCounter, updateEl) => {
  // ...

  return {
    updateView() {
      updateEl.innerHTML = clickCounter.getValue()
    },

    increaseAndUpdateView() {
      clickCounter.increase()
      this.updateView()
    },
  }
}
```
