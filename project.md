# 项目

## canvas上传图片

通过 `canvas.toDataURL()` 获取 图片的dataurl

 ```js
 
/**
 * @description base64转换为文件
 * @param {string} dataurl base64
 * @param {string} filename 文件名
 */
export function dataURLtoFile(dataurl, filename) {
  let arr = dataurl.split(","),
    mime = arr[0].match(/:(.*?);/)[1],
    bstr = atob(arr[1]),
    n = bstr.length,
    u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new File([u8arr], filename, { type: mime });
}
 ```

```js

 const file = dataURLtoFile(dataUrl, time + "share.png");

 ```

## 工单组件

## 微前端改造

## taro

### 两个小程序合并

利用环境变量来区分,

app.config.js

主要是page页面不同 `process.env.APP_TYPE == 'store' ? [] : []`

```js
config = {
  pages: [].concat( mainPackage.concat([
    // 相同的文件
  ]))
}

```

### 小程序最新的api

使用原生模块（文档）
