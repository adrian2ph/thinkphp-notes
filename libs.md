# libs

## vueuse

### Browser

#### useClipboard <a href="#useclipboard" id="useclipboard"></a>



const { copied, copy } = useClipboard({ legacy: true });

> Set `legacy: true` to keep the ability to copy if [Clipboard API](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard\_API) is not available. It will handle copy with [execCommand](https://developer.mozilla.org/en-US/docs/Web/API/Document/execCommand) as fallback.



复制功能的实现主要是依赖于`navigator.clipboard`属性，当浏览器不支持该属性时，如果用户显示声明了`legacy: true`， 则使用降级策略，使用 `document.execCommand('copy')`实现复制功能

* navigator.clipboard

1. **此功能只支持在安全上下文(HTTPS)中使用，而HTTP下不支持，**
2. 本地传递的资源，如那些带有 `http://127.0.0.1`、`http://localhost` 和 `http://*.localhost` 网址（如 `http://dev.whatever.localhost/`）和 `file://` 网址的资源也是认为经过安全传递的。

* ### `document.execCommand('copy')`

`document.execCommand()`是操作剪贴板的传统方法，各种浏览器都支持。逻辑很清晰，这里需要注意的是必须选中当前元素内的文字，否则无法实现复制功能。

```javascript
function legacyCopy(value: string) {
    const ta = document.createElement('textarea')
    ta.value = value ?? ''
    ta.style.position = 'absolute'
    ta.style.opacity = '0'
    document.body.appendChild(ta)
    ta.select()
    document.execCommand('copy')
    ta.remove()
}js
```
