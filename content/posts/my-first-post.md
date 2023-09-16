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

### second intro

wow.

this is code block

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

![image](./test.png)

### comment(footnote)

Here's a sentence with a footnote.[^1]

[^1]: This is unnamed footnote.

### box

{{ < box info >}}  
**This is a bold line**

Hello there, and have a nice day  
{{ < /box >}}
