## 存在哪些问题?

1. 流量问题, 移动端 "首屏" 最重要, 不要页面一加载, 就把页面上所有图片都load出来, 也许用户根本不会滚动到页面下方去看.
2. 用户上传的图片, 大小参差不齐, 相机拍的照片还好, 如果是网络上的图片, 那真是什么尺寸都有, 怎么保持图片在不变形的情况下还能显示的很漂亮?

### 页面结构
html 结构很简单, 给 img 标签包裹一个div容器, 并设置一个自定义属性 data-src 用来保存图片的地址, 由于 img 标签没有设置 src 属性, 所以浏览器并不会发送请求.
```
<div class="img-container">
    ![](images/U6536P1190DT20111029101046.jpg)
</div>
```
### 容器样式
在页面加载出来之后, 图片加载出来之前, 这一段时间, 如果不作任何处理, 用户看到就是一片空白的地方, 然后图片加载完成, 很突兀的出现在用户眼前.
我们的做法是给.img-container, 设置一个背景色或者占位背景图片, 看起来的效果就像这样. 

![背景色](http://upload-images.jianshu.io/upload_images/2261833-f7d4fcc7a473eb62.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![背景图](http://upload-images.jianshu.io/upload_images/2261833-03bef7424e1b21e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
.img-container {
  background-color: #dddddd;
  overflow: hidden;
}
```
### 最简单的懒加载
判断图片是否在可视范围内
把图片的data-src属性的值, 赋值给 src, 然后删掉data-src属性, 以表示这张图片已经完成了加载.
```
UI.lazyLoadImage = function($parent) {
    $parent = $parent || UI.$body;
    $('img[data-src]', $parent).each(function(i, e) {
        if (UI.Utils.isInView(this)) {
            this.src = this.getAttribute('data-src');
            this.removeAttribute('data-src');
        }
    });
}
 
UI.lazyLoadImage();
 
// listen to scroll and resize
UI.$doc.on("scroll", UI.Utils.debounce(UI.lazyLoadImage, 50));
UI.$win.on("load resize orientationchange", UI.Utils.debounce(UI.lazyLoadImage, 50));
```
注意这段代码只是最基本的懒加载, 无法做到和imageAdapt的配合. 

### 自适应图片

假设现在页面上有一组缩略图, img-container的宽和高相等都是180px, 我们理想的情况是图片宽高比1:1

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2261833-4720de847000cdac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

而实际中用户上传的image的尺寸分别为180 * 180, 360 * 180, 180 * 360

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2261833-37b401b2cee5dd76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这种情况如果设置 width 或者 height: 100%, 会导致图片无法填满, 都设置成100%会导致图片变形. 而我们想要的应该是cover的效果图片自动适应容器的大小做等比例的缩放. 最简单的办法, 利用background的特性来实现.
```
.img-container{
    background-position: center center;
}
```
下面这段脚本, 遍历 .img-contaier, 如果img有src属性, 就把img 的src 设置成 .img-container的背景.
```
UI.imageAdapt = function($imgContainer) {
 
    var $img = $imgContainer.children('img[src]');
    //default: width = height
    if (!$imgContainer.is('.no-square')) {
        $imgContainer.height($imgContainer.width());
    }
    if ($img.length) {
        $imgContainer.css('background-image', 'url(' + $img[0].src + ')');
        $img.hide();
    }
 
};
 
$('.img-container').each(function() {
    UI.imageAdapt($(this));
});
```
除了这种做法, 你还可以通过设置图片的margin-top, margin-left, 来达到同样的效果.
```
//css
.img-container > img {
  display: block;
  width: 100%;
}
 
.img-container > .fix-height {
  width: auto;
  height: 100%;
}
//js
if ($img[0].height <= $imgContainer.height()) {
    $img.addClass('fix-height');
    $img.css('margin-left', '-' + (($img[0].width - $imgContainer.width()) / 2) + 'px');
    $img.css('margin-top', '');
}
if ($img[0].height > $imgContainer.height() || $img[0].width < $imgContainer.width()) {
    $img.removeClass('fix-height');
    $img.css('margin-left', '');
    $img.css('margin-top', '-' + (($img[0].height - $imgContainer.height()) / 2) + 'px');
}
```
最后把两者相结合. 大功告成.
```
UI.lazyLoadImage = function($parent) {
 
    $parent = $parent || UI.$body;
    $('img[data-src]', $parent).each(function() {
 
        var $self = $(this);
        if (UI.Utils.isInView(this)) {
 
            var src = $self.data('src');
            var image = new Image();
            image.onload = function() {
                var $imgContainer = $self.parent().append(image);
                $self.remove();
                UI.imageAdapt($imgContainer);
            }
            image.src = src;
 
        }
 
    });
 
}
```
