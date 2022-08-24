```js
function exprotImg() {
  let image = new Image()
  image.src = this.$refs.canvasMap.toDataURL("image/png")
  let blob = this.base64ToBlob(image.src)
  blobDownlod(blob, '轨道图')
}

function base64ToBlob(code) {
  let parts = code.split(";base64,");
  let contentType = parts[0].split(":")[1];
  let raw = window.atob(parts[1]);
  let rawLength = raw.length;
  let uint8Array = new Uint8Array(rawLength);
  for (var i = 0; i < rawLength; i++) {
      uint8Array[i] = raw.charCodeAt(i);
  }
  return new Blob([uint8Array], {type: contentType});
}

// 下载
const blobDownlod = (params, excelName) =>  {
  let url = window.URL.createObjectURL(params)
  let link = document.createElement('a')
  link.style.display = 'none'
  link.href = url
  link.setAttribute('download', excelName)
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link) // 下载完成移除元素
  window.URL.revokeObjectURL(url) // 释放掉blob对象
}
```

