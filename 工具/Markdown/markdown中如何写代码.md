---
date: 2023-11-20 09:57
modified: 2023-12-27 09:29
---

在 markdown 中添加组件，如：[示例](https://www.redblobgames.com/making-of/draggable/)

![[Pasted image 20231120100141.png]]

如何做到？

使用 [mdx](https://mdxjs.com/)

代码：
```js
import {Chart} from './snowfall.js'
export const year = 2023

# Last year’s snowfall

In {year}, the snowfall was above average. It was followed by a warm spring which caused flood conditions in many of the nearby rivers.

<Chart year={year} color="#fcb32c" />
```

渲染：

![[Pasted image 20231120100345.png|350]]
