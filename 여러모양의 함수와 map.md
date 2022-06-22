# 여러모양의 함수와 map

**부분 적용** : 여러 매개변수를 받는 함수에 일부 인자만 미리 적용해서 나머지 인자를 받는 함수 만들기

```ts
(a: A, b: B, c: C) => D

(a: A, b: B) => (c: C) => D
```

**커링** : 여러 매개변수를 받는 함수를 한개의 인자만 받는 단인자 함수들의 함수열로 만들기

```ts
(a: A, b: B, c : C) => D

(a: A) => (b: B) => (c: C) => D
```

## 1. Partial Application(부분 적용)

부분함수와는 다르고 커링과는 약간 비슷하다.
인자가 여러 개인 함수의 전체 인자 중에 인자 몇 개를 고정하여 더 작은 개수의 인자를 가지는 또 다른 함수를 생성하는 프로세스. 함수의 인자를 앞에서 부터 꼭 채울 필요는 없다.

```ts
const delivery = (present: string, from: string, to: string) => {
  return `
    보내는 물건: ${present}
    보내는 사람: ${from}
    받는 사람: ${to}
   `;
};

// 불편함
delivery("상품권", "엄마", "아들");

delivery("상품권", "엄마", "딸");

delivery("상품권", "엄마", "할머니");
```

```ts
const delivery => (present: string, from: string) => (to: string) => {
    return `
    보내는 물건: ${present}
    보내는 사람: ${from}
    받는 사람: ${to}
    `
}

const momsPressent = delivery('상품권', '엄마')
momsPressent("아들");
momsPressent("딸");
momsPressent("할머니");

```

```ts
//compose 함수 partial application의 일종
//<A,B,C>((B) => C, (A) => B) => (A) => C
export const compose =
  <A, B, C>(g: (y: B) => C, f: (s: A) => B) =>
  (x: A) => {
    return g(f(x));
  };

export function getPrice(name: string): number | undefiend {
  if (name === "tomato") return 7000;
  else if (name === "orange") return 15000;
  else if (name === "apple") return 10000;
}

//getPrice의 공역과 아래 함수의 정의역을 일치 시키기
export const isExpensive = (price: number | undefiend) => {
  if (price === undefiend) return false;
  return price > 10000;
};

export const isExpensivePrice = compose(isExpensive, getPrice);
```

## 2. Currying 커링

인자가 여러개인 함수를 인자가 하나인 함수들의 함수열(sequence of functions)로 만들기
커링이란 임의의 함수를 또다른 함수열로 변환하는 작업을 의미한다. 함수열은 함수가 함수를 반환하는 것이 연속되는 것을 뜻한다. 이렇게 커링이 완료된 함수를 curried function이라고 한다.

```ts
const delivery = (present: string) => (from: string) => (to: string) => {
  return `
    보내는 물건: ${present}
    보내는 사람: ${from}
    받는 사람: ${to}
    `;
};
```

임의의함수를 curried function으로 바꾸는 예시
가변적인 인자에 대한 커링함수를 만드는 것은 까다롭기 때문에 3개 인자로 고정된 커리함수 구현

```ts
const delivery = (present: string, from: string, to: string) => {
  return `
    보내는 물건: ${present}
    보내는 사람: ${from}
    받는 사람: ${to}
   `;
};

const curry3 =
  <A, B, C, D>(f: (a: A, b: B, c: C) => D) =>
  (a: A) =>
  (b: B) =>
  (c: C) =>
    f(a, b, c);

const curriedDelivery = curry3(delivery);

const momsPresent = curriedDelivery("상품권")("엄마");
console.log(momsPresent("아들"));
```
