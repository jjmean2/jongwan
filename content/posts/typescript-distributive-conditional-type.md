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

타입스크립트 conditional type으로 타입 유틸을 만들다가 `T`가 `never`인지에 따라 분기하는 타입을 만들려고 하는데 생각한 대로 동작하지 않은 경우를 발견했다. 예를 들어 [다음 예제][ex1]을 보자.
```ts
type IsNever<T> = T extends never ? true : false

type ShouldBeFalse = IsNever<string>
// type ShouldBeFalse = false

type ShouldBeTrue = IsNever<never>
// type ShouldBeTrue = never
```


`IsNever<T>`는 `T`가 `never`인 경우, `true`, `never`가 아닌 경우, `false`로 평가되기를 기대한 타입이다. 그런데 결과는 `T`가 `never`일 때 `true`가 아니라 `never`가 된다. 이유가 뭘까?



## Distributive Conditional Type - 조건부 타입이 제네릭 타입에 사용될 때 특수한 규칙

타입스크립트의 [conditional type 문서][doc-cond-type][^doc-cond-type]에 이 동작의 원인과 해결책이 모두 나와 있다. conditional type 구문이 특정 조건을 만족하면 distributive conditional type이 되기 때문인데, 문서에는 다음과 같이 나와 있다.

> ### Distributive Conditional Types
> When conditional types act on a generic type, they become distributive when given a union type.

Conditional type의 조건절이 제네릭 타입 파라미터에 적용되고, 타입 파라미터로 union 타입이 주어진 경우, 조건절이 분배가 된다는 말이다.

Conditional type이 도입된 [타입스크립트 2.8 배포 노트][doc-release-note-2.8][^doc-release-note-2.8]의 설명도 보자.

> ### Distributive conditional types
>
> Conditional types in which the checked type is a naked type parameter are called *distributive conditional types*. Distributive conditional types are automatically distributed over union types during instantiation. For example, an instantiation of `T extends U ? X : Y` with the type argument `A | B | C` for `T` is resolved as `(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`.


여기서 조건절이 분배된다는 말은 conditional type이 다음과 같을 때
```ts
T extends U ? X : Y
```
`T`에 `A | B | C` 같이 union 타입이 들어오는 경우, 다음과 같이 해석되지 않고,
```ts
(A | B | C) extends U ? X : Y
```
다음과 같이 해석된다는 것이다.
```ts
(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)
```
즉, union 타입 전체에 대해 조건을 검사하는 게 아니라 union 타입에 속한 개별 타입마다 조건을 검사해서 conditional type을 적용한 후 결과들을 다시 union 타입으로 모으는 방식으로 해석되는 것이다.

타입스크립트가 기본 제공하는 타입 유틸인 [`Exclude`][type-exclude], [`Extract`][type-extract] 같은 타입도 이 동작에 기반한다.

```ts
type Exclude<T, U> = T extends U ? never : T;
type Extract<T, U> = T extends U ? T : never;
```

예를 들어 `Exclude` 타입의 동작을 보자.
```ts
// Value가 union 타입일 때
type Value = string | number | boolean | object
// Value에서 object를 제외한다.
type Primitive = Exclude<Value, object>
// string | number | boolean
```
[`Exclude`의 동작을 풀어서 써 보자][ex2]. 만약 distributive 동작이 없었다면, 다음과 같은 형태가 될 것이다.
```ts
// Exclude 정의 T 자리에 Value를, U 자리에 object를 그대로 대입하는 경우,
// Value는 object에 할당할 수 없기 때문에 false 분기를 타서 결과는 Value 그대로가 된다.
type Primitive_v1 = Value extends object ? never : Value
// string | number | boolean | object
```
Distributive 동작을 적용하면 다음과 같이 된다. `Value` union을 구성하는 요소 타입 각각에 조건절을 적용한 후 결과를 합치는 것이다. 이 중 `object` 타입만 `extends object`를 만족시켜서 `never`가 되고, 나머지는 원래 타입을 유지한다. `never` 타입은 다른 타입과 union했을 때 아무 타입도 더하지 않으므로 결과적으로 `Value` union에서 `object`만 제거한 타입이 된다.
```ts
// Distributive 방식으로 동작하는 경우, Exclude라는 이름대로 Value에서 object만 제외한 타입이 된다.
type Primitive_v2 =
    | (string extends object ? never : string)   // string
    | (number extends object ? never : number)   // number
    | (boolean extends object ? never : boolean) // boolean
    | (object extends object ? never : object)   // never

// string | number | boolean | never
```

