+++
date = "2026-03-22T16:55:40+09:00"
draft = true
title = 'Details Summary Shadow Dom'
+++

upnote://x-callback-url/openNote?noteId=019d1473-1302-7684-b6f7-c1d37c0fd474


### HTML 코드

```xml
<details>
  <summary>More Info</summary>
  <p>This is the detailed information that is hidden by default.</p>
</details>

```

<br>

위와 같이 details와 summary 태그를 쓰면, 접었다 펴는 태그가 자동으로 만들어진다.

![](http://localhost:9425/images/019d1478-e3ac-77ed-b179-d4e9ddeb27ec.png)

이를 크롬 devtool에서 열어보면, 다음과 같이 표시가 된다. details 가 열려 있을 때는 `open` Attribute가 붙어 있다. 그리고 `slot` 이라는 태그가 붙어 있다. 이 slot은 Shadow DOM에서 slotted 된, 즉, Shadow DOM 내의 slot 안으로 옮겨서 렌더된 Light DOM 엘리먼트에 붙는 태그다.

### Light DOM

```xml
<details open="">
  <summary>More Info</summary>
  <p>This is the detailed information that is hidden by default.</p>
</details>
```

![](http://localhost:9425/images/019d1476-7337-713a-b454-3a45b7d6a251.png)

<br>

사실 details는 내부적으로 Shadow DOM을 쓰는 태그라고 한다. 이는 브라우저가 자체적으로 만든 Shadow DOM인데 모양을 살펴보면 다음과 같다. 

### Shadow DOM

다음은 Shadow DOM 모양만 표시한 것이다. 여기에는 두 개의 `slot` 이 있는데, 하나는 `summary` 태그가 들어올 곳이고, 하나는 나머지 자식들이 들어올 곳이다.

```xml
<slot id="details-summary">
  <summary>세부정보</summary>
</slot>
<slot id="details-content" pseudo="details-content" style="display: block"></slot>
<style>
  :host summary {
    display: list-item;
    counter-increment: list-item 0;
    list-style: disclosure-closed inside;
  }

  :host([open]) summary {
    list-style-type: disclosure-open;
  }
</style>


```

<br>

여기에 다음과 같이 Light DOM에서 온 것들이 slotted 되면서 접을 수 있는 섹션이 되는 것이다.

![](http://localhost:9425/images/019d1473-b7f3-7516-89db-cbb06b01652c.png)

![](http://localhost:9425/images/019d1487-ad9d-72b1-a394-a9635bd39d0b.png)

위와 같이 브라우저에서 만든 Shadow DOM은 기본적으로는 표시가 안 되는데 devtool 설정으로 표시되게 바꿀 수 있다. 개발자 도구 설정에서 

`Preferences` → `Elements` → `Show user agentn shadow DOM` 으로 설정을 켜고 끌 수 있다.

![](http://localhost:9425/images/019d1475-ea62-746b-9434-6572def816e3.png)

<br>
