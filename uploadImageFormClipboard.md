## 实现在输入框中通过Ctrl+V粘贴剪切板中的图片进行上传


### 如何获取剪贴板的数据
在Chrome中, 可以在*window.addEventListener('paste', e => console.log(e.clipboardData))*的回调函数中通过*ClipboardEvent.clipboardData*访问剪切板中的数据, 该属性实际上是一个*DataTransfer*类型的对象, 它可以保存一项或多项数据、一种或者多种数据类型。

[在IE11, Edge中, 以上做法并不会按预期的效果执行](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/dev-guides/dn265023(v=vs.85)?redirectedfrom=MSDN#clipboard-support-for-images), 我们不得不在dom中添加一个*contenteditable*的容器去兼容IE11, 只有这个区域, 才会触发*onpaste*事件.

### 如何处理图片类型的数据
在paste事件的回调函数中, 根据*item.type*过滤出图片文件, 这里对于Chrome, IE, Edge又有不同的处理方式, 需要额外做一些兼容性处理.

* Chrome `ClipboardEvent.clipboardData.files`
* IE11 `window.clipboardData.files`
* Edge `ClipboardEvent.clipboardData.items`

```
function getClipboardImage(e) {
  let file;
  // ie only have window.clipboardData.files
  let files = (e.clipboardData || window.clipboardData).files ||
    // edge only have e.clipboardData.items
    (e.clipboardData || window.clipboardData).items;
  for (let i = 0; i < files.length; i++) {
    let item = files[i++];
    if (item.type.indexOf('image/') !== -1) {
      file = item;
      break;
    }
  }
  // need transform e.clipboardData.items to file
  return (file && file.getAsFile) ? file.getAsFile() : file;
};
```
得到File对象后, 通过FileReader获取图片的Base64编码, 然后上传.

```
function getBase64(img, callback) {
  const reader = new FileReader();
  reader.addEventListener('load', () => callback(reader.result));
  reader.readAsDataURL(img);
}
```
至此, 基本上实现了通过Ctrl+V粘贴剪切板的图片进行上传的功能. 但是我们通过*contenteditable*容器模拟*textare*的行为带来了一些副作用. 

### 解决模拟Textare的副作用

1. 没有*placeholder*
2. 我们并不需要一个富文本编辑器, 但是*contenteditable*区域可以粘贴html
3. 没有*onchange*事件, 并且IE11不支持*oninput*事件
4. 在react中的受控组件, 会在输入时, 光标位置错乱

#### 添加*placeholder*
巧妙的通过css3选择器实现占位字符的效果, 代码如下.

```
// html
<div class="content-editable" placeholder="Comments here (Max 300 length)..." contenteditable></div>

// css
.content-editable:empty:not(:focus):before {
    content: attr(placeholder);
    cursor: text;
}
```

#### 去除粘贴html的能力
在paste事件的回调函数中, 调用*DataTransfer.getData(type)*, 获取所有的文本数据, 阻止浏览器默认的粘贴行为, 手动添加文本节点.

```
function transformClipboardText(e) {
  let paste = (e.clipboardData || window.clipboardData).getData('text') || '';
  // paste = paste.toUpperCase();

  const selection = window.getSelection();
  if (!selection.rangeCount) return false;
  // delete selected content
  selection.deleteFromDocument();
  selection.getRangeAt(0).insertNode(document.createTextNode(paste));
  selection.collapseEnd();
  // prevent append tag
  e.preventDefault();
};
```

#### 模拟onchange事件
对于*contenteditable*节点, 其实并不像*textarea*一样具备*value*属性, 要想获取它的值, 必须通过*innerHTML*, 根据这一特性, 创建一个单独的*ContentEditable*组件, 利用*MutationObserver*监控dom节点的变化, 主动触发*onChange*事件.

```
componentDidMount() {
  this.mutationObserver = new MutationObserver(this.onChange);
  this.mutationObserver.observe(this.rootRef.current, {
    childList: true, // To check for new lines
    subtree: true, // To check for nested elements
    characterData: true // To check for text modifications
  });
}

onChange = () => {
  if (isFunction(this.props.onChange)) {
    this.props.onChange({
      target: {
        value: this.rootRef.current.innerText
      }
    });
  }
};

```

#### 解决输入时光标位置错乱
添加*shouldComponentUpdate*, 阻止不必要的渲染.

```
  shouldComponentUpdate = (nextProps) => {
    try {
      // We need not rerender if the change of props simply reflects the user's edits.
      // Rerender in this case would make the cursor/caret jump
      // ... if html really changed... (programmatically, not by user edit)
      return nextProps.value !== this.rootRef.current.innerText;
    } catch (e) {
      // Rerender if there is no element yet... (somehow?)
      return true;
    }
  };
```

### 运用到的技术
* https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/onpaste
* https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/contenteditable
* https://developer.mozilla.org/en-US/docs/Web/API/ClipboardEvent/clipboardData
* https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer
* https://developer.mozilla.org/en-US/docs/Web/API/Selection
* https://developer.mozilla.org/en-US/docs/Web/API/Range
* https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver

