---
date: 2023-12-27 09:30
modified: 2023-12-30 11:57
---

Obsidian 扩展了 Markdown 语法，其中之一便是 Callouts（标注）

# 语法

```markdown
> [!类型] 标题
> 正文
```

# 支持的类型

除非 [[#自定义标注|自定义标注]]，否则任何不受支持的类型都默认为 `note` 类型。类型标识符不区分大小写

> [!note] 备注
> 
> ```markdown
> > [!note] 备注
> > BaLa BaLa
> ```

> [!abstract] 摘要
> 
> ```markdown
> > [!abstract] 摘要
> > 别名：summary、tldr
> ```

> [!info] 信息
> 
> ```markdown
> > [!info] 信息
> > BaLa BaLa
> ```

> [!todo] TODO
> 
> ```markdown
> > [!todo] TODO
> > BaLa BaLa
> ```

> [!tip] 小提示
> 
> ```markdown
> > [!tip] 小提示
> > 别名：hint、important
> ```

> [!success] 成功
> 
> ```markdown
> > [!success] 成功
> > 别名：check、done
> ```

> [!failure] 失败
> 
> ```markdown
> > [!failure] 失败
> > 别名：fail、missing
> ```

> [!question] 问题
> 
> ```markdown
> > [!question] 问题
> > 别名：help、faq
> ```

> [!warning] 警告
> 
> ```markdown
> > [!warning] 警告
> > 别名：caution、attention
> ```

> [!danger] 危险
> 
> ```markdown
> > [!danger] 危险
> > 别名：error
> ```

> [!bug] BUG
> 
> ```markdown
> > [!bug] BUG
> > BaLa BaLa
> ```

> [!example] 例子
> 
> ```markdown
> > [!example] 例子
> > BaLa BaLa
> ```

> [!quote] 引用
> 
> ```markdown
> > [!quote] 引用
> > 别名：cite
> ```

# 可折叠标注

通过在类型标识符后直接添加加号 （`+`） 或减号 （`-`） 来使注解可折叠。加号会展开标注，减号会折叠标注

```
> [!faq]- 


> [!node]+
```

# 嵌套标注

可以将标注嵌套在多个级别中

```markdown
> [!question] Can callouts be nested?
> > [!todo] Yes!, they can.
> > > [!example]  You can even use multiple layers of nesting.
```

# 自定义标注

CSS 片段和社区插件可以定义自定义标注，甚至可以覆盖默认配置

要定义自定义标注，请创建以下 CSS 块：

```css
.callout[data-callout="custom-question-type"] {
    --callout-color: 0, 0, 0;
    --callout-icon: lucide-alert-circle;
}
```

- `data-callout` 属性的值是要使用的类型标识符，例如 `[!custom-question-type]`
- `--callout-color` 使用数字（0–255）定义红色、绿色和蓝色的背景颜色
- `--callout-icon` 可以是来自 [lucide.dev](https://lucide.dev/) 的图标 ID，也可以是 SVG 元素
	- 使用 SVG：`-callout-icon: '<svg>...custom svg...</svg>';`
