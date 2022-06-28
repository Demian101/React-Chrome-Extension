源码作者 : https://github.com/shenmaxg/antd-chrome-extension-boilerplate

最近没空搞 Google Extension , 先 Mark 吧



# antd-chrome-extension-boilerplate

使用 react + antd 开发 chrome-extension 的脚手架工程，具有热加载功能。

## 特性

1. 使用 antd + react 开发 chrome extension。
2. 解决了当前页面与 content-script 间的 css 污染问题。
3. 增加开发模式，提供 chrome extension 热更新功能。

## 使用

```
npm run start
```

启动开发模式，在 chrome 浏览器中加载该应用的 dist 目录，此时 dist 目录下的文件具有热更新功能，并且会自动刷新当前页面。

```
npm run build
```

打包模式，不赘述。

## 相关文章

1. [如何使用 antd + react 开发 chrome extension](https://zhuanlan.zhihu.com/p/393004855)
2. [如何实现 chrome extension 的热更新](https://zhuanlan.zhihu.com/p/399937088)





#  开发流程

1. 根据脚手架开发文件

2. chrome://extensions/  中 , 加载已解压的拓展程序
   1. 导入工程的 `./dist`  文件夹即可



## Docs

https://zhuanlan.zhihu.com/p/393004855

https://zhuanlan.zhihu.com/p/399937088

https://github.com/shenmaxg/antd-chrome-extension-boilerplate/issues/1

https://support.google.com/chrome/a/answer/2714278?hl=zh-Hans





# Background Knowledge

## Chrome Extension的组成

简单来说，开发Chrome Extension主要涉及以下四个部分：

1. manifest.json （插件配置文件）
2. popup (点击插件图标弹出的页面)
3. content script (插入到目标页面中执行的js)
4. background script (在chrome后台中运行的程序)



【manifest.json】

manifest.json 必须放在插件项目根目录，里面包含了插件的各种配置信息，其中也包括了popup、content script、background script等路径。



【popup】

作为一个独立的弹出页面，有自己的html、css、js，可以按照常规项目来开发。



【content script】

content script是驻入到目标页面中执行的js脚本，可以获取目标页面的Dom并进行修改。但是，content script的JavaScript与目标页面是互相隔离的。也就是说，content script与目标页面不会出现互相污染的问题，同时，也不能调用对方的方法。

> 注意，以上只是js作用域的隔离，通过content script向目标页面加入的DOM会是可以使用目标页面的css，从而造成css互相污染。



【background script】

background script 常驻在浏览器后台运行，没有实际页面（当然也可以通过manifest.json指定一个页面，如果不设置，chrome会自动生成一个），它的生命周期随着浏览器打开而开始，随着浏览器关闭而结束。一般把全局的、需要一直运行的代码放在这里。重要的是，background script的权限非常高，除了可以调用几乎所有Chrome Extension API外，还可以跨域发请求。



在了解Chrome Extension的基本组成后，需要按照Chrome Extension官方开发文档以及manifest.json的要求，按以下结构build最终的目录。

<img src="http://imagesoda.oss-cn-beijing.aliyuncs.com/Sodaoo/2022-06-28-070923.png" style="zoom:67%;" />



### 配置manifest.json

在开发Chrome Extension之前，要先配置好manifest.json。

public/manifest.json

```js
{
  "name": "Chrome插件Demo",
  "version": "1.0",
  "description": "React开发chrome插件Demo。",
  // 图标，图省事的话，所有尺寸都用一个图也行
  "icons": {
    "16": "images/icon.png",
    "48": "images/icon.png",
    "128": "images/icon.png"
  },
  // manifest.json的文件版本，必须是2
  "manifest_version": 2,
  // popup页面配置
  "page_action": {
    // 浏览器插件按钮的图标
    "default_icon": "images/icon.png",
    // 浏览器插件按钮hover显示的文字
    "default_title": "React CRX",
    // popup页面的路径（根目录为最终build生成的插件包目录）
    "default_popup": "index.html"
  },
  // background script配置
  "background": {
    // background script路径（根目录为最终build生成的插件包目录）
    "scripts": [
      "static/js/background.js"
    ],
    // 官方强烈建议这里为false，乖乖写就行了
    "persistent": false
  },
  // content script配置
  "content_scripts": [
    {
      // 应用于哪些页面地址（可以使用正则，<all_urls>表示匹配所有地址）
      "matches": [
        "<all_urls>"
      ],
      // 注入的css，注意不要污染样式
      "css": [
        "static/css/content.css"
      ],
      // 注入的js
      "js": [
        "static/js/content.js"
      ],
      // 代码注入的时间，可选document_start、document_end、document_idle（默认）
      "run_at": "document_end"
    }
  ],
  // 权限申请（需要background发起跨域请求的url也放在这里）
  "permissions": [
    // 标签
    "tabs",
    // 根据定制的网页规则，采取相应的措施（例如只在baidu.com启动组件）
    "declarativeContent",
    // 插件本地存储
    "storage",
    // 通知
    "notifications"
  ],
  // 如果向目标页面插入js，需要在这里声明下才能获得执行的权限
  "web_accessible_resources": ["insert.js"]
}
```

manifest的配置项还有很多，可前往官网查阅：

> **manifest：**[https://developer.chrome.com/extensions/manifest](https://link.zhihu.com/?target=https%3A//developer.chrome.com/extensions/manifest)
> **manifest_version：**[https://developer.chrome.com/extensions/manifest/manifest_version](https://link.zhihu.com/?target=https%3A//developer.chrome.com/extensions/manifest/manifest_version)
> **background script：**[https://developer.chrome.com/extensions/background_pages](https://link.zhihu.com/?target=https%3A//developer.chrome.com/extensions/background_pages)
> **content script：**[https://developer.chrome.com/extensions/content_scripts](https://link.zhihu.com/?target=https%3A//developer.chrome.com/extensions/content_scripts)
> **permissions：**[https://developer.chrome.com/extensions/declare_permissions](https://link.zhihu.com/?target=https%3A//developer.chrome.com/extensions/declare_permissions)