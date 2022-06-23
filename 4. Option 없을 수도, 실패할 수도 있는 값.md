# Option 없을 수도, 실패할 수도 있는 값

입력에 대응되는 출력이 없을 때, 타입스크립트 컴파일러는 undefiend를 반환하면서
부분함수를 전함수로 바꾸었다. 하지만 undefiend는 여러 맥락에서 사용될 수 있기 때문에 또 다른 혼동을
불러 일으킬 수 있으며, undefiend를 확인하는 코드가 여러 함수에서 중복될 수 있다.

## 1. Option 타입

값이 있을수도, 없을수도 있는 자료구조. 함수형 프로그래밍은 값의 변경을 허용하지 않기 때문에 모든 필드를
readonly로 설정

```ts
// undefiend가 여러맥락에서 사용되기 때문에 또다른 혼동이 생길 수 있다.
export type Option<A> = A | undefiend;

export type Some<A> = {
  readonly _tag: "Some";
  readonly value: A;
};
export type None = {
  readonly _tag: "None";
};

export type Option<A> = Some<A> | None;

const n1: Option<number> = { _tag: "Some", value: 1 };

export const some = <A>(value: A): Option<A> => ({ _tag: "Some", value });

export const none = (): Option<never> => ({ _tag: "None" });

export const isSome = <A>(oa: Option<A>): oa is Some<A> => oa._tag === "Some";

export const isNone = <A>(oa: Option<A>): oa is None => oa._tag === "None";
```

태그드 유니언은 adt 대수 자료구조에서의 합 타입의 일종이다.

## 2. If

If문이 나쁘다는 것이 아니다.
값의 부재를 확인하는 용도로 사용할 때는 정확하게 그러한 목적을 표현하는 타입을
사용하는 것이 부수적인 맥락을 파악하기 위해 덜 신경써도 된다는 것이 핵심이다. 왜냐하면 If의 용도가 다양하기 때문이다.
또한 for문 처럼 명령적인 구문이 사용되는 구문은 합성에서 사용하기 까다롭다.

```ts
export const fromUndefiend = <A>(a: A | undefiend): Option<A> => {
  if (a === undefiend) return none();
  return some(a);
};

export const getOrElse = <A>(oa: Option<A>, defaultValue: A): A => {
  if (isNone(oa)) return defaultValue;
  return oa.value;
};
```

## 3. Option의 Map

배열을 처리하는 map은 N개의 값이 입력되면 N개의 값이 출력되어야 한다.

Option은 값이 없을 수도 있을 수도 있는 타입이기 때문에 값이 있다면 즉, some이라면 option에 있는 값을
함수를 적용한 후에 그 결과를 다시 some으로 반환하게 될 것이다. 그리고 값이 없다면 함수에 적용할 값이 없기 때문에 함수를 실행하지 않고 그대로 none을 반환하게 된다.

배열의 map함수는 인자로 주어진 함수를 원소의 갯수에 따라서 0번이상 몇번 실행될 지 모르는 맥락을 처리해서 입력된 배열에 원소 갯수와 같은 갯수의 원소를 가지는 새로운 배열을 리턴하는 것 처럼 option의 map 함수는 인자로 주어진 함수를 option의 상태가 none인지 some인지에 따라 0번 혹은 1번만 실행하는 모습이 되어야 한다.

그리고 none이거나 some인 상태는 변경하지 않고 그대로 돌려주어야 한다.

배열의 map함수가 입력된 인자의 원소 갯수를 그대로 유지해서 그대로 반환되어야 하는 것과 같은 원리이다.

```ts
// 값이 없으면 none을 그대로 리턴하고 값이 있으면 값의 함수를 적용한 후 옵션으로 만들어서 돌려준다.
export const map = <A, B>(oa: Option<A>, f: (a: A) => B): Option<B> => {
  if (isNone(oa)) return oa;
  return some(f(oa.value));
};
```

```ts
export const mapOrElse = <A, B>(
  oa: Option<A>,
  f: (a: A) => B,
  defaultValue: B
): B => {
  return getOrElse(map(oa, f), defaultValue);
};
```

## 4. functor

기존에 경험했던 것처럼 Array의 맵과 Option의 맵은 한 단계 더 일반화가 가능하다.

맵함수는 입력값의 자료구조의 성질에 따라서 반드시 특정 성질이 변하지 않아야 한다. 이것을 구조를 보존한다고 한다.

맵함수의 의미를 다시 정리해보자면 배열이나 Option같이 특정 부수효과를 동반할 수 있는 자료구조에
부수효과가 동반되지 않는 함수를 적용해 준다. 이 때 출력값은 입력값에 대한 특정 부수효과에 대한 구조는 변경하지 않은 채로 함수를 적용한 결과가 된다.

배열이라면 입력된 원소의 갯수가 유지되어야 하고 옵션이라면 값이 없는지 여부가 유지되어야 한다.
구조를 보존하는 맵이라는 함수를 구현할 수 있는 자료구조를 functor라고 한다. 엄밀한 의미의 functor가 되기 위해서는 구조를 보존한다는 것이 무엇인지 정확하게 정의해야 하고, 맵 함수의 구현은 이러한 정의를 엄격하게 지켜서 구현해야 한다. 이것을 functor의 법칙이라고 한다.

제네릭 타입의 자료구조는 functor의 후보가 될 수 있다. Try, Promise, observable도 functor의 일종이다.
그렇다고 모든 제네릭 타입의 자료구조가 functor가 되는 것이 아니다. functor의 법칙을 만족하지 않는 자료구조가 있다. 대표적으로 Set이 그러한 자료구조 중 하나이다. 원소가 중복될 수 없기 때문에 Map함수가 구조를 보존하도록
구현할 수 없다.

```ts
//Array의 map이나 Option의 map은 f에 의해 내부의 값은 변경되어도 구조는 변경되지 않는다.
Type<A>, f, f: (a: A) => B) => Type<B>
```
