---
title: 前端单文件入口发布新版本 缓存问题
date: 2022-04-18T14:35:46+08:00
Description:
Tags: 
Categories:
DisableComments: false
---

在现代 `javascript`框架项目开发中，一直有一个令人都疼的问题，就是缓存问题；每次发版完之后由于浏览器缓存机制，用户端不会实时获取新的项目页面，甚至有可能出现静态文件获取报404。


## 方法思路

1. 在入口文件中配置文件更新后 缓存同步更新
2. 打包的时候 生成一个唯一的版本号，并添加到 `入口目录/config.json`
3. 每次 `路由` 发生变更的时候，判断版本号是否发生变化，如果发生变化，则刷新当前文件



### 以 `vue` 项目为例

1. 在项目 `public` 文件夹下的 `index.html` 入口文件中添加如下代码
```html
<meta http-equiv="pragma" content="no-cache">
<meta http-equiv="cache-control" content="no-cache, no-store, must-revalidate">
<meta http-equiv="expires" content="0">
```

2. 生成版本号 在vue.config.js中添加如下配置

```js
const Timestamp = (new Date()).getTime()
const gitRevisionPlugin = new GitRevisionPlugin() // 依赖 git-revision-webpack-plugin
const VERSION = `${gitRevisionPlugin.branch()}_${gitRevisionPlugin.version()}_${gitRevisionPlugin.commithash()}_${Timestamp}` // git分支+时间戳；这里可以根据自己喜欢的方式加上随机版本号
process.env.VUE_APP_VERSION = VERSION // 记录到env，并在vuex中记录，用于后面版本号对比校验

const configJSON = require(resolve('public/config.json')) // public文件夹下新建config.json
const configFile = path.resolve(__dirname, 'public/config.json')
fs.writeFileSync(configFile, JSON.stringify({
  ...configJSON,
  version: VERSION
}, null, 2))
```

3. 在utils下，新建systemUpdate.js

```js
import axios from 'axios'
import store from '@/store'
import { removeLocalStorage, setLocalStorage } from '@/utils/localStorage'
import { MessageBox } from '@/element-ui'

const getConfig = () => {
  return new Promise((resolve) => {
    axios.get(`${process.env.VUE_APP_DOMAIN}/config.json`, {
      params: {
        _t: (new Date()).getTime()
      }
    }).then(res => {
      resolve(res.data)
    })
  })
}

export async function isNewVersion () {
  if (process.env.NODE_ENV === 'development') {
    return false
  }
  const config = await getConfig()
  let newVersion = config.version
  let oldVersion = store.getters.version
  let isUpdated = oldVersion !== newVersion
  if (isUpdated) { // 如果version不一致，则清除本地基础数据
    removeLocalStorage('AREADATA')
    MessageBox.alert(
      '系统检测到有新版本，请刷新页面！',
      '系统提示',
      {
        showClose: false,
        confirmButtonText: '刷新页面'
      }
    ).then(() => {
      setLocalStorage('VERSION', { version: newVersion })
      window.location.reload()
    }).catch(() => {
    })
  }
  return isUpdated
}
```

4. 判定版本号

```js
router.beforeEach(async (to, from, next) => {
  // 判断版本号，如果不一致则提示用户刷新页面
  const isUpdate = await isNewVersion()
  if (isUpdate) return false
...
```