`T extends never`가 생각한 대로 동작하지 않았던 것은 이 distributive 방식 때문이다.

## `never` 타입은 empty union 즉, 안에 아무 타입도 들어있지 않은 union 타입이다.

Stack Overflow에서 [`never` 타입이 왜 union 타입에서 아무 타입을 더하지 않는지 묻는 글][so-never-type][^so-never-type]이 있다.





[^doc-cond-type]: [TypeScript: Documentation - Conditional Types][doc-cond-type]
[^doc-release-note-2.8]: [TypeScript: Documentation - TypeScript 2.8][doc-release-note-2.8]

[^so-never-type]: [typescript - Why is the type "never" meaningless in union types? - Stack Overflow][so-never-type]


[type-exclude]: https://github.com/microsoft/TypeScript/blob/55423abe4d029017f19b6e4c32097591994836b4/src/lib/es5.d.ts#L1599-L1602
[type-extract]: https://github.com/microsoft/TypeScript/blob/55423abe4d029017f19b6e4c32097591994836b4/src/lib/es5.d.ts#L1604-L1607
[doc-cond-type]: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#distributive-conditional-types
[doc-release-note-2.8]: https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html#distributive-conditional-types
[so-never-type]: https://stackoverflow.com/questions/64230626/why-is-the-type-never-meaningless-in-union-types
[github-23182]: https://github.com/microsoft/TypeScript/issues/23182#issuecomment-379094672


[ex1]: https://www.typescriptlang.org/play/?ssl=7&ssc=24&pln=7&pc=29#code/C4TwDgpgBAkgzgOQgNwgJwDwBUB8UC8UWUEAHsBAHYAmcUlK6UA-FMGgK7QBcUAZgEMANnAgAoMaEhQAygAsA9hyHUAQhABiw0QViJGmOOwCWlAOZ4xAeittw0eUpXqtI6IUFuJUh4uVqILE53PSRUTAZwnGtbH1k-Z0Dg3Uj0IA
[ex2]: https://www.typescriptlang.org/play/?#code/PTAEDUEMBsFcFNAANaWA7Alge1aQAwuFDxwH3HRAZVoCgAXATwAd4IYFQBeUAZwoCd1UBzUAD6hUsALYAjeJ0GhxmTNHiQcQzOIBW8AMYUyIBnHiAF0cA4g6DWadgH07QgHAnAHmOAdVcAnTQDpKtegAVuo9BXQAN3pWAFEADy04ABN4AB4oQwAacw1tCgA+PTAObj4ZEQkpGTkFJVQybNAIqNhY0EBUCcANcdAAFVBABPHAGs6jAwQrFIBVTp7UywobQA-awABmwB0O0Gn8QA1VwBSm0EBemsAGsaSqxIQ1i3TewANVwE+m49BADCHQQDHRwAY60kAObt6AMxg2ekAQ3rubXDNABprAD81az29Bms2QgBwWtweOigHzoPwBYIAfUCAEYWH16PBwhR4KhomwxulQAB+YTwYLSABc2KquR4-CEhUk0iEpUUyhkhx0lX0ABF0IzxLBkfRAJQ9gF2hwA84-NAJvNgETx1YbbbVSIxeCAH3a1oAXccAIZ1zbGmEk6QAznXYnHh8LrQND3NQ4QikUF4GiAEwsMigGQACkZ+Vx+MJxN5FHJlOpoDpAd4AEpQPpY96-azikGCUTTeGKagqcU6WnOAn9EWU0JfZzyqAMyHsxG81G6VXlCWwC2Kj6K2Ga3jM6G0joG-nadm25GpAyuEyCmI2SV5FyVBPOEA




---

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
