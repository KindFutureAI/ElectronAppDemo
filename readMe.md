# Vue3学习
 
## Vite+Electron快速构建一个VUE3桌面应用
借鉴了这位的博客资料：[参考Vue学习链接](https://github.com/hunter-ji/Blog/issues/52)
### 1 创建一个Vite项目
1. 安装 vite
```shell
yarn create vite
```

2. 创建项目
```shell
yarn create vite <your-vue-app-name> --template vue

如
yarn create vite kuari --template vue
```
 
3. 生成dist文件夹
```bash
yarn build
```
4. 进入且运行
进入项目，在运行前需要先安装下依赖。
```shell
cd kuari
yarn install
yarn dev
```
这样就可以在浏览器看到初始化页面
### 2 配置Electron
```bash
# 首先安装electron至vite应用
yarn add --dev electron
```

####  配置文件
##### 1）vite.config.js
```bash
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path' 										// 新增

// https://vitejs.dev/config/
export default defineConfig({
  base: path.resolve(__dirname, './dist/'),	// 新增
  plugins: [vue()]
})
```

##### 2）main.js
创建一个新的文件main.js，需要注意的是，该内容中index.html的加载路径跟electron官网给的配置不同。
```bash
// main.js

// 控制应用生命周期和创建原生浏览器窗口的模组
const { app, BrowserWindow } = require('electron')
const path = require('path')

function createWindow () {
  // 创建浏览器窗口
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // 加载 index.html
  mainWindow.loadFile('dist/index.html') // 此处跟electron官网路径不同，需要注意

  // 打开开发工具
  // mainWindow.webContents.openDevTools()
}

// 这段程序将会在 Electron 结束初始化
// 和创建浏览器窗口的时候调用
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(() => {
  createWindow()

  app.on('activate', function () {
    // 通常在 macOS 上，当点击 dock 中的应用程序图标时，如果没有其他
    // 打开的窗口，那么程序会重新创建一个窗口。
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// 除了 macOS 外，当所有窗口都被关闭的时候退出程序。 因此，通常对程序和它们在
// 任务栏上的图标来说，应当保持活跃状态，直到用户使用 Cmd + Q 退出。
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

// 在这个文件中，你可以包含应用程序剩余的所有部分的代码，
// 也可以拆分成几个文件，然后用 require 导入。
```

##### 3）preload.js 
```bash
// preload.js

// 所有Node.js API都可以在预加载过程中使用。
// 它拥有与Chrome扩展一样的沙盒。
window.addEventListener('DOMContentLoaded', () => {
  const replaceText = (selector, text) => {
    const element = document.getElementById(selector)
    if (element) element.innerText = text
  }

  for (const dependency of ['chrome', 'node', 'electron']) {
    replaceText(`${dependency}-version`, process.versions[dependency])
  }
})
```

##### 4）package.json 
为了确保能够运行相关electron的命令，需要修改package.json文件。

首先需要去设置main属性，electron默认会去在开始时寻找项目根目录下的index.js文件，此处我们使用的是main.js，所以需要去定义下。
```bash
// package.json

{
  "name": "kuari",
  "version": "0.0.0",
  "main": "main.js", 			// 新增
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "serve": "vite preview",    
    "electron:serve": "electron ." // 新增electron的运行命令。
  },
  "dependencies": {
    "vue": "^3.2.16"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^1.9.3",
    "electron": "^15.1.2",
    "vite": "^2.6.4"
  }
}

```

### 3 运行electron应用
接着我们就可以看到我们桌面应用就出来咯！
```bash
yarn electron:serve
```



### 4 实现热更新
#### 编辑main.js
将mainWindow.loadFile('dist/index.html')更新为mainWindow.loadURL("http://localhost:3000")，更新后的文件如下所示：
```bash
// main.js

// 控制应用生命周期和创建原生浏览器窗口的模组
const { app, BrowserWindow } = require('electron')
const path = require('path')

function createWindow () {
  // 创建浏览器窗口
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // 加载 index.html
  // mainWindow.loadFile('dist/index.html') 将该行改为下面这一行，加载url
  mainWindow.loadURL("http://localhost:3000")

  // 打开开发工具
  // mainWindow.webContents.openDevTools()
}

// 这段程序将会在 Electron 结束初始化
// 和创建浏览器窗口的时候调用
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(() => {
  createWindow()

  app.on('activate', function () {
    // 通常在 macOS 上，当点击 dock 中的应用程序图标时，如果没有其他
    // 打开的窗口，那么程序会重新创建一个窗口。
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// 除了 macOS 外，当所有窗口都被关闭的时候退出程序。 因此，通常对程序和它们在
// 任务栏上的图标来说，应当保持活跃状态，直到用户使用 Cmd + Q 退出。
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

// 在这个文件中，你可以包含应用程序剩余的所有部分的代码，
// 也可以拆分成几个文件，然后用 require 导入。

```

#### 编辑vite.config.js
 修改文件vite.config.js的base，修改后的文件如下所示：
```bash
// vite.config.js

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

// https://vitejs.dev/config/
export default defineConfig({
  base: "./",	// 新增
  plugins: [vue()]
})
```

#### 同时开启vite和electron服务
为了使vite和electron正常运行，需要先运行vite，使得其开发服务器的url可以正常访问，然后再开启electron去加载url。

此处需要安装两个库：

- **concurrently**：阻塞运行多个命令，`-k`参数用来清除其它已经存在或者挂掉的进程
- **wait-on**：等待资源，此处用来等待url可访问

首先来安装。
```bash
yarn add -D concurrently wait-on
```

接着更新文件`package.json`，`scripts`新增两条命令：

```bash
{
  "name": "kuari",
  "version": "0.0.0",
  "main": "main.js",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "serve": "vite preview",
    //新增下面两条
    "electron": "wait-on tcp:3000 && electron .",
    "electron:serve": "concurrently -k \"yarn dev\" \"yarn electron\""
  },
  "dependencies": {
    "vue": "^3.2.16"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^1.9.3",
    "concurrently": "^6.3.0",
    "cross-env": "^7.0.3",
    "electron": "^15.1.2",
    "vite": "^2.6.4",
    "wait-on": "^6.0.0"
  }
}
```
#### 运行electron
执行命令yarn electron:serve即可，当修改项目文件时，桌面应用也将自动更新。

### 5 应用打包
#### 1 思路
先说结论，重点还是在于mainWindow.loadURL()。

打包后还是加载http://localhost:3000是无法运行的，因此，此处需要先用vite打包好，然后使用electron-builder加载vite打包后的文件进行打包。

为了代码能够根据不同环境在运行时加载http://localhost:3000，在打包时加载文件，此处需要使用环境变量来切换生产和开发环境。
#### 2 环境变量
此处使用环境变量NODE_ENV来切换生产和开发环境，生产环境为NODE_ENV=production，开发环境为NODE_ENV=development，若有其它如release等环境可在此基础上拓展。
#### 3 创建electron文件夹

在项目根目录下创建文件夹`electron`，将`main.js`和`preload.js`文件移动进来。其结构如下所示：

```
.
├── README.md
├── electron
│   ├── main.js
│   └── preload.js
...
```

若还是不太明白可以看看[源码](https://github.com/Kuari/Blog/tree/master/Examples/vite_electron/vite_electron_3)中文件结构。

#### 4. 编辑electron/main.js

该文件主要是需要根据环境变量切换electron加载的内容，修改内容如下：

```js
mainWindow.loadURL(
  NODE_ENV === 'development'
  ? 'http://localhost:3000'
  :`file://${path.join(__dirname, '../dist/index.html')}`
);
```

修改后的完整内容如下：

```js
// electron/main.js

// 控制应用生命周期和创建原生浏览器窗口的模组
const { app, BrowserWindow } = require('electron')
const path = require('path')

const NODE_ENV = process.env.NODE_ENV

function createWindow () {
  // 创建浏览器窗口
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      preload: path.join(__dirname, 'preload.js')
    }
  })

  // 加载 index.html
  // mainWindow.loadFile('dist/index.html') 将该行改为下面这一行，加载url
  mainWindow.loadURL(
    NODE_ENV === 'development'
      ? 'http://localhost:3000'
      :`file://${path.join(__dirname, '../dist/index.html')}`
  );

  // 打开开发工具
  if (NODE_ENV === "development") {
    mainWindow.webContents.openDevTools()
  }

}

// 这段程序将会在 Electron 结束初始化
// 和创建浏览器窗口的时候调用
// 部分 API 在 ready 事件触发后才能使用。
app.whenReady().then(() => {
  createWindow()

  app.on('activate', function () {
    // 通常在 macOS 上，当点击 dock 中的应用程序图标时，如果没有其他
    // 打开的窗口，那么程序会重新创建一个窗口。
    if (BrowserWindow.getAllWindows().length === 0) createWindow()
  })
})

// 除了 macOS 外，当所有窗口都被关闭的时候退出程序。 因此，通常对程序和它们在
// 任务栏上的图标来说，应当保持活跃状态，直到用户使用 Cmd + Q 退出。
app.on('window-all-closed', function () {
  if (process.platform !== 'darwin') app.quit()
})

// 在这个文件中，你可以包含应用程序剩余的所有部分的代码，
// 也可以拆分成几个文件，然后用 require 导入。
```

#### 5. 编辑package.json
首先修改`main` 属性，将`main: main.js`改为`main: electron/main.js`。

```json
{
  "name": "kuari",
  "version": "0.0.0",
  "main": "electron/main.js",
 	... 
}
```

接着，编辑`build`属性：

```json
"build": {
  "appId": "com.your-website.your-app",
  "productName": "ElectronApp",
  "copyright": "Copyright © 2021 <your-name>",
  "mac": {
    "category": "public.app-category.utilities"
  },
  "nsis": {
    "oneClick": false,
    "allowToChangeInstallationDirectory": true
  },
  "files": [
    "dist/**/*",
    "electron/**/*"
  ],
  "directories": {
    "buildResources": "assets",
    "output": "dist_electron"
  }
}
```

然后，更新`scripts`属性。

此处需要先安装两个库：

- **`cross-env`**: 该库让开发者只需要注重环境变量的设置，而无需担心平台设置
- **`electron-builder`**: electron打包库

```shell
yarn add -D cross-env electron-builder
```

更新后的`scripts`如下：

```json
{
  "dev": "vite",
  "build": "vite build",
  "serve": "vite preview",
  "electron": "wait-on tcp:3000 && cross-env NODE_ENV=development electron .", // 此处需要设置环境变量以保证开发时加载url
  "electron:serve": "concurrently -k \"yarn dev\" \"yarn electron\"",
  "electron:build": "vite build && electron-builder" // 新增打包命令
}
```

最后，更新后的`package.json`完整内容如下：

```json
{
  "name": "vproj2",
  "private": true,
  "version": "0.0.0",
  "main": "electron/main.js",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "electorn": "wait-on tcp:3000 && cross-env NODE_ENV=development electron .",
    "electron:serve": "concurrently -k \"yarn dev\" \"yarn electron\"",
    "electron:bulid": "vite build && electron-builder"
  },
  "dependencies": {
    "vue": "^3.5.10"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^5.1.4",
    "concurrently": "^9.0.1",
    "vite": "^5.4.8",
    "wait-on": "^8.0.1"
  },
  "build": {
    "appId": "com.your-website.your-app",
    "productName": "ElectronApp",
    "copyright": "Copyright © 2021 <your-name>",
    "mac": {
      "category": "public.app-category.utilities"
    },
    "nsis": {
      "oneClick": false,
      "allowToChangeInstallationDirectory": true
    },
    "files": [
      "dist/**/*",
      "electron/**/*"
    ],
    "directories": {
      "buildResources": "assets",
      "output": "dist_electron"
    }
  }
}
```
#### 6. 打包
直接执行打包命令即可开始打包。
```bash
yarn electron:build

```
