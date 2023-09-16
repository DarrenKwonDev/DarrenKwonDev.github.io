+++
title = 'post place holder'
date = 2023-09-16T16:20:16+09:00
math = true
toc = true
bold = true
tags = ["tag1", "tag2"]
next = true
draft = false
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

all image lazy loaded

![what](./test.png)
![oh](./output.jpg)

### comment(footnote)

Here's a sentence with a footnote.[^1]

[^1]: This is unnamed footnote.

### box

check this : https://gohugo.io/content-management/syntax-highlighting/#highlight-hugogo-template-code

{{ < box info >}}
**This is a bold line**

Hello there, and have a nice day
{{ < /box >}}

<img src="./test.png" />

### collapse

<details>
  <summary>클릭하여 내용 보기/감추기</summary>
  
  이곳에 숨겨진 내용을 작성하세요.
</details>
