---
abbrlink: '0'
hide: true
---
# JavaScript 常用方法

## 文件导出

```js
function saveFile (content, fileName) {
  const blob = new Blob([content]);
  if (window.navigator.msSaveOrOpenBlob) {
    navigator.msSaveBlob(blob, fileName);
  } else {
    const elink = document.createElement('a');
    elink.download = fileName;
    elink.style.display = 'none';
    elink.href = URL.createObjectURL(blob);
    document.body.appendChild(elink);
    elink.click();
    document.body.removeChild(elink);
  }
},
```