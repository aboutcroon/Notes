## 简介

The `position` property can help you manipulate the location of an element



There are **six values** for the `position` property：

- `static`: every element has a static position by default, so the element will stick to the normal page flow. So if there is a [left](https://css-tricks.com/almanac/properties/l/left/)/[right](https://css-tricks.com/almanac/properties/r/right/)/[top](https://css-tricks.com/almanac/properties/t/top/)/[bottom](https://css-tricks.com/almanac/properties/b/bottom/)/[z-index](https://css-tricks.com/almanac/properties/z/z-index/) set then there will be no effect on that element.
- `relative`: an element’s original position remains in the flow of the document, just like the `static` value. But now [left](https://css-tricks.com/almanac/properties/l/left/)/[right](https://css-tricks.com/almanac/properties/r/right/)/[top](https://css-tricks.com/almanac/properties/t/top/)/[bottom](https://css-tricks.com/almanac/properties/b/bottom/)/[z-index](https://css-tricks.com/almanac/properties/z/z-index/) will work. The positional properties “nudge” the element from the original position in that direction.
- `absolute`: the element is removed from the flow of the document and other elements will behave as if it’s not even there whilst all the other positional properties will work on it.
- `fixed`: the element is removed from the flow of the document like absolutely positioned elements. In fact they behave almost the same, only fixed positioned elements are always relative to the document, not any particular parent, and are unaffected by scrolling.
- `sticky` (experimental): the element is treated like a `relative` value until the scroll location of the viewport reaches a specified threshold, at which point the element takes a `fixed` position where it is told to stick.
- `inherit`: the `position` value doesn’t cascade, so this can be used to specifically force it to, and `inherit` the positioning value from its parent.



## position: sticky

`position:sticky`表现符合粘性的表现。基本上，可以看出是`position:relative`和`position:fixed`的结合体——当元素在屏幕内，表现为relative，就要滚出显示器屏幕的时候，表现为fixed。

chrome 已经支持该特性，Safari 目前还需要-webkit-私有前缀。

示例：

```css
nav {
  position: -webkit-sticky;
  position: sticky;
  top: 0;
}
```

