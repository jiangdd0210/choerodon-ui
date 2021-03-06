---
order: 4
title: 在 create-react-app 中使用
---

[create-react-app](https://github.com/facebookincubator/create-react-app) 是业界最优秀的 React 应用开发工具之一，本文会尝试在 create-react-app 创建的工程中使用 `choerodon-ui` 组件，并自定义 webpack 的配置以满足各类工程化需求。

---

## 安装和初始化

我们需要在命令行中安装 create-react-app 工具，你可能还需要安装 [yarn](https://github.com/yarnpkg/yarn/)。

```bash
$ yarn create react-app choerodon-ui-demo

# or

$ npx create-react-app choerodon-ui-demo

# or use in typescript

$ yarn create react-app choerodon-ui-demo-ts --template typescript
```

工具会自动初始化一个脚手架并安装 React 项目的各种必要依赖，如果在过程中出现网络问题，请尝试配置代理或使用其他 npm registry。

然后我们进入项目并启动。

```bash
$ cd choerodon-ui-demo
$ yarn start
```

此时浏览器会访问 http://localhost:3000/ ，看到 `Welcome to React` 的界面就算成功了。

## 引入 choerodon-ui

这是 create-react-app 生成的默认目录结构。

```
├── README.md
├── package.json
├── public
│   ├── favicon.ico
│   └── index.html
├── src
│   ├── App.css
│   ├── App.js
│   ├── App.test.js
│   ├── index.css
│   ├── index.js
│   └── logo.svg
└── yarn.lock
```

现在从 yarn 或 npm 安装并引入 choerodon-ui 以及相关的依赖

```bash
$ yarn add choerodon-ui mobx mobx-react axios
```

> 请注意安装的时候的waring,如果出现了未知的错误,请确定mobx以及mobx-react是否为正确的版本

修改 `src/App.js`，引入 choerodon-ui 的按钮组件。

```jsx
import React, { Component } from 'react';
import { Button } from 'choerodon-ui/pro';
import './App.css';

class App extends Component {
  render() {
    return (
      <div className="App">
        <Button color="primary">Button</Button>
      </div>
    );
  }
}

export default App;
```

修改 `src/App.css`，在文件顶部引入 `choerodon-ui/dist/choerodon-ui.css` 以及 `choerodon-ui/dist/choerodon-ui-pro.css`。

```css
@import '~choerodon-ui/dist/choerodon-ui.css';
@import '~choerodon-ui/dist/choerodon-ui-pro.css';

.App {
  text-align: center;
}

...
```

好了，现在你应该能看到页面上已经有了 choerodon-ui 的蓝色按钮组件，接下来就可以继续选用其他组件开发应用了。其他开发流程你可以参考 create-react-app 的[官方文档](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md)。

## 高级配置

这个例子在实际开发中还有一些优化的空间，比如无法进行主题配置。

此时我们需要对 create-react-app 的默认配置进行自定义，这里我们使用 [craco](https://github.com/gsoft-inc/craco) （一个对 create-react-app 进行自定义配置的社区解决方案）。

现在我们安装 craco 并修改 `package.json` 里的 `scripts` 属性。

```bash
$ yarn add @craco/craco
```

```diff
/* package.json */
"scripts": {
-   "start": "react-scripts start",
-   "build": "react-scripts build",
-   "test": "react-scripts test",
+   "start": "craco start",
+   "build": "craco build",
+   "test": "craco test",
}
```

然后在项目根目录创建一个 `craco.config.js` 用于修改默认配置。


```js
/* craco.config.js */
module.exports = {
  // ...
};
```

### 使用 babel-plugin-import

[babel-plugin-import](https://github.com/ant-design/babel-plugin-import) 是一个用于按需加载组件代码和样式的 babel 插件（[原理](/docs/react/getting-started#按需加载)），现在我们尝试安装它并修改 `craco.config.js` 文件。在`choerodon-ui`中，用了less进行样式处理，我们指定了 style 为 true 所以需要引入 `less`, `less-loader` 以及 `craco-less`。需要注意，**必须将cssLoaderOptions中的url设置为false**

```bash
$ yarn add babel-plugin-import --dev
$ yarn add less less-loader craco-less
```

修改`craco.config.js`为

```js
const CracoLessPlugin = require('craco-less');

module.exports = {
  babel: {
    plugins: [
      [
        'import', {
          libraryName: 'choerodon-ui',
          style: true,
        }, 'c7n',
      ],
      [
        'import', {
          libraryName: 'choerodon-ui/pro',
          style: true,
        }, 'c7n-pro',
      ],
    ],
  },
  plugins: [
    {
      plugin: CracoLessPlugin,
      options: {
        lessLoaderOptions: {
          lessOptions: {
            javascriptEnabled: true,
          },
        },
        cssLoaderOptions: {
          url: false,
        },
      },
    },
  ],
};
```

然后移除前面在 `src/App.css` 里全量添加的 `@import '~choerodon-ui/dist/choerodon-ui.css';` 以及 `choerodon-ui/dist/choerodon-ui-pro.css` 样式代码

重启 `yarn start` 访问页面，choerodon-ui 组件的 js 和 css 代码都会按需加载，你在控制台也不会看到这样的[警告信息](https://zos.alipayobjects.com/rmsportal/vgcHJRVZFmPjAawwVoXK.png)。关于按需加载的原理和其他方式可以阅读[这里](/docs/react/getting-started#按需加载)。

### 自定义主题

按照 [配置主题](/docs/react/customize-theme) 的要求，自定义主题需要用到 less 变量覆盖功能。我们可以修改 `craco.config.js` 中的 `lessOptions` 


```diff
const CracoLessPlugin = require('craco-less');

module.exports = {
  babel: {
    plugins: [
      [
        'import', {
          libraryName: 'choerodon-ui',
          style: true,
        }, 'c7n'
      ],
      [
        'import', {
          libraryName: 'choerodon-ui/pro',
          style: true,
        }, 'c7n-pro'
      ]
    ]
  },
  plugins: [
    {
      plugin: CracoLessPlugin,
      options: {
        lessLoaderOptions: {
          lessOptions: {
            javascriptEnabled: true,
+            modifyVars: {
+              '@primary-color': '#1DA57A'
+            },
          },
        },
        cssLoaderOptions: {
          url: false
        }
      },
    },
  ],
};
```

这里利用了 [less-loader](https://github.com/webpack/less-loader#less-options) 的 `modifyVars` 来进行主题配置，
变量和其他配置方式可以参考 [配置主题](/docs/react/customize-theme) 文档。

修改后重启 `yarn start`，如果看到一个绿色的按钮就说明配置成功了。

> 在以前我们使用 customize-cra 以及 react-app-rewired 来进行自定义配置,但是因为其对less-loader@6的支持不是很好，因此更换为craco

## eject

你也可以使用 create-react-app 提供的 [yarn run eject](https://github.com/facebookincubator/create-react-app#converting-to-a-custom-setup) 命令将所有内建的配置暴露出来。不过这种配置方式需要你自行探索，不在本文讨论范围内。
