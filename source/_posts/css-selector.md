---
title: CSS 选择器
date: 2019-06-11 07:38:54
tags: CSS
---

> CSS 选择器

<!-- more -->

1，id选择 

```html
#my_input {
    color:red;
}

<input id="my_input" >

```

2，class选择

```html
.my_input {
    color:red;
}

<input class="my_input" >

```

3，所有后代选择，包括子元素，子子元素

```html
div span {
    color:red;
}

<div>
    <span>子元素变红</span>
    <p><span>子子元素变红</span></p>
</div>


```

4，仅限子元素选择，无法深度嵌套选择

```html
div>p {
    color:red;
}
<div>
    <span>子元素变红</span>
    <p><span>子子元素无效</span></p>
</div>
```

5，属性选择器，包含 name 属性的标签

```html

[name] {
    color:red;
}

<input name="my_input">
```

6，属性和指定值选择器

```html
[name=password]{
    color:red;
}

<input name="password">

```

7，属性值模糊匹配选择器

```html
[name*=password]{
    color:red;
}

<input name="password">
<input name="password_confirm">

```


8，标签和属性选择器

```html
input[name='password']{
    color:red;
}

<input name="password">

```


9，属性值空格分隔选择器

```html
[title~=one]{
    color:red;
}

<a title="one">
<a title="one two">

```

10，属性值中杠连接符选择器

```html
[title|=one]{
    color:red;
}

<a title="one">
<a title="one-two">
```

11，属性值正则匹配选择器

```html
[title^=one]{
    color:red;
}

<a title="onetwo">


[title$=two]{
    color:red;
}
<a title="onetwo">

```