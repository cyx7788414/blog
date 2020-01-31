# 问题描述
有一段代码是这么个效果，父元素里有一个display:none的子元素，  
当父元素hover时，子元素display:block，  
同时子元素设有最大高度，内容过多时出现垂直滚动条，可以用鼠标拖动滚动条。  
但当升级Chrome到56.0.2924.87后，问题出现了，鼠标移到滚动条上时，触发的hover消失了，子元素隐藏。  
# 示例代码
```html
<div class="parent">
    <div class="child">
        <!--something text-->
    </div>
</div>
```
```css
.parent {
    width: 100px;
    height: 100px;
}
.child {
    width: 100px;
    max-height: 200px;
    overflow-y: auto;
    display: none;
}
.parent:hover .child {
    display: block;
}
```
# 解决方案
stackoverflow上有人提供了如下解决方案，通过为父元素增加事件控制style来强化效果。  
```html
<div class="parent" onmouseenter="this.style.display='block';" onmouseleave="this.style.display='';">
```
# 参考文献
1. [Chrome 56 doesn't apply :hover styling when over a scrollbar?](https://productforums.google.com/forum/#!topic/chrome/7_ov6OeagZY)  
2. [CSS hover with display block on chrome 56](http://stackoverflow.com/questions/41963235/css-hover-with-display-block-on-chrome-56)