+++
title = 'post placeholder and examples'
date = 2023-09-16T16:20:16+09:00
math = true
toc = true
bold = true
draft = false
tags = ["tag1", "tag2"]
+++

## Introduction

This is **bold** text, and this is _emphasized_ text.

Visit the [Hugo](https://gohugo.io) website!

### html tags (set renderer first!)

<div>
	<mark>and this is mark tag!</mark>
	<u>this is u tag!</u>
</div>
<div>aksjfd;lajs;fjalskjflk;asdjasfjhaksdhfjaksdhf</div>

### code block

wow.

this is code block.  
[highlight](https://gohugo.io/content-management/syntax-highlighting/)  
[highlighting-in-code-fences](https://gohugo.io/content-management/syntax-highlighting/#highlighting-in-code-fences)

```go {linenos=inline,hl_lines=[2, "6-9"],linenostart=19}
package main

import "fmt"

func main() {
	fmt.Println("Hello, 世界")
}
// ... code
```

### math katex

$$
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\\
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
$$

### img

all image lazy loaded.
[page scoped img](https://gohugo.io/content-management/page-resources/)

![what](./test.png)

### file tress

```
.
├── 01_helloworld
│   ├── hello_world.c
│   └── 함수_선언_정의_차이점.c
├── 02_자료형
│   ├── bool_data_type.c
│   ├── enum_data_type.c
│   ├── float_data_type.c
│   └── int_data_type.c
├── 03_변수_연산
│   └── declare_var.c
├── README.md
└── a.out
```

### table

| wow | col1 | col2 | col3 | col 4 |
| --- | ---- | ---- | ---- | ----- |
|     | 1    | 2    | 3    | 4     |
|     | 11   | 22   | 33   | 44    |

### comment(footnote)

Here's a sentence with a footnote.[^1]

[^1]: This is unnamed footnote.

### box

check this : https://gohugo.io/content-management/syntax-highlighting/#highlight-hugogo-template-code

info, tip, warning, important

tip
{{< box tip >}}

Hello there, and have a nice day
{{< /box >}}

info
{{< box info >}}

Hello there, and have a nice day
{{< /box >}}

warning
{{< box warning >}}

Hello there, and have a nice day
{{< /box >}}

important
{{< box important >}}

Hello there, and have a nice day
{{< /box >}}

### collapse

<details>
  <summary>클릭하여 내용 보기/감추기</summary>
  
  이곳에 숨겨진 내용을 작성하세요.
</details>

### blockquote

<blockquote>
some quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdfsome quote  asdf
some quote  
some quote  
some quote  
some quote  
some quote  
some quote  
some quote  
some quote  
</blockquote>
