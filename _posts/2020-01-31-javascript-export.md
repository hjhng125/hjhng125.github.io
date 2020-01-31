---
title:  "[Javascript] export, import"
excerpt: "export 와 import 에 관하여 공부한 것을 기록합니다."

categories:
  - Javascript
tags:
  - Javascript
  - export
  - import
last_modified_at: 2020-01-31T017:32:00
---

Javascript ES6 문에서는 export, import 로 모듈을 손쉽게 제작하고 가져다 쓸 수 있다.
`export` : 함수, 객체, 값을 내보냄
`import` : export 로 내보낸 값을 가져옴

# export
* export 에는 named 와 default 가 있다.
* export named: 모듈을 특정 이름으로 export 한다.

```
export { myObj };
export const foo = bar;
```

* export default: 모듈을 이름 없이 export 한다.

```
export default function() {}
export default class {}
```

* 하나의 모듈은 하나의 default export 만 가능하다.
* named export 는 동일한 이름으로 가져올 수 있고, 이름을 바꾸려면 as 를 사용해야 한다.

# import
* export 와 쌍으로 동작한다.
* 다른 모듈에서 내보낸 바인딩을 가져올 때 사용한다.