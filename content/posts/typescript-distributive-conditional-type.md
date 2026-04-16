+++
date = 2026-04-16T21:32:50+09:00
draft = true
title = 'IsNever<T> 구현하기, Distributive Conditional Type'
summary = ''
tags = []
categories = []
+++

upnote://x-callback-url/openNote?noteId=e430ae38-d608-461b-9922-f629215786ec
upnote://x-callback-url/openNote?noteId=c6a3b48c-1713-449f-8176-6432b9ea003e


- conditional type 소개
- distributive conditional type 동작 소개
- distributive 동작이 방해가 되는 케이스 소개 (T가 never인지 확인하는 조건)
- Tuple 문법으로 distributive 동작을 막는 방법
- Distributive 동작을 이용해 DistributiveOmit 만들기

타입스크립트는 타입을 조작하는 다양한 문법을 지원해서, 제네릭(generic) 타입을 이용해 원하는 타입을 만드는 다양한 타입 유틸들이 존재한다. 그러다가 종종 예상한 것과 다르게 동작하는 경우를 만나게 되는데, 그 중 하나가 `IsNever<T>`이다.


## 삼항 연산자를 이용해 조건에 따라 다른 타입으로 해석되는 Conditional Type

타입스크립트에는 Conditional Type이라는 문법이 있다. [타입스크립트 문서][conditional-type]에 따르면 다음과 같은 형식으로 표현된 타입이다.

> Conditional types take a form that looks a little like conditional expressions `(condition ? trueExpression : falseExpression)` in JavaScript:
>
> ```typescript
>   SomeType extends OtherType ? TrueType : FalseType;
> ```

`SomeType extends OtherType`는 [제네릭 제약 조건](https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints)에도 쓰이는 구문인데, "`SomeType`이 `OtherType`은 확장한다." 즉, "`SomeType`은 적어도 `OtherType`의 구조를 다 만족시켜야 한다."는 의미다. 제네릭에서는 이게 `SomeType`으로 넣을 수 있는 타입을 제한하는 역할을 하는데, Conditional Type 문법에서는 이게 `SomeType`에 넣는 타입이 제약 조건을 만족하느냐에 따라 결과 타입을 분기하는 역할을 한다.

보통 `SomeType`의 값을 `OtherType`의 변수에 할당할 수 있느냐에 따라 `SomeType extends OtherType` 조건절의 참/거짓이 결정되고, 이게 참인 경우, 평가 결과가 `TrueType`이 되고, 거짓인 경우, `FalseType`이 되는 타입이라고 할 수 있다.

[타입스크립트 문서][conditional-type]의 예시를 보면, 다음과 같이 포함 관계에 있는 타입이 있을 때
```typescript
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}
```
다음과 같이 `extends` 앞에 오는 타입이 뒤에 오는 타입에 할당될 수 있으면, 결과가 `:` 앞의 타입이 되고, 할당될 수 없으면, `:` 뒤의 타입이 된다.
```typescript
type Example1 = Dog extends Animal ? number : string;
// type Example1 = number

type Example2 = RegExp extends Animal ? number : string;
// type Example2 = string
```

그런데 위와 같이 조건에 구체적인 타입을 쓰면, 조건(`Dog` extends `Animal`)의 결과가 이미 참 또는 거짓으로 고정되어 있으므로 결과도 두 분기 중 하나로 정해져 있다. 따라서 그냥 바로 `number` 또는 `string`으로 지정하는 것과 동일하므로 그다지 유용하지 않다.

따라서 이 구절이 주로 쓰이는 곳은 조건절에 어떤 타입이 올지 모르는 제네릭 타입이다. 다음 타입은 이 제네릭 타입을 사용하는 곳에서 어떤 타입을 넣느냐에 따라 결과가 달라진다.
```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel
```

이를 이용해서 `T`가 `never`이냐에 따라 결과를 분기하고 싶은 상황이 있었다. `never`에 할당할 수 있는 것은 `never` 뿐이므로 `T extends never`라는 조건으로 검사하면, `T`가 `never`일 때만 true가 될 것이라고 생각했다.
```typescript
interface A {}
interface B {}

type SomeType<T> = T extends never ? A : B
```

그런데 실제로 이 타입을 사용해보면, `T`가 `never`가 아닐 때는 기대한 대로 `B`가 나오는데, `T`가 `never`일 때는 `A`가 아니라 `never`라는 결과가 된다.
```typescript
type ShouldBeB = SomeType<string> // B
type ShouldBeA = SomeType<never> // never
```

왜 이런 현상이 일어날까? 이는 Conditional Type에서 특수한 규칙이 적용되는 Distributive Conditional Type 때문이었다.

## Distributive Conditional Type
이는 [타입스크립트 Conditional Type 문서][distributive-conditional-type]에 설명이 되어 있는 기능이다.
> When conditional types act on a generic type, they become distributive when given a union type. For example, take the following:

조건절이 제네릭 타입에 붙고, 제네릭 타입 파라미터에 Union 타입이 들어오는 경우, 조건절이 분배(distributive)가 된다는 말이다. 여기서 조건절이 분배가 된다는 게 무슨 말일까?

다음과 같은 조건절이 있을 때
```typescript
T extends U ? X : Y
```
만약, `T`가 `A | B | C`와 같이 Union Type이라면, 다음과 같이 `T` 자리에 `A | B | C`를 넣은 것처럼 해석되는 게 아니라
```typescript
(A | B | C) extends U ? X : Y
```
다음과 같이 Union의 개별 요소에 대해 조건절을 적용해서 각각의 결과를 다시 Union으로 합치는 방식으로 해석한다는 것이다.
```typescript
(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)
```

`Extract<T, U>`나 `Exclude<T, U>` 같은 기본으로 제공되는 타입 유틸도 이 성질을 기반으로 동작한다. `Exclude<T, U>` 타입을 예로 들면,

```typescript
type Exclude<T, U> = T extends U ? never : T;

type Result = Exclude<'name' | 'id' | 'value', 'id'>


```

[conditional-type]: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
[distributive-conditional-type]: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types

---

짧은 도입. 이 글에서 다루는 문제와 얻을 수 있는 결과를 2~4문장으로 정리한다.

## 문제

문제가 발생한 상황, 증상, 목표를 짧게 적는다.

## 핵심 원인

왜 이런 현상이 생기는지 한두 문단으로 설명한다.

## 해결 방법

적용한 방법을 순서대로 정리한다.

1. 첫 번째 단계
2. 두 번째 단계
3. 세 번째 단계

```sh
# 필요한 명령이나 예제를 넣는다.
```

## 확인

어떤 결과가 나오면 정상인지, 어디를 보면 되는지 적는다.

## 정리

핵심 takeaway를 한두 문장으로 마무리한다.
