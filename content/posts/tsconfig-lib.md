+++
date = "2026-03-24T12:27:13+09:00"
draft = true
title = 'Tsconfig Lib'
+++

upnote://x-callback-url/openNote?noteId=4f4d3fcc-5fd1-417b-aad1-a5ce1696c838

upnote://x-callback-url/openNote?noteId=a34ba10c-9621-4e8e-b551-65b7e225bd94

# Typescript tsconfig.json의 lib 설정과 @types/node 에 대해서

* * *

<br>

TypeScript 자체는 아무런 내장 타입(builtin type)이 정의되어 있지 않다. 즉, Array, String 같은 기본적인 클래스도 정의되어 있지 않다는 말이다. `string` 같은 원시값은 언어에 포함된 것이라 정의되어 있는 것 같은데, `String` 같은 내장된 클래스는 정의되어 있지 않다. 이런 것들은 보통 브라우저나 NodeJs 같은 런타임 환경(플랫폼, 호스트)에서 정의하고 있는 것들이다.

<br>

예를 들어, tsconfig.json에서 `"noLib": true` 설정을 하거나, `"lib": []` 설정을 하는 경우, 타입스크립트가 library 타입을 하나도 포함하지 않도록 할 수 있는데, 이 때 타입 체크되는 것을 보면 다음과 같다.

