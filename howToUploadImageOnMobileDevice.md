## 遇到的问题
1. 移动设备拍的照片大小通常都在1M以上, 如果直接上传原图, 在移动数据网络的情况下, 先不说上传的乌龟速度, 光是流量唰唰唰的跑, 也会让用户疯掉的. 所以我们必须考虑在前端对图片进行压缩.
2. 有些设备(例如IOS)的照片在本地看是正的, 但是上传到后台, 显示出来并不是正的. 为什么会有这种情况? 怎么解决?(坑)
3. 性能差的设备在绘制大图时, 响应时间会非常长, 甚至无响应. 那怎么办呢? 
4. 如何实现图片缩略图预览
5. 图片压缩之后, 我们拿到的是一段base64的编码, 那我们怎么与后台交互? 我们有两种选择, 

 * 直接上传base64编码给后台, 在后台进行转换
 * 在前端把base64编码转换成二进制对象, 在传到后台(坑)

6. android微信浏览器对 WebkitBlobBuilder 的实现有bug, 导致编码与二进制对象的转换不能顺利进行.

### 如何实现前端压缩图片
1. 计算压缩比例, 绘制在canvas上, 然后导出, 在社群项目中, 把图片大小限定在了800 * 800, 也就是说宽和高最大不能超过800px, 以长度较大的边为准. ( 至于800这个值怎么来的, 目前没有通过科学合理的方式计算, 而是估计了一个大概的值.这点也算是以后需要改进的地方. )
```
getRatio: function(nw, nh){
    var ratiow = (nw / 800) > 1 ? (nw / 800) : 1;
    var ratioh = (nh / 800) > 1 ? (nh / 800) : 1;
    return Math.max(ratiow, ratioh);
}
...
//可指定压缩比
var imgsrc = canvas.toDataURL('image/jpeg',.8);
```

### 图片上传后不正
这类图片中包含有 EXIF 信息, 记录了拍照方向、相机设备型号、拍摄时间、ISO 感光度、GPS 地理位置等数据. 
在设备上看时, 查看器会根据拍照方向自动进行旋转. 所以看到的图永远是正的. 我们现在需要做的就是通过exif.js得到图片的Orientation值, 然后根据该值适当的旋转context, 从而保持图片是正向的.
需要注意的是, context如果被旋转了, 即坐标系发生旋转，x,y 坐标会相应的发生变化
例如: 图片向右旋转了90度，即坐标系旋转了90度，那么起始位置应该是 x = 0, y = img.height. 其他方向依次类推.
如果对旋转有疑惑的话, 参考这篇文章 http://blog.csdn.net/jia20003/article/details/9235813
```
rotate: function(orientation, cw, ch, canvas, ctx){
    var degree = 0, w = cw, h = ch, sp = [0, 0];
    switch(orientation){
        case 1:
            degree = 0;
            break;
        case 6:
            degree = 90;
            w = ch;
            h = cw;
            sp = [0, -ch] ;
            break;
        case 3:
            degree = 180;
            sp = [-cw, -ch];
            break;
        case 8:
            degree = -90;
            w = ch;
            h = cw;
            sp = [-cw, 0];
            break;
    }
    degree = degree * Math.PI / 180;
    canvas.width = w;
    canvas.height = h;
    ctx.rotate(degree);
    return sp;
}
```

### 绘制图片卡顿
把img绘制到convas上,  并不是简单的调用drawImage, 这里涉及到一个切片绘制, 
就是把大图片划分为一块一块的小图, 然后在绘制 ( 理论上可以缓解性能压力. 有待进一步测试. )
项目中采用400 * 400 为一个单元 ( 有待提高, 得出更合理的值 )
对 drawImage api 的解释 http://www.w3school.com.cn/tags/canvas_drawimage.asp
```
drawImg: function(img, sp, cw, ch, ratio, canvas, ctx){
    var col = Math.ceil(canvas.width / 400);
    var row = Math.ceil(canvas.height / 400);
    //slice image into pieces
    if(col > 1 || row > 1){
        var uw = cw / col;
        var uh = ch / row;
        for (var i = 0; i < col; i++) {
            for (var j = 0; j < row; j++) {
                ctx.drawImage(img, i * uw * ratio, j * uh * ratio, uw * ratio, uh * ratio, sp[0] + i * uw, sp[1] + j * uh, uw, uh);
            }
        }
    }
    else{
        //draw image according to the shrunken size
        ctx.drawImage(img, sp[0], sp[1], cw, ch);
    }
}
```

### 如何实现图片缩略图预览
通过FileReader 的 readAsDataURL的方法, 把选中的图片转化成base64的编码
new Image()对象, 并把base64编码赋值给src属性
```
var fReader = new FileReader();
//read complete listener
fReader.onload = function (fREvent) {
    var img = new Image();
    img.src = fREvent.target.result;
    if(img.complete){
        callback();
    }else{
        img.onload = callback;
    }
    function callback() {
        $('body').append('![](' + imgsrc + ')');
    }

};

//start reading file
fReader.readAsDataURL(f); 
```

### 如何把base64转成二进制对象
这个需要单独开一个页面来说. 涉及到很多不常用的构造函数, 例如 Unit8Array, WebkitBlobBuilder, Blob
```
//convert base64 to blob object
var type = self.files[fIndex].type;
var imgsrc = window.atob(data.split(',')[1]);//data为base64编码
var buffer = new Uint8Array(imgsrc.length);
for(var i = 0; i < imgsrc.length; i++) buffer[i] = imgsrc.charCodeAt(i);
var blob = self.getBlob(buffer, type);
getBlob: function(buffer, format){
    //obsoleted by w3c, android wechat browser still support this feture, but can't get a correct blob
    var Builder = window.WebKitBlobBuilder;
    if(Builder){
        var builder = new Builder;
        builder.append(buffer);
        return builder.getBlob(format);
    } else {
        //new standard, ios、 pc、 android、 will use this constructor
        return new window.Blob([buffer], {type: format});
    }
}
```

最后因为android微信浏览器不能正确支持blob, 还是采用了直接上传base64的方案. 
