# 如何查询字符的字节大小

总所周知，每个字符的大小都是不一致的，使用string.length字符长度很难具体判断内存大小，所以有以下方法：

## Blob
```js
new Blob(['哈哈哈']).size

// 输出 9
```

## TextEncoder

```js
var textEncoder = new TextEncoder()
textEncoder.encode('哈哈哈')

// 输出 Uint8Array[......] length: 9
```

输出Uint8Array后，可以通过`.length`获取字节大小

## 总结

~~如果只是获取字符字节大小，TextEncoder是比Blob要好太多的，性能最好~~

~~但是Blob更加全面，可以获取其他类型的字节大小，比如Object, Map之类的~~
```js
var map = new Map()
map.set('1', 1)

new Blob([map, '哈哈哈', 1234]).size // 字节大小: 25
```

用TexteEncoder！对象什么的也能搞定，还比Blob性能好
```js
var text = new TextEncoder()
text.encode(map)
```

# 如何调用复制

## documentAPI

```js
function copyToClipboard(text) {
  const tempInput = document.createElement('textarea');
  tempInput.value = text;
  document.body.appendChild(tempInput);
  tempInput.select();
  document.execCommand('copy');
  document.body.removeChild(tempInput);
}

// 用法：
copyToClipboard('要复制的文本内容');
```
documentAPI是一套过时的API，但是浏览器兼容最好

## Clipboard API
```js
function copyToClipboard(text) {
  navigator.clipboard.writeText(text)
    .then(() => {
      console.log('已成功复制到剪贴板');
    })
    .catch(err => {
      console.error('无法复制到剪贴板：', err);
    });
}

// 用法：
copyToClipboard('要复制的文本内容');
```

Clipboard API是一套现代的API，是趋势，但是老旧的浏览器可能不支持。需要授予权限。

# 如何获取包围盒的八个顶点

仔细想想就能知道结果，但是自己想没什么意义，很简单，直接拿来主义即可
```ts
public getBoxCorners(box: THREE.Box3) {
    const min = box.min;
    const max = box.max;

    const boxVertices = [
        new THREE.Vector3(min.x, min.y, min.z),
        new THREE.Vector3(min.x, min.y, max.z),
        new THREE.Vector3(min.x, max.y, min.z),
        new THREE.Vector3(min.x, max.y, max.z),
        new THREE.Vector3(max.x, min.y, min.z),
        new THREE.Vector3(max.x, min.y, max.z),
        new THREE.Vector3(max.x, max.y, min.z),
    ]

    return boxVertices
}
```

# RGBA和十六进制互相转化
```js
function hexToRGBA(hex: string, alpha?: number) {
  // 去掉可能包含的 "#" 符号
  hex = hex.replace(/^#/, '');

  // 将十六进制颜色分解成R、G、B三个部分
  let r = parseInt(hex.substring(0, 2), 16);
  let g = parseInt(hex.substring(2, 4), 16);
  let b = parseInt(hex.substring(4, 6), 16);

  // 如果提供了alpha值，则将其应用
  if (typeof alpha === 'number' && alpha >= 0 && alpha <= 1) {
    return `rgba(${r},${g},${b},${alpha})`;
  } else {
    // 如果没有提供alpha值，默认为1（不透明）
    return `rgb(${r},${g},${b})`;
  }
}
```