![](http://localhost:9425/images/d40dcf75-ea2e-455a-978a-68785571fdc5.png)<br>

<br>

`lib`는 프로젝트에 어떤 타입 정의가 포함되어야 할지를 지정하는 것이다. 

### TypeScript: TSConfig Reference - Docs on every TSConfig option

[https://www.typescriptlang.org/tsconfig/#lib](https://www.typescriptlang.org/tsconfig/#lib)

<br>

> ypeScript includes a default set of type definitions for built-in JS APIs (like `Math`), as well as type definitions for things found in browser environments (like `document`). TypeScript also includes APIs for newer JS features matching the [`target`](https://www.typescriptlang.org/tsconfig/#target) you specify; for example the definition for `Map` is available if [`target`](https://www.typescriptlang.org/tsconfig/#target) is `ES6` or newer.
>
> You may want to change these for a few reasons:
>
> - Your program doesn’t run in a browser, so you don’t want the `"dom"` type definitions
> - Your runtime platform provides certain JavaScript API objects (maybe through polyfills), but doesn’t yet support the full syntax of a given ECMAScript version
> - You have polyfills or native implementations for some, but not all, of a higher level ECMAScript version
>
> In TypeScript 4.5, lib files can be overridden by npm modules, find out more [in the blog](https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta/#supporting-lib-from-node_modules).
>
> <br>

<br>

<br>

그럼 `lib`에는 어떤 값이 들어가야 할까?

먼저 tsconfig.json의 `target`설정을 보자.

### Target - TypeScript: TSConfig Reference - Docs on every TSConfig option

[https://www.typescriptlang.org/tsconfig/#target](https://www.typescriptlang.org/tsconfig/#target)

> Modern browsers support all ES6 features, so `ES6` is a good choice. You might choose to set a lower target if your code is deployed to older environments, or a higher target if your code is guaranteed to run in newer environments.
>
> The `target` setting changes which JS features are downleveled and which are left intact. For example, an arrow function `() => this` will be turned into an equivalent `function` expression if `target` is ES5 or lower.
>
> Changing `target` also changes the default value of [`lib`](https://www.typescriptlang.org/tsconfig/#lib). You may “mix and match” `target` and `lib` settings as desired, but you could just set `target` for convenience.
>
> For developer platforms like Node there are baselines for the `target`, depending on the type of platform and its version. You can find a set of community organized TSConfigs at [tsconfig/bases](https://github.com/tsconfig/bases#centralized-recommendations-for-tsconfig-bases), which has configurations for common platforms and their versions.
>
> The special `ESNext` value refers to the highest version your version of TypeScript supports. This setting should be used with caution, since it doesn’t mean the same thing between different TypeScript versions and can make upgrades less predictable.
>
> <br>

<br>

<br>

### What does the TypeScript "lib" option really do? - Stack Overflow

[https://stackoverflow.com/questions/50986494/what-does-the-typescript-lib-option-really-do](https://stackoverflow.com/questions/50986494/what-does-the-typescript-lib-option-really-do)

> Remember, TS **never** injects polyfills in your code. It's [not its goal](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Design-Goals#non-goals). Complementing the accepted anwer:
>
> `target` tells TS which ES specification you want the final/transpiled code to support. If you configure it as `ES5`, TS will down compile the _syntactic_ features to ES5, so any arrow functions `() => {}` in your code will be transformed to `function () {}`.
>
> Whatever you choose for `target` affects the default value of `lib` which in turn tells TS what **type definitions** to include in your project. If you have `"target": "es5"`, the default value of `lib` will be `["dom", "es5", "ScriptHost"]`. It's assuming which _functional_ features the browser will support at runtime. Adding things to `lib` it's just to make TS happy - you still need to import the polyfill yourself in the project.
>
> So in short: configure `target` first, and if you need any extra polyfill in your project OR you **know** your browser(s) will support this little extra feature, `lib` is how to make TS happy about it.
>
> Example: You need to support IE11 but also you would like to use promises. IE11 supports ES5, but promises is an ES6 feature. You import a promises polyfill, but TS is still giving an error. Now you just need to tell TypeScript that your code will target ES5 and it's safe to use promises in the codebase:
>
> ```
> "target": "es5",
> "lib": ["dom", "es5", "ScriptHost", "es2015.promise"]
> ```
>
> <br>

<br>

> Typescript does not have any built-in types all types come from a set of base definitions (located in the `lib` folder in the typescript install directory). By default the `target` defines which `libs` are included. For example the [docs](https://www.typescriptlang.org/docs/handbook/compiler-options.html) state:
>
> > Note: If --lib is not specified a default list of librares are injected. The default libraries injected are:
> >
> > ► For `--target ES5: DOM,ES5,ScriptHost`
> >
> > ► For `--target ES6: DOM,ES6,DOM.Iterable,ScriptHost`
>
> The basic idea is that while target is deals with language features (more specifically which language features need to be down compiled, ex: for-of, or arrow functions), the `lib` option deals with what facilities the runtime environment has (ie. what built-in objects look like, what they are).
>
> Ideally the default `libs` for a given `target` should be used. We may, however, have an environment which supports some of the runtime facilities but not the language features, or we may target runtime with a lower `es` version and poly-fill some of the runtime facilities, which can be in general done for some things (ex: Promises).
>
> <br>

<br>

<br>

TypeScript는 코드를 `target`에 지정한 ES 버전의 문법으로 트랜스파일한다. (참고로 이 과정에서 결코 polyfill을 하지는 않는다.) 즉, `target`에 지정한 버전의 코드로 실행된다는 것을 의미한다. 보통 코드를 실행시키려는 대상은 브라우저 또는 NodeJS일 것이다. 브라우저로 치면, 대상 브라우저가 ES6 스펙까지 지원하면, `target` 을 `ES6`로 지정할 것이다. 그러면, `ES6` 브라우저에서 지원하는 내장 타입을 모두 쓸 수 있게 하는 것이 바람직하다. 그래서 `target` 값은 `lib`의 기본값에 영향을 준다. `lib`를 직접 설정하지 않는 경우, `target`에 설정한 버전에 따라 `lib`가 자동 지정된다는 것이다. (참고로 `target` 의 기본값은 `es5`이다.)

> Changing `target` also changes the default value of [`lib`](https://www.typescriptlang.org/tsconfig/#lib). You may “mix and match” `target` and `lib` settings as desired, but you could just set `target` for convenience.

예를 들어, `target`이 `es5`이면, `lib`의 기본값은 `["dom", "es5", "ScriptHost"]`이다. 

<br>

만약, `lib`를 직접 설정하는 경우, 직접 설정하는 lib만 포함이 된다. 따라서 `lib: []`로 설정항면, 아무 라이브러리 타입이 포함되지 않고, 처음 봤던 것처럼 `Array` 같은 기본 타입조차 인식하지 못하게 된다.

<br>

`lib`를 직접 설정한다는 것은 내가 실행한 런타임에 대해 더 잘 알기 때문에, 내가 원하는 것만 직접 포함시키겠다는 의미다.

예를 들어, ES5 브라우저에서 실행시킬 것이지만, ES5에서 지원하지 않는 Promise를 polyfill로 지원할 예정이라면 `lib`를 `["dom", "es5", "ScriptHost", "es2015.promise"]`로 직접 지정할 수 있는 것이다.

<br>

`lib`에 명시할 수 있는 값의 목록은 다음 소스코드를 참고하여 알 수 있다. 이건 ES API 업데이트에 따라 계속 업데이트 된다.

<br>

참고로 `lib`에서 다루는 도메인은 다음과 같은 것들이 있다고 한다.

- ECMAScript 언어 기능: `es5`,`es6`(`es2015`), `es2016`, `es2017`, ..., `esnext`
- DOM API (브라우저 API): `dom`
- Intl API (ECMA-402 스펙으로 정의된 Intl 관련 API): `es2016.intl`, ... 

### TypeScript/src/lib at main · microsoft/TypeScript

[https://github.com/microsoft/TypeScript/tree/main/src/lib](https://github.com/microsoft/TypeScript/tree/main/src/lib)

> # Read this!
>
> The files within this directory are copied and deployed with TypeScript as the set of APIs available as a part of the JavaScript language.
>
> There are three main domains of APIs in `src/lib`:
>
> - **ECMAScript language features** - e.g. JavaScript APIs like functions on Array etc which are documented in [ECMA-262](https://tc39.es/ecma262/)
> - **DOM APIs** - e.g. APIs which are available in web browsers
> - **Intl APIs** - e.g. APIs scoped to `Intl` which are documented in [ECMA-402](https://www.ecma-international.org/publications-and-standards/standards/ecma-402/)
>
> ## How do we figure out when to add something?
>
> TypeScript has a rule-of-thumb to only add something when it has got far enough through the standards process that it is more or less confirmed. For JavaScript APIs and language features, that means the proposal is at stage 3 or later.
>
> You can find the source of truth for modern language features and Intl APIs in these completed proposal lists:
>
> - [JavaScript](https://github.com/tc39/proposals/blob/master/finished-proposals.md)
> - [Intl](https://github.com/tc39/proposals/blob/master/ecma402/finished-proposals.md)
>
> For the DOM APIs, which are a bit more free-form, we have asked that APIs are available un-prefixed/flagged in at least 2 browser _engines_ (i.e. not just 2 chromium browsers.)
>
> ## Generated files
>
> The DOM files ending in `.generated.d.ts` aren't meant to be edited by hand.
>
> If you need to make changes to such files, make a change to the input files for [**our library generator**](https://github.com/microsoft/TypeScript-DOM-lib-generator).

<br>

### proposals/finished-proposals.md at main · tc39/proposals

[https://github.com/tc39/proposals/blob/main/finished-proposals.md](https://github.com/tc39/proposals/blob/main/finished-proposals.md)

### proposals/ecma402/finished-proposals.md at main · tc39/proposals

[https://github.com/tc39/proposals/blob/main/ecma402/finished-proposals.md](https://github.com/tc39/proposals/blob/main/ecma402/finished-proposals.md)

<br>

이 목록을 보면, 흥미로운 패턴이 있다.

`lib/libs.json` 을 보면, 이 목록을 좀더 보기 좋게 분류해 두었는데 다음과 같다.

[https://github.com/microsoft/TypeScript/blob/main/src/lib/libs.json](https://github.com/microsoft/TypeScript/blob/main/src/lib/libs.json)

- JavaScript 언어 기능
    - 특정 버전 전체 포함:`es5`, `es2015`, `es2016`, ...`es2024`, `esnext`
    - 세부 기능별: `es2015.core`, `es2015.collection`, ...
    - Host 관련: `dom.generated`, `dom.iterable`, `webworker.generated`, `scripthost`, ...
    - full (기본 라이브러리): `es5.full`, `es2015.full`, ...

### lib/libs.json

```typescript
{
    "libs": [
        // JavaScript only
        "es5",
        "es2015",
        "es2016",
        "es2017",
        "es2018",
        "es2019",
        "es2020",
        "es2021",
        "es2022",
        "es2023",
        "es2024",
        "esnext",
        // Host only
        "dom.generated",
        "dom.iterable.generated",
        "dom.asynciterable.generated",
        "webworker.generated",
        "webworker.importscripts",
        "webworker.iterable.generated",
        "webworker.asynciterable.generated",
        "scripthost",
        // By-feature options
        "es2015.core",
        "es2015.collection",
        "es2015.generator",
        "es2015.iterable",
        "es2015.promise",
        "es2015.proxy",
        "es2015.reflect",
        "es2015.symbol",
        "es2015.symbol.wellknown",
        "es2016.array.include",
        "es2016.intl",
        "es2017.arraybuffer",
        "es2017.date",
        "es2017.object",
        "es2017.sharedmemory",
        "es2017.string",
        "es2017.intl",
        "es2017.typedarrays",
        "es2018.asyncgenerator",
        "es2018.asynciterable",
        "es2018.regexp",
        "es2018.promise",
        "es2018.intl",
        "es2019.array",
        "es2019.object",
        "es2019.string",
        "es2019.symbol",
        "es2019.intl",
        "es2020.bigint",
        "es2020.date",
        "es2020.promise",
        "es2020.sharedmemory",
        "es2020.string",
        "es2020.symbol.wellknown",
        "es2020.intl",
        "es2020.number",
        "es2021.string",
        "es2021.promise",
        "es2021.weakref",
        "es2021.intl",
        "es2022.array",
        "es2022.error",
        "es2022.intl",
        "es2022.object",
        "es2022.string",
        "es2022.regexp",
        "es2023.array",
        "es2023.collection",
        "es2023.intl",
        "es2024.arraybuffer",
        "es2024.collection",
        "es2024.object",
        "es2024.promise",
        "es2024.regexp",
        "es2024.sharedmemory",
        "es2024.string",
        "esnext.decorators",
        "esnext.intl",
        "esnext.disposable",
        "esnext.collection",
        "esnext.array",
        "esnext.iterator",
        "esnext.promise",
        "esnext.float16",
        "decorators",
        "decorators.legacy",
        // Default libraries
        "es5.full",
        "es2015.full",
        "es2016.full",
        "es2017.full",
        "es2018.full",
        "es2019.full",
        "es2020.full",
        "es2021.full",
        "es2022.full",
        "es2023.full",
        "es2024.full",
        "esnext.full"
    ],
    "paths": {
        "dom.generated": "lib.dom.d.ts",
        "dom.iterable.generated": "lib.dom.iterable.d.ts",
        "dom.asynciterable.generated": "lib.dom.asynciterable.d.ts",
        "webworker.generated": "lib.webworker.d.ts",
        "webworker.iterable.generated": "lib.webworker.iterable.d.ts",
        "webworker.asynciterable.generated": "lib.webworker.asynciterable.d.ts",
        "es5.full": "lib.d.ts",
        "es2015.full": "lib.es6.d.ts"
    }
}
```

<br>

위 저장소의 lib 파일 이름은 

`es5.d.ts`, `es2015.d.ts` 인데 프로젝트에서 실제 참조하는 lib 파일은 `lib.es5.d.ts`, `lib.es2025.d.ts`이다. 

<br>

![](http://localhost:9425/images/7b5155d1-0fe5-4d75-9c2d-cfadedef322c.png)<br>

<br>

이는 `lib/libs.json`의 `paths` 부분을 보면 짐작할 수 있는데, 저장소의 파일을 이용해서 한번 빌드 과정을 거쳐서 `lib.`가 붙은 파일들을 만들어내는 것으로 보인다. 만드는 과정에서 `dom.generated.d.ts` 는 `lib.dom.d.ts`로 `generated` suffix 를 떼고, `es5.full.d.ts`은 `lib.d.ts`라는 좀더 일반적인 이름을 부여하며, `es2015.full.d.ts`은 `lib.es6.d.ts`로 바꾸는 것 같다.

```typescript
    "paths": {
        "dom.generated": "lib.dom.d.ts",
        "dom.iterable.generated": "lib.dom.iterable.d.ts",
        "dom.asynciterable.generated": "lib.dom.asynciterable.d.ts",
        "webworker.generated": "lib.webworker.d.ts",
        "webworker.iterable.generated": "lib.webworker.iterable.d.ts",
        "webworker.asynciterable.generated": "lib.webworker.asynciterable.d.ts",
        "es5.full": "lib.d.ts",
        "es2015.full": "lib.es6.d.ts"
    }
```

<br>

각 lib 들의 구성을 보는 것도 재밌는데 `es5.d.ts`는 가장 길고,JavaScript 언어에서 정의하는 기본적인 API를 대부분 포함한다.

[https://github.com/microsoft/TypeScript/blob/main/src/lib/es5.d.ts](https://github.com/microsoft/TypeScript/blob/main/src/lib/es5.d.ts)

```typescript
/// <reference lib="decorators" />
/// <reference lib="decorators.legacy" />

/////////////////////////////
/// ECMAScript APIs
/////////////////////////////

declare var NaN: number;
declare var Infinity: number;
```

<br>

그리고 `es2015.d.ts`부터는 이전 버전(`es5.d.ts`) + 그 버전에서 추가된 기능들을 triple-slash 주석으로 import하는 식으로 작성된다.

[https://github.com/microsoft/TypeScript/blob/main/src/lib/es2015.d.ts](https://github.com/microsoft/TypeScript/blob/main/src/lib/es2015.d.ts)

```typescript
/// <reference lib="es5" />
/// <reference lib="es2015.core" />
/// <reference lib="es2015.collection" />
/// <reference lib="es2015.iterable" />
/// <reference lib="es2015.generator" />
/// <reference lib="es2015.promise" />
/// <reference lib="es2015.proxy" />
/// <reference lib="es2015.reflect" />
/// <reference lib="es2015.symbol" />
/// <reference lib="es2015.symbol.wellknown" />
```

<br>

위와 같이 `es2015.d.ts`는 wrapper 정의 파일 같은 것으로 이전 버전과 ES2015의 코어 기능, 개별 기능들을 추가하는 식으로 정의되었다. 이렇게 나눔으로써 tsconfig.json을 설정할 때 `"es2015"`로 언어 기능 전체를 포함할 수도 있고, `"es2015.promise"`처럼 언어의 특정 기능만 포함하는 식으로도 할 수 있다.

<br>

그리고 `es2016.full.d.ts` 같이 `full` suffix가 붙은 파일은 다음과 같은 패턴이다.

```typescript
/// <reference lib="es2016" />
/// <reference lib="dom" />
/// <reference lib="webworker.importscripts" />
/// <reference lib="scripthost" />
/// <reference lib="dom.iterable" />
```

이는 `target`을 그 버전(여기서는 `es2016`)으로 선택했을 때 기본 설정되는`lib`값을 의미하는 것으로 보이는데, 언어 기능 (`es2016.d.ts`)과 Runtime 환경 기능 (`dom.d.ts`, `webworker.importscripts.d.ts`, `scripthost.d.ts`, `dom.iterable.d.ts`)을 모두 포함한다. 즉, 위의 예는 ES2016을 지원하는 브라우저 환경을 타깃으로 하는 타입 선언이 되는 것이다.

<br>

테스트해보니 tsconfig.json의 `lib`에 `ES2016.Full` 같은 값을 지정하는 것은 제대로 인식하지 못했다. (`lib.es2016.full.d.ts` 파일이 있음에도) 따라서 이 값은 사용자가 직접 할당하기 위한 값은 아니고, `target`에 의한 기본값 설정에 사용되기 위한 타입 선언 파일인 것으로 보인다.

(`lib.es6.d.ts`은 `es2015.full.d.ts` 파일 내용을 담은 파일이다. 즉, 이 파일에는 `dom` 선언도 포함되어 있다. 그런데 `"lib": ["es6"]`와 같이 지정해도, 타입 체크시 `dom` 타입을 인식하지 못하는 것으로 보였다. 그냥 es2015의 언어 기능만 인식하는 것으로 보였다. 즉, `es6`가 `lib.es6.d.ts`를 직접 바라보는 게 아닐 수도 있다는 생각이 들었다.)

<br>

* * *

## 자, `lib`의 기능은 대강 알았으니 이제 이를 어떻게 세팅하는 게 좋을지 생각해 보자.

<br>

`lib`는 런타임에 어떤 기능을 쓸 수 있냐에 대한 이야기로 볼 수 있다. TypeScript가 ES2024 를 ES5 로 바꿔 줄 수 있지만, 그건 문법적인 것이고, 새로운 버전에서 추가된 내장 타입을 만들어주진 않는다. 즉, polyfill을 해주지는 않는다는 것이다. ES2019 에서 Array에 새로 들어온 flatMap 메소드를 ES5 코드로 바꾸면서 polyfill 하거나 코드를 바꿔주지 않는다는 것이다. `array.flatMap(...)` 은 트랜스파일하고 나서도 `array.flatMap(...)`이다.

<br>

따라서 `target`이 실행시킬 런타임이 지원하는 버전에 맞춰야 하는 것처럼 `lib`도 런타임이 지원하는 버전에 맞춰야 한다. `target`에 따라 `lib`가 자동 설정되는 만큼 보통은 `lib`를 따로 지정하지 않아도 괜찮을 수 있지만, 특정 기능만 직접 polyfill을 한다거나 할 때는 `lib`를 직접 지정해야 한다.

<br>

## 브라우저 환경이라면, `target`에 지정한 값의 full 타입 선언을 먼저 지정하고, 추가로 직접 polyfill할 기능에 대한 타입을 추가하면 좋다.

예를 들어, 타깃 브라우저가 `es5` 까지 지원하고, Promise를 polyfill 할 예정이라면 다음과 같이 지정할 수 있을 것이다.

```typescript
"target": "es5",
"lib": ["dom", "es5", "webworker.importscripts", "scripthost", "es2015.promise"]
```

<br>

## NodeJS 환경이라면 어떨까?

NodeJS 환경에서만 존재하는 내장 타입도 있다. 대표적으로 `process`가 있다. 환경 변수를 얻기 위해 `process.env.SOMETHING` 같은 문법을 자주 쓰지만, `process`의 타입은 타입스크립트 기본`lib` 설정에 들어있지 않다.

`process`를 사용했을 때 TypeScript 에러 메시지를 보면, `@types/node` 패키지를 설치하라고 가이드한다.

![](http://localhost:9425/images/6bf9d566-9da2-420b-bd6b-4b6cfecc846e.png)<br>

이 패키지를 누가 관리하는 건지 정확히 모르겠다.

[https://www.npmjs.com/package/@types/node](https://www.npmjs.com/package/@types/node)

다만 패키지 [README.md](https://README.md "https://README.md") 에 보면, 여러 기여자가 있는데, TypeScript 팀, NodeJS 팀에서도 관여하는 것으로 보인다.

![](http://localhost:9425/images/8f026ba8-cae5-4d1d-a051-229234eb10d7.png)<br>

<br>

브라우저 내장 타입은 TypeScript에 내장된 lib 타입 선언으로 지원하지만, NodeJS 용 내장 타입은 별도의 @types 패키지로 지원하는 것이다.

<br>

따라서 NodeJS 런타임을 타깃으로 한다면, `@types/node`를 설치해야 한다.

<br>

### Deno 환경이라면? `@types/deno`라는 것도 있긴 한데, deno 런타임 자체에서 타입스크립트 관련 툴링을 지원하므로 그 쪽의 가이드를 따르면 된다. `@types/deno` 를 사용하는 것 같진 않다.

[https://www.npmjs.com/package/@types/deno](https://www.npmjs.com/package/@types/deno)

<br>

## 내장 `lib` 선언 대신에 커스텀 타입 선언 사용하기

타입스크립트 4.5 버전부터 `lib`로 명시한 라이브러리 내장 타입이 타입스크립트에 내장된 `lib.something.d.ts` 파일을 바라보는 대신에 `node_modules`에 직접 설치한 타입 선언 파일을 바라보도록 설정할 수 있는 방법이 생겼다.

다음 블로그에 내용이 소개된다.

### Announcing TypeScript 4.5 Beta - TypeScript

[https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta/#supporting-lib-from-node\_modules](https://devblogs.microsoft.com/typescript/announcing-typescript-4-5-beta/#supporting-lib-from-node_modules)

### TypeScript: Documentation - TypeScript 4.5

[https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html#supporting-lib-from-node\_modules](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-5.html#supporting-lib-from-node_modules)

> To ensure that TypeScript and JavaScript support works well out of the box, TypeScript bundles a series of declaration files (`.d.ts` files). These declaration files represent the available APIs in the JavaScript language, and the standard browser DOM APIs. While there are some reasonable defaults based on your [`target`](https://www.typescriptlang.org/tsconfig#target), you can pick and choose which declaration files your program uses by configuring the [`lib`](https://www.typescriptlang.org/tsconfig#lib) setting in the `tsconfig.json`.

### <br>

위 설명은 `lib`에 대한 설명이다. 타입스크립트는 일련의 타입 정의 파일들을 내장(번들링)하고 있고, 이는 JavaScript 언어 API나 표준 브라우저 DOM API 등을 선언한다. `target`에 따라 합리적인 기본값이 있지만, `lib`설정으로 원하는 타입 정의가 포함되도록 설정할 수 있다.

다만, `lib`는 키워드로만 설정하는 것이기 때문에 실제로 어떤 타입 정의가 포함될지 디테일하게 지정할 수는 없었다.

따라서 이런 문제가 생길 수 있다.

- TypeScript를 업그레이드하면, 타입스크립트에 내장된 lib 타입 정의도 새로운 버전의 타입스크립트에 내장된 것으로 대체되는데, 이를 원치 않을 수 있다.
- 프로젝트의 의존성에서 DOM API를 사용하는 것으로 선언한다면, 이를 커스터마이징하기 힘들 수 있다.

<br>

타입스크립트 4.5 버전부터는 내장된 lib 타입도 `@types/` 타입을 사용할 때처럼 node\_modules에 설치해서 사용할 수 있도록 지원한다. 다음처럼 동작한다.

<br>

TypeScript가 어떤 `lib` 파일을 포함해야 한다면, 먼저 node\_modules에서`@typesscript/lib-*`라는 패키지를 찾는다. 예를 들어, `"lib": ["dom"]` 이렇게 `dom` 설정이 되었다면, `node_modules/@typescript/lib-dom`이라는 패키지를 찾아서 있으면 이걸 이용하는 것이다. 만약 이 패키지가 없으면, 타입스크립트에 내장된 버전을 사용한다.

<br>

따라서 특정 lib 의 타입 선언을 커스터마이징하고 싶다면, `@typescript/lib-*` 이라는 alias로 커스터마이징된 버전을 설치하면 된다. 예를 들어서, 타입스크립트 팀에서는 DOM API 타입 선언을 `@types/web` 패키지로 배포하고 있다고 한다. 즉, 다음과 같이 하면 타입스크립트에서 내장한 DOM API 선언과 동일한 선언을 얻을 수 있게 되는데, 이렇게 의존성으로 명시하면, 의존성의 버전이 lock 파일에 고정되므로 타입스크립트를 업데이트해도 DOM API 의 버전은 처음 설치했을 때로 고정되게 된다. 즉, 타입스크립트 버전과는 별개로 원할 때 업데이트를 할 수 있는 것이다.

```typescript
{
 "dependencies": {
    "@typescript/lib-dom": "npm:@types/web"
  }
}
```

[https://www.npmjs.com/package/@types/web](https://www.npmjs.com/package/@types/web)

<br>

이렇게 `@typescript/lib-*` 패키지를 먼저 찾는 동작을 lib replacement라고 한다.

[https://github.com/microsoft/TypeScript/pull/45771](https://github.com/microsoft/TypeScript/pull/45771)

그리고 libReplacement를 할지 말지 여부를 `libReplacement` 설정으로 조절할 수 있다.

[https://www.typescriptlang.org/tsconfig/#libReplacement](https://www.typescriptlang.org/tsconfig/#libReplacement)

<br>

## 플랫폼별로 tsconfig.json 의 권장설정이 npm 패키지로 관리되고 있다.

tsconfig.json `target` 설명에 다음과 같은 내용이 있다.

[https://www.typescriptlang.org/tsconfig/#target:~:text=For%20developer%20platforms,upgrades%20less%20predictable.](https://www.typescriptlang.org/tsconfig/#target:~:text=For%20developer%20platforms,upgrades%20less%20predictable.)

> For developer platforms like Node there are baselines for the `target`, depending on the type of platform and its version. You can find a set of community organized TSConfigs at [tsconfig/bases](https://github.com/tsconfig/bases#centralized-recommendations-for-tsconfig-bases), which has configurations for common platforms and their versions.

<br>

플랫폼과 버전별로 baseline이 될 타입 설정에 대해 기술한 문서가 있다는 것인데, 다음 저장소다. 이는 커뮤니티가 관리하는 것으로 보인다.

<br>

### tsconfig/bases: Hosts TSConfigs to extend in a TypeScript app, tuned to a particular runtime environment

[https://github.com/tsconfig/bases#centralized-recommendations-for-tsconfig-bases](https://github.com/tsconfig/bases#centralized-recommendations-for-tsconfig-bases)

<br>

예를 들어 NodeJS 20 버전에서 실행할 예정이라면 `@tsconfig/node20` 패키지를 설치하면 된다. 

<br>

### @tsconfig/node20 - npm

[https://npmjs.com/package/@tsconfig/node20](https://npmjs.com/package/@tsconfig/node20)

```typescript
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "_version": "20.1.0",

  "compilerOptions": {
    "lib": ["es2023"],
    "module": "nodenext",
    "target": "es2022",

    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "node16"
  }
}
```

<br>

<br>

여기서 `lib`를 보면 NodeJS 20이 ES2023 까지 지원한다는 것을 추측할 수 있다.

[https://node.green/](https://node.green/) 사이트를 보면 대략 맞는 것 같다.

![](http://localhost:9425/images/16e04f40-debd-450b-9dd1-9654a7f3fce1.png)<br>

<br>

다양한 플랫폼(NodeJS, Bun, Deno 같은 런타임 뿐 아니라 NextJS, svelte 같은 프레임워크 관련된 베이스 설정도 있음)을 사용할 때 권장되는 tsconfig.json 설정을 npm 패키지로 배포해둔 것으로 `lib` 적절한 `lib`설정, `module`및 `target` 설정들이 들어있다. 이를 이용해 특정 플랫폼에서 권장되는 tsconfig 를 빠르게 설정할 수 있다.

(`@types/node` 이 제공하는 NodeJS 내장 타입은 이 패키지의 관심 대상은 아닌 듯하다.)

<br>

<br>

* * *

## 참고 자료들

<br>

[https://norux.me/59](https://norux.me/59)

<br>

```typescript
// lib.es2015.d.ts

/// <reference no-default-lib="true" />

/// <reference path="lib.es2015.core.d.ts" />
/// <reference path="lib.es2015.collection.d.ts" />
/// <reference path="lib.es2015.generator.d.ts" />
/// <reference path="lib.es2015.promise.d.ts" />
/// <reference path="lib.es2015.iterable.d.ts" />
/// <reference path="lib.es2015.proxy.d.ts" />
/// <reference path="lib.es2015.reflect.d.ts" />
/// <reference path="lib.es2015.symbol.d.ts" />
/// <reference path="lib.es2015.symbol.wellknown.d.ts" />
/// <reference path="lib.es5.d.ts" />
```

<br>

## \[Typescript\] tsconfig.json의 lib

[https://norux.me/59](https://norux.me/59)

> # \[Typescript\] tsconfig.json의 lib
>
> ## 1\. lib 옵션의 사용
>
> 타입스크립트가 빌드 될 때 참조하는 `tsconfig.json`의 컴파일 옵션중에 `lib`이라는 항목이 있습니다. 이 항목의 의미를 알아봅시다.
>
> ```
> // tsconfig.json
>
> {
>   "CompilerOptions": {
>     "target": "es5",
>     "module": "commonjs",
>     "lib": [
>       "dom",
>       "es5",
>       "es2015.promise"
>     ]
>   }
> }
> ```
>
> 이 `tsconfig.json` 에 등장하는 중간에 `lib`의 내용을 보면 배열형태로 사용할 라이브러리들을 정의하고 있습니다. 만약 `lib` 항목을 정의하지 않았다면 `target` 항목에서 지정한 ECMAScript의 버전에 따라 기본값이 정의됩니다.
>
> - **ES5의 기본 값:** `dom`, `es5`, `scripthost`
> - **ES6의 기본 값:** `dom`, `dom.iterable`, `es6`, `scripthost`
>
> 위의 기본 값 대신에 커스텀하게 라이브러리를 쓰려고 할 때, `lib`을 정의하여 사용합니다.
>
> 사용 예를 하나 들겠습니다. 위 예제처럼 빌드 될 `target`은 `es5`로 정의되어 있습니다. 그런데 작성한 코드나, 코드에서 참조하는 모듈들(`node_modules`)에서 ES6에서 등장한 `Promise` 를 사용하려면 `es2015.promise` 라는 라이브러리를 위처럼 정의하여 라이브러리 인젝션을 해줘야 합니다.
>
> ## 2\. lib 옵션에 사용할 수 있는 값들
>
> lib에는 사용할 수 있는 문자열은 아래와 같습니다.
>
> - ES5
> - ES6
> - ES2015
> - ES7
> - ES2016
> - ES2017
> - ESNext
> - DOM
> - DOM.Iterable
> - WebWorker
> - ScriptHost
> - ES2015.Core
> - ES2015.Collection
> - ES2015.Generator
> - ES2015.Iterable
> - ES2015.Promise
> - ES2015.Proxy
> - ES2015.Reflect
> - ES2015.Symbol
> - ES2015.Symbol.WellKnown
> - ES2016.Array.Include
> - ES2017.object
> - ES2017.SharedMemory
> - ES2017.TypedArrays
> - esnext.asynciterable
> - esnext.array
> - esnext.promise
>
> 아시겠지만, `ES6`는 `ES2015`와 동일하고 `ES7` 는 `ES2016` 와 동일합니다. 따라서 `ES6`를 사용하게 되면, `ES2015`와 동일하며, `ES2015.*` 즉, ES2015의 모든 항목들을 전부 로딩합니다.
>
> ## 3\. 실제로 참조하는 lib은 무엇일까?
>
> 코드를 분석하며 좀 더 딥하게 다이브(_Deep Dive_) 해봅시다.
>
> ### 3.1. lib 파일들은 어디에 있을까요?
>
> 보통 설치한 타입스크립트 모듈에 존재합니다. 예를 들어, `npm install -g typescript` 와 같이 글로벌 영역에 타입스크립트를 설치 했다면, Linux/macOS 에서는 `/usr/local/lib/node_modules/typescript` 에 설치가 됩니다. 죄송하지만 Windows에서는 잘 모르겠습니다. ^\_^;
>
> ```
> $ ls -l /usr/local/lib/node_modules/typescript/lib
> ... (중략)
> lib.d.ts
> lib.dom.d.ts
> ...
> lib.es2015.d.ts
> lib.es2015.promise.d.ts
> ...
> ```
>
> 위 처럼 아까 보셨던 문자열과 거의 동일한 이름으로 `lib` 파일이 정의되어 있습니다. 파일들을 열어보면 ECMAScript 규격에 정의된 자바스크립트 객체들에 대한 인터페이스들이 정의되어 있는 것을 보실 수 있습니다.
>
> ```
> // lib.es5.d.ts
>
> ... (중략)
>
> interface Function {
>   apply(this: Function, thisArgs: any, argArray?: any): any;
>   call(this: Function, thisArgs: any, ...argArray: any[]): any;
>   ...
> }
>
> interface Number {
>   toString(radix?: number): string;
>   ...
>   valueOf(): number;
> }
>
> ...
> ```
>
> 위 처럼 원시 타입(_Primitive Type_)의 인터페이스들도 정의되어 있구요. `Math` 같은 자바스크립트 내장 객체들의 인터페이스도 정의되어 있습니다.
>
> ### 3.2. 라이브러리 로딩 과정
>
> 이제 `tsc.js` 의 코드를 보며 실제 컴파일 과정에서 라이브러리를 읽는 과정을 잠깐 보겠습니다.
>
> ```
> //tsc.js
>
> {
>   name: "target",
>   shortName: "t",
>   type: ts.createMapFromTemplate({
>     "es3": 0,
>     "es5": 1,
>     "es6": 2,
>     "es2015": 2,
>     "es2016": 3,
>     "es2017": 4,
>     "esnext": 5
>   }),
>   ...(후략)
> },
> {
>   name: "lib",
>   type: "list",
>   element: {
>     name: "lib",
>     type: ts.createMapFromTemplate({
>       "es5": "lib.es5.d.ts",
>       "es6": "lib.es2015.d.ts",
>       "es2015": "lib.es2015.d.ts",
>       "es7": "lib.es2016.d.ts",
>       ...
>       "es2015.core": "lib.es2015.core.d.ts",
>       "es2015.generator": "lib.es2015.generator.d.ts",
>        ...(후략)
>     }),
>   }
>   ...(후략)
> }
> ```
>
> 위 코드는 `tsc.js` 컴파일러가 옵션을 파싱하는 부분입니다. `target` 옵션과 `lib` 옵션에 대해 위처럼 매핑되어있습니다. `lib.es2015.d.ts` 파일을 잠깐 열어보면요.
>
> ```typescript
> // lib.es2015.d.ts
>
> /// <reference no-default-lib="true" />
>
> /// <reference path="lib.es2015.core.d.ts" />
> /// <reference path="lib.es2015.collection.d.ts" />
> /// <reference path="lib.es2015.generator.d.ts" />
> /// <reference path="lib.es2015.promise.d.ts" />
> /// <reference path="lib.es2015.iterable.d.ts" />
> /// <reference path="lib.es2015.proxy.d.ts" />
> /// <reference path="lib.es2015.reflect.d.ts" />
> /// <reference path="lib.es2015.symbol.d.ts" />
> /// <reference path="lib.es2015.symbol.wellknown.d.ts" />
> /// <reference path="lib.es5.d.ts" />
> ```
>
> 보시다시피 XML형식으로 다른 `ts` 파일들을 참조하도록 되어있습니다. 실제 구현부는 `lib.es2015.core.d.ts` 같은 파일들에 있습니다.
>
> ```typescript
> // tsc.js
>
> function getDefaultLibFileName(options) {
>   switch (options.target) {
>     case 5: return "lib.esnext.full.d.ts";
>     case 4: return "lib.es2017.full.d.ts";
>     case 3: return "lib.es2016.full.d.ts";
>     case 2: return "lib.es6.d.ts";
>     default: return "lib.d.ts";
>   }
> }
> ```
>
> 이 함수는 서두에 말씀 드렸던 `lib`을 정의하지 않았을 때, `target` 에 대한 기본 라이브러리를 읽어오는 부분입니다. `lib.es2015.d.ts` 파일이 XML 의 참조 형식을 빌렸기 때문에, 실제 구현부의 집합인 `lib.es6.d.ts`라는 파일을 추가 정의하여 사용하는것 같습니다. `lib.es6.d.ts` 파일은 `lib.es2015.*.d.ts` 의 모든 내용을 합친것과 같습니다. 파일이름에 `full` 이 붙은 `esnext`, `es2017`, `es2016` 역시 같은 맥락입니다.
>
> ## 4\. 결론
>
> 타입스크립트를 빌드하며 `lib` 이 도대체 뭘까 궁금했습니다. 그러다가 [rxjs](http://reactivex.io/rxjs/)를 공부하는 와중에 `target: "es5"` 로 컴파일러 옵션을 주고 빌드를 하면 다음과 같은 에러가 발생하는 문제때문에 딥하게 공부해보기 시작했습니다.
>
> ```
> node_modules/@reactivex/rxjs/dist/package/Observable.d.ts(58,60): error TS2693: 'Promise' only refers to a type, but is being used as a value here.
> node_modules/@reactivex/rxjs/dist/package/Observable.d.ts(73,59): error TS2693: 'Promise' only refers to a type, but is being used as a value here.
> ```
>
> 일단은 `lib: ["es5", "dom", "es2015.promise"]` 혹은 `lib: ["es6", "dom"]` 으로 해결이 되는 문제였습니다. 어떻게 `lib`을 사용하는것이 효과적인지, 두 방법간에 어떤 차이가 있는지 궁금해서 시작하게 된 공부였습니다.
>
> 제 생각은 성능상에는 큰 상관 없을 것 같습니다. ^^ `es2015.promise` 처럼 라이브러리를 인젝션하면 모듈화를 할때 조금 더 효율적이긴 할 것 같지만 성능상에 큰 문제를 발생시킬것 같지는 않습니다. 아무래도 공부가 더 필요한 모양입니다.
>
> 문의사항이나 의견이 있으면 언제든 무엇이든 댓글 남겨주세요.
>
> <br>

<br>

### <br>

* * *

## tsconfig.json에서 lib를 명시하지 않았을 때의 기본값?

<br>

### tsc - What is the typescript compiler's default lib value? - Stack Overflow

[https://stackoverflow.com/questions/63943629/what-is-the-typescript-compilers-default-lib-value](https://stackoverflow.com/questions/63943629/what-is-the-typescript-compilers-default-lib-value)

<br>

> In the source code there is a [list of supported libs](https://github.com/microsoft/TypeScript/blob/663b19fe4a7c4d4ddaa61aedadd28da06acd27b6/src/lib/libs.json#L56-L64), which has a section called "Default libraries":
>
> ```
>         // Default libraries
>         "es5.full",
>         "es2015.full",
>         "es2016.full",
>         "es2017.full",
>         "es2018.full",
>         "es2019.full",
>         "es2020.full",
>         "es2021.full",
>         "esnext.full"
> ```
>
> These correspond to your target setting.
>
> Looking at the source code of one of these ([lib.es2017.full.d.ts](https://github.com/microsoft/TypeScript/blob/663b19fe4a7c4d4ddaa61aedadd28da06acd27b6/lib/lib.es2017.full.d.ts)), you can see what it imports:
>
> ```
> /// <reference lib="es2017" />
> /// <reference lib="dom" />
> /// <reference lib="webworker.importscripts" />
> /// <reference lib="scripthost" />
> /// <reference lib="dom.iterable" />
> ```
>
> And FYI, all language levels import the previous. So for example: es2018 imports es2017, which imports es2016, which imports es2015, which imports es5. "es6" is unique because nothing imports it.
>
> There appears to be no "es3" lib file in the source code, so I don't have an answer for that. Try setting `noLib`, see what breaks, and comment below.
>
> <br>

<br>

### javascript - Why are dom and dom.iterable separate? - Stack Overflow

[https://stackoverflow.com/questions/71510368/why-are-dom-and-dom-iterable-separate](https://stackoverflow.com/questions/71510368/why-are-dom-and-dom-iterable-separate)

> I suspect it's because iteration (iterables and iterators) weren't added to JavaScript until ES2015, long after TypeScript was well-established. When ES2015 was released, the JavaScript engines in many browsers didn't support iteration (yet),¹ and of course at the time Internet Explorer was still a...thing...that was never going to get new features. So some projects had to target environments that didn't have iteration, so the libs are separate.
>
> Even here in 2022, IE survives in corporate and government installations (although thankfully that's _finally_ changing), and some folks have to target their apps and pages to those environments, so they wouldn't want to use `dom.iterable`.
>
> * * *
>
> ¹ ES2015 was the last version of the specification to include significant features that didn't already have implementations in the field. These days, [the process](https://tc39.es/process-document/) that TC39 follows generally doesn't land features in the spec until there are (ideally) a couple of implementations of the feature shipping in the field. Instead, the proposal stays at Stage 3 until that happens, before being moved to Stage 4 by consensus at a TC39 meeting, being added to the [editor's draft](https://tc39.es/ecma262/multipage/), and thus being in the next snapshot specification the following June.
>
> <br>

<br>

> I think worth mentioning. And for some of the people that may land here. And to bring some clarification around other `lib` options. As the **same concept of separation is not applied**.
>
> Resume:
>
> > `dom`, `dom.iterable` are **separate**. The **rest of the libs** are **inclusive**. `es2022` includes the rest of all `es2022.*` variants (except for full. `es2022.full` includes `es2022 + others`)
>
> ## dom and dom.iterable
>
> `dom` doesn't include `dom.iterable`. And here is why:
>
> And as it can be shown here \[[lib.dom.iterable.d.ts#L37](https://github.com/microsoft/TypeScript/blob/2a91107f7548eeb5e32673e76277d27264ea88e2/lib/lib.dom.iterable.d.ts#L37), [lib.dom.d.ts#L2726](https://github.com/microsoft/TypeScript/blob/2a91107f7548eeb5e32673e76277d27264ea88e2/lib/lib.dom.d.ts#L2726)\] for example:
>
> [![enter image description here](https://i.sstatic.net/bFPxd.png)](https://i.sstatic.net/bFPxd.png)
>
> You can see the iterable extends the dom lib and add iterable implementation.
>
> ## What about other libs
>
> `es2022`, `es2022.array`, ....
>
> The **same concept doesn't apply**. `es2022` includes all the others as shown here:
>
> [lib.es2022.d.ts](https://github.com/microsoft/TypeScript/blob/main/lib/lib.es2022.d.ts)
>
> es2022 file with stripped comments
>
> ```
> /// <reference no-default-lib="true"/>
>
>
> /// <reference lib="es2021" />
> /// <reference lib="es2022.array" />
> /// <reference lib="es2022.error" />
> /// <reference lib="es2022.intl" />
> /// <reference lib="es2022.object" />
> /// <reference lib="es2022.sharedmemory" />
> /// <reference lib="es2022.string" />
> ```
>
> And the **full version (es2022.full)** does even more:
>
> [lib.es2022.full.d.ts](https://github.com/microsoft/TypeScript/blob/main/lib/lib.es2022.full.d.ts)
>
> It does include even `dom`, `dom.iterable`, and more. Meaning if you use the full version. You don't need to include `dom` nor `dom.iterable`.
>
> ```
> /// <reference no-default-lib="true"/>
>
>
> /// <reference lib="es2022" />
> /// <reference lib="dom" />
> /// <reference lib="webworker.importscripts" />
> /// <reference lib="scripthost" />
> /// <reference lib="dom.iterable" />
> ```
>
> <br>

<br>

---

# TypeScript tsconfig.json의 lib 에 넣을 수 있는 값 목록

* * *

<br>

이쪽에 정의된 것들을 넣을 수 있다.

<br>

### TypeScript/src/lib at main · microsoft/TypeScript

[https://github.com/microsoft/TypeScript/tree/main/src/lib](https://github.com/microsoft/TypeScript/tree/main/src/lib)

<br>

VSCode 에서 tsconfig.json Spec 검사할 때 쓰는 JSON 스키마

### JSON editing in Visual Studio Code

[https://code.visualstudio.com/docs/languages/json#\_intellisense-and-validation](https://code.visualstudio.com/docs/languages/json#_intellisense-and-validation)

> We also perform structural and value verification based on an associated JSON schema giving you red squiggles. To disable validation, use the `json.validate.enable` [setting](https://code.visualstudio.com/docs/getstarted/settings).
>
> <br>

<br>

### JSON Schema

[https://json-schema.org/](https://json-schema.org/)

<br>

### JSON Schema Store

[https://www.schemastore.org/json/](https://www.schemastore.org/json/)

[https://json.schemastore.org/tsconfig.json](https://json.schemastore.org/tsconfig.json)

[https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/tsconfig.json](https://github.com/SchemaStore/schemastore/blob/master/src/schemas/json/tsconfig.json)

### tsconfig.json  JSON Schema 의 `lib` 부분

```
{
  // ...중략
  "lib": {
    "$comment": "The value of 'null' is UNDOCUMENTED (https://github.com/microsoft/TypeScript/pull/18058).",
    "description": "Specify a set of bundled library declaration files that describe the target runtime environment.",
    "type": ["array", "null"],
    "uniqueItems": true,
    "items": {
      "$comment": "The value of 'null' is UNDOCUMENTED (https://github.com/microsoft/TypeScript/pull/18058).",
      "type": ["string", "null"],
      "anyOf": [
        {
          "enum": [
            "ES5",
            "ES6",
            "ES2015",
            "ES2015.Collection",
            "ES2015.Core",
            "ES2015.Generator",
            "ES2015.Iterable",
            "ES2015.Promise",
            "ES2015.Proxy",
            "ES2015.Reflect",
            "ES2015.Symbol.WellKnown",
            "ES2015.Symbol",
            "ES2016",
            "ES2016.Array.Include",
            "ES2017",
            "ES2017.Intl",
            "ES2017.Object",
            "ES2017.SharedMemory",
            "ES2017.String",
            "ES2017.TypedArrays",
            "ES2017.ArrayBuffer",
            "ES2018",
            "ES2018.AsyncGenerator",
            "ES2018.AsyncIterable",
            "ES2018.Intl",
            "ES2018.Promise",
            "ES2018.Regexp",
            "ES2019",
            "ES2019.Array",
            "ES2019.Intl",
            "ES2019.Object",
            "ES2019.String",
            "ES2019.Symbol",
            "ES2020",
            "ES2020.BigInt",
            "ES2020.Promise",
            "ES2020.String",
            "ES2020.Symbol.WellKnown",
            "ESNext",
            "ESNext.Array",
            "ESNext.AsyncIterable",
            "ESNext.BigInt",
            "ESNext.Collection",
            "ESNext.Intl",
            "ESNext.Object",
            "ESNext.Promise",
            "ESNext.Regexp",
            "ESNext.String",
            "ESNext.Symbol",
            "DOM",
            "DOM.AsyncIterable",
            "DOM.Iterable",
            "ScriptHost",
            "WebWorker",
            "WebWorker.AsyncIterable",
            "WebWorker.ImportScripts",
            "Webworker.Iterable",
            "ES7",
            "ES2021",
            "ES2020.SharedMemory",
            "ES2020.Intl",
            "ES2020.Date",
            "ES2020.Number",
            "ES2021.Promise",
            "ES2021.String",
            "ES2021.WeakRef",
            "ESNext.WeakRef",
            "ES2021.Intl",
            "ES2022",
            "ES2022.Array",
            "ES2022.Error",
            "ES2022.Intl",
            "ES2022.Object",
            "ES2022.String",
            "ES2022.SharedMemory",
            "ES2022.RegExp",
            "ES2023",
            "ES2023.Array",
            "ES2024",
            "ES2024.ArrayBuffer",
            "ES2024.Collection",
            "ES2024.Object",
            "ES2024.Promise",
            "ES2024.Regexp",
            "ES2024.SharedMemory",
            "ES2024.String",
            "Decorators",
            "Decorators.Legacy",
            "ES2017.Date",
            "ES2023.Collection",
            "ESNext.Decorators",
            "ESNext.Disposable"
          ]
        },
        {
          "pattern": "^[Ee][Ss]5|[Ee][Ss]6|[Ee][Ss]7$"
        },
        {
          "pattern": "^[Ee][Ss]2015(\\.([Cc][Oo][Ll][Ll][Ee][Cc][Tt][Ii][Oo][Nn]|[Cc][Oo][Rr][Ee]|[Gg][Ee][Nn][Ee][Rr][Aa][Tt][Oo][Rr]|[Ii][Tt][Ee][Rr][Aa][Bb][Ll][Ee]|[Pp][Rr][Oo][Mm][Ii][Ss][Ee]|[Pp][Rr][Oo][Xx][Yy]|[Rr][Ee][Ff][Ll][Ee][Cc][Tt]|[Ss][Yy][Mm][Bb][Oo][Ll]\\.[Ww][Ee][Ll][Ll][Kk][Nn][Oo][Ww][Nn]|[Ss][Yy][Mm][Bb][Oo][Ll]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2016(\\.[Aa][Rr][Rr][Aa][Yy]\\.[Ii][Nn][Cc][Ll][Uu][Dd][Ee])?$"
        },
        {
          "pattern": "^[Ee][Ss]2017(\\.([Ii][Nn][Tt][Ll]|[Oo][Bb][Jj][Ee][Cc][Tt]|[Ss][Hh][Aa][Rr][Ee][Dd][Mm][Ee][Mm][Oo][Rr][Yy]|[Ss][Tt][Rr][Ii][Nn][Gg]|[Tt][Yy][Pp][Ee][Dd][Aa][Rr][Rr][Aa][Yy][Ss]|[Dd][Aa][Tt][Ee]|[Aa][Rr][Rr][Aa][Yy][Bb][Uu][Ff][Ff][Ee][Rr]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2018(\\.([Aa][Ss][Yy][Nn][Cc][Gg][Ee][Nn][Ee][Rr][Aa][Tt][Oo][Rr]|[Aa][Ss][Yy][Nn][Cc][Ii][Tt][Ee][Rr][Aa][Bb][Ll][Ee]|[Ii][Nn][Tt][Ll]|[Pp][Rr][Oo][Mm][Ii][Ss][Ee]|[Rr][Ee][Gg][Ee][Xx][Pp]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2019(\\.([Aa][Rr][Rr][Aa][Yy]|[Ii][Nn][Tt][Ll]|[Oo][Bb][Jj][Ee][Cc][Tt]|[Ss][Tt][Rr][Ii][Nn][Gg]|[Ss][Yy][Mm][Bb][Oo][Ll]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2020(\\.([Bb][Ii][Gg][Ii][Nn][Tt]|[Pp][Rr][Oo][Mm][Ii][Ss][Ee]|[Ss][Tt][Rr][Ii][Nn][Gg]|[Ss][Yy][Mm][Bb][Oo][Ll]\\.[Ww][Ee][Ll][Ll][Kk][Nn][Oo][Ww][Nn]|[Ss][Hh][Aa][Rr][Ee][Dd][Mm][Ee][Mm][Oo][Rr][Yy]|[Ii][Nn][Tt][Ll]|[Dd][Aa][Tt][Ee]|[Nn][Uu][Mm][Bb][Ee][Rr]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2021(\\.([Ii][Nn][Tt][Ll]|[Pp][Rr][Oo][Mm][Ii][Ss][Ee]|[Ss][Tt][Rr][Ii][Nn][Gg]|[Ww][Ee][Aa][Kk][Rr][Ee][Ff]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2022(\\.([Aa][Rr][Rr][Aa][Yy]|[Ee][Rr][Rr][Oo][Rr]|[Ii][Nn][Tt][Ll]|[Oo][Bb][Jj][Ee][Cc][Tt]|[Ss][Tt][Rr][Ii][Nn][Gg]|[Ss][Hh][Aa][Rr][Ee][Dd][Mm][Ee][Mm][Oo][Rr][Yy]|[Rr][Ee][Gg][Ee][Xx][Pp]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2023(\\.([Aa][Rr][Rr][Aa][Yy]|[Cc][Oo][Ll][Ll][Ee][Cc][Tt][Ii][Oo][Nn]))?$"
        },
        {
          "pattern": "^[Ee][Ss]2024(\\.([Aa][Rr][Rr][Aa][Yy][Bb][Uu][Ff][Ff][Ee][Rr]|[Cc][Oo][Ll][Ll][Ee][Cc][Tt][Ii][Oo][Nn]|[Oo][Bb][Jj][Ee][Cc][Tt]|[Pp][Rr][Oo][Mm][Ii][Ss][Ee]|[Rr][Ee][Gg][Ee][Xx][Pp]|[Ss][Hh][Aa][Rr][Ee][Dd][Mm][Ee][Mm][Oo][Rr][Yy]|[Ss][Tt][Rr][Ii][Nn][Gg]))?$"
        },
        {
          "pattern": "^[Ee][Ss][Nn][Ee][Xx][Tt](\\.([Aa][Rr][Rr][Aa][Yy]|[Aa][Ss][Yy][Nn][Cc][Ii][Tt][Ee][Rr][Aa][Bb][Ll][Ee]|[Bb][Ii][Gg][Ii][Nn][Tt]|[Ii][Nn][Tt][Ll]|[Pp][Rr][Oo][Mm][Ii][Ss][Ee]|[Ss][Tt][Rr][Ii][Nn][Gg]|[Ss][Yy][Mm][Bb][Oo][Ll]|[Ww][Ee][Aa][Kk][Rr][Ee][Ff]|[Dd][Ee][Cc][Oo][Rr][Aa][Tt][Oo][Rr][Ss]|[Dd][Ii][Ss][Pp][Oo][Ss][Aa][Bb][Ll][Ee]))?$"
        },
        {
          "pattern": "^[Dd][Oo][Mm](<.([Aa][Ss][Yy][Nn][Cc])?[Ii][Tt][Ee][Rr][Aa][Bb][Ll][Ee]>)?$"
        },
        {
          "pattern": "^[Ss][Cc][Rr][Ii][Pp][Tt][Hh][Oo][Ss][Tt]$"
        },
        {
          "pattern": "^[Ww][Ee][Bb][Ww][Oo][Rr][Kk][Ee][Rr](<.([Ii][Mm][Pp][Oo][Rr][Tt][Ss][Cc][Rr][Ii][Pp][Tt][Ss]|([Aa][Ss][Yy][Nn][Cc])?[Ii][Tt][Ee][Rr][Aa][Bb][Ll][Ee])>)?$"
        },
        {
          "pattern": "^[Dd][Ee][Cc][Oo][Rr][Aa][Tt][Oo][Rr][Ss](\\.([Ll][Ee][Gg][Aa][Cc][Yy]))?$"
        }
      ]
    },
    "markdownDescription": "Specify a set of bundled library declaration files that describe the target runtime environment.\n\nSee more: https://www.typescriptlang.org/tsconfig#lib"
  },
  // ...중략
}
```

<br>

<br>

<br>
