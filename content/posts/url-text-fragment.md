+++
date = "2026-03-24T10:59:38+09:00"
draft = true
title = 'URL Text Fragment 문법에 대해서'
+++

upnote://x-callback-url/openNote?noteId=019d1560-c4d7-7778-aed4-91a781b05c3b

# URL의 Text Fragment 문법

* * *

[https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text\_fragments](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments)

<br>

```bash
https://example.com#:~:text=[prefix-,]textStart[,textEnd][,-suffix]
                    ---===== -------  =========  -------   =======
                    (1) (2)   (3)       (4)        (5)       (6)
```

<br>

> Text fragments are a kind of URL fragment, and is written after the `#`. The key parts to understand are as follows:
>
> [`:~:`](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments#sect)
>
> Otherwise known as _the fragment directive_, this sequence of characters tells the browser that what comes next is one or more user-agent instructions, which are stripped from the URL during loading so that author scripts cannot directly interact with them. User-agent instructions are also called directives.
>
> [`text=`](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments#text)
>
> A text directive. This provides a text fragment to the browser, defining what text is to be linked to in the linked document.
>
> [`textStart`](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments#textstart)
>
> A text string specifying the start of the linked text.
>
> [`textEnd` Optional](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments#textend)
>
> A text string specifying the end of the linked text.
>
> [`prefix-` Optional](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments#prefix-)
>
> A text string followed by a hyphen specifying what text should immediately precede the linked text, only allowing for whitespace in between. This helps the browser to select the correct linked text, in cases where there are multiple matches.
>
> [`-suffix` Optional](https://developer.mozilla.org/en-US/docs/Web/URI/Reference/Fragment/Text_fragments#-suffix)
>
> A hyphen followed by a text string specifying what text should immediately follow the linked text, only allowing for whitespace in between. This helps the browser to select the correct linked text, in cases where there are multiple matches.
